# Workflow Implementation Guide: Feedback Reminder Automation

This document details the configuration and logic required to implement the **Feedback Reminder Workflow**. The solution is decoupled into three separate flows to ensure modularity and scalability.

## 1. Environment Variables & Configuration
To ensure portability across environments (Dev, Test, Prod), the following values should be stored as **Environment Variables** (or a Configuration List) rather than hardcoded in the flow.

| Variable Name | Default Value | Description |
| :--- | :--- | :--- |
| `ENV_SiteUrl` | `https://yourtenant.sharepoint.com/sites/yoursite` | The absolute URL of the SharePoint site. |
| `ENV_RequestList` | `Request Access` | The name of the SharePoint list. |
| `ENV_ThresholdDays` | `14` | Days after download to trigger feedback requirement. |
| `ENV_FollowUpDays` | `7` | Days after first reminder to send a follow-up. |

---

## 2. SharePoint List Configuration

The following columns must be added to the **Request Access List** to support these workflows.
**Note**: All columns use the `fdbx_` prefix.

| Display Name | Internal Name (recommended) | Type | Configuration | Purpose |
| :--- | :--- | :--- | :--- | :--- |
| **Feedback Provided?** | `fdbx_FeedbackProvided` | Yes/No | Default: `No` | Tracks if the user has completed the feedback form. |
| **Download Date** | `fdbx_DownloadDate` | Date & Time | Date Only | Captures when the user clicked/accessed the document. |
| **Feedback Required?** | `fdbx_FeedbackRequired` | Yes/No | Default: `No` | **System Flag**. Calculated by Flow 1. |
| **Last Reminder Sent** | `fdbx_LastReminderSent` | Date & Time | Date & Time | Tracks when the last email was sent. |
| **Email Trigger** | `fdbx_EmailTrigger` | Choice | Options: `None`, `Initial`, `Reminder` | **Trigger Column**. Used by Flow 2 to allow Flow 3 to send the mail. |

---

## 3. Power Automate Workflow Design

### Flow 1: "Set Feedback Required"
**Goal**: Identify items that have matured past the 14-day threshold.
**Trigger**: Recurrence (Schedule) - Daily (e.g., 9:00 AM)

#### Process Definition:
1.  **Initialize Variable**:
    -   Name: `varThresholdDays`
    -   Type: Integer
    -   Value: `@{parameters('ENV_ThresholdDays')}` (Default: 14)

2.  **Get items** (SharePoint):
    -   *Site Address*: `@{parameters('ENV_SiteUrl')}`
    -   *List Name*: `@{parameters('ENV_RequestList')}`
    -   *Filter Query*: `fdbx_DownloadDate ne null and fdbx_FeedbackRequired eq 0`
    -   *Top Count*: 5000 (pagination enabled if needed)

3.  **Apply to each** (Value from 'Get items'):
    -   **Compose ("Calculate Days Diff")**:
        -   *Inputs*: `div(sub(ticks(utcNow()), ticks(items('Apply_to_each')?['fdbx_DownloadDate'])), 864000000000)`
    
    -   **Condition ("Check Threshold")**:
        -   *Left*: `outputs('Calculate_Days_Diff')`
        -   *Operator*: `is greater than`
        -   *Right*: `variables('varThresholdDays')`

    -   **If Yes**:
        -   **Update item** (SharePoint):
            -   *Id*: `items('Apply_to_each')?['ID']`
            -   *fdbx_FeedbackRequired*: `Yes`

---

### Flow 2: "Calculate Reminder Requirements"
**Goal**: Determine if an email needs to be sent (Initial vs Reminder).
**Trigger**: Recurrence (Schedule) - Daily (Runs 30 mins after Flow 1)

#### Process Definition:
1.  **Initialize Variable**:
    -   Name: `varFollowUpDays`
    -   Type: Integer
    -   Value: `@{parameters('ENV_FollowUpDays')}` (Default: 7)

2.  **Get items** (SharePoint):
    -   *Site Address*: `@{parameters('ENV_SiteUrl')}`
    -   *List Name*: `@{parameters('ENV_RequestList')}`
    -   *Filter Query*: `fdbx_FeedbackRequired eq 1 and fdbx_FeedbackProvided eq 0 and fdbx_EmailTrigger eq 'None'`

3.  **Apply to each** (Value from 'Get items'):
    -   **Compose ("Last Reminder Date")**:
        -   *Inputs*: `items('Apply_to_each')?['fdbx_LastReminderSent']`
    
    -   **Condition ("Is Initial Email?")**:
        -   *Left*: `outputs('Last_Reminder_Date')`
        -   *Operator*: `is equal to`
        -   *Right*: `null`

    -   **If Yes (Branch A: Initial)**:
        -   **Update item** (Set Trigger to Initial):
            -   *fdbx_EmailTrigger*: `Initial`

    -   **If No (Branch B: Check Follow-up)**:
        -   **Compose ("Days Since Last Reminder")**:
            -   *Inputs*: `div(sub(ticks(utcNow()), ticks(outputs('Last_Reminder_Date'))), 864000000000)`
        
        -   **Condition ("Is Due for Follow-up?")**:
            -   *Left*: `outputs('Days_Since_Last_Reminder')`
            -   *Operator*: `is greater than`
            -   *Right*: `variables('varFollowUpDays')`
        
        -   **If Yes**:
            -   **Update item** (Set Trigger to Reminder):
                -   *fdbx_EmailTrigger*: `Reminder`

---

### Flow 3: "Send Feedback Email" (Common Sender)
**Goal**: Centralized email sender logic to avoid duplication.
**Trigger**: When an item is created or modified (SharePoint)
**Trigger Conditions**: `@not(equals(triggerOutputs()?['body/fdbx_EmailTrigger/Value'], 'None'))`

#### Process Definition:
1.  **Initialize Variable**:
    -   Name: `varEmailSubject`
    -   Type: String
    -   Value: `Feedback Requested` (Default)

2.  **Switch**:
    -   *On*: `triggerOutputs()?['body/fdbx_EmailTrigger/Value']`

    -   **Case "Initial"**:
        -   **Set Variable**:
            -   *Name*: `varEmailSubject`
            -   *Value*: `Action Required: Please provide feedback for [Title]`
    
    -   **Case "Reminder"**:
        -   **Set Variable**:
            -   *Name*: `varEmailSubject`
            -   *Value*: `Reminder: Pending Feedback for [Title]`

3.  **Send an email (V2)** (Outlook):
    -   *To*: `triggerOutputs()?['body/requestorEmail']`
    -   *Subject*: `variables('varEmailSubject')`
    -   *Body*: (Compose generic HTML body with link to item including `@{parameters('ENV_SiteUrl')}`)

4.  **Update item** (Reset Trigger):
    -   *Id*: `triggerOutputs()?['body/ID']`
    -   *fdbx_LastReminderSent*: `utcNow()`
    -   *fdbx_EmailTrigger*: `None`

