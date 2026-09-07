# AgencyFlow — Automated Client Onboarding System

AgencyFlow is a workflow automation project designed to reduce the repetitive manual work marketing agencies perform when onboarding new clients.

The system automates key onboarding steps—from capturing a new client's details to collecting intake information, creating a client workspace, generating onboarding tasks, and notifying the agency team.

Built with free and open-source tools, AgencyFlow demonstrates an end-to-end, production-style workflow automation solution with validation, duplicate detection, status tracking, and basic error handling.

---

## 📌 The Problem

Marketing agencies often onboard new clients through a manual, multi-step process.

A typical onboarding process includes:

* Sending welcome emails
* Requesting client and business information
* Collecting onboarding details
* Storing client information
* Creating a client workspace
* Creating onboarding tasks
* Notifying the agency team

When these steps are performed manually for every new client, agencies can face:

* Repetitive administrative work
* Delays between onboarding steps
* Forgotten or inconsistent processes
* Duplicate client records
* Difficulty scaling operations as the agency grows

---

## 💡 The Solution

AgencyFlow automates the client onboarding journey using two connected n8n workflows.

### Workflow 1 — New Client Onboarding

The first workflow:

1. Captures new client information
2. Validates required fields
3. Checks for duplicate clients
4. Creates a client record in Supabase
5. Updates the onboarding status
6. Sends a personalized welcome email
7. Provides the client with an onboarding intake form

### Workflow 2 — Client Intake Processing

The second workflow:

1. Captures the completed client intake form
2. Validates the submitted information
3. Finds and links the correct client
4. Stores intake information in Supabase
5. Creates a dedicated Notion client workspace
6. Creates standard onboarding tasks
7. Notifies the agency team
8. Updates the onboarding status to completed

---

# 🔄 How It Works

## Step 1 — New Client Submission

A new client submits their basic information through an n8n form.

The form collects:

* Client name
* Company name
* Email address
* Selected service

```text
New Client Form
       ↓
n8n Workflow
```

---

## Step 2 — Validate Client Information

The automation validates the submitted data before continuing.

It checks that:

* Required fields are provided
* The email address is valid
* Important information is not missing

Invalid submissions are prevented from continuing through the workflow.

---

## Step 3 — Duplicate Client Detection

Before creating a new client record, the workflow checks whether the email already exists in the database.

```text
Check Client Email
        ↓
Does Client Already Exist?
       / \
     Yes  No
      ↓    ↓
    Stop  Create Client
```

This helps prevent duplicate client records.

---

## Step 4 — Create Client Record

If the client does not already exist, their information is stored in Supabase.

The onboarding process is then started and tracked using a status field.

Example:

```text
new
↓
onboarding_started
```

---

## Step 5 — Send Welcome Email

The automation sends a personalized welcome email to the client.

The email includes:

* Client name
* Company name
* Welcome message
* Link to the onboarding intake form

---

## Step 6 — Client Completes Intake Form

The client submits detailed information through the onboarding form.

The form can collect:

* Website
* Industry
* Business description
* Marketing goals
* Target audience
* Current marketing tools
* Additional notes

---

## Step 7 — Store Intake Information

The workflow finds the correct client and links the submitted intake information to their client record.

```text
Client
   ↓
Client ID
   ↓
Client Intake Information
```

This creates a relational data structure inside Supabase.

---

## Step 8 — Create Client Workspace

After successful intake processing, the automation creates a dedicated client workspace in Notion.

The workspace contains:

* Client information
* Business details
* Selected service
* Marketing goals
* Onboarding information
* Onboarding tasks

---

## Step 9 — Create Onboarding Tasks

Standard onboarding tasks are automatically created.

Example:

* Review client intake form
* Review business information
* Request required account access
* Create campaign strategy
* Schedule kickoff meeting

---

## Step 10 — Notify Agency Team

Once the onboarding process is completed, the automation sends a notification to the agency team.

```text
Client Intake Completed
          ↓
Notion Workspace Created
          ↓
Onboarding Tasks Created
          ↓
Team Notification Sent
          ↓
Onboarding Completed
```

---

# 🏗️ System Architecture

```text
                    NEW CLIENT
                        │
                        ▼
                ┌───────────────┐
                │ New Client    │
                │ n8n Form      │
                └───────┬───────┘
                        │
                        ▼
                ┌───────────────┐
                │     n8n       │
                │ Workflow 1    │
                └───────┬───────┘
                        │
                        ▼
                ┌───────────────┐
                │   Supabase    │
                │ Client Record │
                └───────┬───────┘
                        │
                        ▼
                ┌───────────────┐
                │    Gmail      │
                │ Welcome Email │
                └───────┬───────┘
                        │
                        ▼
                CLIENT INTAKE FORM
                        │
                        ▼
                ┌───────────────┐
                │     n8n       │
                │ Workflow 2    │
                └───────┬───────┘
                        │
             ┌──────────┼──────────┐
             ▼          ▼          ▼
        Supabase      Notion      Gmail
       Store Intake   Workspace   Notify Team
```

---

# ⚙️ Tech Stack

| Purpose             | Technology                 |
| ------------------- | -------------------------- |
| Workflow Automation | n8n                        |
| Workflow Hosting    | Self-hosted n8n via Docker |
| Database            | Supabase / PostgreSQL      |
| Client Workspace    | Notion                     |
| Email Delivery      | Gmail                      |
| Forms               | n8n Forms                  |

All tools used in the project can be started using free tiers or open-source software.

---

# ✨ Key Features

* 📋 Web-based client onboarding forms
* ✅ Required field and email validation
* 🔁 Duplicate client detection
* 🗄️ Relational data storage in Supabase
* 📊 Client onboarding status tracking
* 📧 Personalized welcome emails
* 📝 Automated intake information processing
* 📓 Automatic Notion workspace creation
* ✅ Automated onboarding task creation
* 🔔 Agency team notifications
* 🛡️ Basic error handling for failed integrations
* 🔗 Two connected workflows for easier maintenance and debugging

---

# 🗄️ Database Schema

AgencyFlow uses Supabase PostgreSQL to store client and intake information.

## Clients Table

```text
clients
│
├── id (UUID)
├── client_name
├── company_name
├── email
├── service
├── onboarding_status
└── created_at
```

The `clients` table stores the primary information for every new client.

---

## Client Intake Table

```text
client_intake
│
├── id (UUID)
├── client_id (Foreign Key → clients.id)
├── website
├── industry
├── business_description
├── marketing_goals
├── target_audience
├── marketing_tools
├── additional_notes
└── submitted_at
```

---

## Relationship

```text
clients
   │
   │ 1
   │
   └───────────────┐
                   │
                   │ Many / Related Intake Data
                   ▼
             client_intake
```

The `client_id` field connects intake information to the correct client.

---

# 📊 Onboarding Status Flow

The onboarding lifecycle is tracked through the following statuses:

```text
new
 ↓
onboarding_started
 ↓
intake_received
 ↓
completed
```

Status tracking makes the automation easier to monitor, debug, and extend.

---

# 🧩 Workflow Overview

## Workflow 1 — New Client Onboarding

```text
New Client Form
       ↓
Validate Input
       ↓
Check Duplicate Client
       ↓
Create Client Record
       ↓
Update Status
       ↓
Send Welcome Email
       ↓
Send Intake Form
```

---

## Workflow 2 — Client Intake Processing

```text
Client Intake Form
        ↓
Validate Data
        ↓
Find Client
        ↓
Store Intake Data
        ↓
Update Client Status
        ↓
Create Notion Workspace
        ↓
Create Onboarding Tasks
        ↓
Send Team Notification
        ↓
Update Status: Completed
```

---

# 🛡️ Error Handling

The automation handles several important scenarios.

## Duplicate Client

Before creating a client, the system checks whether the email already exists.

```text
Duplicate Email
      ↓
Do Not Create Another Client
```

---

## Missing Required Information

Required fields are validated before processing continues.

Invalid or incomplete submissions are prevented from moving through the workflow.

---

## Client Not Found

If an intake form is submitted without a matching client record:

* The workflow stops safely
* The project/workspace is not created incorrectly
* The issue can be logged or handled through an error notification

---

## Failed Third-Party Integration

If an integration such as Notion fails:

* The workflow should not silently fail
* The failure can be captured
* An administrator notification can be sent

---

# 📂 Project Structure

```text
agency-client-onboarding-automation/
│
├── README.md
│
├── docs/
│   ├── architecture.md
│   ├── workflow.md
│   └── screenshots/
│
└── n8n-workflows/
    ├── workflow-1.json
    └── workflow-2.json
```

---

# 🚀 Setup and Running Locally

## Prerequisites

Before running the project, you need:

* n8n
* Docker (recommended for self-hosting n8n)
* A Supabase account/project
* A Notion account and integration
* A Gmail account for email delivery

---

## 1. Set Up n8n

You can run n8n locally using Docker or another supported installation method.

Import the workflow files:

```text
n8n-workflows/workflow-1.json
n8n-workflows/workflow-2.json
```

---

## 2. Set Up Supabase

Create a Supabase project and create the required tables:

```text
clients
client_intake
```

Configure the relationship:

```text
client_intake.client_id
        ↓
clients.id
```

Add your Supabase credentials to the required n8n nodes.

---

## 3. Set Up Notion

Create a Notion integration and connect it to the workspace/database where client workspaces will be created.

Add the Notion credentials to the relevant n8n nodes.

---

## 4. Configure Gmail

Connect Gmail or configure SMTP/App Password authentication for email delivery.

The project uses Gmail for:

* Welcome emails
* Agency team notifications

---

## 5. Configure n8n Credentials

Connect the required credentials inside n8n:

* Supabase
* Notion
* Gmail

⚠️ Never commit API keys, passwords, access tokens, or credentials to this repository.

---

## 6. Test the Automation

### Test Workflow 1

1. Submit a new client through the form.
2. Verify validation.
3. Test duplicate detection.
4. Check the Supabase client record.
5. Confirm the welcome email is sent.

### Test Workflow 2

1. Submit the client intake form.
2. Verify intake information is stored.
3. Check the client status.
4. Confirm the Notion workspace is created.
5. Verify onboarding tasks.
6. Confirm the agency notification email.

---

# 🖼️ Screenshots

The `docs/screenshots/` directory contains visual proof of the project.

Recommended screenshots include:

### Workflow Automation

* Complete Workflow 1
* Complete Workflow 2

### Forms

* New Client Form
* Client Intake Form

### Database

* Supabase Clients Table
* Supabase Client Intake Table

### Integrations

* Created Notion Client Workspace
* Welcome Email
* Agency Team Notification

### Execution

* Successful workflow execution

---

# 🎯 Business Value

AgencyFlow demonstrates how workflow automation can help marketing agencies:

* Reduce repetitive onboarding tasks
* Standardize client onboarding
* Reduce the risk of forgotten steps
* Maintain structured client information
* Improve operational consistency
* Make onboarding workflows easier to scale

This project is designed as a **portfolio/demo solution** and does not claim specific time or cost savings without real-world measurement.

---

# 🧭 Possible Version 2 Enhancements

Potential improvements include:

* 🤖 AI-generated client intake summaries
* 💬 Slack notifications
* 📁 Automated Google Drive folder creation
* 📅 Automated kickoff meeting scheduling
* 🔗 CRM integration
* 📊 Onboarding dashboard
* 📜 More detailed workflow monitoring and logs

---

# ⚠️ Disclaimer

This is a portfolio/demo project created to demonstrate workflow automation, API integration, database design, and operational automation skills.

It is not presented as real client work.

Credentials, API keys, passwords, and access tokens are never committed to this repository.

---

## 👤 Author

Built by **Farhan Naeem** as an AI and workflow automation portfolio project demonstrating end-to-end business automation for marketing agencies.
