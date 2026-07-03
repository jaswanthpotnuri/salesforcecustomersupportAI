# Project Documentation: AI-Powered Customer Support Ticketing System

This documentation provides an executive overview, architecture specification, and implementation review of the **AI-Powered Customer Support Ticketing System** built on Salesforce DX.

---

## 1. Executive Summary
The system automates the customer support ticket lifecycle by applying Salesforce Service Cloud customizations, Einstein Case Classification logic, proactive Service Level Agreement (SLA) monitoring, and a feedback loop for continuous AI model training. 

By automating case categorization and prioritization, the system reduces Average Handle Time (AHT), improves customer satisfaction (CSAT), and ensures high SLA compliance.

---

## 2. Architecture & System Topology

```mermaid
graph TD
    A[Incoming Case Created] --> B[Record-Triggered Flow: Classification]
    B --> C{Scan Description & Subject}
    C -- "Billing Keywords" --> D[Category: Billing | Priority: Medium | SLA: 12h]
    C -- "Technical Keywords" --> E[Category: Technical | Priority: High | SLA: 2h]
    C -- "Account Keywords" --> F[Category: Account | Priority: Low | SLA: 24h]
    C -- "No Match" --> G[Category: General | Priority: Low | SLA: 24h]
    
    D & E & F & G --> H[Case Saved with SLA Deadline]
    H --> I[Case Lightning Record Page]
    
    I --> J[Sidebar Screen Flow: Capture Feedback]
    J --> K[Update Case Feedback Fields]
    
    H --> L[Record-Triggered Flow: SLA Tracker]
    L -- "SLA Deadline Reached & Case Still Open" --> M[Status -> Escalated | Priority -> High]
```

---

## 3. Data Model & Field Specification

To stay within the free Developer Org limits and avoid object exhaustion errors (`reached maximum number of custom objects`), the feedback loop was architected directly onto the standard **`Case`** object. This saves custom object slots while streamlining Case database queries.

### Case Object Modifications

| Field API Name | Field Label | Data Type | Description |
| :--- | :--- | :--- | :--- |
| `AI_Predicted_Category__c` | AI Predicted Category | Picklist | Einstein AI predicted category (Billing, Technical, Account, General). |
| `AI_Confidence_Score__c` | AI Confidence Score | Percent | Einstein Case Classification prediction confidence. |
| `Einstein_Sentiment__c` | Einstein Sentiment | Picklist | Detected sentiment of the customer ticket (Positive, Neutral, Negative). |
| `SLA_Deadline__c` | SLA Deadline | Date/Time | Deadline timestamp computed based on Priority SLA parameters. |
| `SLA_Status__c` | SLA Status | Formula (Text) | SLA state calculated dynamically:<br>`Within SLA`, `Warning` (< 2 hours remaining), `Breached`, or `Closed`. |
| `Agent_Feedback_Rating__c` | Agent Feedback Rating | Picklist | Rating (1-5 stars) log of Einstein AI recommendation helpfulness. |
| `Agent_Feedback_Comments__c`| Agent Feedback Comments| Long Text | Details and notes provided by the agent. |
| `AI_Response_Used__c` | AI Response Used | Checkbox | Indicates whether the support agent utilized the recommended response. |

---

## 4. Process Automation (Flows)

The system utilizes three specialized Salesforce Flows:

### A. Case Classification & Priority (Before-Save Trigger)
* **Trigger**: Case Creation
* **Type**: Before-Save Record-Triggered Flow (Optimized for speed)
* **Logic**:
  - Scans `Subject` and `Description` for keyword groups.
  - Automatically updates `AI_Predicted_Category__c`, `AI_Confidence_Score__c`, `Einstein_Sentiment__c`, and `Priority`.
  - Calculates the `SLA_Deadline__c` dynamically:
    - **High Priority**: Current Time + 2 Hours.
    - **Medium Priority**: Current Time + 12 Hours.
    - **Low/General**: Current Time + 24 Hours.

### B. SLA Deadline Escalation (After-Save Trigger & Scheduled Path)
* **Trigger**: Case Creation or Update
* **Type**: After-Save Record-Triggered Flow with a **Scheduled Path**
* **Logic**:
  - Sets up a scheduled task to run exactly at `SLA_Deadline__c`.
  - Evaluates if the case is still open (`IsClosed = false`).
  - If open, updates `Status` to **Escalated** and `Priority` to **High** to flag support queues.

### C. Capture Agent Feedback (Screen Flow)
* **Type**: User Interaction Screen Flow
* **Interface**:
  - A clean form displaying a dropdown rating selection (1 to 5 stars), comments textarea, and a checkbox asking if the suggestion was used.
  - Updates the context Case record with the agent's entries upon submission.

---

## 5. UI Layout & Lightning Console Pages

### Case Record Page (`Case_Record_Page`)
A standard, layout-optimized Case workspace page in Lightning Experience:
* **Header**: High-impact Highlights Panel displaying Case Subject, Status, Priority, and SLA Status.
* **Main Area**: Standard Case details, updates tab, and feeds.
* **Sidebar**: Hosts the interactive **Capture Agent Feedback** Screen Flow directly on the workspace page, prompting agents to review the AI suggestions without changing context.

---

## 6. Access Control & Security Model

Two dedicated Permission Sets govern user access:
1. **`Support_Agent`**: 
   - Read/Write access on standard Case fields.
   - Read/Write access on AI predicted categories, SLA details, and Agent Feedback fields.
   - Designed for front-line support staff to handle tickets and log feedback.
2. **`Support_Manager`**:
   - Administrative and editing rights on Case records.
   - Reporting and dashboard viewing rights for supervisor reviews.

---

## 7. Reports & Dashboard Analytics

### Dashboard: `Support Operations & AI Insights`
A 12-column grid-aligned Executive Dashboard compiling reports on:
* **SLA Compliance Overview**: Visualizes met, warning, and breached SLAs using a **Donut Chart** (`SLA_Compliance` report).
* **AI Classification by Category**: Monitors ticket distributions across predicted support queues using a **Grouped Bar Chart** (`AI_Classification_Accuracy` report).
* **Agent Rating Feedback Distribution**: Tracks user satisfaction with Einstein predictions using a **Donut Chart** (`Agent_Feedback` report).

---

## 8. Deployment & Execution
To redeploy these assets or inspect deployment logs, refer to the project's [README.md](file:///C:/Users/potnu/Documents/antigravity/clever-raman/README.md).
