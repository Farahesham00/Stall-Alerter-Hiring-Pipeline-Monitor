# Stall Alerter -- Hiring Pipeline Monitor

## Overview

Stall Alerter is an automated, AI-powered workflow built in n8n to help
recruiters monitor a hiring pipeline and follow up with candidates whose
records have not been updated recently.

The workflow runs weekly, reads candidate records from a Google Sheet,
calculates how many days have passed since each candidate's last update,
filters candidates based on a stalling threshold, and sends the
recruiter an email report. An AI Agent ranks the filtered candidates by
urgency, assigns a priority, explains why follow-up is needed, and
recommends a next action.

## What the Workflow Does

1.  **Schedule Weekly Trigger** -- starts the workflow on a weekly
    schedule.
2.  **Get hiring board sheet** -- retrieves candidate records from the
    configured Google Sheet.
3.  **Calculate Days Since Last Update** -- calculates the number of
    days since each candidate's `Last Updated` date and adds a
    `days_since_update` field.
4.  **5 days threshold** -- filters the records using the configured day
    threshold.
5.  **Aggregate** -- combines the filtered candidate records into one
    input for the AI Agent.
6.  **AI Agent** -- uses Google Gemini to:
    -   Return every candidate received.
    -   Rank candidates by stage urgency, then by days stalled within
        the same stage.
    -   Assign High, Medium, or Low priority.
    -   Provide a short reason and a practical recommended action.
    -   Keep the reason and action consistent for candidates with the
        same stage and priority.
7.  **Structured Output Parser** -- validates the AI Agent's response
    against the expected structured output.
8.  **email formating** -- sorts the candidates by priority and days
    stalled, then builds an HTML email report.
9.  **Send a message** -- sends the report through Gmail to the
    configured recipient.

## Input Data

The workflow expects a Google Sheet containing candidate records with
these columns:

  Column           Description
  ---------------- --------------------------------------------
  `Candidate`      Candidate name
  `Stage`          Candidate's current hiring stage
  `Last Updated`   Date the candidate record was last updated

The workflow adds `days_since_update` during execution. This value is
calculated from `Last Updated`; it does not need to be entered manually
in the sheet.

## AI Prioritization Rules

The AI Agent receives candidates that have already passed the workflow's
stalling filter. It uses these rules:

### Stage urgency

`Offer Sent > Final Interview > Technical Interview > HR Interview > Applied`

### Priority categories

-   **High:** `Offer Sent`, or any candidate stalled for 10 days or
    more.
-   **Medium:** `Final Interview` or `Technical Interview` candidates
    stalled for fewer than 10 days.
-   **Low:** `HR Interview` or `Applied` candidates stalled for fewer
    than 10 days.

Within the same stage, a candidate with more days stalled is ranked as
more urgent.

The AI Agent is instructed to return every candidate and provide: -
`candidate` - `days_stalled` - `stage` - `priority` - `reason` -
`recommended_action`

Candidates with the same stage and priority should receive exactly the
same reason and recommended action.

## Requirements

Before running the workflow, make sure you have:

-   An n8n instance.
-   Access to the Google Sheet used as the hiring board.
-   A Google Sheets OAuth2 connection in n8n.
-   A Google Gemini API credential configured for the Gemini Chat Model
    node.
-   A Gmail OAuth2 connection in n8n with permission to send email.

## Setup

1.  Download the exported workflow JSON file.
2.  In n8n, select **Import from File** and import the workflow JSON.
3.  Open **Get hiring board sheet** and connect your Google Sheets
    account. Confirm that the correct hiring board document and
    worksheet are selected and accessible.
4.  Open **Google Gemini Chat Model** and configure your Gemini
    credential.
5.  Open **Send a message** and connect your Gmail account. Set the
    `Send To` field to the recruiter's email address.
6.  Review the weekly schedule in **Schedule Weekly Trigger** and set
    the desired schedule if needed.
7.  Review the stalling threshold in **5 days threshold**.
8.  Save the workflow.

> **Important threshold note:** In the supplied workflow JSON, the
> filter uses `days_since_update >= 5` (greater than or equal to five
> days). If the intended rule is strictly *more than five days*, change
> the filter operator to `greater than` (`gt`) with a value of `5`.

## How to Run

1.  Keep the workflow inactive while configuring credentials and
    testing.
2.  Test the workflow manually in n8n and check the output of each node.
3.  Confirm that the date calculation and threshold filter select the
    expected candidates.
4.  Check that the AI Agent returns all filtered candidates in the
    expected structure.
5.  Verify that the formatted email contains the correct candidate
    details, priorities, reasons, and recommended actions.
6.  Confirm the Gmail recipient is correct before allowing the workflow
    to send an email.
7.  Once testing is successful, activate the workflow so the weekly
    schedule can run automatically.

## Expected Result

The recruiter receives a structured email titled:

`Hiring Stall Alert – [number] Candidates Need Follow-up`

The email includes a table with each candidate's name, hiring stage,
number of days stalled, priority, reason for follow-up, and recommended
action.

## Scope and Limitations

Stall Alerter is a decision-support tool for recruiters. It helps
identify candidates who may need attention and suggests follow-up
actions.

It does not: - Contact candidates directly. - Update or change candidate
stages in the hiring board. - Make hiring decisions or accept/reject
candidates.

The recruiter remains responsible for reviewing the report and deciding
what action to take.

## Notes

-   The workflow export is initially **inactive**. Activate it only
    after credentials, the spreadsheet, the recipient, and the schedule
    have been checked.
-   Credential connections are specific to an n8n instance and may need
    to be selected again after importing the workflow.
-   The workflow uses the current hiring board data as its source of
    truth. Keep the column names consistent with the expected input
    fields.
