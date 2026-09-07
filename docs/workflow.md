# Workflow Breakdown

This document describes the two n8n workflows used in AgencyFlow, including each node's role, the order in which data flows, validation logic, status updates, and error handling.

AgencyFlow uses two independent workflows because client onboarding happens in stages. A client may submit their basic information first and complete the detailed intake form later.

---

# Workflow Overview

## Workflow 1 — New Client Onboarding

```text
New Client Form
       ↓
Validate Input
       ↓
Check for Duplicate
       ↓
Create Client Record
       ↓
Send Welcome Email
```

## Workflow 2 — Client Intake Processing

```text
Client Intake Form
        ↓
Find Client
        ↓
Validate Client
        ↓
Store Intake Information
        ↓
Update Client Status
        ↓
Create Notion Workspace
        ↓
Send Team Notification
        ↓
Mark Onboarding Completed
```

---

# Workflow 1 — New Client Onboarding

**Trigger:** n8n Form Trigger, hosted natively in n8n. No external form service is required.

This workflow handles the first stage of onboarding by collecting a new client's basic information, validating it, checking for duplicates, storing the client record, and sending a welcome email.

| Step | Node                                     | Purpose                                                                                                                                                                                                  |
| ---- | ---------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | **New Client Onboarding** (Form Trigger) | Collects Client Name, Company Name, Email, and Selected Service from a new client.                                                                                                                       |
| 2    | **If** (Validation)                      | Confirms Client Name, Company Name, and Email are present and verifies that the email matches a valid email format using regex. Invalid submissions are routed to a `No Operation` node and stop safely. |
| 3    | **Get many rows** (Supabase)             | Searches the `clients` table for an existing row with the same email address. `Always Output Data` is enabled so the workflow continues even when zero matches are found.                                |
| 4    | **Check Duplicate** (If)                 | Determines whether the lookup returned an existing client. Duplicate submissions are routed to a `No Operation` node and stop safely, preventing duplicate client records.                               |
| 5    | **Create a row** (Supabase)              | Creates a new row in the `clients` table using the submitted details and sets `onboarding_status` to `onboarding_started`.                                                                               |
| 6    | **Send Email** (SMTP)                    | Sends a personalized welcome email to the client, dynamically including their name and company information and inviting them to complete the client intake form.                                         |

## Workflow 1 Result

After Workflow 1 completes successfully:

```text
Client Record Created
        ↓
Status: onboarding_started
        ↓
Welcome Email Sent
        ↓
Client Receives Intake Form
```

The client can complete the intake form immediately or at a later time.

---

# Workflow 2 — Client Intake Processing

**Trigger:** A second independent n8n Form Trigger.

This workflow handles the detailed client intake process. It can run after Workflow 1 whenever the client completes the intake form.

| Step | Node                                       | Purpose                                                                                                                                                                                                                                                                           |
| ---- | ------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | **On form submission** (Form Trigger)      | Collects Email to identify the client, along with Website URL, Industry, Business Description, Marketing Goals, Target Audience, Marketing Tools, and Additional Notes.                                                                                                           |
| 2    | **Find Client** (Supabase — Get many rows) | Searches the `clients` table using the submitted email address to find the matching client record. `Always Output Data` is enabled so the workflow can detect a non-match instead of stopping unexpectedly.                                                                       |
| 3    | **Client Found?** (If)                     | Checks whether a matching client record was found. If no matching client exists, the workflow safely stops at a `No Operation` node rather than creating an orphaned intake record. This handles the **Client Not Found** error case.                                             |
| 4    | **Create a row** (Supabase)                | Creates a new row in the `client_intake` table and links it to the matched client using `client_id`. All submitted business and marketing details are stored.                                                                                                                     |
| 5    | **Update** (Supabase)                      | Updates the matched client's `onboarding_status` to `intake_received`.                                                                                                                                                                                                            |
| 6    | **Create a page** (Notion)                 | Creates a dedicated client sub-page under the **AgencyFlow Clients** parent page. The page is titled using the client's company name and contains their service, website, marketing goals, onboarding status, and a five-item onboarding checklist as native Notion to-do blocks. |
| 7    | **Send Email** (SMTP)                      | Sends a notification to the agency team confirming that the client's onboarding is complete. The notification includes the company name, selected service, and website.                                                                                                           |
| 8    | **Update** (Supabase)                      | Updates the client's `onboarding_status` to `completed`, marking the successful completion of the onboarding pipeline.                                                                                                                                                            |

---

# Workflow 2 Result

After Workflow 2 completes successfully:

```text
Client Intake Submitted
        ↓
Client Identified
        ↓
Intake Data Stored
        ↓
Status: intake_received
        ↓
Notion Workspace Created
        ↓
Team Notification Sent
        ↓
Status: completed
```

---

# Error Handling

AgencyFlow includes validation and error paths to prevent failures from silently creating incorrect or incomplete onboarding data.

## 1. Invalid Client Submission

Workflow 1 validates:

* Client Name
* Company Name
* Email address
* Email format

If the required information is missing or invalid:

```text
Invalid Submission
        ↓
No Operation
        ↓
Workflow Stops Safely
```

No client record is created.

---

## 2. Duplicate Client Detection

Before creating a new client, Workflow 1 checks whether the submitted email address already exists.

```text
Check Client Email
        ↓
Duplicate Found?
      /         \
    Yes          No
     ↓            ↓
Stop Safely   Create Client
```

This prevents duplicate client records.

---

## 3. Client Not Found

Workflow 2 searches for the client using the submitted email address.

If no matching client record is found:

```text
Find Client
        ↓
Client Found?
      /         \
    No           Yes
     ↓            ↓
Stop Safely   Continue Workflow
```

This prevents orphaned intake records from being created.

---

## 4. Failed Notion Integration

The **Create a page** (Notion) node uses the `Continue Using Error Output` setting.

If the Notion API call fails because of issues such as:

* Invalid credentials
* API errors
* Rate limits
* Temporary integration failures

The workflow follows a dedicated error branch.

### Error Handling Flow

```text
Create Notion Workspace
          │
     ┌────┴────┐
     │         │
  Success     Error
     │         │
     ▼         ▼
Send Team    Send Error
Notification Notification
     │         │
     ▼         ▼
Mark        Agency Admin
Completed
```

The error notification includes relevant information such as:

* Affected client
* Company name
* Failed workflow step
* Error message

This ensures that failures are visible and actionable instead of silent.

---

# Status Lifecycle

The client's `onboarding_status` tracks progress through the onboarding process.

```text
new
 ↓
onboarding_started
 ↓
intake_received
 ↓
completed
```

### Status Definitions

| Status               | Meaning                                                                                                     |
| -------------------- | ----------------------------------------------------------------------------------------------------------- |
| `new`                | Initial client state before onboarding begins, if configured as a database default.                         |
| `onboarding_started` | Client record has been created and the onboarding process has started.                                      |
| `intake_received`    | The client intake form has been successfully processed and stored.                                          |
| `completed`          | The Notion workspace and onboarding tasks were successfully created and the onboarding process is complete. |

If a client cannot be found during intake processing or an external integration fails, the workflow follows its dedicated error path rather than silently advancing the client's onboarding status.

---

# Workflow Design Principles

## Decoupled Workflows

The two workflows are intentionally separated.

```text
Workflow 1
Client Registration
        ↓
Client Completes Intake Later
        ↓
Workflow 2
Intake Processing
```

This reflects a real agency onboarding process because clients may complete the intake form hours or days after their initial registration.

---

## Single Source of Truth

Supabase stores the structured client and intake data.

```text
Workflow 1
     ↓
Supabase
     ↓
Workflow 2
     ↓
Supabase
```

Notion and Gmail are used for workspace management and communication, while Supabase remains the central database for client records and onboarding status.

---

## Safe Processing

The workflows include checks for:

* Missing required fields
* Invalid email formats
* Duplicate clients
* Missing clients
* Failed Notion integrations

This helps prevent incorrect records and silent workflow failures.

---

# Summary

AgencyFlow uses two connected but independently triggered n8n workflows to automate the client onboarding process.

```text
New Client
     ↓
Workflow 1
     ↓
Validate Client
     ↓
Check Duplicate
     ↓
Create Client Record
     ↓
Send Welcome Email
     ↓
Client Completes Intake
     ↓
Workflow 2
     ↓
Find Client
     ↓
Store Intake Data
     ↓
Create Notion Workspace
     ↓
Notify Agency Team
     ↓
Mark Onboarding Completed
```

The workflow design focuses on reliability, clear data ownership, independent processing stages, and safe handling of common onboarding errors.
