# AgencyFlow — Automated Client Onboarding System

AgencyFlow is a workflow automation project that eliminates the repetitive manual work marketing agencies face when onboarding new clients. Built entirely with free and open-source tools, it demonstrates a complete, production-style automation pipeline — from first contact to a fully set-up client workspace — with real error handling and reliability built in.

## The Problem

Marketing agencies typically onboard new clients through a manual, multi-step process: sending welcome emails, requesting business information, creating project workspaces, setting up onboarding tasks, and notifying the team — all done by hand, for every single client. This leads to:

- Repetitive administrative work
- Delays between steps
- Forgotten steps and inconsistent onboarding
- Difficulty scaling as the agency grows

## The Solution

AgencyFlow automates the entire onboarding journey using two orchestrated workflows:

1. **New Client Onboarding** — captures a new client's details, validates the input, checks for duplicates, creates a client record, and sends a personalized welcome email.
2. **Client Intake Processing** — captures detailed business and marketing information, links it to the correct client, creates a dedicated Notion workspace with an onboarding task checklist, and notifies the agency team once everything is ready.

## Tech Stack (100% Free Tier)

| Purpose | Tool |
|---|---|
| Workflow automation / orchestration | [n8n](https://n8n.io) (self-hosted via Docker) |
| Database | [Supabase](https://supabase.com) (PostgreSQL, free tier) |
| Client workspace & task management | [Notion](https://notion.so) (free plan + API integration) |
| Email delivery | Gmail (SMTP with App Password) |

No paid APIs, subscriptions, or credits are required to run this project.

## Key Features

- 📋 Web-based intake forms (hosted natively by n8n — no third-party form service needed)
- ✅ Input validation (required fields, email format)
- 🔁 Duplicate client detection by email
- 🗄️ Relational data storage in Supabase (clients ↔ client_intake, linked by `client_id`)
- 📓 Automatic Notion workspace creation per client, complete with a 5-item onboarding checklist
- 📧 Dynamic, personalized email notifications (welcome email + team notification)
- 📊 Full onboarding status tracking (`new` → `onboarding_started` → `intake_received` → `completed`)
- 🛡️ Built-in error handling:
  - Gracefully stops if an intake form is submitted with no matching client
  - Catches failed third-party integrations (e.g. Notion API errors) and sends an admin alert instead of failing silently

## Project Structure

```
agency-client-onboarding-automation/
│
├── README.md                  ← you are here
├── docs/
│   ├── architecture.md        ← system architecture overview
│   ├── workflow.md            ← step-by-step breakdown of both workflows
│   └── screenshots/           ← visual proof of each working component
└── n8n-workflows/
    ├── workflow-1.json        ← New Client Onboarding
    └── workflow-2.json        ← Client Intake Processing
```

## Status

Version 1 (MVP) is complete and fully functional, covering every "Must Have" requirement from the project spec: both workflows, validation, duplicate protection, status tracking, and reliability/error handling.

Potential Version 2 additions: an AI-generated client intake summary, Slack notifications, automated Google Drive folder creation, and a simple dashboard.

## Disclaimer

This is a portfolio/demo project built to showcase workflow automation and integration skills. Credentials and API keys are never committed to this repository.