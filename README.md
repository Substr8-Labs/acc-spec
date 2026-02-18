# Agent Capability Control (ACC)

**Declarative Authorization Framework for Autonomous AI Ecosystems**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

---

## Overview

Agent Capability Control (ACC) is a declarative authorization framework designed specifically for AI agent ecosystems. It addresses the fundamental mismatch between traditional RBAC systems and the requirements of autonomous agents that operate continuously at machine speed.

## The Problem

Traditional RBAC assumes human users who:
- Act intermittently at human speeds
- Follow predictable patterns
- Can be reasoned with when exceeding bounds

AI agents break these assumptions through:
- **Continuous operation** — 24/7 without fatigue
- **High velocity** — Thousands of actions per hour
- **Emergent behavior** — Unpredictable tool chains
- **Delegation depth** — Spawning sub-agents creates permission chains

## The Solution

ACC provides a three-layer authorization model using markdown-native configuration:

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
```

### Key Features

- **Markdown-native** — Configuration lives in `.md` files with YAML frontmatter
- **Declarative** — Permissions are stated, not computed
- **Attenuating** — Sub-agents can only receive subset of parent permissions
- **Auditable** — Every authorization decision is traceable
- **Zero-trust** — Default deny; explicit grants required

## Documents

| Document | Description |
|----------|-------------|
| [ACC_SPEC.md](./ACC_SPEC.md) | Technical specification (v1.0.0) |
| [ACC_WHITEPAPER.md](./ACC_WHITEPAPER.md) | Academic whitepaper (10,454 words) |
| [ACC_WHITEPAPER.tex](./ACC_WHITEPAPER.tex) | LaTeX version for publication |
| [references.bib](./references.bib) | BibTeX citations |

## Quick Example

**SKILL.md** (skill declares requirements):
```yaml
---
name: publish-twitter
acc:
  required:
    - social:write
    - external:post
  denied_roles:
    - guest
    - reader
---
```

**SOUL.md** (agent declares capabilities):
```yaml
---
acc:
  role: agent
  capabilities:
    - social:*
    - external:*
  denied:
    - infra:*
---
```

**Authorization check:**
```
Agent wants: publish-twitter (needs social:write, external:post)
Agent has: social:*, external:*

social:write ⊆ social:* ✓
external:post ⊆ external:* ✓

Result: ALLOWED
```

## Prior Art

ACC builds upon proven patterns:

- **[Macaroons](https://research.google/pubs/pub41892/)** (Google, 2014) — HMAC caveats, capability attenuation
- **[Biscuits](https://www.biscuitsec.org/)** (Eclipse Foundation) — Datalog policies, Ed25519 signing

## Integration

ACC is designed to integrate with the [FDAA](https://github.com/Substr8-Labs/fdaa-spec) (Fully Deterministic Agent Architecture) verification pipeline:

- **Tier 2:** Guard Model validates ACC declarations match skill behavior
- **Tier 4:** Registry stores ACC metadata with cryptographic signatures

## License

MIT License — see [LICENSE](./LICENSE)

---

**Substr8 Labs** — Building provable agent infrastructure.
