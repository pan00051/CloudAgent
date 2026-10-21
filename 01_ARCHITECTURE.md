# Personal Agent System V1
# 01 · Architecture Specification

**Document ID:** ARCH-001  
**Filename:** `01_ARCHITECTURE.md`  
**Version:** 0.2.0  
**Status:** DRAFT — Pending Independent Review  
**Date:** 2026-10-21  
**Owner:** Human System Owner  
**Applies To:** Personal Agent System V1  
**Acceptance Baseline:** GOV-ACCEPT-001 v0.2  
**Authority:** System Architecture, Functional Boundaries & Architecture Invariants

---

## 1. Purpose, Scope & Authority

### 1.1 Purpose

This document defines the architectural foundation of Personal Agent System V1.

Its primary objectives are to:

1. Establish the system's functional structure and architectural boundaries.
2. Define L0–L6 responsibilities and cross-layer components.
3. Prevent unauthorized privilege inheritance between modules.
4. Preserve system maintainability and independent module evolution.
5. Establish architectural invariants that future implementations MUST preserve.
6. Provide a stable reference for new LLM maintainers and architecture reviewers.

### 1.2 Authority

This document is the authoritative source for:

- System architecture.
- Functional layer definitions.
- Component responsibilities.
- Architectural trust boundaries.
- Permitted architectural dependencies.
- Architecture invariants.
- Architectural upgrade constraints.

It does not independently authorize runtime actions.

The following documents hold authority over their respective subjects:

| Document | Authority |
|---|---|
| `02_SECURITY_POLICY.md` | Identity, credentials, authorization, isolation and security enforcement |
| `03_MODULE_CONTRACTS.md` | API contracts, schemas, state machines and transactional behavior |
| `04_OPERATIONS_RUNBOOK.md` | Deployment, maintenance, diagnostics, backup and recovery |
| `05_CHANGE_CONTROL.md` | Governance approval, rule publication, activation and rollback |
| `06_VALIDATION_STANDARD.md` | Tests, implementation acceptance and release requirements |

Cross-document conflicts MUST be reported and the affected change MUST be blocked pending resolution.

### 1.3 Normative Language

- **MUST:** Mandatory requirement.
- **MUST NOT:** Prohibited behavior.
- **SHOULD:** Recommended unless a justified exception is documented.
- **MAY:** Optional permitted behavior.

### 1.4 Implementation Status

This document describes the target architecture.

It MUST NOT be interpreted as evidence that the architecture has already been implemented or validated.

Unless explicitly verified, described mechanisms remain design requirements rather than deployed capabilities.

---

## 2. System Mission & V1 Scope

### 2.1 Mission

Build a personally operated AI system that can understand natural-language requests, develop plans, retrieve information, generate deliverables and execute selected actions under explicit authorization.

The system prioritizes:

1. Security and bounded authority.
2. Maintainability and modularity.
3. Reliable execution.
4. Intelligent planning.
5. Operational and financial efficiency.

**Core principle:**

Intelligence may evolve. Authority must remain explicitly controlled.

### 2.2 V1 Functional Scope

| ID | Capability | V1 Requirement |
|---|---|---|
| F1 | Chat & Planning | Multi-turn conversation and structured task planning |
| F2 | Memory | Controlled storage, retrieval, correction and deletion |
| F3 | Restricted Fetch | Approved URL, RSS and API retrieval with source evidence |
| F4 | Independent Audit | Independent review of plans, evidence and results |
| F5 | Controlled Delivery | Email drafting and explicitly approved delivery |
| F6 | Document Generation | CSV, XLSX, DOCX and Markdown generation |

### 2.3 Explicit Non-Goals

V1 does not include:

- Fully autonomous multi-agent operation.
- Autonomous production deployment.
- Autonomous governance or permission modification.
- Unrestricted web browsing or computer control.
- Automated bulk customer outreach.
- Full CRM functionality.
- AI avatar meeting participation.
- Advanced interactive dashboards.
- Multi-server automatic failover.
- Large local LLM inference on the primary server.

Future integration interfaces MAY be anticipated, but these functions MUST NOT be treated as implemented or authorized.

---

## 3. Architectural Style & Design Principles

### 3.1 Modular Monolith

V1 SHOULD use a modular monolith.

Business functionality SHOULD be maintained in independently testable modules with explicit contracts.

Logical modularity does not imply that every module needs its own process or container.

However, modules with conflicting trust requirements MUST have enforceable isolation, even when this requires separate runtime processes.

### 3.2 Deterministic Enforcement

LLMs MAY perform:

- Language understanding.
- Planning.
- Draft generation.
- Semantic analysis.
- Evidence review.
- Suggested workflow improvements.

Deterministic software MUST enforce:

- Authorization decisions.
- Resource limits.
- State transition validity.
- Approval validity.
- Execution restrictions.
- Protected credential access.
- Controlled persistence.

LLM responses MUST NOT be treated as inherently trusted authorization decisions.

### 3.3 No Implicit Privilege Inheritance

One module calling another MUST NOT inherit that module's privileged capabilities.

An Orchestrator that can request an operation MUST NOT automatically gain permission to perform that operation.

### 3.4 Runtime Independence

GitHub is a governance version source, not a mandatory online runtime dependency.

Production decisions MUST use the currently active, verified local policy snapshot.

### 3.5 Least Complexity

Complexity MUST be justified by actual V1 requirements.

The architecture SHOULD avoid unnecessary:

- Always-on agents.
- Independent model providers.
- Microservices.
- Message brokers.
- Vector databases.
- Kubernetes deployments.
- Cross-server synchronization.

Security-critical separation remains mandatory even when the rest of the application is consolidated.

### 3.6 Failure Containment

A failure in one module SHOULD NOT automatically grant privileges, corrupt unrelated modules or trigger uncontrolled retries.

Protected operations MUST fail closed when required security conditions are unavailable.

---

## 4. Logical System Architecture

### 4.1 Functional Structure

```text
Human Owner
    |
    v
L1 · Interface
    |
    v
Orchestrator [PROPOSED: n8n]
    |
    +----> L2 · Intelligence
    |          |
    |          +----> L6 · Authorized Memory Read
    |
    +----> L3 · Restricted Acquisition
    |          |
    |          +----> External Sources
    |
    +----> L4 · Independent Verification
    |
    +----> Protected Operation Request
                  |
                  v
           Policy Gate
                  |
             If required:
           Trusted Approval
                  |
                  v
          L5 · Restricted Execution
                  |
                  v
          L6 · Persistence
                  |
                  v
          L1 · Result Display


L0 · Governance
    |
    +----> Policies & Authorization Rules
    +----> Trusted Approval Authority
    +----> Governance Release Control
    +----> Emergency Stop Control
```

This diagram describes responsibilities and control boundaries, not physical container count.

### 4.2 Control Plane vs Data Plane

The architecture MUST separate two conceptual planes.

**Control Plane**

Responsible for:

- Identity.
- Authorization.
- Policy versions.
- Approval records.
- Risk classification.
- Tool permissions.
- Emergency control.
- Governance release decisions.

**Data Plane**

Responsible for:

- Conversations.
- External information.
- Task planning content.
- Memory records.
- Documents.
- Tool results.
- Business data.

Data Plane content MUST NOT modify Control Plane authority.

External text, model outputs and retrieved memories remain data unless independently authenticated through an authorized control mechanism.

---

## 5. L0–L6 Functional Layer Definitions

Layer numbers describe functional domains.

They do not represent privilege inheritance or a hierarchy where higher numbers have greater authority.

A runtime component MAY serve more than one functional layer only when the applicable security boundaries remain enforceable.

### 5.1 L0 · Governance

**Primary Responsibility:** Define and control system authority.

**Actors and components:**

- Human System Owner.
- Governance policies.
- Trusted approval authority.
- Governance release authority.
- Emergency control authority.

**Inputs:**

- Change requests.
- Security findings.
- Approval requests.
- Governance release candidates.
- Emergency reports.

**Outputs:**

- Authorized decisions.
- Approved policy versions.
- Release decisions.
- Emergency stop or recovery authorization.

**Allowed:**

- Approve governed changes.
- Define permitted capabilities.
- Authorize protected actions.
- Trigger emergency restrictions.

**Prohibited:**

- Treating an LLM-generated approval statement as Owner authorization.
- Delegating unrestricted governance authority to an ordinary Agent.
- Automatically activating proposed changes without required approval.

L0 is a governance function, not a single application process with unrestricted access to every component.

### 5.2 L1 · Interface

**Primary Responsibility:** Present the system to the authenticated human user.

**Inputs:**

- User instructions.
- Task results.
- Audit findings.
- Approval requests.

**Outputs:**

- Authenticated task submissions.
- Trusted approval or rejection requests.
- Result presentations.

**Allowed:**

- Display plans, reports and artifacts.
- Accept human instructions.
- Present protected operations for approval.

**Prohibited:**

- Treating ordinary chat text as a valid approval token.
- Allowing Planner or Orchestrator to impersonate the Owner.
- Storing protected execution credentials in client-side UI state.

Approval must pass through a trusted, authenticated channel independent of LLM-generated messages.

### 5.3 L2 · Intelligence

**Primary Responsibility:** Interpret, plan and reason.

**Components:**

- Chat Context Manager.
- Planner.
- LLM Provider Adapter.
- Authorized Memory Retrieval Interface.

**Inputs:**

- User requests.
- Authorized context.
- Retrieved data.
- Permitted memory references.

**Outputs:**

- Structured plans.
- Proposed tool calls.
- Drafts.
- Analyses.
- Suggested improvements.

**Allowed:**

- Recommend actions.
- Request authorized information.
- Propose task refinements.
- Suggest that a task requires higher risk classification.

**Prohibited:**

- Reducing assigned risk classification.
- Expanding its own `allowed_tools`.
- Generating binding authorization records.
- Directly using protected execution credentials.
- Publishing governance changes.
- Treating untrusted external content as privileged instructions.

LLM-generated risk labels and tool lists are proposals, not authoritative permission decisions.

### 5.4 L3 · Acquisition

**Primary Responsibility:** Retrieve external information from permitted sources.

**Components:**

- URL Fetch Adapter.
- RSS Adapter.
- Approved API Adapter.
- Source Metadata Collector.

**Inputs:**

- Authorized acquisition requests.
- Target and destination constraints.
- Request budgets.
- Timeouts and size limits.

**Outputs:**

- Retrieved content.
- Source identifiers.
- Retrieval metadata.
- Error and timeout information.
- Trust classification.

**Allowed:**

- Retrieve content from authorized sources.
- Preserve source provenance.
- Normalize results for downstream processing.

**Prohibited:**

- Accessing internal or otherwise prohibited network destinations.
- Accessing cloud metadata endpoints.
- Expanding approved network targets.
- Converting retrieved instructions into tool authority.
- Executing hidden write operations disguised as Fetch.

Outbound Fetch is a controlled capability, even when the underlying HTTP method is read-oriented.

### 5.5 L4 · Verification

**Primary Responsibility:** Independently evaluate proposed plans, results and supporting evidence.

**Components:**

- Auditor.
- Evidence Verifier.
- Structural Validation Interface.

**Inputs:**

- Original task requirements.
- Proposed outputs.
- Relevant raw evidence.
- Tool execution summaries.
- Applicable policy references.

**Outputs:**

- Audit findings.
- Evidence evaluation.
- Revision requirements.
- Review conclusions.

The audit outcome vocabulary is:

- `AUDIT_PASS`
- `REVISE`
- `NEEDS_REVIEW`
- `BLOCK`

This vocabulary is separate from the task state machine.

**Allowed:**

- Read only the evidence necessary for review.
- Identify inconsistencies.
- Recommend correction.
- Identify conditions requiring a block.

**Prohibited:**

- Changing original task data.
- Modifying audited output.
- Calling protected execution tools.
- Creating approval records.
- Modifying existing audit entries.

Auditor MUST use an independently restricted runtime identity and read-only data access for audited resources.

The Auditor itself MUST treat external evidence as untrusted input.

For operations requiring an audit, `BLOCK`, `NEEDS_REVIEW` and unresolved `REVISE` MUST prevent progression to protected execution.

`AUDIT_PASS` is a necessary condition where an audit is required, but never a sufficient authorization condition.

### 5.6 L5 · Execution

**Primary Responsibility:** Perform authorized actions through restricted mechanisms.

**Components:**

- Restricted Executor.
- Document Generator.
- Delivery Adapter.
- Protected Tool Interface.

**Inputs:**

- Validated operation requests.
- Applicable policy information.
- Required approvals.
- Authorized execution parameters.

**Outputs:**

- Files.
- Tool results.
- Delivery status.
- Execution evidence.
- Error or uncertain-result records.

**Allowed:**

- Create files within approved locations.
- Perform authorized local operations.
- Perform approved external actions.
- Record execution outcomes.

**Prohibited:**

- Using broader credentials than required.
- Executing actions that bypass Policy Gate.
- Modifying permissions to complete a blocked task.
- Blindly retrying uncertain external deliveries.
- Modifying or deleting protected audit history.

Protected credentials MUST remain within the trusted authorization and execution boundary, under separately restricted storage and access rules.

The sending credential SHOULD be readable only by the dedicated Delivery Executor, not by every component in the boundary.

### 5.7 L6 · Persistence

**Primary Responsibility:** Preserve authorized task state, memory, evidence and generated artifacts.

**Components:**

- Task Store.
- Memory Store.
- Evidence Store.
- Artifact Store.
- Audit Event Store.
- Governance Snapshot Store.

**Inputs:**

- Authorized write requests.
- Task transitions.
- Generated artifacts.
- Audit events.
- Governance snapshots.

**Outputs:**

- Durable records.
- Authorized query results.
- File references.
- Integrity metadata.
- Recovery data.

**Allowed:**

- Store and retrieve data through scoped service roles.
- Maintain version and source references.
- Support recovery and review.

**Prohibited:**

- Granting all modules database administrator privileges.
- Allowing ordinary Agents to edit protected audit records.
- Confusing user memory deletion with unrestricted audit deletion.
- Activating governance rules simply because a new file was stored.

Audit history MUST support Append-Only semantics.

Detailed retention, redaction, administrator access and tamper-evidence requirements belong to D02 and D04.

---

## 6. Cross-Layer Components & Trusted Boundaries

### 6.1 Orchestrator · n8n

**Status:** PROPOSED  
**Role:** Unprivileged orchestration principal.

n8n is a candidate implementation, not an architectural requirement.

Its proposed value is:

- Workflow scheduling.
- Integration coordination.
- Retry and task sequencing.
- Human-readable workflow management.
- Reduced development effort for ordinary automation.

Its costs include:

- Persistent memory usage.
- Plugin and node attack surface.
- Workflow governance requirements.
- Additional credential management.

Because the primary server has limited memory, adoption MUST be justified through actual resource testing and comparison with a simpler FastAPI-based scheduler.

If adopted, n8n MUST:

1. Operate with an independent low-privilege service identity.
2. Remain unavailable through a public administrative interface.
3. Use only approved nodes and network destinations.
4. Have arbitrary shell and unrestricted code execution capabilities disabled or otherwise rendered inaccessible.
5. Avoid mounting the Docker daemon socket.
6. Avoid host-root privileges.
7. Hold no protected delivery, governance-release or production-administrator credentials.
8. Submit all protected operation requests exclusively through the authorized Policy Gate interface.

n8n MUST NOT directly invoke the underlying protected Delivery Executor.

Node allowlists, network policy and credential specifications belong to D02.

### 6.2 Policy Gate

**Role:** Trusted authorization enforcement boundary.

Policy Gate implements policies authorized under L0.

It is not an ordinary Agent and does not accept LLM-generated permission decisions as authoritative.

The architecture MUST place authorization enforcement on every protected operation path.

Protected operations include, at minimum:

- Controlled outbound network requests.
- Memory creation, modification and deletion.
- Persistent file writes and modifications.
- External email delivery.
- Workflow modifications.
- Production deployments.
- Governance updates.
- Sensitive configuration changes.

These operations do not necessarily require human approval.

Authorization checking and human approval are separate concepts.

For example:

- A permitted Fetch request may pass deterministic checks without human intervention.
- An authorized local report generation may execute without explicit approval.
- An external email send requires both policy authorization and an applicable valid human approval.

Policy Gate MUST deterministically determine effective:

- Operation permissions.
- `allowed_tools`.
- Risk classification.
- Applicable budgets.
- Approval requirements.

Planner MAY suggest increasing a risk classification, but MUST NOT unilaterally reduce the effective classification or expand allowed tools.

A request outside permitted policy scope MUST be rejected or referred to the Human Owner.

### 6.3 Protected Credentials

All credentials enabling protected operations MUST remain inside the trusted authorization and execution boundary or an associated access-controlled secret store.

The following principals MUST NOT have general read access to these credentials:

- Planner.
- Auditor.
- Orchestrator.
- Fetch result processors.
- Ordinary Maintainer sessions.
- Governance Synchronizer.

Policy Gate SHOULD hold only the credentials required for identity and authorization verification.

Individual restricted executors SHOULD exclusively hold the credentials needed for their specific downstream operations.

The architecture MUST NOT rely on the secrecy of prompts to protect credentials.

Privileged human break-glass access, if implemented, must use a separate controlled mechanism defined by D02 and D04.

### 6.4 Trusted Approval Channel

Human approval MUST originate through an authenticated, non-LLM approval interface.

The approval interface MUST:

- Render the actual canonical operation payload from trusted storage.
- Present all relevant target identities and side effects.
- Clearly show recipient, subject, body and attachments for email.
- Bind the decision to the exact payload version.
- Reject unauthenticated or impersonated approvals.

Planner and Orchestrator MUST NOT be able to invoke the Owner-only approval action.

A model-generated message saying that the user approved an action is not an approval.

The interface SHOULD prioritize a simple, unambiguous representation of what will actually be sent or modified.

### 6.5 Auditor Isolation

Auditor MUST operate using:

- Its own restricted service identity.
- Independent credentials.
- A read-only database role for protected evidence.
- An isolated runtime process or container.
- No access to delivery, governance release or privileged execution tools.

Sharing a physical host is permitted.

Sharing credentials that would defeat read-only restrictions is not permitted.

This isolation is a minimum runtime boundary, not a claim of protection against an attacker with unrestricted host administrator control.

### 6.6 Maintainer

**Role:** Restricted maintenance principal.

Maintainer may inspect or modify authorized parts of the system only within a specifically granted maintenance task.

Maintainer MUST:

- Use a distinct, restricted access path.
- Have an explicit modification scope.
- Have time-bounded or otherwise revocable authorization.
- Preserve audit evidence.
- Follow formal release controls for production changes.

Ordinary Maintainer access MUST NOT include:

- Host root.
- Unrestricted Docker Socket access.
- General production secret retrieval.
- Direct governance activation.
- Unapproved production deployment.

Maintenance access MAY use controlled SSH accounts or approved tool interfaces, as detailed in D02 and D04.

### 6.7 Docker Host Boundary

No ordinary Agent or Orchestrator container may mount the Docker daemon socket or obtain unrestricted host-root capabilities.

A component able to control the host Docker daemon could undermine otherwise valid container isolation.

Privileged administrative operations, where necessary, MUST use a separately controlled and auditable human-authorized maintenance path.

### 6.8 Governance Synchronizer

Governance Synchronizer is a low-privilege synchronization component.

It MUST:

- Use read-only repository credentials when authentication is needed.
- Retrieve candidate governance content.
- Write only into a designated candidate or staging area.
- Record retrieved revision identifiers.
- Perform permitted integrity checks.

It MUST NOT:

- Approve its own retrieved rule changes.
- Activate production rules.
- Modify protected execution policies directly.
- Hold production release credentials.

---

## 7. Trust Boundaries & External Content

### 7.1 Trust Categories

The system MUST distinguish:

1. Authenticated human commands.
2. Approved governance rules.
3. Internal trusted service metadata.
4. LLM-generated proposals.
5. Retrieved external content.
6. Stored business knowledge originating from external sources.

A source becoming stored in L6 does not automatically make it trustworthy.

### 7.2 Untrusted Fetch Content

L3 MUST preserve provenance and untrusted-source classification.

Retrieved material MUST be passed to L2 and L4 as task data, not as system authority.

Instructions contained within retrieved pages, RSS items or API responses MUST NOT:

- Override governance rules.
- Increase `allowed_tools`.
- Change effective risk classification.
- Generate valid approval records.
- Authorize external delivery.
- Change system configuration.

Sensitive tool execution MUST remain separately authorized regardless of the model's interpretation of external content.

### 7.3 Memory Trust Boundary

Memory content MUST preserve its origin and trust classification.

User preference memory MUST require user confirmation before becoming a persistent preference.

Externally obtained business information MUST retain provenance and appropriate untrusted-data markers when stored.

L2 may propose memory writes, but the authorized Memory Service and Policy Gate decide whether a write is permitted.

LLM-generated assertions MUST NOT automatically become verified facts.

### 7.4 Auditor as an Injection Surface

Auditor may encounter malicious instructions embedded in source evidence.

Therefore, its evidence context MUST remain separated from its operational instructions and tool authority.

Audit conclusions MUST be based on the defined review contract, not on instructions embedded in audited material.

### 7.5 External Model Data Boundary

Before sending data to an external model API, the system MUST determine whether that class of data is approved for external processing.

At a minimum, the architecture distinguishes:

- Public or non-sensitive information.
- Personal or internal operational information.
- Confidential business information.
- Credentials and security secrets.

Credentials and security secrets MUST NOT be placed into ordinary model prompts.

The detailed classification, redaction and transfer controls belong to D02.

---

## 8. Protected Execution Lifecycle

### 8.1 General Workflow

```text
Authenticated Request
        |
        v
Task Creation
        |
        v
Deterministic Policy Classification
        |
        v
Planning / Authorized Fetch
        |
        v
Audit (where required)
        |
        v
Protected Operation Proposal
        |
        v
Policy Gate Validation
        |
        +----> BLOCKED
        |
        v
Human Approval (if required)
        |
        v
Approval & Payload Validation
        |
        v
Durable Execution Intent
        |
        v
Restricted Executor
        |
        v
Result / Unknown Status
        |
        v
Append-Only Audit Record
```

Every protected operation MUST pass the relevant deterministic authorization checks.

### 8.2 Risk Classification Authority

Planner outputs MUST NOT serve as authoritative risk classifications.

The trusted policy enforcement system determines the effective risk classification according to approved policies.

Planner may identify additional risks and recommend escalation.

A task MUST NOT gain broader tools or permissions merely because an LLM rephrased its objective.

### 8.3 Audit and Authorization

When a task requires audit:

- The audit result MUST be evaluated before protected execution.
- `BLOCK` MUST prevent execution.
- `NEEDS_REVIEW` MUST prevent execution pending appropriate review.
- `REVISE` MUST prevent execution until the required revision and review are complete.
- `AUDIT_PASS` only satisfies the applicable audit requirement.

Policy Gate MUST independently evaluate authorization even after `AUDIT_PASS`.

### 8.4 Durable Intent Before Execution

Before any protected operation creates a relevant external side effect, the system MUST successfully persist an execution-intent audit record.

That record MUST identify, at minimum:

- Task.
- Requested action.
- Target.
- Payload or immutable payload reference.
- Relevant policy version.
- Authorization reference.
- Operation or idempotency identifier.
- Timestamp.

If the required intent record cannot be durably written, execution MUST NOT proceed.

For operations requiring one-time approval, the system MUST atomically coordinate approval consumption, operation reservation and durable intent recording within the trusted transactional boundary.

The full transaction model belongs to D03.

### 8.5 Email Approval Integrity

For email delivery, the approved object MUST include the canonical representation of:

- Sending identity.
- To recipients.
- CC and BCC recipients.
- Subject.
- Body.
- Attachments and their content versions.
- Task reference.
- Applicable policy and operation identifiers.

The trusted approval interface MUST present the operation to the Owner using data bound to this exact canonical payload.

Policy Gate and the restricted sender MUST verify the approved payload hash against the actual sending payload.

Approval MUST have an explicit expiry.

Expired, rejected, previously consumed or invalidated approvals MUST NOT authorize sending.

Approval consumption MUST be enforced by the trusted transactional mechanism, not merely by n8n workflow state.

### 8.6 Delivery Uncertainty

A remote delivery action cannot generally be atomically committed in the same transaction as the local database.

A network timeout or local process failure may therefore leave uncertainty about whether the external provider accepted the action.

The architecture MUST recognize this explicitly.

Where delivery outcome cannot be confirmed:

- The task enters `DELIVERY_UNKNOWN`.
- Automatic duplicate sending is forbidden.
- The original operation identity is retained.
- Reconciliation or human intervention is required.

Detailed transitions and provider-specific reconciliation belong to D03.

### 8.7 Document Generation

Document generation is a controlled operation even when it does not involve external delivery.

File paths, output size, permitted formats and resource budgets MUST be enforced outside the LLM.

Generated output MUST retain task and artifact references.

External distribution of a generated file is a separate operation requiring its own authorization.

### 8.8 Memory Operations

Memory reads and writes MUST be distinguished.

A module authorized to retrieve memory is not automatically authorized to create, replace or delete it.

User-initiated correction or deletion MUST pass through the applicable memory governance and data lifecycle rules.

---

## 9. Governance Repository & Rule Snapshots

### 9.1 Rule States

The governance system distinguishes:

- `LATEST`: Newest available repository revision.
- `APPROVED`: Version approved for release.
- `ACTIVE`: Version currently enforced.
- `ARCHIVED`: Historical version retained for traceability or rollback.

`LATEST` MUST NOT automatically become `ACTIVE`.

### 9.2 Local-First Governance

The running system MUST consume the approved active local policy snapshot.

It MUST NOT require a successful real-time GitHub request to authorize every task.

Each task SHOULD record the policy version and active rule revision under which it was accepted.

### 9.3 Snapshot Integrity

An approved-to-active transition MUST validate:

- Source revision identity.
- Approved release manifest.
- Expected content hashes.
- Applicable approval evidence.
- Compatibility and required validation results.
- Authorization of the activating principal.

Hash comparison alone does not prove that a rule is trustworthy or authorized.

The trusted release process MUST establish both integrity and authorization.

### 9.4 Activation Authority

Ordinary Agents, including Planner, Auditor, Orchestrator and Governance Synchronizer, MUST NOT activate governance policy changes.

Activation requires the release authority and approval process defined in D05.

### 9.5 Atomic Publication

New snapshots SHOULD be prepared and validated in staging before activation.

The active pointer MUST be changed through a controlled, atomic publication mechanism.

Failed publication MUST preserve the previous valid active snapshot wherever feasible.

### 9.6 Drift Detection

Background checks SHOULD compare active governance specifications with actual security-relevant runtime configuration.

Detection MAY be automated.

Permission-expanding remediation MUST NOT be performed by an ordinary Agent without the required authorization.

---

## 10. Operational Modes & Failure Boundaries

### 10.1 Operational Modes

The system defines five conceptual runtime modes.

| Mode | Architectural Meaning |
|---|---|
| `NORMAL` | Required controls and services are available |
| `DEGRADED` | Selected noncritical capabilities are unavailable |
| `READ_ONLY` | Only permitted read and diagnostic operations may continue |
| `EXECUTION_BLOCKED` | Protected execution is disabled |
| `EMERGENCY_STOP` | Emergency restrictions prevent new protected side effects |

The exact allowed actions and transition conditions belong to D02 and D03.

### 10.2 Restrictive Transitions

Trusted deterministic health and security controls MAY automatically transition the system toward a more restrictive mode.

A transition toward a less restrictive mode MUST require the authorization specified by L0 governance; recovery from security-critical restrictions requires explicit Owner authorization.

Ordinary Planner or Orchestrator output cannot authorize recovery.

### 10.3 Emergency Stop

The system MUST provide an emergency stop mechanism independent of the LLM and ordinary orchestration path.

It MUST:

- Be triggerable through a trusted out-of-band management path.
- Prevent new protected side-effect operations within its scope.
- Remain enforceable when Planner or n8n is unavailable.
- Preserve available diagnostic and audit evidence.
- Require authorized recovery.

Failure or uncertain state of the emergency control mechanism MUST cause protected execution to fail closed.

Emergency stop cannot retroactively cancel a message already accepted by an external email provider.

### 10.4 Failure Response Table

| Failure | Required Architectural Response |
|---|---|
| GitHub unavailable | Continue using a valid active local snapshot |
| LLM provider unavailable | Suspend model-dependent work; no automatic privilege expansion |
| Fetch unavailable | Record failure and prevent unsupported factual conclusions |
| Auditor unavailable | Block operations requiring audit clearance |
| Policy Gate unavailable | Block protected operations |
| Approval service unavailable | Do not generate substitute approvals |
| Active policy corrupted | Block operations lacking a valid policy |
| Audit intent persistence fails | Do not execute the protected operation |
| Database unavailable | Block operations requiring durable state or authorization records |
| Delivery result uncertain | Enter `DELIVERY_UNKNOWN`; do not automatically resend |
| Resource budget exceeded | Block additional out-of-budget operations |
| Emergency stop activated | Prevent new protected side effects |
| Emergency controller unavailable | Fail closed for affected protected operations |

The precise relationship between failures, runtime modes and task states MUST be defined in D03 and operationalized in D04.

---

## 11. Deployment Architecture & Resource Constraints

### 11.1 Known Primary Server

The following configuration is user-reported and has not yet been independently verified on the host.

| Resource | Reported Value |
|---|---|
| Region | Shanghai |
| CPU | 2 vCPU |
| Memory | 4 GB |
| Storage | 70 GB SSD |
| OS | Ubuntu 24.04 |
| Docker | Version 29 |
| Network | 6 Mbps, 600 GB/month |

**Evidence Status:** USER_REPORTED / NOT VERIFIED

### 11.2 Secondary Server

A Hong Kong server exists, but its deployment condition and detailed specifications are not yet verified.

It MUST NOT be treated as a production dependency until its state, security configuration and operational role have been established.

### 11.3 Candidate Runtime Layout

```text
Shanghai Host
|
+-- Private Access Boundary
|
+-- FastAPI Core
|   +-- Chat / Planning
|   +-- Memory Interfaces
|   +-- Fetch Coordination
|   +-- Document Coordination
|
+-- Orchestrator [PROPOSED: n8n]
|
+-- Auditor Restricted Process
|
+-- Trusted Authorization / Execution Boundary
|   +-- Policy Gate
|   +-- Restricted Executors
|   +-- Protected Credentials
|
+-- PostgreSQL
|   +-- Task State
|   +-- Memory
|   +-- Evidence
|   +-- Audit Log
|
+-- Restricted Artifact Storage
|
+-- Local Governance Snapshots
```

This is a logical topology.

A separate network or container boundary MUST be introduced where required to enforce authorization or credential separation.

### 11.4 Isolation Classes

| Component | Minimum Architectural Separation |
|---|---|
| Planner | Logical module separation; no protected execution credentials |
| Fetch | Restricted network destinations, with private-address and cloud-metadata protections |
| Auditor | Separate restricted process, service identity and read-only database access |
| Orchestrator | Separate low-privilege identity and controlled network/tool access |
| Policy Gate | Trusted enforcement boundary not bypassable by ordinary workflow calls |
| Restricted Executor | Credential and capability isolation from ordinary Agent components |
| Maintainer | Distinct authorized maintenance channel; no default root or Docker Socket access |
| Governance Synchronizer | Repository read-only and candidate-area write-only scope |
| PostgreSQL | Per-service database roles and controlled privileged administration |
| Docker Host | No Agent or Orchestrator access to unrestricted daemon control or host root |

These are architectural minimums.

D02 MUST translate them into concrete technical isolation requirements.

### 11.5 Resource Management

Given the reported 4GB memory limit:

- A single-host, low-concurrency deployment SHOULD be preferred.
- External model APIs SHOULD be evaluated before local model hosting.
- Unnecessary always-on services SHOULD be avoided.
- Memory consumption MUST be measured before production activation.
- Resource and cost limits MUST be enforced by trusted software.
- Backup and recovery requirements MUST be considered before choosing storage layout.

The target architecture does not guarantee that the current host can operate every candidate component simultaneously.

### 11.6 Network & Administrative Access

V1 is intended for personal use.

Administrative interfaces SHOULD remain privately accessible and MUST NOT be exposed publicly without an approved access and compliance design.

Lack of ICP filing and the planned server location remain deployment constraints requiring verification before introducing internet-facing services.

### 11.7 Model Provider Feasibility Gate

A model provider MUST NOT be promoted to a production dependency until the following have been evaluated:

- Supported service and account regions.
- Eligibility under provider terms.
- Actual network availability.
- Latency and timeout behavior.
- Usage pricing and hard-budget controls.
- Approved data classes for external transmission.
- Failure and degradation behavior.

A secondary server MUST NOT be assumed to solve provider eligibility restrictions merely by forwarding requests.

Model provider selection remains open.

---

## 12. Architecture Invariants

The following invariants apply to all V1 implementations and extensions.

### INV-01 · No Self-Authorization

An Agent MUST NOT grant, approve or extend its own privileges.

### INV-02 · No Privilege Inheritance Through Orchestration

Workflow coordination MUST NOT grant underlying protected execution privileges.

### INV-03 · Protected Credential Ownership

Protected credentials MUST remain within the trusted authorization/execution boundary or authorized secret storage, inaccessible to ordinary Agent principals.

### INV-04 · Deterministic Risk and Tool Authority

Effective risk classification and allowed tools MUST be determined by approved deterministic policy, not LLM declarations.

### INV-05 · Trusted Human Approval

Human approval MUST originate from an authenticated, non-LLM-controlled Owner channel and bind to the actual operation payload.

### INV-06 · Approval Integrity

Approved content, targets, attachments and relevant versions MUST match the operation actually executed.

### INV-07 · Audit–Execution Separation

Auditor MUST NOT control protected execution or modify the audited material.

### INV-08 · Untrusted Data Containment

External or otherwise lower-trust information MUST NOT become control-plane authority.

### INV-09 · Protected Memory Boundary

External information MUST retain provenance and trust status; user preferences require confirmation before persistent adoption.

### INV-10 · Durable Intent Before Side Effects

A protected operation MUST NOT execute unless its required execution-intent audit record has been durably committed.

### INV-11 · Append-Only Audit History

Planner, Auditor and Executor MUST NOT modify or delete existing protected audit events.

### INV-12 · No Blind Delivery Retry

Uncertain external delivery MUST NOT cause automatic duplicate sending.

### INV-13 · No Ordinary Agent Host Control

Ordinary Agent and Orchestrator containers MUST NOT receive host-root authority or unrestricted Docker daemon access.

### INV-14 · Independent Emergency Stop

A trusted out-of-band emergency stop mechanism MUST be available and independent of LLM decisions.

### INV-15 · Runtime Governance Independence

Production authorization MUST NOT depend on live GitHub availability.

### INV-16 · Controlled Rule Activation

New governance rules MUST NOT become active merely because they were retrieved or committed.

### INV-17 · Resource and Financial Limits

Protected execution MUST honor enforceable task-level and global limits.

### INV-18 · Verified State Representation

Planned, implemented and verified capabilities MUST remain distinguishable.

### INV-19 · Controlled Module Evolution

New capabilities MUST preserve existing governance and authorization boundaries unless a formally approved governance change explicitly modifies them.

Changes affecting these invariants require formal governance review and Human Owner authorization.

---

## 13. Module Evolution & Compatibility

### 13.1 Replaceable Components

Subject to interface compatibility and security validation, the following MAY be replaced:

- LLM providers.
- Planner implementation.
- Fetch adapters.
- Document generators.
- Chat UI.
- Workflow scheduling and orchestration engine.
- Business-specific task templates.

### 13.2 Mandatory Extension Requirements

Any proposed new module MUST declare:

1. Functional-layer ownership.
2. Inputs and outputs.
3. Data trust classification.
4. Required capabilities.
5. Expected side effects.
6. Failure behavior.
7. Audit requirements.
8. Compatibility implications.
9. Resource impact.
10. Required validation and approvals.

### 13.3 Prohibited Shortcuts

A module upgrade MUST NOT be accepted solely because it makes development easier if it:

- Consolidates all protected credentials into n8n.
- Allows Planner to directly invoke the mail provider.
- Gives Auditor write access to audited records.
- Replaces human approval with an LLM decision.
- Enables unrestricted command execution from an ordinary Agent.
- Automatically activates new GitHub governance content.
- Disables durable intent logging.
- Removes uncertain-delivery handling.
- Grants unrestricted host administration to maintenance Agents.

### 13.4 Compatibility

Public module contracts SHOULD remain backward compatible when possible.

Breaking changes MUST follow the versioning, migration, testing and release rules assigned to D03, D05 and D06.

---

## 14. Open Decisions & Implementation Dependencies

| Item | Current Status | Primary Follow-Up |
|---|---|---|
| n8n necessity and final adoption | PROPOSED | D01 feasibility review / D04 |
| n8n node and credential allowlist | PLANNED | D02 |
| Policy Gate identity and enforcement design | PLANNED | D02 |
| Protected credential storage | PLANNED | D02 |
| Maintainer access mechanism | PLANNED | D02 / D04 |
| Trusted approval UI and endpoint | PLANNED | D02 / D03 |
| Approval payload canonicalization | PLANNED | D03 |
| Atomic approval consumption and intent logging | PLANNED | D03 |
| Delivery reconciliation and state transitions | PLANNED | D03 |
| Runtime-mode transition table | PLANNED | D03 |
| Auditor runtime and database access | PLANNED | D02 / D04 |
| Fetch network restrictions | PLANNED | D02 |
| External-data and model-input classification | PLANNED | D02 |
| Append-Only log retention and integrity | PLANNED | D02 / D04 |
| Kill Switch implementation | PLANNED | D02 / D04 |
| Governance manifest and activation procedure | PLANNED | D05 |
| Concrete risk levels and allowed-tool policy | PLANNED | D02 / D03 |
| CSV/XLSX formula-injection validation | PLANNED | D06 |
| Model API provider and regional feasibility | NOT VERIFIED | D04 feasibility gate |
| Actual primary server resource measurements | NOT VERIFIED | D04 |
| Hong Kong secondary server condition | NOT VERIFIED | D04 |
| Real backup and recovery procedures | NOT VERIFIED | D04 / D06 |

These open decisions MUST NOT be silently treated as implemented features.

---

## 15. Architecture Acceptance Checklist

Acceptance must be evaluated against GOV-ACCEPT-001 v0.2.

### Functional Structure

- [ ] All L0–L6 layers have explicit responsibilities and boundaries.
- [ ] Logical layers are distinguished from runtime processes.
- [ ] All six V1 capabilities are represented.
- [ ] Orchestrator, Policy Gate and Maintainer have explicit architectural placement.
- [ ] No critical capability lacks an identified responsible component.

### Security Boundaries

- [ ] Orchestrator cannot directly perform protected execution.
- [ ] Protected credentials have a clearly defined trusted ownership boundary.
- [ ] Planner cannot modify effective risk classification or allowed tools.
- [ ] Approval originates from a trusted Owner channel.
- [ ] Auditor has an independently enforceable read-only boundary.
- [ ] External content and memory trust classifications are defined.
- [ ] Audit intent must be durable before protected execution.
- [ ] Docker Socket and host-root exposure are prohibited for ordinary Agents.
- [ ] Emergency stop is independent of LLM and Orchestrator availability.

### Execution Integrity

- [ ] Approval payload integrity is specified.
- [ ] Single-use approval is required for applicable operations.
- [ ] Uncertain delivery is distinguished from confirmed failure.
- [ ] No blind automatic resend is permitted.
- [ ] Resource and financial limits are architectural requirements.

### Governance & Operations

- [ ] GitHub is not a runtime dependency.
- [ ] Only approved local governance rules can become active.
- [ ] Rule activation requires authorization and integrity verification.
- [ ] Failure modes and restrictive transitions are defined.
- [ ] Known server constraints are identified without unsupported capacity claims.
- [ ] Pending design decisions are separated from implemented capabilities.

### Modularity

- [ ] Replaceable modules are identified.
- [ ] New module entry requirements are defined.
- [ ] Architecture invariants constrain future upgrades.
- [ ] No document improperly duplicates the detailed authority of D02–D06.

**Acceptance Status:** NOT VERIFIED

All checklist items remain unchecked until independent review is completed.

---

## 16. Review, Approval & Freeze Conditions

This document MUST remain in DRAFT until:

1. Independent review is completed.
2. D01 acceptance criteria are met.
3. Critical architectural findings are resolved.
4. Major findings are resolved or explicitly accepted under documented conditions.
5. D02 and D03 are checked for cross-document consistency.
6. The Human Owner approves the architecture baseline.

A Frozen architecture remains changeable only through the approved change-control process.

A Frozen document does not mean the system has been deployed or verified.

---

## 17. Final Architectural Principles

**Planning is not authorization.**

**Orchestration is not privilege inheritance.**

**Auditing is not execution.**

**Model output is not policy.**

**Retrieved content is not trusted instruction.**

**A displayed approval request is not a valid approval unless the authenticated Owner actually approves the bound payload.**

**An operation without durable execution intent must not proceed.**

**A newer governance revision is not automatically an active revision.**

**A more capable Agent is not automatically a more privileged Agent.**

The architecture is designed to evolve in intelligence and functionality while keeping authority explicitly bounded, enforceable and auditable.

---

**End of ARCH-001 · Draft v0.2.0**
