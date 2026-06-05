# Corporate Mailer Technical Design

## Introduction

This document defines the technical design for a privacy-first corporate mailer.

The system helps employees process business email by summarizing incoming messages and drafting replies with a local AI model. All AI processing runs on local infrastructure. No email body, attachment text, prompt, summary, draft, or model context is sent to an external AI service.

Core workflow:

1. Mail ingestion
2. Local LLM summary and reply drafting
3. User review and edit
4. Send approved email

Key goals:

- Keep AI processing 100% local using Gemma 4 2B, 4B, or 12B through `llama.cpp`
- Protect corporate email data and user privacy
- Use a modular backend and frontend architecture
- Require human approval before sending any AI-assisted message
- Support auditing, security controls, and future model upgrades

## Architecture

### High-Level Architecture

```text
[Mail Server / IMAP / Graph API]
            |
            v
[Mail Ingestion Service]
            |
            v
[Backend API - FastAPI]
            |
            +--> [Local Mail Store / Metadata DB]
            |
            +--> [LLM Orchestrator]
                    |
                    v
              [llama.cpp Server]
                    |
                    v
          [Gemma 4 Local Model]
            |
            v
[User Review UI - Next.js]
            |
            v
[Send Service - SMTP / Graph API]
```

### Main Modules

#### Mail Ingestion Service

Responsible for receiving and normalizing mail data.

Responsibilities:

- Fetch incoming mail from IMAP, Microsoft Graph, or another corporate mail API
- Parse headers, body, attachments, recipients, and thread metadata
- Store only required metadata and content
- Queue messages for local summarization
- Deduplicate messages by message ID and thread ID

#### Backend API

Implemented with Python and FastAPI.

Responsibilities:

- Provide authenticated APIs for the frontend
- Manage mail state, summaries, drafts, and review status
- Enforce access control
- Call the local LLM orchestrator
- Record audit logs for review and send actions

#### LLM Orchestrator

Responsible for safe and repeatable local AI execution.

Responsibilities:

- Build prompts from mail content and policy templates
- Call `llama.cpp` locally
- Select Gemma 4 model size based on workload
- Apply output validation
- Return summaries, suggested actions, and draft replies
- Prevent direct sending from AI output

Model selection example:

- Gemma 4 2B: quick summaries, low-resource devices
- Gemma 4 4B: balanced summary and drafting
- Gemma 4 12B: higher-quality drafting and complex threads

#### User Review UI

Implemented with React and Next.js.

Responsibilities:

- Show inbox, thread details, summaries, and AI-generated drafts
- Let users edit drafts before sending
- Clearly mark AI-generated content
- Require explicit user action before sending
- Show confidence notes, missing context warnings, and policy flags

#### Send Service

Responsible for sending only user-approved messages.

Responsibilities:

- Send email through SMTP, Microsoft Graph, or approved provider API
- Verify sender identity and recipient list
- Prevent AI-generated drafts from being sent automatically
- Store send status and audit metadata

## Tech Stack

### Backend

- Language: Python
- Framework: FastAPI
- API style: REST, with optional WebSocket support for live processing status
- Background jobs: Celery, Dramatiq, RQ, or FastAPI background workers
- Database: PostgreSQL for production, SQLite for local development
- Cache or queue: Redis if asynchronous scale is required
- Auth: SSO/OIDC, SAML, or corporate identity provider integration

### Frontend

- Framework: React with Next.js
- UI responsibilities:
  - Mail list
  - Thread view
  - Summary panel
  - Draft editor
  - Approval and send controls
- Security:
  - Server-side session validation
  - CSRF protection
  - Role-aware rendering
  - No sensitive mail content stored in browser local storage

### Local LLM

- Runtime: `llama.cpp`
- Model format: GGUF
- Models: Gemma 4 2B, 4B, and 12B
- Deployment mode:
  - Local machine for small teams
  - Internal server for company-wide deployment
  - No external AI API calls

### Storage

- Mail metadata: PostgreSQL
- Mail body: encrypted database field or encrypted object storage
- Attachments: encrypted storage with strict access control
- Audit logs: append-only table or dedicated log store

## Data Privacy

### Local-Only AI Policy

All AI processing must run inside the company-controlled environment.

The system must not send the following data to external AI services:

- Email body
- Email subject
- Recipient or sender addresses
- Attachments
- Prompt text
- Summaries
- Draft replies
- Embeddings
- Conversation history

### Data Minimization

The system stores only what it needs.

Recommended rules:

- Store message metadata separately from message content
- Avoid storing full prompts unless audit policy requires it
- Keep temporary LLM context in memory when possible
- Delete generated drafts when users discard them
- Apply retention limits for summaries and audit records

### Encryption

Required controls:

- TLS for all internal HTTP traffic
- Encryption at rest for database and attachment storage
- Secret management for mail credentials and signing keys
- Per-environment keys for development, staging, and production

### Access Control

Required controls:

- SSO login
- User-level mailbox permissions
- Role-based admin access
- Strict tenant or department boundaries if used across teams
- Audit logs for mail access, draft generation, edits, and sends

### Human Approval

AI must never send email directly.

The send flow must require:

1. User opens the draft
2. User reviews or edits the content
3. User confirms recipients
4. User clicks send
5. Backend records approval and sends the message

## Implementation Plan

### Phase 1: Foundation

- Create FastAPI backend project
- Create Next.js frontend project
- Define database schema for users, messages, summaries, drafts, and audit logs
- Add authentication and session handling
- Set up local development environment

### Phase 2: Mail Ingestion

- Implement IMAP or corporate mail API connector
- Parse and normalize incoming messages
- Store mail metadata and encrypted body content
- Add background ingestion jobs
- Add inbox and thread APIs

### Phase 3: Local LLM Integration

- Deploy `llama.cpp` locally
- Load Gemma 4 2B, 4B, and 12B GGUF models
- Implement LLM orchestrator service
- Create prompt templates for:
  - Message summary
  - Thread summary
  - Reply draft
  - Action item extraction
- Add timeout, retry, and output validation

### Phase 4: User Review Workflow

- Build inbox UI
- Build thread detail view
- Display AI summaries and draft replies
- Add draft editor
- Add explicit approval and send action
- Show warnings for uncertain or incomplete AI output

### Phase 5: Secure Sending

- Implement SMTP or Graph API send service
- Validate sender, recipients, and approval state
- Prevent unreviewed AI drafts from being sent automatically
- Record send audit logs
- Add failure handling and retry policy

### Phase 6: Security and Operations

- Add encryption at rest
- Add audit log review tools
- Add role-based admin controls
- Add monitoring for ingestion, LLM latency, and send failures
- Add backup and retention policies
- Run security review before production rollout

### Phase 7: Validation

Required checks:

- Unit tests for mail parsing, prompt building, and send validation
- Integration tests for ingestion, LLM generation, review, and send flow
- Frontend tests for review and approval screens
- Security tests for access control and data leakage
- Manual review with real corporate email scenarios in a controlled test environment

## Final Design Principles

- AI is an assistant, not an autonomous sender
- Privacy is enforced by architecture, not only by policy
- Local models are swappable through the LLM orchestrator
- Mail data access is auditable
- The user remains responsible for final communication
