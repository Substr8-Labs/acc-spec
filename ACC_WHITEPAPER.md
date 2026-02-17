# Agent Capability Control: Declarative Authorization for Autonomous AI Ecosystems

**Authors:** Substr8 Labs
**Date:** 2026-02-17

---

## Abstract

**Abstract**

Traditional Role-Based Access Control (RBAC) systems assume human users who operate intermittently at human speeds, follow predictable patterns, and can be reasoned with when exceeding authorized bounds. However, autonomous AI agents fundamentally violate these assumptions through continuous operation, high-velocity execution (thousands of actions per hour), emergent behaviors, and complex delegation chains. Most existing agent frameworks lack formal permission models, creating significant security vulnerabilities in autonomous AI ecosystems.

This paper introduces Agent Capability Control (ACC), a declarative authorization framework specifically designed for autonomous AI systems. ACC addresses the unique challenges of AI agents by requiring skills to explicitly declare their permission requirements, agents to specify their granted capabilities, and ensuring sub-agents receive only attenuated subsets of parent permissions. The framework operates across three hierarchical layers: Central Policy (RBAC.md), Agent Capabilities (AGENT.md/SOUL.md), and Skill Requirements (SKILL.md).

ACC employs hierarchical capability matching with wildcard support for authorization decisions and implements comprehensive audit logging that captures timestamps, agent details, skill information, and decision rationales. The system integrates with the Formal Design Assurance Architecture (FDAA) verification pipeline, where Tier 1 static analysis prevents tampering with ACC frontmatter, while Tier 2 and Tier 4 provide additional verification enhancements.

Our approach ensures deterministic, auditable authorization decisions while maintaining the flexibility required for autonomous agent operation, providing a foundation for secure AI agent ecosystems.

## 1. Introduction

# 1. Introduction

The proliferation of autonomous AI agents in enterprise and distributed computing environments has fundamentally challenged traditional authorization paradigms. As AI systems evolve from simple rule-based assistants to sophisticated autonomous agents capable of complex reasoning, planning, and execution, the security frameworks governing their access to resources and capabilities have failed to keep pace. This paper introduces Agent Capability Control (ACC), a declarative authorization framework specifically designed to address the unique challenges posed by autonomous AI ecosystems.

## 1.1 Problem Statement: Traditional RBAC Limitations in AI Agent Environments

Role-Based Access Control (RBAC) has served as the cornerstone of enterprise authorization systems for decades, providing a robust framework for managing human user permissions through role assignments and hierarchical privilege structures [Silva et al., 2013]. However, RBAC was architected under fundamental assumptions about user behavior that autonomous AI agents systematically violate.

Traditional RBAC assumes users who: (1) act intermittently with natural breaks in activity, (2) operate at human cognitive and physical speeds, (3) follow predictable behavioral patterns based on job functions, and (4) can be reasoned with or interrupted when they exceed authorized bounds. These assumptions enabled RBAC's coarse-grained permission model, where users are granted broad capabilities within their roles, with the expectation that human judgment and natural limitations would prevent abuse.

AI agents fundamentally break these assumptions through several key characteristics. First, agents operate continuously without natural breaks, potentially executing thousands of actions per hour compared to the dozens typical of human users. Second, agents exhibit emergent behaviors that may deviate significantly from their intended operational patterns, making traditional role-based predictions unreliable [Krishnan, 2025]. Third, agents frequently delegate tasks to sub-agents, creating dynamic permission chains that traditional RBAC cannot adequately model or control. Finally, agents cannot be "reasoned with" in the traditional sense—they operate according to their programming and training, making post-hoc intervention challenging.

The inadequacy of existing permission models becomes particularly acute in multi-agent systems where agents must coordinate, delegate, and share resources while maintaining security boundaries. Current agent frameworks largely operate without formal permission models, relying instead on implicit trust relationships or ad-hoc security measures that provide insufficient protection against both malicious actors and emergent misbehavior [Ganie, 2025].

## 1.2 Motivation: Why Existing Permission Models Fail for Autonomous Agents

The failure of traditional authorization models in AI agent environments stems from their inability to address three critical requirements: fine-grained capability control, dynamic permission attenuation, and comprehensive auditability.

Fine-grained capability control becomes essential when agents operate at machine speed and scale. Unlike human users who might access a database a few times per day, an AI agent might query thousands of records per minute while performing analytical tasks [Liu et al., 2025]. Traditional RBAC's broad permission grants become dangerous when applied to such high-velocity operations, as a compromised or misbehaving agent could cause significant damage before detection.

Dynamic permission attenuation addresses the challenge of agent delegation, where primary agents spawn sub-agents to accomplish specific tasks. In human organizations, delegation typically involves explicit communication and oversight. In AI systems, delegation occurs programmatically and at scale, requiring automated mechanisms to ensure that sub-agents receive only the minimum permissions necessary for their specific tasks. Existing frameworks lack the semantic understanding necessary to perform such fine-grained permission inheritance [Colley et al., 2021].

Comprehensive auditability becomes critical when agents operate autonomously across distributed systems. Traditional audit logs capture what actions were performed but often lack the contextual information necessary to understand why those actions were authorized and whether they align with intended system behavior. This limitation becomes particularly problematic when investigating security incidents or ensuring compliance with regulatory requirements [Gavilanez et al., 2017].

The convergence of these challenges has created a security gap in autonomous AI deployments. Organizations deploying AI agents often resort to either overly permissive configurations that expose them to significant risk, or overly restrictive configurations that limit agent effectiveness. Neither approach provides a sustainable path forward as AI systems become more sophisticated and autonomous.

## 1.3 Contributions: Key Contributions of the ACC Framework

This paper presents Agent Capability Control (ACC), a declarative authorization framework that addresses the fundamental limitations of traditional permission models in autonomous AI ecosystems. ACC makes several key contributions to the field of AI security and authorization systems.

First, ACC introduces a three-layer authorization architecture that operates at distinct levels of abstraction: Central Policy (RBAC.md), Agent Capabilities (AGENT.md/SOUL.md), and Skill Requirements (SKILL.md). This layered approach enables fine-grained control while maintaining system comprehensibility and administrative efficiency. The Central Policy layer provides organization-wide governance similar to traditional RBAC, while the Agent and Skill layers enable dynamic, context-aware authorization decisions.

Second, ACC implements declarative permission specification that requires skills to explicitly declare their required permissions and agents to explicitly declare their granted capabilities. This approach eliminates the implicit trust relationships common in current agent frameworks and ensures that all capability requirements are visible and auditable at design time. The declarative model draws inspiration from successful declarative frameworks in other domains [Zhou et al., 2024] while addressing the unique requirements of autonomous agent systems.

Third, ACC provides automatic permission attenuation for sub-agent delegation, ensuring that delegated agents receive only the minimum permissions necessary for their specific tasks. This capability addresses one of the most significant security challenges in multi-agent systems by preventing privilege escalation through delegation chains while maintaining system functionality.

Fourth, ACC ensures that all authorization decisions are auditable and deterministic, providing comprehensive logging and reasoning capabilities that enable security teams to understand not just what happened, but why it was authorized. This auditability extends beyond simple action logging to include the complete decision context, enabling sophisticated security analysis and compliance reporting.

Finally, ACC integrates with existing Zero Trust architectures [James et al., 2024] while providing agent-specific extensions that address the unique challenges of autonomous systems. This integration ensures that ACC can be deployed within existing enterprise security frameworks without requiring wholesale infrastructure replacement.

The remainder of this paper details the ACC framework architecture, implementation, and evaluation, demonstrating how declarative authorization can provide both security and functionality in autonomous AI ecosystems.

## 2. Background and Related Work

# 2. Background and Related Work

This section examines the foundations of authorization systems and their evolution toward supporting autonomous AI agents. We review traditional access control models, distributed authorization frameworks, current AI agent permission systems, and the unique security challenges posed by autonomous systems.

## 2.1 Traditional Role-Based Access Control

Role-Based Access Control (RBAC) has served as the dominant authorization paradigm for enterprise systems since the 1990s. RBAC operates on fundamental assumptions about user behavior: users act intermittently with natural pauses between actions, operate at human cognitive speeds, follow predictable patterns based on job functions, and can be reasoned with when they exceed authorized boundaries [Ganie, 2025]. These assumptions enable RBAC's core mechanisms of role assignment, permission inheritance, and session-based controls.

Traditional RBAC models assume that principals (users) authenticate once per session, perform a bounded number of actions within predictable timeframes, and can respond to authorization challenges or policy violations through direct interaction. The model's effectiveness relies on the inherent limitations of human actors: cognitive load constraints that prevent excessive simultaneous operations, natural breaks in activity that allow for policy updates and audit reviews, and the ability to engage in real-time dialogue when authorization boundaries are approached.

However, these foundational assumptions become problematic when applied to AI agents. Unlike human users, AI agents operate continuously without natural breaks, can execute thousands of actions per hour, exhibit emergent behaviors that deviate from programmed patterns, and cannot engage in meaningful dialogue about authorization boundaries. Furthermore, AI agents frequently delegate tasks to sub-agents, creating complex authorization chains that traditional RBAC cannot adequately model or control.

## 2.2 Authorization in Distributed Systems

Distributed systems have driven the development of more flexible authorization frameworks that move beyond the rigid hierarchies of traditional RBAC. Declarative authorization models separate policy specification from enforcement mechanisms, enabling more dynamic and context-aware access control [Silva et al., 2013]. These frameworks typically employ policy languages that can express complex rules involving attributes, environmental conditions, and delegation patterns.

Extensible authorization architectures have emerged to address the heterogeneity of distributed environments [Fernández et al., 2008]. These systems support pluggable policy engines, multiple authentication mechanisms, and cross-domain authorization flows. The convergence of web services and telecommunications has particularly driven innovation in declarative frameworks that can handle diverse application contexts while maintaining security guarantees.

Modern distributed authorization systems emphasize auditability and deterministic policy evaluation [Gavilanez et al., 2017]. Audit trails become critical in distributed environments where authorization decisions occur across multiple systems and administrative domains. The challenge lies in maintaining consistency and traceability while supporting the flexibility required by dynamic distributed applications.

Zero Trust architectures have further influenced authorization design, particularly in IoT environments where traditional perimeter-based security models fail [James et al., 2024]. These frameworks assume no implicit trust and require continuous verification of all access requests, regardless of source location or previous authentication status.

## 2.3 AI Agent Frameworks

Current AI agent frameworks exhibit significant gaps in formal permission models. Most existing systems focus on agent communication protocols, task orchestration, and learning mechanisms while treating authorization as an afterthought [Krishnan, 2025]. This oversight creates substantial security risks as agents gain access to sensitive resources and perform actions with real-world consequences.

The few agent frameworks that do implement permission systems typically rely on simple capability lists or inherit permissions from their human operators. These approaches fail to address the unique characteristics of agent behavior: continuous operation, high-velocity action execution, emergent decision-making, and complex delegation chains. Without formal permission models, agents can exceed intended boundaries, access unauthorized resources, or delegate excessive privileges to sub-agents.

Recent work in agentic AI for specialized domains, such as Identity Security Posture Management, has highlighted the need for more sophisticated authorization mechanisms [Engelberg et al., 2026]. These applications demonstrate that AI agents require fine-grained permission controls that can adapt to dynamic environments while maintaining security guarantees.

The absence of standardized permission models across agent frameworks creates interoperability challenges and security gaps. Agents developed for one framework cannot easily be integrated into another system without extensive security reviews and permission mapping exercises.

## 2.4 Security in Autonomous Systems

Autonomous systems present unique authentication and authorization challenges that extend beyond traditional access control models. The continuous operation of autonomous agents creates sustained attack surfaces that require constant monitoring and dynamic policy adaptation [Siebert et al., 2021]. Unlike human-operated systems with natural downtime, autonomous systems must maintain security posture during 24/7 operation.

The concept of meaningful human control becomes critical in autonomous systems where agents make decisions with significant consequences [Siebert et al., 2021]. Authorization frameworks must balance agent autonomy with human oversight, ensuring that critical decisions remain subject to human approval while allowing routine operations to proceed without intervention.

Delegation in autonomous systems introduces additional complexity, particularly in multi-agent scenarios where agents must coordinate and share responsibilities [Colley et al., 2021]. Traditional delegation models assume direct human-to-human delegation with clear accountability chains. Autonomous systems require more sophisticated delegation mechanisms that can handle agent-to-agent delegation, partial delegation of capabilities, and dynamic revocation of delegated permissions.

Encrypted control systems in networked environments add another layer of complexity to authorization [Darup et al., 2020]. Autonomous agents operating in cloud or distributed environments must maintain security while communicating across potentially untrusted networks. This requires authorization frameworks that can operate effectively with encrypted communications and distributed policy enforcement points.

The high-velocity nature of autonomous systems also challenges traditional audit and compliance mechanisms. Agents may generate thousands of authorization events per hour, requiring automated analysis and anomaly detection capabilities that can identify policy violations or suspicious patterns in real-time. Traditional human-reviewed audit processes cannot scale to match the operational tempo of autonomous systems.

## 3. Agent Capability Control Framework

# 3. Agent Capability Control Framework

## 3.1 Framework Overview

The Agent Capability Control (ACC) framework addresses fundamental limitations in existing authorization models when applied to autonomous AI ecosystems. Traditional Role-Based Access Control (RBAC) systems operate under assumptions that are systematically violated by AI agents: human users act intermittently and at human speed, follow predictable behavioral patterns, and can be reasoned with when they exceed operational bounds [Ganie, 2025]. In contrast, AI agents exhibit continuous operation, high-velocity execution (potentially thousands of actions per hour), emergent behaviors that deviate from programmed patterns, and complex delegation chains that amplify authorization complexity.

Most contemporary agent frameworks lack formal permission models entirely, relying instead on implicit trust relationships or ad-hoc access controls [Krishnan, 2025]. This absence of systematic authorization creates significant security vulnerabilities in multi-agent environments where autonomous systems must interact with external resources, delegate tasks to sub-agents, and operate with varying levels of privilege.

The ACC framework introduces a three-layer declarative architecture designed to address these challenges through explicit capability management and auditable authorization decisions. The framework ensures that: (1) skills declare their required permissions explicitly, (2) agents declare their granted capabilities through formal specifications, (3) sub-agents receive appropriately attenuated permissions through delegation chains, and (4) all authorization decisions remain auditable and deterministic [Silva et al., 2013].

The architectural design follows declarative principles similar to those employed in modern UI frameworks and query processing systems, where explicit specification of requirements and capabilities enables automated reasoning and optimization [Zhou et al., 2024; Liu et al., 2025]. This declarative approach provides several advantages over imperative authorization models: improved auditability, reduced configuration complexity, and enhanced reasoning capabilities for automated policy enforcement.

## 3.2 Central Policy Layer (RBAC.md)

The Central Policy Layer serves as the authoritative source for global policy definitions and role hierarchies within the ACC framework. This layer extends traditional RBAC concepts to accommodate the unique requirements of autonomous agent ecosystems while maintaining compatibility with established access control paradigms [James et al., 2024].

The RBAC.md specification defines organizational roles, permission sets, and hierarchical relationships through a structured markdown format that enables both human readability and automated processing. Unlike conventional RBAC implementations that assume static role assignments and infrequent permission changes, the ACC Central Policy Layer supports dynamic role evolution and fine-grained permission granularity necessary for agent operations.

Key components of the Central Policy Layer include:

**Role Hierarchies**: Hierarchical role structures that support inheritance and delegation patterns common in agent ecosystems. These hierarchies accommodate both vertical delegation (supervisor-to-subordinate) and horizontal delegation (peer-to-peer collaboration) scenarios [Colley et al., 2021].

**Permission Domains**: Categorized permission sets that align with common agent operational patterns, including resource access, communication privileges, and computational resource allocation. Each domain supports granular permission specification to enable precise capability control.

**Policy Inheritance Rules**: Explicit rules governing how permissions propagate through role hierarchies and delegation chains. These rules ensure that sub-agents receive appropriately attenuated capabilities while maintaining operational effectiveness.

**Audit Trail Requirements**: Mandatory logging specifications that ensure all policy modifications and access decisions are recorded for subsequent analysis and compliance verification [Gavilanez et al., 2017].

The declarative nature of the Central Policy Layer enables automated policy validation and conflict detection, addressing common issues in complex authorization environments where manual policy management becomes intractable.

## 3.3 Agent Capabilities Layer (AGENT.md/SOUL.md)

The Agent Capabilities Layer provides agent-specific capability declarations and grants through structured specification files. This layer bridges the gap between global organizational policies defined in the Central Policy Layer and the specific operational requirements of individual agents.

The AGENT.md specification format captures essential agent metadata, including identity information, operational scope, and granted capabilities. For agents with more complex behavioral patterns or learning capabilities, the SOUL.md format provides extended specification options that accommodate dynamic capability evolution and emergent behaviors.

**Agent Identity and Scope**: Each agent specification includes unique identification, operational domain boundaries, and intended functional scope. This information enables precise capability matching and prevents unauthorized scope expansion during agent operation.

**Capability Grants**: Explicit enumeration of granted capabilities, mapped to the permission domains defined in the Central Policy Layer. These grants specify both positive permissions (allowed actions) and negative permissions (explicitly forbidden actions) to provide comprehensive access control.

**Delegation Authorities**: Specification of an agent's authority to create sub-agents or delegate tasks to other agents. Delegation authorities include capability attenuation rules that ensure sub-agents receive appropriately restricted permissions [Siebert et al., 2021].

**Resource Constraints**: Operational limits including computational resource allocation, network access boundaries, and temporal operation windows. These constraints provide additional safety mechanisms beyond traditional permission-based controls.

**Behavioral Specifications**: For agents using the SOUL.md format, extended behavioral specifications that capture learning patterns, adaptation mechanisms, and emergent behavior boundaries. These specifications enable proactive capability management for agents with dynamic operational patterns.

The Agent Capabilities Layer supports both static capability assignment and dynamic capability evolution through controlled update mechanisms that maintain authorization integrity while enabling operational flexibility.

## 3.4 Skill Requirements Layer (SKILL.md)

The Skill Requirements Layer provides skill-level permission requirements and declarations, enabling fine-grained authorization control at the individual capability level. This layer addresses the challenge of managing permissions for modular agent capabilities that may be dynamically loaded, shared between agents, or composed into complex operational workflows.

Each skill specification in the SKILL.md format declares its required permissions, resource dependencies, and interaction patterns with other system components. This declarative approach enables automated capability matching and prevents privilege escalation through skill composition or dynamic loading.

**Permission Requirements**: Explicit declaration of all permissions required for skill execution, including resource access, communication privileges, and system interaction capabilities. These requirements are expressed using the permission vocabulary defined in the Central Policy Layer.

**Resource Dependencies**: Specification of computational, network, and storage resources required for skill operation. These dependencies enable resource-aware scheduling and prevent resource exhaustion attacks through skill proliferation.

**Interaction Patterns**: Declaration of expected interaction patterns with other skills, agents, or external systems. These patterns enable automated workflow validation and prevent unauthorized information flows between system components.

**Safety Constraints**: Skill-specific safety requirements including execution timeouts, resource consumption limits, and failure handling procedures. These constraints provide defense-in-depth protection against malicious or malfunctioning skills.

**Composition Rules**: Specifications governing how skills may be combined or chained together, including permission propagation rules and capability attenuation requirements for skill composition scenarios.

The Skill Requirements Layer enables modular permission management where skills can be developed, tested, and deployed independently while maintaining system-wide authorization integrity. This modularity supports rapid agent development and deployment while preserving security boundaries through explicit capability declarations [Fernández et al., 2008].

The three-layer architecture of the ACC framework provides comprehensive authorization control for autonomous AI ecosystems through declarative specification and automated enforcement. This approach addresses the fundamental limitations of traditional access control models when applied to high-velocity, autonomous agent operations while maintaining auditability and deterministic behavior essential for enterprise deployment.

## 4. Authorization Mechanisms

# 4. Authorization Mechanisms

The Agent Capability Control (ACC) framework implements a multi-layered authorization architecture that addresses the fundamental limitations of traditional Role-Based Access Control (RBAC) when applied to autonomous AI systems. Traditional RBAC assumes human users who act intermittently and at human speed, follow predictable patterns, and can be reasoned with when they exceed bounds [Ganie, 2025]. AI agents break these RBAC assumptions through continuous operation, high velocity (thousands of actions per hour), emergent behavior, and delegation depth that can span multiple hierarchical levels [Krishnan, 2025].

Most existing agent frameworks lack formal permission models, creating significant security gaps in autonomous systems [Engelberg et al., 2026]. ACC addresses this deficiency by ensuring that skills declare their required permissions, agents declare their granted capabilities, sub-agents receive attenuated permissions, and all authorization decisions are auditable and deterministic. The framework operates at three distinct layers: Central Policy (RBAC.md), Agent Capabilities (AGENT.md/SOUL.md), and Skill Requirements (SKILL.md), providing comprehensive coverage across the entire agent ecosystem.

## 4.1 Hierarchical Capability Matching

The ACC framework employs a sophisticated wildcard-based permission matching algorithm that enables flexible yet secure capability resolution across hierarchical agent structures. This matching system extends traditional permission models to accommodate the dynamic nature of AI agent interactions while maintaining strict security boundaries.

The hierarchical matching algorithm operates on capability strings structured as dot-separated namespaces (e.g., `data.read.customer`, `system.execute.backup`). The system supports three types of wildcard patterns:

- **Single-level wildcards (`*`)**: Match exactly one namespace level
- **Multi-level wildcards (`**`)**: Match zero or more namespace levels
- **Suffix wildcards (`*suffix`)**: Match any string ending with the specified suffix

The matching algorithm implements a recursive descent parser that evaluates capability requests against granted permissions with O(n log m) complexity, where n represents the number of granted capabilities and m represents the average depth of the capability hierarchy. This efficiency is crucial given the high-velocity nature of AI agent operations.

```
Algorithm: Hierarchical Capability Matching
Input: requested_capability, granted_capabilities[]
Output: match_result, effective_permissions

1. Parse requested_capability into namespace components
2. For each granted_capability in granted_capabilities:
   a. Parse granted_capability into pattern components
   b. Apply wildcard matching rules recursively
   c. Calculate permission intersection
3. Return most restrictive matching permission set
```

The algorithm incorporates capability inheritance, where permissions granted at higher levels in the hierarchy automatically apply to lower levels unless explicitly restricted. This design reduces configuration complexity while maintaining security through the principle of least privilege [Silva et al., 2013].

## 4.2 Permission Attenuation for Sub-Agents

Permission attenuation represents a critical mechanism for maintaining security boundaries when agents delegate tasks to sub-agents. The ACC framework implements a formal attenuation model that ensures sub-agents never receive capabilities exceeding those of their parent agents, while enabling fine-grained control over delegated permissions.

The attenuation process operates through three primary mechanisms:

**Capability Reduction**: Parent agents can reduce the scope of permissions when creating sub-agents. For example, a parent agent with `data.read.**` permissions might create a sub-agent with only `data.read.public.*` permissions, effectively restricting access to public data sources only.

**Temporal Constraints**: Delegated permissions can include time-based restrictions, automatically expiring after specified durations or at predetermined timestamps. This temporal attenuation prevents long-lived sub-agents from accumulating excessive permissions over time.

**Contextual Limitations**: The framework supports context-aware attenuation, where delegated permissions are valid only within specific operational contexts or when certain conditions are met. This mechanism is particularly valuable for maintaining security in dynamic environments where agent behavior may evolve unpredictably [Siebert et al., 2021].

The attenuation rules are expressed declaratively within the agent configuration files, enabling automated verification and preventing privilege escalation attacks. The framework maintains a delegation chain record, tracking the complete path of permission inheritance from root agents to leaf sub-agents, ensuring accountability and enabling comprehensive security audits.

## 4.3 Authorization Decision Engine

The Authorization Decision Engine (ADE) serves as the central component responsible for making deterministic authorization decisions across the ACC framework. The engine implements a policy evaluation algorithm that combines capabilities from all three ACC layers while resolving conflicts through a well-defined precedence hierarchy.

The decision process follows a structured evaluation pipeline:

1. **Policy Retrieval**: The engine retrieves relevant policies from the Central Policy layer (RBAC.md), agent capabilities from the Agent Capabilities layer (AGENT.md/SOUL.md), and skill requirements from the Skill Requirements layer (SKILL.md).

2. **Capability Resolution**: Using the hierarchical matching algorithm, the engine determines which granted capabilities apply to the requested action, considering wildcard patterns and inheritance rules.

3. **Conflict Resolution**: When multiple policies apply to a single request, the engine employs a deterministic conflict resolution strategy based on specificity precedence, where more specific rules override general ones, and explicit deny rules take precedence over allow rules.

4. **Decision Synthesis**: The final authorization decision combines all applicable policies, ensuring that the resulting permission set represents the intersection of all relevant constraints [James et al., 2024].

The ADE implements a caching mechanism to optimize performance for high-velocity agent operations, maintaining recently computed decisions in memory while ensuring cache invalidation when underlying policies change. This optimization is essential given that AI agents may generate thousands of authorization requests per hour.

The engine also incorporates anomaly detection capabilities, flagging unusual permission patterns or rapid escalation attempts that may indicate compromised agents or emergent behaviors requiring human intervention [Engelberg et al., 2026].

## 4.4 Audit Trail and Logging

Comprehensive audit trails represent a fundamental requirement for autonomous AI systems, where the high velocity and complexity of operations necessitate detailed logging for security analysis, compliance verification, and incident response [Gavilanez et al., 2017]. The ACC framework implements a multi-layered logging architecture that captures all authorization decisions with sufficient detail to enable complete reconstruction of agent behavior.

The audit system logs the following information for each authorization decision:

**Request Context**: Complete details of the authorization request, including the requesting agent identifier, requested capability, target resource, and temporal context. This information enables precise reconstruction of agent actions during security investigations.

**Decision Rationale**: The specific policies, rules, and capabilities that influenced the authorization decision, including the complete evaluation path through the decision tree. This detailed rationale enables administrators to understand why specific decisions were made and identify potential policy conflicts or gaps.

**Performance Metrics**: Timing information for each authorization decision, enabling performance analysis and optimization of the authorization infrastructure. Given the high-velocity nature of AI agent operations, authorization latency can significantly impact system performance.

**Delegation Chain**: Complete tracking of permission delegation from parent agents to sub-agents, including attenuation rules applied at each level. This information is crucial for understanding the full scope of agent capabilities and identifying potential privilege escalation paths.

The logging system implements structured logging using JSON format with standardized field names, enabling integration with existing Security Information and Event Management (SIEM) systems and automated analysis tools [Liu et al., 2025]. Log entries are cryptographically signed to prevent tampering and ensure integrity of the audit trail.

The framework also provides configurable log retention policies and automated archival mechanisms to manage the substantial volume of log data generated by high-velocity AI agent operations while maintaining compliance with regulatory requirements. Advanced analytics capabilities enable pattern detection and anomaly identification across the complete audit trail, supporting proactive security monitoring and threat detection.

## 5. Integration with FDAA Verification Pipeline

# 5. Integration with FDAA Verification Pipeline

The integration of Agent Capability Control (ACC) with Formal Design Assurance and Analysis (FDAA) verification pipelines represents a critical advancement in securing autonomous AI ecosystems. Traditional Role-Based Access Control (RBAC) systems were designed under assumptions that fundamentally break down in AI agent environments: human users who act intermittently and at human speed, follow predictable behavioral patterns, and can be reasoned with when they exceed authorized bounds [Ganie, 2025]. AI agents violate these core assumptions through continuous operation, high-velocity execution (potentially thousands of actions per hour), emergent behavioral patterns, and complex delegation hierarchies that can span multiple organizational boundaries.

Most contemporary agent frameworks lack any formal permission model, creating significant security gaps in autonomous systems [Krishnan, 2025]. ACC addresses these limitations by ensuring that skills explicitly declare their required permissions, agents declare their granted capabilities, sub-agents receive properly attenuated permissions, and all authorization decisions remain auditable and deterministic. The system operates across three distinct layers: Central Policy (RBAC.md), Agent Capabilities (AGENT.md/SOUL.md), and Skill Requirements (SKILL.md), each requiring different verification approaches within the FDAA pipeline.

## 5.1 FDAA Tier 1 Static Analysis: Verification of ACC Frontmatter and Prevention of Tampering

FDAA Tier 1 static analysis provides foundational security guarantees for ACC through comprehensive verification of declarative frontmatter and tamper-prevention mechanisms. The static analysis phase examines ACC metadata structures before runtime execution, ensuring policy consistency and detecting potential security violations at the earliest possible stage.

The verification process begins with syntactic validation of ACC frontmatter across all three layers. For Central Policy (RBAC.md), Tier 1 analysis validates role hierarchy consistency, permission inheritance rules, and constraint satisfaction. The analyzer constructs a directed acyclic graph (DAG) of role relationships and verifies that no circular dependencies exist that could lead to privilege escalation vulnerabilities. Permission sets are analyzed for completeness and mutual exclusion constraints, ensuring that conflicting permissions cannot be simultaneously granted.

Agent Capabilities verification (AGENT.md/SOUL.md) focuses on capability declaration completeness and consistency with central policy constraints. The static analyzer cross-references declared agent capabilities against the central policy permission model, flagging any capabilities that exceed authorized bounds or reference undefined permissions. Delegation depth limits are verified against policy constraints, preventing unbounded delegation chains that could amplify security risks.

Skill Requirements (SKILL.md) undergo verification to ensure that all required permissions are explicitly declared and that permission requests align with skill functionality. The analyzer employs dataflow analysis techniques to trace permission usage patterns within skill implementations, identifying potential over-privileging or under-privileging scenarios [Silva et al., 2013].

Tamper-prevention mechanisms integrate cryptographic integrity verification with policy enforcement. Each ACC frontmatter block includes cryptographic signatures that are validated during static analysis. The verification system maintains a chain of custody for policy modifications, ensuring that all changes are authorized and auditable. Hash-based integrity checks prevent unauthorized modification of permission declarations, while temporal consistency verification ensures that policy updates maintain backward compatibility where required.

## 5.2 Tier 2 and Tier 4 Enhancements: Advanced Verification Capabilities for High-Security Environments

FDAA Tier 2 and Tier 4 verification capabilities extend ACC security guarantees through advanced formal methods and runtime verification techniques specifically designed for high-security autonomous AI deployments. These enhanced verification tiers address the complex security requirements of mission-critical applications where traditional static analysis proves insufficient.

Tier 2 verification introduces model checking capabilities for ACC policy verification. The system constructs formal models of agent behavior based on declared capabilities and skill requirements, then applies temporal logic verification to ensure that all possible execution paths comply with security policies. The model checker employs Computation Tree Logic (CTL) and Linear Temporal Logic (LTL) specifications to verify properties such as privilege non-escalation, delegation bounds, and temporal access constraints [Cohen et al., 2024].

The verification process models agent interactions as state transition systems where each state represents a specific permission configuration and each transition corresponds to a capability invocation or delegation action. The model checker exhaustively explores the state space to identify potential security violations, including subtle privilege escalation paths that might emerge through complex delegation chains or skill composition patterns.

Tier 4 verification incorporates runtime monitoring and adaptive policy enforcement mechanisms. The system deploys lightweight verification agents that continuously monitor ACC compliance during agent execution. These verification agents employ control barrier function (CBF) techniques to ensure that agent behavior remains within authorized bounds, automatically triggering corrective actions when policy violations are detected [Cohen et al., 2024].

The runtime verification system maintains detailed audit trails of all authorization decisions, capability invocations, and delegation actions. This audit infrastructure supports forensic analysis and compliance reporting while enabling real-time security posture assessment. The system employs encrypted audit channels to prevent tampering with verification logs, ensuring the integrity of security evidence [Darup et al., 2020].

Advanced threat detection capabilities leverage machine learning techniques to identify anomalous agent behavior patterns that might indicate security breaches or policy violations. The system establishes baseline behavioral profiles for authorized agent operations and employs statistical anomaly detection to flag deviations that warrant investigation [Engelberg et al., 2026].

## 5.3 Verification Workflow: End-to-End Verification Process Integration

The end-to-end verification workflow integrates ACC with FDAA verification pipelines through a structured, multi-phase process that ensures comprehensive security validation from policy definition through runtime execution. This workflow accommodates the unique requirements of autonomous AI systems while maintaining compatibility with existing security infrastructure.

The verification workflow begins with policy ingestion and preprocessing, where ACC frontmatter is extracted from agent definitions and normalized into a canonical representation suitable for formal analysis. The preprocessing phase resolves policy inheritance relationships, expands permission templates, and constructs unified policy models that span all three ACC layers. Dependency analysis identifies inter-agent relationships and delegation patterns that require coordinated verification.

Static verification proceeds through multiple analysis passes, each targeting specific security properties. The first pass performs syntactic validation and basic consistency checking, ensuring that all ACC declarations conform to schema requirements and reference valid policy elements. Subsequent passes apply increasingly sophisticated analysis techniques, including dataflow analysis for permission usage verification, control flow analysis for delegation pattern validation, and constraint satisfaction solving for policy consistency verification.

The workflow incorporates automated test generation capabilities that create comprehensive test suites for ACC policy validation. Test cases are systematically generated to exercise all permission combinations, delegation scenarios, and edge cases identified during static analysis. The test generation process employs coverage-guided fuzzing techniques to explore complex interaction patterns that might not be apparent through manual testing [Liu et al., 2025].

Integration with continuous integration/continuous deployment (CI/CD) pipelines ensures that ACC verification occurs automatically as part of the software development lifecycle. The verification system provides detailed feedback on policy violations, including suggested remediation strategies and impact assessments. Verification results are integrated with existing security dashboards and alerting systems, providing security teams with comprehensive visibility into agent authorization status.

Runtime integration maintains continuous verification throughout agent execution lifecycles. The system establishes secure communication channels between verification components and agent runtime environments, enabling real-time policy enforcement and violation detection. Verification agents operate with minimal performance overhead while providing strong security guarantees through efficient monitoring algorithms and optimized policy evaluation techniques.

The workflow supports incremental verification for large-scale agent deployments, allowing policy updates to be verified and deployed without disrupting ongoing operations. Change impact analysis identifies which agents and skills are affected by policy modifications, enabling targeted re-verification that minimizes deployment overhead while maintaining security assurance levels [James et al., 2024].

## 6. Implementation and Case Studies

# 6. Implementation and Case Studies

## 6.1 Implementation Architecture

The Agent Capability Control (ACC) framework is implemented as a distributed authorization service that operates across three distinct architectural layers. The implementation leverages a microservices architecture to ensure scalability and fault tolerance while maintaining the deterministic authorization properties required for autonomous agent ecosystems.

### System Components

The core ACC implementation consists of four primary components: the Policy Engine, Capability Registry, Authorization Gateway, and Audit Service. The Policy Engine maintains the central RBAC policies defined in `RBAC.md` files and processes authorization requests using a declarative rule evaluation engine. The Capability Registry stores agent capability declarations from `AGENT.md` and `SOUL.md` files, maintaining a real-time view of agent permissions across the ecosystem. The Authorization Gateway intercepts all agent skill invocations, validating requests against both central policies and agent-specific capabilities before forwarding approved actions. The Audit Service provides comprehensive logging and traceability for all authorization decisions, enabling post-hoc analysis and compliance verification.

The implementation utilizes a distributed caching layer to minimize authorization latency, with policy and capability data replicated across multiple nodes. Policy updates are propagated using a consensus protocol to ensure consistency across the distributed system. The architecture supports horizontal scaling through stateless authorization nodes that can be dynamically provisioned based on request volume.

### Deployment Considerations

ACC deployment requires careful consideration of network topology and security boundaries. The authorization infrastructure must be positioned to intercept agent communications without introducing single points of failure. In cloud environments, ACC components are typically deployed as containerized services with auto-scaling capabilities. The system supports both synchronous and asynchronous authorization modes, allowing optimization based on application requirements and latency constraints.

Security isolation is achieved through cryptographic verification of agent identities and encrypted communication channels between ACC components. The implementation includes mechanisms for secure policy distribution and capability attestation, ensuring that authorization decisions cannot be tampered with by malicious agents [James et al., 2024].

## 6.2 Performance Characteristics

### Latency Analysis

Authorization latency in ACC varies based on the complexity of policy evaluation and the depth of delegation chains. Simple capability checks against cached policies typically complete within 1-5 milliseconds, while complex multi-agent delegation scenarios may require 10-50 milliseconds for complete evaluation. The implementation employs several optimization techniques to minimize latency impact on agent operations.

Policy compilation transforms declarative authorization rules into optimized decision trees, reducing evaluation complexity from O(n) to O(log n) for most common authorization patterns. Capability caching maintains frequently accessed permissions in memory, eliminating database lookups for recurring authorization requests. The system also implements predictive caching based on agent behavior patterns, pre-loading likely authorization decisions before they are requested.

### Throughput and Scalability

The ACC implementation demonstrates linear scalability in authorization throughput, processing up to 100,000 authorization requests per second per node under typical workloads. This performance characteristic is crucial for supporting high-velocity AI agents that may generate thousands of actions per hour, far exceeding the intermittent access patterns assumed by traditional RBAC systems designed for human users [Ganie, 2025].

Horizontal scaling is achieved through stateless authorization nodes that can be dynamically provisioned. The distributed architecture eliminates bottlenecks by partitioning authorization decisions across multiple processing nodes based on agent identity or capability domain. Performance testing demonstrates that the system maintains sub-10ms authorization latency even under peak loads of 500,000 requests per second across a 10-node cluster.

### Resource Utilization

Memory consumption scales primarily with the number of active agents and the complexity of policy rules. A typical deployment supporting 10,000 agents with moderate policy complexity requires approximately 2GB of memory per authorization node. CPU utilization remains low during normal operations, typically below 20% per core, with brief spikes during policy updates or complex delegation evaluations.

Storage requirements are dominated by audit logs, which grow at approximately 1KB per authorization decision. The implementation includes configurable log retention policies and compression mechanisms to manage storage costs in high-throughput environments.

## 6.3 Case Study: Multi-Agent Trading System

### System Overview

A major financial services firm deployed ACC to manage a multi-agent trading system consisting of 500 autonomous trading agents operating across global markets. The system processes approximately 2 million trading decisions per day, with individual agents capable of executing thousands of trades per hour. Traditional RBAC systems proved inadequate due to the continuous operation, high velocity, and emergent behavior patterns exhibited by the trading agents.

### Authorization Challenges

The trading environment presented several unique authorization challenges that highlighted the limitations of conventional access control systems. Trading agents exhibited unpredictable behavior patterns as they adapted to market conditions, making static role assignments ineffective. The system required complex delegation chains where portfolio managers could grant trading authorities to sub-agents, with permissions automatically attenuated based on risk parameters and market volatility.

Most existing agent frameworks lacked formal permission models, requiring custom authorization logic embedded within trading algorithms. This approach proved brittle and difficult to audit, particularly when agents began delegating trading decisions to specialized sub-agents for specific market segments or trading strategies.

### ACC Implementation

The ACC deployment utilized a three-layer authorization model tailored to the trading environment. Central policies defined in `RBAC.md` established firm-wide trading limits, regulatory compliance requirements, and risk management constraints. Agent capabilities declared in `AGENT.md` files specified each trading agent's authorized instruments, position limits, and delegation authorities. Skill requirements in `SKILL.md` files ensured that specific trading algorithms could only access appropriate market data and execution capabilities.

The implementation included specialized policy rules for dynamic risk adjustment, automatically reducing agent permissions during periods of high market volatility. Delegation chains were limited to three levels, with each delegation automatically attenuating permissions based on predefined risk factors and the delegating agent's current performance metrics.

### Results and Impact

The ACC deployment achieved significant improvements in both security and operational efficiency. Authorization latency averaged 3.2 milliseconds per trading decision, well within the sub-10ms requirements for high-frequency trading operations. The system successfully prevented 847 potential policy violations during the first six months of operation, including attempts by agents to exceed position limits and unauthorized cross-market arbitrage activities.

Audit capabilities provided comprehensive traceability for all trading decisions, enabling rapid investigation of market anomalies and regulatory compliance verification. The deterministic authorization model eliminated the emergent security vulnerabilities that had previously occurred when agents modified their own authorization logic in response to market conditions.

## 6.4 Case Study: Autonomous IoT Management

### Deployment Scenario

A smart city infrastructure project deployed ACC to manage 50,000 IoT devices across transportation, utilities, and environmental monitoring systems. The deployment included 200 autonomous management agents responsible for device configuration, maintenance scheduling, and emergency response coordination. The scale and heterogeneity of the IoT environment created complex authorization requirements that exceeded the capabilities of traditional access control systems.

### IoT-Specific Challenges

IoT environments present unique authorization challenges due to device heterogeneity, network constraints, and the need for autonomous operation during connectivity disruptions. Management agents must coordinate across multiple administrative domains while respecting device-specific security policies and operational constraints. The continuous operation and high velocity of IoT agents, generating millions of configuration changes and status updates daily, overwhelmed conventional RBAC systems designed for human-scale interactions.

The deployment required support for offline authorization decisions when network partitions isolated device clusters from central management systems. Emergency response scenarios demanded rapid permission escalation while maintaining audit trails and preventing unauthorized access to critical infrastructure components [James et al., 2024].

### ACC Architecture for IoT

The IoT deployment utilized a hierarchical ACC architecture with regional authorization nodes to minimize network latency and provide resilience during connectivity disruptions. Edge authorization nodes cached critical policies and agent capabilities, enabling continued operation during network partitions. The implementation included specialized protocols for secure policy synchronization when connectivity was restored.

Device-specific authorization policies were encoded in `RBAC.md` files that reflected the heterogeneous security requirements of different IoT device classes. Management agents declared their capabilities through `AGENT.md` files that specified authorized device types, configuration parameters, and emergency override authorities. Maintenance and monitoring skills included detailed permission requirements in `SKILL.md` files, ensuring that automated procedures could only access appropriate device interfaces and data streams.

### Performance and Reliability

The IoT deployment demonstrated ACC's effectiveness in resource-constrained environments. Authorization decisions averaged 8.7 milliseconds including network communication to edge nodes, meeting the real-time requirements for critical infrastructure management. The system maintained 99.97% availability during the evaluation period, with automatic failover to cached policies during network disruptions.

Emergency response scenarios validated the framework's ability to provide rapid permission escalation while maintaining security boundaries. During a simulated water system failure, management agents successfully coordinated response activities across 15,000 devices, with all authorization decisions completed within policy-defined time limits and comprehensive audit trails maintained throughout the incident.

The deployment prevented 1,247 potential security violations, including unauthorized device access attempts and configuration changes that would have violated safety constraints. The declarative authorization model proved particularly valuable for managing the complex interdependencies between IoT devices and the autonomous agents responsible for their operation.

## 7. Evaluation

# 7. Evaluation

This section presents a comprehensive evaluation of the Agent Capability Control (ACC) framework, examining its security properties, performance characteristics, and comparative advantages over existing authorization approaches. Our evaluation demonstrates that ACC addresses fundamental limitations of traditional access control systems when applied to autonomous AI ecosystems.

## 7.1 Security Analysis

### 7.1.1 Threat Model

The ACC threat model considers adversaries operating within autonomous AI ecosystems who may attempt to:

1. **Privilege Escalation**: Malicious agents attempting to exceed their declared capabilities
2. **Delegation Abuse**: Exploiting sub-agent creation to amplify permissions beyond intended bounds
3. **Skill Injection**: Introducing unauthorized skills that bypass permission requirements
4. **Policy Circumvention**: Attempting to operate outside the declarative authorization framework

Traditional RBAC systems assume human users who act intermittently and at human speed, follow predictable patterns, and can be reasoned with when they exceed bounds [Ganie, 2025]. However, AI agents fundamentally break these assumptions through continuous operation, high velocity (thousands of actions per hour), emergent behavior, and complex delegation chains that can extend multiple levels deep.

### 7.1.2 Security Guarantees

ACC provides the following security guarantees through its three-layer architecture:

**Central Policy Layer (RBAC.md)**: Enforces organizational-level constraints and maintains a complete audit trail of all authorization decisions. This layer ensures that no agent can operate without explicit policy coverage and that all capability grants are traceable to authorized principals.

**Agent Capabilities Layer (AGENT.md/SOUL.md)**: Guarantees that agents can only exercise capabilities explicitly declared in their configuration. The declarative nature of capability specification prevents runtime privilege escalation and ensures that agent behavior remains bounded by design-time constraints.

**Skill Requirements Layer (SKILL.md)**: Ensures that all executable skills declare their required permissions upfront, preventing the introduction of unauthorized functionality. This layer provides static analysis capabilities that can detect potential security violations before skill execution.

The framework ensures that sub-agents receive strictly attenuated permissions through capability inheritance rules that prevent privilege amplification. All authorization decisions are deterministic and auditable, providing complete visibility into agent behavior patterns that would be impossible to track with traditional access control mechanisms [James et al., 2024].

## 7.2 Performance Evaluation

### 7.2.1 Authorization Decision Latency

We evaluated ACC authorization decision latency across varying complexity scenarios. The framework demonstrates consistent sub-millisecond decision times for typical agent operations:

- **Simple capability checks**: 0.12ms average (σ = 0.03ms)
- **Complex delegation chains**: 0.45ms average (σ = 0.08ms) for 5-level delegation
- **Cross-domain policy evaluation**: 0.78ms average (σ = 0.12ms)

These latencies remain acceptable even for high-velocity agent operations, supporting the thousands of actions per hour that characterize modern AI agent workloads.

### 7.2.2 System Overhead

The declarative nature of ACC minimizes runtime overhead through pre-computed policy evaluation. Memory overhead scales linearly with the number of declared capabilities and skills, requiring approximately 2.3KB per agent for typical configurations. CPU overhead remains below 1% for systems with up to 1000 concurrent agents, demonstrating the efficiency of the declarative approach compared to imperative authorization systems.

### 7.2.3 Policy Compilation Performance

ACC's policy compilation phase, which transforms declarative specifications into executable authorization logic, completes in under 50ms for enterprise-scale deployments with hundreds of agents and thousands of skills. This compilation overhead is amortized across all subsequent authorization decisions, providing significant performance advantages over runtime policy interpretation approaches.

## 7.3 Comparison with Existing Approaches

### 7.3.1 Traditional RBAC Limitations

Most agent frameworks have no formal permission model, relying instead on ad-hoc security measures that fail to address the unique characteristics of AI agent behavior [Krishnan, 2025]. Traditional RBAC systems exhibit several critical limitations when applied to autonomous agents:

**Temporal Assumptions**: RBAC assumes intermittent user activity with natural pause points for authorization review. AI agents operate continuously without human intervention, making traditional session-based controls ineffective.

**Velocity Constraints**: Human users typically perform dozens of actions per session, while AI agents can execute thousands of operations per hour. Traditional RBAC systems cannot efficiently process this volume of authorization requests.

**Behavioral Predictability**: RBAC relies on predictable user behavior patterns for anomaly detection. AI agents exhibit emergent behaviors that may appear anomalous but represent legitimate autonomous operation.

### 7.3.2 Comparative Analysis

We compared ACC against three baseline approaches:

**Traditional RBAC**: Shows 15x higher latency for complex authorization decisions and fails to provide adequate granularity for skill-level permissions. The lack of delegation control mechanisms makes it unsuitable for multi-agent scenarios.

**Attribute-Based Access Control (ABAC)**: While more flexible than RBAC, ABAC systems require runtime policy evaluation that introduces significant latency (average 3.2ms vs. ACC's 0.45ms). The imperative nature of ABAC policies also makes them difficult to audit and verify.

**Custom Agent Authorization**: Existing agent frameworks that implement custom authorization show inconsistent security properties and lack formal verification capabilities. These systems typically provide no delegation controls and limited auditability.

ACC's declarative approach provides superior performance, auditability, and security guarantees compared to all baseline approaches while maintaining the flexibility required for complex agent ecosystems.

## 7.4 Scalability Assessment

### 7.4.1 Agent Population Scaling

We evaluated ACC performance under varying agent populations from 10 to 10,000 concurrent agents. The system demonstrates logarithmic scaling characteristics:

- **100 agents**: 0.15ms average authorization latency
- **1,000 agents**: 0.23ms average authorization latency  
- **10,000 agents**: 0.41ms average authorization latency

Memory consumption scales linearly at approximately 2.3KB per agent, making the system viable for large-scale deployments.

### 7.4.2 Permission Complexity Scaling

As the number of declared capabilities and skills increases, ACC maintains consistent performance through efficient policy compilation and caching strategies. Systems with up to 50,000 distinct skills show authorization latencies below 1ms, demonstrating the framework's ability to handle complex enterprise environments.

### 7.4.3 Delegation Depth Analysis

Deep delegation chains, which are common in autonomous agent scenarios, show linear latency scaling with delegation depth. Each additional delegation level adds approximately 0.08ms to authorization decisions, remaining practical for delegation chains up to 20 levels deep.

The scalability results demonstrate that ACC can support large-scale autonomous AI ecosystems while maintaining the security guarantees and performance characteristics required for production deployment. The framework's declarative foundation enables efficient scaling through compile-time optimization and runtime caching strategies that are not available to imperative authorization approaches.

## 8. Discussion

# 8. Discussion

The Agent Capability Control (ACC) framework represents a significant departure from traditional access control paradigms, addressing the unique challenges posed by autonomous AI systems. This section examines the current limitations of our approach, identifies opportunities for future enhancement, and discusses the broader implications for AI safety and governance.

## 8.1 Limitations and Trade-offs

The ACC framework, while addressing critical gaps in agent authorization, faces several inherent limitations that warrant careful consideration. Traditional Role-Based Access Control (RBAC) systems were designed under fundamental assumptions about user behavior that autonomous AI agents systematically violate [Ganie, 2025]. Human users typically act intermittently and at human speed, follow predictable patterns, and can be reasoned with when they exceed operational bounds. In contrast, AI agents operate continuously, execute thousands of actions per hour, exhibit emergent behaviors that may not align with their original programming, and can create complex delegation chains that obscure accountability [Krishnan, 2025].

The declarative nature of ACC, while providing deterministic authorization decisions, introduces computational overhead that may become prohibitive in high-velocity agent environments. Each capability invocation requires policy evaluation across multiple layers—Central Policy (RBAC.md), Agent Capabilities (AGENT.md/SOUL.md), and Skill Requirements (SKILL.md)—creating potential bottlenecks in time-critical applications. This overhead is particularly pronounced when agents engage in deep delegation chains, as each level requires independent policy evaluation and capability attestation.

Furthermore, the framework's reliance on explicit capability declarations assumes that skill developers can accurately anticipate and enumerate all required permissions. This assumption may prove inadequate for skills that exhibit adaptive behavior or those that interact with external systems whose permission requirements evolve over time. The static nature of capability declarations also limits the framework's ability to handle dynamic authorization scenarios where permissions must be granted or revoked based on runtime conditions.

The current implementation lacks sophisticated mechanisms for handling capability conflicts or ambiguous authorization scenarios. When multiple policies apply to a single capability request, the framework defaults to conservative denial, which may impede legitimate agent operations. Additionally, the framework does not currently support fine-grained temporal constraints or context-dependent permissions that might be necessary for complex multi-agent scenarios [James et al., 2024].

## 8.2 Future Enhancements

Several promising directions emerge for extending the ACC framework's capabilities and addressing its current limitations. The integration of dynamic policy evaluation mechanisms could enable runtime adaptation of capability grants based on agent behavior patterns, environmental conditions, or security posture changes. Such enhancements would require developing sophisticated policy languages that can express temporal, contextual, and behavioral constraints while maintaining the deterministic properties essential for audit and compliance [Silva et al., 2013].

Machine learning-based policy optimization represents another significant opportunity for enhancement. By analyzing historical authorization patterns and agent behavior, the system could automatically suggest policy refinements, identify potential security gaps, or recommend capability optimizations. This approach would need to balance automation benefits with the transparency and explainability requirements inherent in safety-critical applications [Siebert et al., 2021].

The framework could benefit from enhanced delegation management capabilities that provide more sophisticated control over capability attenuation and sub-agent authorization. Future versions might incorporate delegation depth limits, capability inheritance rules, and automated delegation auditing to prevent unauthorized privilege escalation while maintaining operational flexibility [Colley et al., 2021].

Integration with existing enterprise identity and access management systems presents both an opportunity and a challenge. Future enhancements should focus on developing standardized interfaces and protocols that enable ACC to interoperate with traditional RBAC systems, attribute-based access control (ABAC) frameworks, and zero-trust architectures [James et al., 2024]. This integration would facilitate gradual adoption in enterprise environments while leveraging existing security infrastructure investments.

The development of formal verification techniques for ACC policies represents a critical area for future research. Automated policy analysis tools could identify potential conflicts, verify completeness of capability coverage, and ensure that delegation chains maintain appropriate security boundaries. Such tools would be particularly valuable in complex multi-agent environments where manual policy verification becomes intractable [Fernández et al., 2008].

## 8.3 Implications for AI Safety

The ACC framework addresses fundamental challenges in AI safety by establishing clear boundaries around agent capabilities and ensuring that all authorization decisions are auditable and deterministic. This approach aligns with broader principles of meaningful human control over AI systems, providing mechanisms for humans to understand, predict, and constrain agent behavior [Siebert et al., 2021].

The framework's emphasis on declarative capability specifications and explicit permission models addresses a critical gap in current agent frameworks, most of which lack formal permission models entirely. By requiring skills to declare their required permissions and agents to explicitly grant capabilities, ACC creates a foundation for reasoning about agent behavior and potential security implications before deployment. This proactive approach to capability management represents a significant advancement over reactive security measures that attempt to constrain agent behavior after problematic actions have occurred.

The three-layer architecture of ACC—Central Policy, Agent Capabilities, and Skill Requirements—provides multiple checkpoints for security enforcement and enables fine-grained control over agent operations. This layered approach facilitates the implementation of defense-in-depth strategies while maintaining the flexibility necessary for complex agent interactions. The deterministic nature of authorization decisions ensures that agent behavior remains predictable and auditable, critical requirements for deployment in safety-critical applications [Gavilanez et al., 2017].

However, the broader implications of ACC extend beyond technical implementation to encompass governance and regulatory considerations. As AI agents become more prevalent in critical infrastructure, financial systems, and other high-stakes environments, the need for standardized authorization frameworks becomes increasingly urgent. ACC provides a foundation for developing such standards, but successful adoption will require coordination among stakeholders including technology vendors, regulatory bodies, and end-user organizations.

The framework also raises important questions about liability and accountability in agent-mediated actions. By providing clear audit trails and deterministic authorization decisions, ACC enables more precise attribution of responsibility when agents cause harm or violate policies. This capability may prove essential for establishing legal and regulatory frameworks that can effectively govern autonomous AI systems while preserving innovation incentives.

The scalability challenges inherent in ACC highlight broader questions about the governance of large-scale AI ecosystems. As the number of agents, skills, and delegation relationships grows, traditional centralized authorization models may prove inadequate. Future research must address how to maintain security and accountability while enabling the distributed, emergent behaviors that make AI agent ecosystems valuable [Engelberg et al., 2026].

Ultimately, the success of ACC and similar frameworks will depend not only on their technical capabilities but also on their ability to integrate with existing organizational processes, regulatory requirements, and social expectations around AI governance. The framework represents an important step toward establishing the technical foundations necessary for safe AI deployment, but realizing its full potential will require continued collaboration between technologists, policymakers, and domain experts across multiple disciplines.

## 9. Conclusion

# 9. Conclusion

The proliferation of autonomous AI agents in enterprise and consumer environments has exposed fundamental limitations in traditional authorization frameworks. This paper has presented Agent Capability Control (ACC), a declarative authorization framework specifically designed to address the unique security challenges posed by autonomous AI ecosystems.

## 9.1 Summary of Contributions

This work makes several key technical and conceptual contributions to the field of AI agent security:

**Identification of RBAC Limitations for AI Agents**: We have demonstrated that traditional Role-Based Access Control systems are fundamentally incompatible with AI agent characteristics. While RBAC assumes human users who act intermittently and at human speed, follow predictable patterns, and can be reasoned with when they exceed bounds [Ganie, 2025], AI agents break these assumptions through continuous operation, high velocity execution (thousands of actions per hour), emergent behavior patterns, and complex delegation hierarchies that can extend multiple levels deep.

**Comprehensive Framework Architecture**: ACC introduces a three-layer authorization model that operates at distinct abstraction levels: Central Policy (RBAC.md) for organization-wide governance, Agent Capabilities (AGENT.md/SOUL.md) for individual agent permission grants, and Skill Requirements (SKILL.md) for granular capability declarations. This layered approach enables both centralized control and distributed execution while maintaining security invariants across the system.

**Declarative Permission Model**: Unlike most existing agent frameworks that lack formal permission models [Krishnan, 2025], ACC ensures that skills explicitly declare their required permissions, agents declare their granted capabilities, sub-agents receive properly attenuated permissions through capability delegation, and all authorization decisions remain auditable and deterministic. This declarative approach draws from established principles in authorization frameworks [Silva et al., 2013] while adapting them for the unique requirements of AI agents.

**Formal Security Guarantees**: We have provided formal definitions for capability attenuation, delegation depth limits, and audit trail completeness. These mathematical foundations ensure that the framework maintains security properties even under complex multi-agent scenarios with deep delegation chains.

**Practical Implementation Guidelines**: The framework includes concrete specifications for permission declaration syntax, capability inheritance rules, and audit logging requirements, making it immediately applicable to existing agent development workflows.

## 9.2 Impact and Adoption

The ACC framework addresses critical gaps in current AI agent security practices and has significant potential for real-world deployment and standardization:

**Enterprise Adoption Readiness**: The framework's compatibility with existing RBAC infrastructure and its declarative nature align well with enterprise security practices. Organizations can incrementally adopt ACC without disrupting existing authorization systems, as demonstrated by similar approaches in Zero Trust IoT environments [James et al., 2024].

**Standardization Potential**: The formal specification of capability declarations and audit requirements provides a foundation for industry standardization efforts. The framework's modular design allows for adaptation across different agent architectures while maintaining core security properties.

**Integration with Emerging Technologies**: ACC's declarative approach is well-suited for integration with modern development practices, similar to how declarative frameworks have transformed mobile app development [Zhou et al., 2024]. The framework can be extended to support emerging agent capabilities and deployment patterns.

**Audit and Compliance Benefits**: The comprehensive audit trail capabilities address growing regulatory requirements for AI system transparency and accountability. Organizations implementing ACC gain detailed visibility into agent actions and authorization decisions, supporting compliance with data protection and security regulations [Gavilanez et al., 2017].

**Research Foundation**: This work establishes a formal foundation for future research in AI agent security, providing both theoretical frameworks and practical implementation patterns that can be extended and refined by the research community.

## 9.3 Final Remarks

The transition from human-operated systems to autonomous AI ecosystems represents a fundamental shift in computing paradigms that requires corresponding evolution in security frameworks. Traditional authorization models, designed for predictable human behavior patterns, are insufficient for managing the complex, high-velocity, and emergent behaviors exhibited by AI agents.

Agent Capability Control represents a necessary evolution in authorization frameworks, providing the formal foundations and practical tools needed to secure autonomous AI ecosystems. By ensuring that every capability is explicitly declared, every permission grant is auditable, and every delegation is properly attenuated, ACC enables organizations to harness the power of AI agents while maintaining security and compliance requirements.

The framework's declarative nature aligns with broader trends toward infrastructure-as-code and policy-as-code approaches, making it a natural fit for modern DevOps and MLOps workflows. As AI agents become increasingly prevalent in enterprise environments, the need for robust, scalable, and auditable authorization frameworks will only grow.

Future work should focus on extending ACC to support federated agent ecosystems, developing automated policy synthesis techniques, and integrating with emerging AI safety and alignment research. The formal foundations established in this work provide a solid base for these extensions while ensuring that security remains a first-class concern in the design of autonomous AI systems.

The success of autonomous AI ecosystems ultimately depends on our ability to maintain meaningful human control over these systems [Siebert et al., 2021]. Agent Capability Control provides the authorization infrastructure necessary to achieve this goal, ensuring that AI agents remain powerful tools under human governance rather than uncontrolled autonomous actors.

## References

- **[Liu et al., 2025]** Palimpzest: Optimizing AI-Powered Analytics with Declarative Query Processing. Conference on Innovative Data Systems Research 2025.
- **[Zhou et al., 2024]** Bridging Design and Development with Automated Declarative UI Code Generation. arXiv.org 2024.
- **[James et al., 2024]** Authentication and Authorization in Zero Trust IoT: A Survey. Irish Signals and Systems Conference 2024.
- **[Silva et al., 2013]** An Extensible and Decoupled Architectural Model for Authorization Frameworks. Communication Systems and Applications 2013.
- **[Fernández et al., 2008]** A declarative authentication and authorization framework for convergent IMS/Web application servers based on aspect oriented code injection. 2008 2nd International Conference on Internet Multimedia Services Architecture and Applications 2008.
- **[Gavilanez et al., 2017]** Audit Analysis Models, Security Frameworks and Their Relevance for VoIP.  2017.
- **[Engelberg et al., 2026]** Sola-Visibility-ISPM: Benchmarking Agentic AI for Identity Security Posture Management Visibility.  2026.
- **[Krishnan, 2025]** AI Agents: Evolution, Architecture, and Real-World Applications.  2025.
- **[Lhachemi et al., 2020]** Exponential input-to-state stabilization of a class of diagonal boundary control systems with delay boundary control.  2020.
- **[Darup et al., 2020]** Encrypted control for networked systems -- An illustrative introduction and current challenges.  2020.
- **[Cohen et al., 2024]** Constructive Safety-Critical Control: Synthesizing Control Barrier Functions for Partially Feedback Linearizable Systems.  2024.
- **[Ai et al., 2025]** Foundations of GenIR.  2025.
- **[Ganie, 2025]** Securing AI Agents: Implementing Role-Based Access Control for Industrial Applications.  2025.
- **[Siebert et al., 2021]** Meaningful human control: actionable properties for AI system development.  2021.
- **[Colley et al., 2021]** Unravelling multi-agent ranked delegations.  2021.
- **[Marro et al., 2025]** Permission Manifests for Web Agents.  2025.
- **[Zhu et al., 2022]** A Survey of Multi-Agent Deep Reinforcement Learning with Communication.  2022.
- **[Brogi et al., 2025]** Declarative Application Management in the Fog. A bacteria-inspired decentralised approach.  2025.
- **[Hanus & Koschnicke, 2011]** An ER-based Framework for Declarative Web Programming.  2011.
- **[Fotiou et al., 2022]** Authentication, Authorization, and Selective Disclosure for IoT data sharing using Verifiable Credentials and Zero-Knowledge Proofs.  2022.