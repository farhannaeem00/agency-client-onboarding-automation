# Architecture

## Overview

AgencyFlow is built around a single automation engine (n8n) that orchestrates data flow between a form-based front door, a relational database, a client-facing workspace tool, and email notifications. Each service plays exactly one role, keeping the system simple, debuggable, and free to run.

## High-Level Data Flow

AgencyFlow uses two connected workflows that automate different stages of the client onboarding journey.

### Complete Onboarding Journey

```
                         NEW CLIENT
                              │
                              ▼
                    ┌─────────────────┐
                    │ New Client Form │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │ Workflow 1      │
                    │ Client Onboarding│
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │    Supabase     │
                    │ Client Record   │
                    └────────┬────────┘
                             │
                  ┌──────────┴──────────┐
                  ▼                     ▼
          Welcome Email          Update Status
          + Intake Link        waiting_for_intake
                  │
                  ▼
           ┌───────────────┐
           │ Client Intake │
           │     Form      │
           └───────┬───────┘
                   │
                   ▼
          ┌──────────────────┐
          │   Workflow 2     │
          │ Intake Processing│
          └────────┬─────────┘
                   │
                   ▼
          ┌──────────────────┐
          │    Supabase      │
          │ Store Intake +   │
          │ Update Status    │
          └────────┬─────────┘
                   │
        ┌──────────┼──────────┐
        ▼          ▼          ▼
   ┌────────┐ ┌──────────┐ ┌─────────┐
   │ Notion │ │ Supabase │ │  Gmail  │
   │Client  │ │  Status  │ │  Team   │
   │Workspace│ │Completed │ │Notification│
   └────────┘ └──────────┘ └─────────┘
```

### Error Handling Flow

```
Any Workflow Step
        │
        ▼
Integration or Processing Error
        │
        ▼
Error Handling Logic
        │
        ▼
Error Notification
        │
        ▼
Agency Admin
```

This architecture keeps Supabase as the central source of truth while allowing Workflow 1 and Workflow 2 to operate independently at different stages of the onboarding process.


## Components

### n8n — Orchestration Layer
Self-hosted via Docker, n8n is the brain of the system. It hosts both intake forms directly (no third-party form tool needed), runs all validation and branching logic, and calls out to every other service via its native nodes. Running self-hosted keeps the entire automation layer free indefinitely, with no trial expiry.

### Supabase — Data Layer
A free-tier PostgreSQL database holds two related tables:

- **`clients`** — one row per client, including contact info, selected service, and current `onboarding_status`.
- **`client_intake`** — one row per intake submission, linked back to its client via a `client_id` foreign key.

Supabase's REST API is called directly from n8n for all read/write operations — no custom backend required.

### Notion — Client Workspace Layer
Each client gets an individual page (nested under a parent "AgencyFlow Clients" page) summarizing their service, website, and goals, plus a 5-item onboarding checklist as native Notion to-do blocks. This gives the agency team a ready-to-use project space the moment intake is processed.

### Gmail — Notification Layer
SMTP (via a Gmail App Password) sends two kinds of emails:
1. A personalized **welcome email** to the client immediately after their record is created.
2. A **team notification** to the agency once a client's Notion workspace and tasks are ready.

If any external call in the pipeline fails (e.g. a Notion API error), a separate **error notification** is sent to the agency admin instead of the workflow failing silently.

## Why This Design

- **Single source of truth**: Supabase holds all structured client data, so status and history are always queryable, independent of email or Notion state.
- **Decoupled workflows**: Workflow 1 (client creation) and Workflow 2 (intake processing) are separate and independently triggerable, mirroring how a real agency's process unfolds over time — a client may sign up today and complete their intake form days later.
- **Free-tier friendly**: Every component was deliberately chosen to have a genuinely free, non-expiring tier suitable for real (if small-scale) production use, not just a trial.
- **Fail-safe by design**: Rather than assuming every step succeeds, the workflows explicitly check for duplicate clients, missing clients, and failed third-party calls, and respond to each case deliberately instead of crashing.
