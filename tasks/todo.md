# Task Plan

## 2026-06-06 Rabbit Mail Detailed Design Master Update

### Plan
- [x] Confirm plain Japanese document rules.
- [x] Review `docs/detailed_design.md`, `tasks/todo.md`, and `tasks/lessons.md`.
- [x] Rename the design identity to `Rabbit Mail`.
- [x] Define the High-Brand visual direction, rabbit silhouette, white-base theme, and refined dark mode.
- [x] Replace the previous web-app architecture with `Tauri 2 + Rust + Canvas`.
- [x] Remap Sassuru-kokoro, Haikei, Kizukai, Action Kanban, and Self-Improvement Loop to the native desktop architecture.
- [x] Add Part 4 for the Canvas-based Master Artisan's Workspace.
- [x] Record progress, verification notes, and next tasks.

### Review Notes
- Keep Rabbit Mail as the canonical product name.
- Treat the product as a local PC installed native desktop application, not a browser-delivered web app.
- Keep all AI inference, prompts, summaries, drafts, lessons, and risk validation on local hardware.
- Use Rust for orchestration, local inference control, storage, encryption, audit, and state transitions.
- Make Action Kanban a Canvas-native flow, not a standard list or table view.

### Progress
- Rewrote `docs/detailed_design.md` across Parts 1-4.
- Updated the project identity, brand aesthetic, visual identity, architecture, feature mapping, and workspace design.
- Replaced endpoint-centered backend language with Tauri Commands / Events and Rust module responsibilities.
- Added acceptance criteria for Rabbit Mail, local-only inference, and Canvas-based workspace behavior.

### Verification
- Confirmed `docs/detailed_design.md` contains Rabbit Mail and Parts 1-4.
- Confirmed no `FastAPI`, `Next.js`, `PostgreSQL`, or old project name references remain in `docs/detailed_design.md`.
- Ran Markdown whitespace validation with `git diff --check -- docs\detailed_design.md tasks\todo.md tasks\lessons.md`.

### Next Tasks
- Define Rust module interfaces for `rabbit_mail`, `rabbit_ai`, `rabbit_kanban`, `rabbit_audit`, and `rabbit_storage`.
- Prototype the Canvas Action Kanban, Draft Studio, and Context Inspector.
- Define installer behavior, model directory handling, and local hardware detection.
- Finalize the rabbit silhouette logo and design tokens.

## 2026-06-06 Detailed Design Part 3: Data Schema & Backend Architecture

### Plan
- [x] Confirm plain Japanese document rules.
- [x] Review `docs/detailed_design.md` Parts 1 and 2.
- [x] Define database tables for messages, summaries, drafts, audit logs, Action Kanban, and user lessons.
- [x] Define FastAPI REST and WebSocket endpoints for inbox, threads, drafts, send, and Kanban.
- [x] Define the LLM Orchestrator protocol, Craftsman templates, confidence parsing, and contextual hints.
- [x] Define AES-256 at-rest encryption and local-only backend boundaries.
- [x] Update `docs/detailed_design.md` with progress and next tasks.

### Review Notes
- Treat `docs/detailed_design.md` as the canonical detailed design file.
- Keep all message bodies, prompts, summaries, drafts, lessons, and model context on the local PC.
- Preserve immutable audit logging for access, generation, edits, approvals, and send actions.
- Make schema portable between PostgreSQL and SQLite where possible.

### Progress
- Added `Detailed Design - Part 3: Data Schema & Backend Architecture` to `docs/detailed_design.md`.
- Defined portable schema guidance for local PostgreSQL and SQLite.
- Defined primary FastAPI REST endpoints and WebSocket events.
- Defined local `llama.cpp` / Gemma 4 communication contracts.
- Defined AES-256-GCM at-rest encryption and local-only enforcement.

### Verification
- Confirmed the Part 3 heading and required sections exist in `docs/detailed_design.md`.
- Confirmed no trailing whitespace in `docs/detailed_design.md`, `tasks/todo.md`, and `tasks/lessons.md`.
- Ran `git diff --check -- docs\detailed_design.md tasks\todo.md tasks\lessons.md`.

### Next Tasks
- Use Part 3 as the source for migrations, repository interfaces, and API route implementation.

## 2026-06-06 Detailed Design Part 2: Core Functionality & LLM Workflow

### Plan
- [x] Confirm plain Japanese document rules.
- [x] Review `docs/detailed_design.md` Part 1 and `docs/design.md`.
- [x] Define the Artisan LLM workflow for analysis, crafting, validation, and confidence scoring.
- [x] Define Auto-Reply triggers and the Action Kanban state machine.
- [x] Define the Hermes-influenced self-improvement loop and learning data boundaries.
- [x] Update the target document with progress and next tasks.

### Review Notes
- Keep all AI processing local to the user's PC.
- Treat confidence as a work-risk indicator, not as an absolute truth score.
- Allow auto-reply only inside pre-approved rules with high confidence, low risk, and durable audit logs.
- Keep learning local, scoped, visible, approved by the user, and reversible.
- Ensure Hermes Agent organizes learning only; it must not send messages or expand automation scope.

### Progress
- Added `Detailed Design - Part 2: Core Functionality & LLM Workflow` to `docs/detailed_design.md`.
- Defined Analysis Phase, Crafting Phase, confidence scoring, hard caps, and UI mapping.
- Defined auto-reply rule contracts, execution flow, Action Kanban states, and card data mapping.
- Defined feedback collection, lesson approval, style profiles, refinement frequency, and Hermes Agent responsibilities.

### Next Tasks
- Create Part 3 for screen information architecture and transitions.
- Create wireframes for Action Kanban, Context Rail, and Learning Ledger.
- Convert Part 2 data contracts into API and database schema requirements.
- Add acceptance tests for low-confidence output, scheduled-send interruption, and lesson approval.

## 2026-06-06 Detailed Design Part 1: UI/UX & Philosophy

### Plan
- [x] Confirm plain Japanese document rules.
- [x] Review `docs/design.md` for current privacy-first and human-approval principles.
- [x] Create `docs/detailed_design.md` with the requested Part 1 section.
- [x] Define visual language, Sassuru-kokoro UX, Auto-Action Kanban, and Artisan Feedback Loop.
- [x] Keep the automatic action concept aligned with explicit approval and auditability.

### Review Notes
- The product must feel like a quiet artisan, not an intrusive autonomous agent.
- Auto-action UX must show flow, conditions, confidence, and audit state.
- AI learning must be local, visible, scoped, and reversible.
- The document should use clear Japanese while preserving the requested English section title.

### Progress
- Added `docs/detailed_design.md`.
- Wrote `Detailed Design - Part 1: UI/UX & Philosophy`.
- Included plan, next steps, progress, and next tasks in the target document.
- Documented the work in `tasks/todo.md`.

### Next Tasks
- Draft Part 2 for information architecture and screen transitions.
- Convert the color, typography, spacing, and state rules into design tokens.
- Create wireframes for Context Rail, Auto-Action Kanban, and Learning Ledger.

## 2026-06-06 Design Document Refinement

### Plan
- [x] Review `docs/design.md` structure.
- [x] Confirm plain Japanese response rules.
- [x] Add a User Experience Philosophy section.
- [x] Expand robustness requirements for audit logs, backup, and error handling.
- [x] Add Gemma 4 model-to-artisan-task mapping.
- [x] Verify the final diff for consistency.

### Review Notes
- Preserve the existing English technical design style.
- Keep changes scoped to design requirements.
- Avoid implementation details that conflict with the current architecture.

### Progress
- Updated `docs/design.md` with UX philosophy, robust audit requirements, backup and degradation rules, and Gemma 4 artisan assignment.
- Verified that required headings exist in the document.
- Ran `git diff --check` for Markdown whitespace validation.

### Next Tasks
- Convert the AuditLogs requirements into a concrete schema.
- Add acceptance tests for local LLM downtime and send retry behavior.
- Prepare restore runbooks for the first operational drill.

## 2026-06-06 Robustness Detail Pass

### Plan
- [x] Confirm that UX philosophy, robustness, and Gemma 4 model mapping are present.
- [x] Add confidence-band behavior for AI output presentation.
- [x] Add audit write path requirements for critical actions.
- [x] Add degraded-mode matrix and robustness acceptance criteria.
- [x] Verify Markdown diff and whitespace.

### Progress
- Expanded `docs/design.md` with confidence presentation rules.
- Added fail-closed audit writer requirements for mail access, AI generation, approval, and send.
- Added degraded-mode handling for local LLM, audit writer, database, mail provider, and object storage failures.

### Next Tasks
- Convert the audit write path into API and database schema requirements.
- Define health-check thresholds for the local LLM circuit breaker.
- Add acceptance tests for degraded-mode UI states.

## 2026-06-06 Business Model & Monetization Section

### Plan
- [x] Review the existing `docs/design.md` tone and section order.
- [x] Add a coherent business model section that preserves the privacy-first philosophy.
- [x] Integrate consulting, managed local operations, enterprise edition, and white-label routes.
- [x] Verify the Markdown diff and record progress.

### Review Notes
- Keep the section aligned with the existing English technical design style.
- Emphasize trust, reliability, and auditability as the paid value.
- Avoid weakening the local-only AI and human-approval principles.

### Progress
- Added `Business Model & Monetization` to `docs/design.md`.
- Connected the new section to existing `Next Steps` and `Final Design Principles`.
- Verified Markdown whitespace with `git diff --check`.

### Next Tasks
- Define concrete packaging and pricing metrics for OSS Core, Enterprise Edition, Managed Local Operations, and partner offerings.
- Draft the enterprise support and SLA tiers.
- Decide which audit and retention features belong in OSS Core versus Enterprise Edition.

## 2026-06-06 Local PC Installation Design Update

### Plan
- [x] Review `docs/design.md` for centralized local-server assumptions.
- [x] Update the deployment model to local PC installation.
- [x] Add hardware dependency and recommended specifications.
- [x] Adjust architecture, privacy, tech stack, and implementation plan sections.
- [x] Record progress and verify the Markdown diff.

### Review Notes
- Preserve the existing English technical design tone.
- Keep external connectivity limited to mail ingestion, mail sending, and optional identity integration.
- State that AI processing, prompts, summaries, drafts, and model context stay on the user's local PC.
- Avoid reintroducing centralized inference servers or cloud AI fallbacks.

### Progress
- Updated `docs/design.md` to show the Backend API, LLM Orchestrator, `llama.cpp`, and Gemma 4 model running on the client's local machine.
- Added hardware-tier guidance for integrated graphics, mid-range dedicated GPUs, and high-end dedicated GPUs.
- Replaced centralized operations language with managed local operations language where it conflicted with the local PC model.
- Updated privacy, backup, tech stack, implementation, and next-step sections for local PC installation.

### Next Tasks
- Define the installer and local service startup flow.
- Add hardware detection details for CPU, GPU, system memory, and VRAM.
- Decide the exact model compatibility thresholds for Gemma 4 2B, 4B, and 12B.
