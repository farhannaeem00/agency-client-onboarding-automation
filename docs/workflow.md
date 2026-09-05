# Workflow Breakdown

This document describes each node's role in both n8n workflows, in the order data flows through them.

---

## Workflow 1 — New Client Onboarding

**Trigger:** n8n Form Trigger, hosted natively — no external form service required.

| Step | Node | Purpose |
|---|---|---|
| 1 | **New Client Onboarding** (Form Trigger) | Collects Client Name, Company Name, Email, and Selected Service from a new client. |
| 2 | **If** (Validation) | Confirms Client Name, Company Name, and Email are all present, and that the Email matches a valid email format via regex. Invalid submissions are routed to a `No Operation` node and stop safely. |
| 3 | **Get many rows** (Supabase) | Searches the `clients` table for any existing row with the same email. "Always Output Data" is enabled so the workflow continues even when zero matches are found. |
| 4 | **Check Duplicate** (If) | Confirms whether the lookup returned an empty result (no duplicate) or an existing record (duplicate). Duplicates are routed to a `No Operation` node and stop safely, preventing double client records. |
| 5 | **Create a row** (Supabase) | Inserts a new row into `clients` with the submitted details and `onboarding_status` set to `onboarding_started`. |
| 6 | **Send Email** (SMTP) | Sends a personalized welcome email to the client, dynamically including their name and company, inviting them to complete the intake form. |

---

## Workflow 2 — Client Intake Processing

**Trigger:** A second, independent n8n Form Trigger — this can run at any point after Workflow 1, since a client's intake form is typically completed separately.

| Step | Node | Purpose |
|---|---|---|
| 1 | **On form submission** (Form Trigger) | Collects Email (to identify the client) plus Website URL, Industry, Business Description, Marketing Goals, Target Audience, Marketing Tools, and Additional Notes. |
| 2 | **Find Client** (Supabase, Get many rows) | Looks up the `clients` table by email to find the matching client record. "Always Output Data" is enabled so the workflow can detect a non-match instead of stalling. |
| 3 | **Client Found?** (If) | Checks whether a matching client was actually found. If not, the workflow safely stops at a `No Operation` node rather than attempting to create orphaned records — this is the **Client Not Found** error case from the reliability requirements. |
| 4 | **Create a row** (Supabase) | Inserts a new row into `client_intake`, linked to the matched client via `client_id`, with all submitted business/marketing details. |
| 5 | **Update** (Supabase) | Updates the matched client's `onboarding_status` to `intake_received`. |
| 6 | **Create a page** (Notion) | Creates a new sub-page under the "AgencyFlow Clients" parent page, titled after the client's company name, containing their service, website, marketing goals, and status — followed by a 5-item onboarding checklist (Review intake form, review business info, request account access, create campaign strategy, schedule kickoff meeting) as native Notion to-dos. |
| 7 | **Send Email** (SMTP) | Notifies the agency team that a client's onboarding is complete, including their company name, service, and website. |
| 8 | **Update** (Supabase) | Sets the client's `onboarding_status` to `completed`, marking the end of the pipeline. |

### Error Handling — Failed Integration

The **Create a page** (Notion) node has its `On Error` setting configured to **Continue Using Error Output** rather than stopping the workflow. If the Notion API call fails for any reason (invalid credentials, rate limits, etc.), execution is routed to a dedicated error branch that sends an **error notification email** to the agency admin, including the affected client's name and the specific error message — ensuring failures are surfaced and actionable rather than silent.

---

## Status Lifecycle

A client's `onboarding_status` progresses through the following states across both workflows:

```
new → onboarding_started → intake_received → completed
```

(A client who is never found during intake processing, or whose intake fails partway through an integration, is flagged via the error paths above rather than silently advancing status.)