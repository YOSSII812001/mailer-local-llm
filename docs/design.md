# Corporate Mailer Technical Design (Local PC Installation, Privacy-First)

## Introduction

This document defines the technical design for a privacy-first corporate mailer installed on each user's local PC.

The system helps employees process business email by summarizing incoming messages and drafting replies with a local AI model. The Backend API, LLM Orchestrator, `llama.cpp` inference engine, and Gemma 4 model all run on the user's own hardware. **No email body, attachment text, prompt, summary, draft, or model context is sent to an external AI service.**

Core workflow:

1. Mail ingestion
2. Local LLM summary and reply drafting
3. User review and edit
4. Send approved email

Key goals:

- Keep AI processing 100% local on each user's PC using Gemma 4 (2B, 4B, or 12B) via `llama.cpp`
- Protect corporate email data and user privacy through multi-layer security
- Use a modular backend and frontend architecture
- Require explicit human approval before sending any AI-assisted message
- Support robust auditing, security controls, and future model upgrades
- Let users choose the local model that fits their CPU, GPU, memory, and VRAM

## Architecture

### Deployment Model: Local PC Installation

The default deployment model is a local PC installation. Each user installs and runs the mailer on their own machine. The local installation includes the application UI, Backend API, mail processing services, LLM orchestration layer, inference engine, and model files.

This model maximizes privacy because email processing and AI inference stay on the user's device. External connectivity is limited to required mail-provider operations, such as fetching messages from Xserver, Gmail, Microsoft Graph, or IMAP, and sending approved messages through SMTP or provider APIs.

Centralized inference servers, shared GPU servers, managed cloud AI services, and external AI fallbacks are outside the default architecture. If an enterprise adds device management, backup, or support tooling, those tools must preserve the local-only processing boundary.

### High-Level Architecture

```text
        [External Mail Providers]
 [Xserver / Gmail / IMAP / SMTP / Microsoft Graph]
          ^                                  |
          | Approved send                    | Mail fetch
          |                                  v
+-------------------------------------------------------------+
| Client's Local Machine                                      |
|                                                             |
| [User Review UI - Next.js]                                  |
|              |                                              |
|              v                                              |
| [Backend API - FastAPI]                                     |
|      |             |                  |                     |
|      |             |                  +--> [Send Service] --+
|      |             |                                        |
|      |             +--> [Mail Ingestion Service] <----------+
|      |                                                      |
|      +--> [Local Mail Store / Metadata DB]                  |
|      |                                                      |
|      +--> [LLM Orchestrator]                                |
|                |                                            |
|                v                                            |
|        [llama.cpp Inference Engine]                         |
|                |                                            |
|                v                                            |
|        [Gemma 4 Local Model (GGUF)]                         |
+-------------------------------------------------------------+
```

All components inside the "Client's Local Machine" boundary run on the user's local hardware. Mail ingestion and sending connect to external providers, but the processing layer remains local. External providers receive only normal mail-protocol traffic and the final user-approved outbound email.

### Main Modules

#### Mail Ingestion Service
Responsible for receiving and normalizing mail data.
- Run locally on the user's PC.
- Fetch incoming mail from Xserver, Gmail, IMAP, Microsoft Graph, or corporate mail APIs.
- Parse headers, body, attachments, recipients, and thread metadata.
- Store only required metadata; encrypted content is stored in the local database or local encrypted content store.
- Queue messages for local summarization and deduplicate by message ID.

#### Backend API (FastAPI)
The central hub for business logic and security.
- Run as a local API service, usually bound to `localhost`.
- Provide authenticated APIs for the local frontend.
- Manage mail state, summaries, drafts, and review statuses.
- Enforce Role-Based Access Control (RBAC).
- Interface with the LLM Orchestrator and audit logging service.

#### LLM Orchestrator
Responsible for safe and repeatable local AI execution.
- **Model Switching**: Select Gemma 4 2B, 4B, or 12B based on the artisan task mapping, workload, thread complexity, and the user's local hardware resources.
- **Prompt Engineering**: Construct prompts from mail content and pre-defined corporate policy templates.
- **Output Validation**: Verify AI outputs for policy compliance and hallucination markers before presenting to users.
- **Context Management**: Maintain minimal, ephemeral context to minimize data footprint in memory.
- **Escalation Rules**: Escalate low-confidence, high-risk, or ambiguous tasks to a larger local model or a human-only review path.

#### User Review UI (Next.js)
The primary interface for human-in-the-loop interaction.
- Display inbox, thread details, AI summaries, and draft replies.
- Provide a rich-text editor for users to refine AI-generated content.
- Clearly distinguish AI-generated text from human-authored text.
- **Mandatory Approval**: Require explicit user action for "Send" operations.
- Display confidence scores, source context notes, model identity, and "Missing Context" warnings.
- Present AI outputs with helpful context, such as "why this summary was generated" and "what may need human review."

#### Send Service
Handles the final delivery of approved messages.
- Run locally and send email via SMTP, Microsoft Graph, Gmail APIs, or other approved providers.
- Validate sender identity and recipient list against internal policies.
- Ensure AI-generated drafts are never sent without an explicit "Approved" flag in the database.
- Record send status and audit metadata.
- Send only the final user-approved message body to the external mail provider.

## Hardware Dependency & Performance

The AI experience depends directly on the user's local PC specifications. CPU performance, system memory, GPU capability, and especially GPU VRAM determine which Gemma 4 model can run smoothly.

Users can choose Gemma 4 2B, 4B, or 12B based on local hardware. Smaller models provide faster responses on modest devices. Larger models can produce stronger summaries and drafts, but they require more memory, more VRAM, and longer processing time.

Performance also depends on quantization level, context length, attachment size, and the number of concurrent jobs. The application must expose model compatibility checks and make model selection understandable to users.

### Recommended Specifications

The following tiers are guidance for local PC installations. Exact requirements depend on model quantization, context window size, and runtime settings.

| Hardware Tier | Typical Local PC Specs | Recommended Gemma 4 Models | Expected Experience |
| --- | --- | --- | --- |
| Entry: integrated graphics or CPU-only | Modern multi-core CPU, 16 GB RAM, integrated graphics or shared memory, no dedicated VRAM | 2B quantized. 4B only for short tasks if latency is acceptable. | Best for mail triage, short summaries, labels, and lightweight reply suggestions. Drafting may be slower and less nuanced. |
| Mid-range: dedicated GPU | NVIDIA GPU with about 8 GB VRAM, 16-32 GB RAM, modern CPU | 2B and 4B quantized. 12B only with aggressive quantization and short context, if supported. | Smooth local summarization and standard reply drafting. Good default tier for daily business use. |
| High-end: larger dedicated GPU | NVIDIA GPU with 12-16 GB or more VRAM, 32-64 GB RAM, strong CPU | 4B and 12B quantized | Best experience for long threads, complex summaries, policy-sensitive drafts, and attachment-heavy workflows. |

VRAM is the strongest constraint for larger models. If the selected model does not fit the user's hardware, the system must recommend a smaller model or route the task to manual drafting. The system must not switch to an external AI service.

## User Experience Philosophy

The product experience must make privacy and control visible to users. The AI should feel like a reliable artisan that handles routine work carefully. It must not feel like an autonomous agent that can run ahead of the user.

### Safety Net of Assurance
- The system must make local-only AI processing visible in the UI and operational logs.
- AI must never send, forward, delete, archive, or move mail without explicit user action.
- Every generated summary or draft must show its status, model, time, and review requirement.
- High-risk actions must require confirmation, including external recipients, sensitive attachments, and policy-sensitive topics.
- Users must be able to discard AI output without changing the source email or draft history.

### Frictionless UX
- The UI must buffer raw AI behavior with human-readable context notes.
- Summaries must include confidence scores, missing context warnings, and cited source areas when possible.
- Drafts must include tone notes, assumptions, and unresolved questions instead of blunt AI responses.
- Low-confidence output must guide the user toward review, regeneration, or manual drafting.
- The system must keep review steps short, clear, and reversible.

Confidence presentation rules:

| Confidence Band | UI Behavior | System Behavior |
| --- | --- | --- |
| High | Show concise summary or draft with source notes. | Allow normal review flow. Still require human approval before send. |
| Medium | Show assumptions, missing context, and review hints. | Allow regeneration or edit. Record warning acknowledgement if the user proceeds. |
| Low | Show a caution state and recommend manual review. | Block auto-filled final drafts for high-risk mail. Escalate to a larger local model or human-only path. |

### Systematic Maintainability
- Prompt templates, model routing rules, and validation rules must live in versioned shared configuration.
- Each AI output must record the model version, prompt template version, policy version, and validator version.
- Teams must use shared runbooks for model updates, prompt changes, incident response, and audit review.
- Golden test cases must cover common mail types, sensitive scenarios, and failure cases.
- Quality must not depend on one maintainer's private knowledge.

## Business Model & Monetization

The commercial model must preserve the product's role as a privacy-first safety net and a craftsman-grade mail assistant. The product does not monetize email content, model training data, public AI usage, or opaque automation. It monetizes trust, reliability, governance, and operational confidence.

Core commercial principles:

- Keep the core mailer open source so customers can inspect the safety model, local AI boundaries, and human-approval workflow.
- Attach paid value to assurance: secure local deployment, governance depth, expert guidance, fleet operations, and accountable support.
- Preserve the local-only AI policy across all offerings. Paid tiers must not introduce external AI fallbacks or hidden data sharing.
- Make trust and reliability the primary products. The AI feature is valuable because the surrounding system is stable, auditable, and controllable.
- Treat local PC installation as the default product boundary. Commercial services may support installation, configuration, updates, and hardware sizing, but they must not centralize AI processing.

### Revenue Routes

| Route | Offer | Target Customer | Monetization Logic | Philosophy Alignment |
| --- | --- | --- | --- | --- |
| Companion-style Implementation Consulting | High-touch setup, security review, SSO/OIDC or SAML integration, mail-system integration, policy template design, model sizing, maintenance, and periodic governance alignment. | Companies that want the strongest security posture and will pay for expert guidance. | Upfront implementation package, ongoing maintenance retainer, and change-project fees for policy, infrastructure, or compliance updates. | Sells trust directly. The customer buys a reliable implementation partner, not just software access. |
| Managed Local Operations | Installer packaging, endpoint rollout support, local model distribution, device health checks, backup guidance, update channels, and support runbooks for user PCs. | Companies that want local PC privacy with enterprise-grade rollout and support. | Monthly support fee based on device count, support level, governance scope, and update-management needs. | Sells operational confidence while keeping AI processing on each user's PC. |
| Enterprise Edition | Paid governance layer on top of the OSS core, including advanced RBAC, granular audit logging, advanced data retention policies, compliance exports, administrative policy controls, and dedicated technical support. | Large organizations with strict governance, audit, and compliance requirements. | Annual subscription by organization size, support tier, and compliance module scope. | Monetizes governance and assurance. The OSS core remains useful, while regulated customers pay for stronger controls. |
| White Label Partnership | Rebrandable platform, deployment templates, partner documentation, enablement, and support for DX consultants and System Integrators. | Consultants and SIs that want to bundle the mailer into their own client solutions. | Partner license, revenue share, certification fees, and partner support contracts. | Scales the craftsman approach through trusted implementation partners instead of generic AI wrappers. |

### Packaging Boundaries

The OSS Core should include the essential safety contract:

- Local LLM orchestration and model routing.
- Mail ingestion, review, drafting, and mandatory human approval.
- Baseline RBAC, audit events, encrypted storage, and no external AI fallback.
- Transparent prompts, model identity, confidence notes, and review status.

The Enterprise Edition may extend these controls, but it must not remove them. Advanced paid features should add governance depth, such as role hierarchies, policy-specific permissions, audit export workflows, legal hold, tenant-specific retention, and dedicated support response targets.

Managed Local Operations should use the same local PC architecture as the OSS Core. The difference is operational responsibility: the vendor helps the customer package installers, size models, distribute updates, verify device health, and maintain runbooks. Mail content, prompts, summaries, drafts, and model context remain on the user's machine.

### Go-to-Market Motion

The preferred customer journey starts with a trust-building engagement. A consulting assessment maps the customer's mail systems, identity provider, data-retention rules, hardware tiers, and security policies. From there, the customer chooses the OSS Core, Enterprise Edition, or Managed Local Operations for local PC rollout.

White-label partners extend this motion. Partners can package the platform with their own DX consulting, workflow redesign, and system-integration services. The vendor should certify partners on privacy boundaries, audit requirements, and safe deployment patterns so the brand promise remains consistent.

### Differentiation

The product should compete as a dependable operating system for private AI-assisted email, not as a generic AI wrapper. Its differentiators are:

- Auditable per-device local AI execution.
- Explicit human control before send.
- Stable integrations with enterprise identity, mail, logging, and compliance systems.
- High-quality implementation practices that reduce operational risk.
- Clear proof that customer mail content is not used to train public models.

This is the craftsman position: the system is valuable because it is carefully built, predictable, and inspectable. The business earns revenue by making that reliability easier to adopt and safer to operate.

## Tech Stack

### Backend & Infrastructure
- **Language**: Python
- **Framework**: FastAPI
- **Database**: Local PostgreSQL or SQLite, depending on installation size and enterprise policy
- **Task Queue**: Local Celery, Redis Queue, or lightweight local queue for asynchronous ingestion and LLM processing
- **Auth**: Local user session, optional SSO/OIDC (SAML) integration for corporate identity

### Frontend
- **Framework**: React with Next.js (App Router)
- **Styling**: Tailwind CSS
- **State Management**: React Query for server-state synchronization

### Local LLM Engine
- **Runtime**: `llama.cpp`
- **Model Format**: GGUF
- **Models**: Gemma 4 (2B, 4B, 12B)
- **Deployment**: Installed on each user's local PC. The default design does not require a shared inference host.

### Gemma 4 Model Mapping (Artisan Assignment)

| Model | Artisan Role | Primary Tasks | Local Hardware Fit | Guardrails |
| --- | --- | --- | --- | --- |
| Gemma 4 2B | Fast triage artisan | Mail classification, priority labels, short summaries, action extraction, missing-context detection | Entry PCs, integrated graphics, CPU-only fallback, or low-VRAM devices | No long-form drafting. Escalate low confidence, sensitive content, or long threads. |
| Gemma 4 4B | Balanced drafting artisan | Standard summaries, first-pass replies, tone adjustment, meeting follow-ups, customer support drafts | Mid-range PCs with dedicated GPU VRAM or strong CPU and sufficient memory | Require confidence notes and source context. Escalate ambiguous or high-stakes drafts. |
| Gemma 4 12B | Senior analysis artisan | Complex analysis, long-thread synthesis, policy-sensitive drafts, negotiation support, long-form drafting | High-end PCs with larger dedicated GPU VRAM and enough system memory | Slower queue with stricter validation. Send still requires explicit human approval. |

### Storage & Security
- **Metadata**: Local PostgreSQL or SQLite with encryption enabled
- **Content/Attachments**: Local encrypted content store or encrypted DB fields (AES-256)
- **Audit Logs**: Local immutable append-only store for mail access, AI generation, human review, and send approvals

## Data Privacy & Security

### Local-Only AI Policy
**All AI processing must run on the user's local PC.**
The system is prohibited from sending the following to external services:
- Email body, subject, recipients, or attachments
- Prompt text, summaries, or draft replies
- Embeddings or conversation history

Mail providers receive messages only through normal mail protocols. The final outbound email body leaves the local PC only after explicit user approval.

### Encryption & Connectivity
- **In-Transit**: TLS 1.3 for external mail-provider and identity-provider connections.
- **At-Rest**: AES-256 encryption for local database records and attached files.
- **Secret Management**: Use an OS-backed local credential store, such as Windows Credential Manager or macOS Keychain. Enterprise deployments may add approved secret-management tooling if decrypted secrets remain local.

### Access Control & Auditing
- **RBAC**: Granular permissions for users, admins, and auditors.
- **Audit Trail**: Immutable logs for all user, system, AI, and administrator actions.
- **Retention**: Configurable data retention policies for summaries and audit logs.

#### Audit Log Requirements

Audit logging must be granular, immutable, and privacy-preserving.

Required event coverage:

| Event Area | Required Events |
| --- | --- |
| Mail access | Mail ingested, message opened, thread opened, attachment metadata viewed, attachment content accessed, search performed |
| AI processing | Summary requested, summary generated, draft requested, draft generated, model selected, validation completed, regeneration requested |
| Human review | Draft opened, manual edit saved, AI text accepted, AI text discarded, confidence warning acknowledged |
| Send approval | Approval screen opened, recipient list confirmed, final "Send" approved, send attempted, send succeeded, send failed |
| Administration | Policy changed, prompt template changed, model version changed, retention job executed, audit export requested |

Each audit event must include:
- `event_id`, `request_id`, UTC timestamp, actor ID, role, tenant, source IP, and device fingerprint when available.
- `message_id`, `thread_id`, `draft_id`, and attachment IDs when applicable.
- Action, outcome, failure code, and user-visible error message.
- Model ID, model checksum, prompt template version, policy version, validator version, and confidence score for AI events.
- Draft content hash and recipient list hash for final send approval events.

Immutability controls:
- Audit records are append-only. The application must not update or delete existing audit rows.
- Each record stores `previous_event_hash` and `event_hash` to create a tamper-evident chain.
- Production logs should be replicated to write-once storage or a database role that denies update and delete.
- Full email bodies and raw prompts must not be stored in audit logs. Store references, hashes, and redacted snippets only.
- A scheduled verifier must check hash-chain continuity and report gaps or unexpected mutations.

Audit write path requirements:
- Critical actions must write an audit record before the action starts and after it ends.
- Critical actions include mail body access, attachment content access, summary generation, draft generation, approval, and send.
- If the audit writer is unavailable, the system must fail closed for critical actions.
- Each audit write must use an idempotency key so retries do not create false duplicate actions.
- The audit service must expose verifier status to administrators and alert on hash-chain gaps.
- Audit export must include a manifest with record count, time range, hash root, exporter identity, and export checksum.

### Human-in-the-Loop (HITL)
- **Approval Gate**: AI cannot initiate a "Send" action.
- **Review Flow**: 1. User views summary -> 2. User edits draft -> 3. User confirms -> 4. System sends.

### Backup, Recovery & Graceful Degradation

The system must fail closed for AI actions and fail gracefully for user workflows.

#### Backup Strategy
- Local PostgreSQL or SQLite must use encrypted backups with tested restore procedures.
- Local content storage must use encrypted snapshots or encrypted backup targets for mail bodies and attachments.
- Secret and key material must be backed up through the approved local or enterprise secret-management process, not copied into application backups.
- Audit logs must be exported to immutable storage on a fixed schedule if enterprise policy requires it.
- Any off-device backup must encrypt content before it leaves the user's PC.
- Restore drills must verify database state, object integrity, audit-chain continuity, and key availability.
- Target recovery goals should be defined per deployment. Initial targets are RPO of 15 minutes and RTO of 4 hours.

#### Recovery Strategy
- Recovery must support full environment restore, tenant-level restore, and message/thread-level restore.
- Restored content must pass checksum verification before the UI displays it.
- Corrupted or unverifiable records must be quarantined and hidden from normal workflows.
- Send approvals must not be reconstructed from summaries alone. The system must restore or re-request the human approval state.
- Recovery events must create their own audit records.

#### Local LLM Failure Handling
- If `llama.cpp` or a model is unavailable, the system must disable AI generation and keep manual mail review available.
- The UI must show a clear local AI unavailable state, without offering an external AI fallback.
- Pending AI jobs must remain queued with retry metadata and idempotency keys.
- A circuit breaker must stop repeated model calls after configured failures.
- Existing summaries and drafts may remain readable if policy allows, but regeneration must stay disabled until health checks pass.
- High-risk or failed AI tasks must route to manual drafting instead of lower-quality automatic output.

#### Mail and Send Failure Handling
- Mail ingestion failures must retry with backoff and then move to a dead-letter queue with audit records.
- Duplicate ingestion must be prevented with message IDs and idempotency keys.
- Send failures must preserve the approved draft and show the delivery status to the user.
- If the database is unavailable, the UI must enter a read-only or maintenance mode and disable send actions.
- If the mail provider is unavailable, the system must keep drafts local and resume delivery only after provider health returns.

#### Degraded Mode Matrix

| Failure Mode | User Experience | System Constraint |
| --- | --- | --- |
| Local LLM unavailable | Mail reading and manual drafting remain available. AI controls show an unavailable state. | No external AI fallback. Queue AI jobs with retry metadata. |
| Smaller model available only | Fast triage can continue for low-risk mail. Drafting may be disabled or escalated. | Do not downgrade high-risk drafting from 12B to 2B or 4B without policy approval. |
| Audit writer unavailable | The UI shows that protected actions are temporarily unavailable. | Block mail body access, attachment access, AI generation, approval, and send. |
| Database read-only | Users can view permitted cached or replicated data where safe. | Disable edits, approvals, sends, and retention mutations. |
| Mail provider unavailable | Drafts remain local and delivery status is visible. | Do not mark mail as sent until provider confirmation is durable. |
| Object storage checksum mismatch | The affected message or attachment is hidden from normal use. | Quarantine the record and create a recovery audit event. |

Robustness acceptance criteria:
- No email may be sent unless the final approval audit event is durably written.
- No AI output may be shown unless model ID, prompt version, and validation outcome are recorded.
- Restore drills must prove that a sent-message approval trail can be reconstructed from immutable logs.
- Failure states must be explicit in the UI and must never silently switch to an external service.
- Every retryable job must have a bounded retry policy, a dead-letter path, and an operator-visible status.

## Implementation Plan

### Phase 1: Foundation
- Setup FastAPI & Next.js project structures for local PC installation.
- Define local service startup, localhost binding, and installer assumptions.
- Define DB schema (Users, Messages, Summaries, Drafts, AuditLogs).
- Implement local session management and optional SSO/OIDC authentication.

### Phase 2: Mail Ingestion & Storage
- Develop IMAP/Graph API connectors.
- Implement encrypted storage for mail bodies.
- Build background ingestion workers.

### Phase 3: Local LLM Integration
- Install `llama.cpp` on the user's local PC and load Gemma 4 GGUF models.
- Add local hardware detection and model compatibility checks.
- Build the LLM Orchestrator with prompt templates.
- Implement model switching and output validation logic.
- Implement Gemma 4 artisan routing for 2B, 4B, and 12B tasks.

### Phase 4: User Review Workflow
- Develop Inbox and Thread Detail UI.
- Integrate the draft editor and summary panel.
- Implement the "Approval" gate and "Send" actions.
- Add confidence scores, context notes, missing-context warnings, and reversible review actions.

### Phase 5: Secure Sending & Auditing
- Implement the SMTP/Graph API Send Service.
- Configure immutable audit logging with hash chaining and write-once replication.
- Implement data retention and cleanup jobs.
- Implement backup, restore, dead-letter queues, and degraded-mode behavior.

### Phase 6: Security Audit & UAT
- **Security Review**: Penetration testing and audit of encryption/access controls.
- **UAT**: Testing with real corporate email scenarios in a sandboxed environment.
- **Compliance Check**: Verify no data leakage to external APIs.

### Next Steps
- Define the local PC installer, startup flow, and service health checks.
- Add hardware profiling for CPU, GPU, system memory, and VRAM.
- Define user-facing model selection rules for Gemma 4 2B, 4B, and 12B.
- Define the concrete AuditLogs schema and hash-chain verifier.
- Create prompt, policy, validator, and model versioning conventions.
- Build failure-mode acceptance tests for local LLM downtime, mail provider downtime, and send retry behavior.
- Prepare restore runbooks and schedule the first restore drill.
- Define commercial packaging boundaries for OSS Core, Enterprise Edition, Managed Local Operations, and partner offerings.

## Final Design Principles
- **Human-Centric**: AI is an assistant, not an autonomous actor.
- **Assuring**: The UI must reduce anxiety by making privacy boundaries and human control visible.
- **Frictionless**: AI output must include context, confidence, and next-step cues.
- **Security-First**: Privacy is enforced by per-device local execution and architectural boundaries, not just policy.
- **Trust-Oriented**: Monetization must center on reliable deployment, governance, auditability, and support.
- **Robust**: Logs, backups, and degraded modes must make failures recoverable and auditable.
- **Systematically Maintainable**: Shared configuration, tests, and runbooks must prevent siloed knowledge.
- **Scalable**: Modular design allows each local installation to swap LLM models or mail providers.
- **Auditable**: Every AI interaction and final action is logged for transparency.
