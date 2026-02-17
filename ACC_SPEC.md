# Agent Capability Control (ACC)

**Version:** 0.1.0-draft  
**Status:** RFC  
**Author:** Ada (Substr8 Labs)  
**Date:** 2026-02-17

---

## Abstract

Agent Capability Control (ACC) is a declarative authorization framework for AI agent ecosystems. It defines how agents acquire, delegate, and exercise permissions over skills and tools using markdown-native configuration files.

ACC ensures that:
- Skills declare their required permissions
- Agents declare their granted capabilities
- Sub-agents receive attenuated (reduced) permissions
- All authorization decisions are auditable and deterministic

---

## 1. Problem Statement

### 1.1 The Agent Permission Problem

Traditional RBAC assumes human users who:
- Act intermittently and at human speed
- Follow predictable patterns
- Can be reasoned with when they exceed bounds

AI agents break these assumptions:
- **Continuous operation** — Agents run 24/7 without fatigue
- **High velocity** — Thousands of actions per hour
- **Emergent behavior** — Tool chains create unexpected permission combinations
- **Delegation depth** — Agents spawn sub-agents, creating permission inheritance chains

### 1.2 Current State

Most agent frameworks have no formal permission model. An agent with access to a skill can invoke it unconditionally. This creates risks:

| Risk | Example |
|------|---------|
| **Privilege escalation** | Sub-agent uses parent's Twitter credentials |
| **Lateral movement** | Research agent accesses infrastructure tools |
| **Unintended delegation** | Guest agent spawns admin-level sub-agents |
| **Audit gaps** | No record of why an agent had access |

### 1.3 Design Goals

1. **Markdown-native** — Configuration lives in `.md` files with YAML frontmatter
2. **Declarative** — Permissions are stated, not computed
3. **Attenuating** — Sub-agents can only receive subset of parent permissions
4. **Auditable** — Every authorization decision is traceable
5. **Zero-trust** — Default deny; explicit grants required

---

## 2. Architecture

ACC operates at three layers:

```
┌─────────────────────────────────────────────────────────┐
│                    RBAC.md (Central)                    │
│         Role definitions, capability mappings           │
└─────────────────────────────────────────────────────────┘
                            │
            ┌───────────────┴───────────────┐
            ▼                               ▼
┌─────────────────────┐         ┌─────────────────────────┐
│   AGENT.md / SOUL.md │         │       SKILL.md          │
│   Agent capabilities │         │   Required permissions  │
└─────────────────────┘         └─────────────────────────┘
            │                               │
            └───────────────┬───────────────┘
                            ▼
                ┌─────────────────────┐
                │   Runtime Enforcer   │
                │  agent.caps ⊇ skill  │
                └─────────────────────┘
```

### 2.1 Layer 1: Central Policy (RBAC.md)

Defines the organization's role hierarchy and capability mappings.

**Location:** `RBAC.md` in workspace root or `~/.openclaw/RBAC.md`

```markdown
# RBAC.md

## Roles

Roles form a hierarchy. Each role inherits capabilities from roles it extends.

| Role | Extends | Description |
|------|---------|-------------|
| `owner` | — | Full system access, cannot be attenuated |
| `admin` | `agent` | Administrative capabilities |
| `agent` | `worker` | Main session agent (e.g., Ada) |
| `worker` | `reader` | Task-focused sub-agents |
| `reader` | — | Read-only access |
| `guest` | — | Minimal access, sandboxed execution |

## Capabilities

Capabilities use a hierarchical namespace with wildcards.

| Capability | Description | Default Roles |
|------------|-------------|---------------|
| `*` | All capabilities | `owner` |
| `data:*` | All data operations | `admin`, `agent` |
| `data:read` | Read files, databases | `worker`, `reader` |
| `data:write` | Write files, databases | `admin`, `agent`, `worker` |
| `data:delete` | Delete files, databases | `admin` |
| `social:*` | All social media | `admin`, `agent` |
| `social:read` | Read social feeds | `worker` |
| `social:write` | Post to social media | `agent` |
| `social:dm` | Send direct messages | `admin` |
| `infra:*` | Infrastructure management | `admin` |
| `infra:read` | View infra status | `agent` |
| `infra:restart` | Restart services | `admin` |
| `infra:provision` | Create/destroy resources | `owner` |
| `external:*` | External API calls | `admin`, `agent` |
| `external:fetch` | HTTP GET requests | `worker` |
| `external:post` | HTTP POST/PUT/DELETE | `agent` |
| `spawn:*` | Agent spawning | `admin`, `agent` |
| `spawn:worker` | Spawn worker-level agents | `agent`, `worker` |
| `spawn:reader` | Spawn read-only agents | `worker` |

## Attenuation Rules

When an agent spawns a sub-agent, capabilities are attenuated:

| Parent Role | Can Spawn | Max Delegation |
|-------------|-----------|----------------|
| `owner` | Any role | `admin` |
| `admin` | `agent`, `worker`, `reader`, `guest` | `agent` |
| `agent` | `worker`, `reader`, `guest` | `worker` |
| `worker` | `reader`, `guest` | `reader` |
| `reader` | `guest` | — |
| `guest` | — | — |

## Explicit Denials

Some capabilities are never delegatable:

- `infra:provision` — Owner only, no delegation
- `social:dm` — Requires explicit per-session grant
- `data:delete` on paths matching `/secrets/*` — Admin only
```

### 2.2 Layer 2: Agent Capabilities (AGENT.md / SOUL.md)

Each agent declares its capabilities in frontmatter.

**Location:** `SOUL.md`, `AGENT.md`, or `IDENTITY.md`

```yaml
---
# Agent Capability Declaration
acc:
  role: agent
  capabilities:
    - data:*
    - social:read
    - social:write
    - external:fetch
    - spawn:worker
  denied:
    - infra:*
    - social:dm
  constraints:
    max_spawn_depth: 3
    require_approval:
      - social:write
      - external:post
    rate_limits:
      social:write: 10/hour
      spawn:*: 50/day
---
```

**Schema:**

| Field | Type | Description |
|-------|------|-------------|
| `acc.role` | string | Base role from RBAC.md |
| `acc.capabilities` | string[] | Granted capabilities (additive to role) |
| `acc.denied` | string[] | Explicitly denied (overrides role grants) |
| `acc.constraints.max_spawn_depth` | int | How many levels deep this agent can spawn |
| `acc.constraints.require_approval` | string[] | Capabilities that need human approval |
| `acc.constraints.rate_limits` | map | Rate limits per capability |

### 2.3 Layer 3: Skill Requirements (SKILL.md)

Skills declare the permissions they require.

**Location:** `SKILL.md` in skill directory

```yaml
---
name: publish-twitter
version: 1.2.0
description: Post tweets and threads to X/Twitter

# ACC Requirements
acc:
  required:
    - social:write
    - external:post
  optional:
    - data:read  # For reading draft files
  denied_roles:
    - guest
    - reader
  scope: |
    This skill posts content to Twitter. It requires write access
    to social media and the ability to make external API calls.
    It does NOT require access to DMs or infrastructure.
---
```

**Schema:**

| Field | Type | Description |
|-------|------|-------------|
| `acc.required` | string[] | Must have ALL of these |
| `acc.optional` | string[] | May use if available |
| `acc.denied_roles` | string[] | Roles that cannot use this skill |
| `acc.scope` | string | Human-readable scope description |

---

## 3. Runtime Enforcement

### 3.1 Authorization Algorithm

When an agent attempts to invoke a skill:

```python
def authorize(agent: Agent, skill: Skill) -> AuthResult:
    # 1. Check role denial
    if agent.role in skill.acc.denied_roles:
        return Denied(reason="role_denied", 
                      detail=f"Role {agent.role} cannot use {skill.name}")
    
    # 2. Resolve agent's effective capabilities
    effective_caps = resolve_capabilities(agent)
    
    # 3. Check required capabilities
    for required in skill.acc.required:
        if not matches(required, effective_caps):
            return Denied(reason="missing_capability",
                          detail=f"Requires {required}")
    
    # 4. Check explicit denials
    for required in skill.acc.required:
        if matches(required, agent.acc.denied):
            return Denied(reason="explicit_denial",
                          detail=f"Agent explicitly denies {required}")
    
    # 5. Check rate limits
    if exceeds_rate_limit(agent, skill.acc.required):
        return Denied(reason="rate_limited")
    
    # 6. Check approval requirements
    pending = []
    for cap in skill.acc.required:
        if cap in agent.acc.constraints.require_approval:
            pending.append(cap)
    
    if pending:
        return PendingApproval(capabilities=pending)
    
    return Allowed(effective_caps=effective_caps)
```

### 3.2 Capability Resolution

Capabilities use hierarchical matching with wildcards:

```python
def matches(required: str, granted: set[str]) -> bool:
    # Exact match
    if required in granted:
        return True
    
    # Wildcard match (e.g., "data:*" grants "data:read")
    parts = required.split(":")
    for i in range(len(parts)):
        wildcard = ":".join(parts[:i+1]) + ":*"
        if wildcard in granted:
            return True
    
    # Global wildcard
    if "*" in granted:
        return True
    
    return False
```

### 3.3 Spawn Attenuation

When spawning a sub-agent:

```python
def attenuate(parent: Agent, requested_caps: list[str]) -> set[str]:
    # Get parent's effective capabilities
    parent_caps = resolve_capabilities(parent)
    
    # Sub-agent can only have subset of parent
    granted = set()
    for cap in requested_caps:
        if matches(cap, parent_caps):
            granted.add(cap)
    
    # Apply attenuation rules from RBAC.md
    max_role = get_max_delegation(parent.role)
    granted = filter_by_role(granted, max_role)
    
    # Never delegate parent's explicit denials
    granted -= parent.acc.denied
    
    # Decrement spawn depth
    if parent.acc.constraints.max_spawn_depth <= 0:
        raise SpawnDenied("max_spawn_depth exceeded")
    
    return granted
```

---

## 4. Audit Trail

Every authorization decision is logged:

```json
{
  "timestamp": "2026-02-17T10:45:00Z",
  "trace_id": "acc_8f3a2b1c",
  "agent": {
    "id": "ada",
    "role": "agent",
    "session": "main"
  },
  "skill": {
    "name": "publish-twitter",
    "version": "1.2.0"
  },
  "decision": "allowed",
  "required_caps": ["social:write", "external:post"],
  "granted_caps": ["social:*", "external:*"],
  "parent_chain": ["owner:raza"],
  "constraints_checked": {
    "rate_limit": "2/10 per hour",
    "approval": "not_required"
  }
}
```

---

## 5. Integration with FDAA

ACC integrates with the FDAA verification pipeline:

### 5.1 Tier 2 Enhancement

The Guard Model validates ACC declarations:

```
SKILL CAPABILITY ANALYSIS
─────────────────────────
Skill: publish-twitter v1.2.0

Required Capabilities:
  ✓ social:write — Matches skill scope (posting tweets)
  ✓ external:post — Required for Twitter API

Scope Consistency: PASS
  - Skill claims to post tweets → requires social:write ✓
  - Makes external API calls → requires external:post ✓
  - Does not claim DM access → does not require social:dm ✓

Capability Minimality: PASS
  - No over-broad capabilities requested
  - No wildcards in requirements
```

### 5.2 Tier 4 Enhancement

The registry stores ACC metadata:

```json
{
  "skill_id": "d5cf0a0cd8cd3bf8",
  "name": "publish-twitter",
  "acc": {
    "required": ["social:write", "external:post"],
    "denied_roles": ["guest", "reader"],
    "risk_level": "medium"
  },
  "signature": "ed25519:..."
}
```

---

## 6. Migration Path

### 6.1 Phase 1: Opt-in Declaration

Skills and agents MAY include ACC frontmatter. Runtime logs warnings but doesn't enforce.

### 6.2 Phase 2: Soft Enforcement

Runtime checks ACC and logs violations, but allows execution. Builds audit trail.

### 6.3 Phase 3: Hard Enforcement

ACC checks are mandatory. Missing declarations default to most restrictive.

---

## 7. Examples

### 7.1 Main Agent (Ada)

```yaml
# SOUL.md
---
acc:
  role: agent
  capabilities:
    - data:*
    - social:*
    - external:*
    - spawn:worker
  denied:
    - infra:provision
    - infra:restart
  constraints:
    max_spawn_depth: 3
    require_approval:
      - social:dm
    rate_limits:
      social:write: 20/hour
---
```

### 7.2 Research Sub-Agent

```yaml
# spawned with attenuated capabilities
---
acc:
  role: worker
  capabilities:
    - data:read
    - external:fetch
  denied:
    - social:*
    - infra:*
    - spawn:*
  constraints:
    max_spawn_depth: 0  # Cannot spawn further
---
```

### 7.3 Skill: Infrastructure Restart

```yaml
# SKILL.md
---
name: restart-gateway
version: 1.0.0
acc:
  required:
    - infra:restart
  denied_roles:
    - guest
    - reader
    - worker
  scope: |
    Restarts the OpenClaw gateway service. Requires infrastructure
    management permissions. Only admin-level agents should have access.
---
```

### 7.4 Authorization Flow

```
Ada (agent) wants to use publish-twitter skill:

1. Check role: agent not in denied_roles [guest, reader] ✓
2. Resolve capabilities: [data:*, social:*, external:*, spawn:worker]
3. Check required:
   - social:write ⊆ social:* ✓
   - external:post ⊆ external:* ✓
4. Check denials: none match ✓
5. Check rate limit: 3/20 this hour ✓
6. Check approval: social:write not in require_approval ✓

Result: ALLOWED
```

```
Research-Agent (worker) wants to use publish-twitter skill:

1. Check role: worker not in denied_roles [guest, reader] ✓
2. Resolve capabilities: [data:read, external:fetch]
3. Check required:
   - social:write ⊈ [data:read, external:fetch] ✗

Result: DENIED (missing_capability: social:write)
```

---

## 8. Security Considerations

### 8.1 Frontmatter Tampering

ACC frontmatter is verified by FDAA Tier 1 (static analysis). Any modification after signing invalidates the skill.

### 8.2 Capability Explosion

Wildcard capabilities (`*`) should be rare. The Guard Model flags skills requesting wildcards.

### 8.3 Delegation Attacks

An agent cannot grant capabilities it doesn't have. The attenuation algorithm is monotonically decreasing.

### 8.4 Rate Limit Bypass

Rate limits are tracked per-agent, per-capability, in persistent storage. Clock manipulation is detected via signed timestamps.

---

## 9. Future Work

- **Temporal capabilities** — Permissions that expire after N minutes
- **Contextual capabilities** — Permissions scoped to specific resources (e.g., "social:write on @substr8labs only")
- **Capability certificates** — Cryptographically signed capability tokens for cross-system authorization
- **Revocation** — Real-time capability revocation broadcast

---

## Appendix A: Capability Namespace Registry

| Namespace | Description |
|-----------|-------------|
| `data:` | File and database operations |
| `social:` | Social media platforms |
| `infra:` | Infrastructure management |
| `external:` | External API calls |
| `spawn:` | Agent spawning |
| `message:` | Messaging (email, chat) |
| `finance:` | Financial operations |
| `code:` | Code execution |
| `browser:` | Browser automation |
| `camera:` | Camera/media capture |

---

## Appendix B: YAML Schema

```yaml
# JSON Schema for ACC in SKILL.md
$schema: "http://json-schema.org/draft-07/schema#"
type: object
properties:
  acc:
    type: object
    properties:
      required:
        type: array
        items:
          type: string
          pattern: "^[a-z]+:[a-z*]+$"
      optional:
        type: array
        items:
          type: string
      denied_roles:
        type: array
        items:
          type: string
      scope:
        type: string
    required: [required]
```

---

*This specification is part of the FDAA (Fully Deterministic Agent Architecture) framework.*
