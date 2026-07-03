# AI-Powered Customer Support Ticketing System (Salesforce DX Metadata)

This project contains the **Salesforce DX (SFDX)** metadata configurations for the **AI-Powered Customer Support Ticketing System**. It incorporates Service Cloud customization, Einstein AI classification, Agentforce-ready flows, SLA tracking, and Agent Feedback collection using standard and custom Salesforce metadata structures.

---

## Project Structure

```text
clever-raman/
├── sfdx-project.json                  # SFDX project configuration
├── README.md                         # Deployment instructions and project summary
├── config/
│   └── project-scratch-def.json      # Scratch Org definition (ServiceCloud, Einstein features enabled)
└── force-app/main/default/
    ├── flexipages/
    │   └── Case_Record_Page.flexipage-meta.xml    # Case Lightning Record Page (custom details, NBA & Screen Flow)
    ├── flows/
    │   ├── Case_Classification_and_Priority.flow-meta.xml  # Automatic case categorization, priority, and SLA setup
    │   ├── Capture_Agent_Feedback.flow-meta.xml            # Screen Flow to capture agent ratings of AI recommendations
    │   └── SLA_Deadline_Escalation.flow-meta.xml           # Scheduled SLA breaches escalations
    ├── objects/
    │   ├── Case/
    │   │   └── fields/
    │   │       ├── AI_Confidence_Score__c.field-meta.xml    # Percent field for classification confidence
    │   │       ├── AI_Predicted_Category__c.field-meta.xml  # Picklist for predicted category (Billing, Tech, Account)
    │   │       ├── Einstein_Sentiment__c.field-meta.xml     # Picklist for case sentiment (Positive, Neutral, Negative)
    │   │       ├── SLA_Deadline__c.field-meta.xml           # SLA target DateTime field
    │   │       └── SLA_Status__c.field-meta.xml             # SLA state indicator (Within SLA, Warning, Breached)
    │   └── Agent_Feedback__c/
    │       ├── Agent_Feedback__c.object-meta.xml            # Custom Object to log AI prediction satisfaction audits
    │       └── fields/
    │           ├── AI_Response_Used__c.field-meta.xml       # Checkbox representing if prediction was selected
    │           ├── Case__c.field-meta.xml                   # Case Lookup link
    │           ├── Comments__c.field-meta.xml               # Long text feedback details
    │           └── Rating__c.field-meta.xml                 # Picklist field (1-5 Star satisfaction)
    ├── permissionsets/
    │   ├── Support_Agent.permissionset-meta.xml             # Read/Write rights on cases, AI fields, and Agent Feedback
    │   └── Support_Manager.permissionset-meta.xml           # Admin and reporting privileges on all features
    ├── reports/
    │   ├── SupportAIReports-meta.xml                        # Folder metadata
    │   └── SupportAIReports/
    │       ├── AI_Classification_Accuracy.report-meta.xml   # AI Accuracy evaluation report
    │       ├── Agent_Feedback.report-meta.xml               # Agent feedback review report
    │       └── SLA_Compliance.report-meta.xml               # SLA SLA Compliance reports
    └── dashboards/
        ├── SupportAIAnalytics-meta.xml                      # Folder metadata
        └── SupportAIAnalytics/
            └── Support_AI_Analytics.dashboard-meta.xml      # Executive KPI & AI Analytics Dashboard layout
```

---

## Deployment & Setup Guide

Ensure you have the [Salesforce CLI](https://developer.salesforce.com/tools/salesforcecli) installed and have configured a Dev Hub or Developer Org.

### Step 1: Log in to your Salesforce Org
Log in to your Salesforce Developer Hub (Dev Hub) or your Developer Org:
```bash
sf org login web -d
```
*(Use `-r https://test.salesforce.com` if using a Sandbox org).*

### Step 2: Deploy Project Metadata
Deploy all custom objects, custom fields, permission sets, flows, reports, and page layouts directly to your active org:
```bash
sf project deploy start
```

### Step 3: Assign Permission Sets
Assign the permission sets to your users so they can access the new fields and objects:
- For support agents:
  ```bash
  sf org assign permset -n Support_Agent
  ```
- For support supervisors/managers:
  ```bash
  sf org assign permset -n Support_Manager
  ```

---

## Core System Capabilities

### 1. Einstein AI Case Classification & SLA Setup
- **Flow**: [Case Classification and Priority Flow](file:///C:/Users/potnu/Documents/antigravity/clever-raman/force-app/main/default/flows/Case_Classification_and_Priority.flow-meta.xml)
- **Execution**: Runs instantly when a new Case is created.
- **Rules**:
  - Automatically scans case `Subject` and `Description` for triggers.
  - Classifies cases into categories: `Billing`, `Technical`, `Account`, or `General`.
  - Recommends an initial case `Priority` (High, Medium, Low) and calculates an `SLA_Deadline__c`:
    - **High Priority**: 2-hour SLA deadline.
    - **Medium Priority**: 12-hour SLA deadline.
    - **Low Priority**: 24-hour SLA deadline.

### 2. SLA Escalation & Tracking
- **Flow**: [SLA Deadline Escalation Flow](file:///C:/Users/potnu/Documents/antigravity/clever-raman/force-app/main/default/flows/SLA_Deadline_Escalation.flow-meta.xml)
- **Execution**: Features a **Scheduled Path** that executes exactly when `SLA_Deadline__c` is reached.
- **Rule**: Checks if the Case is still open (`IsClosed = false`). If so, status changes to **Escalated**, Priority transitions to **High**, and it highlights critical actions.

### 3. Agent Feedback Collector
- **Flow**: [Capture Agent Feedback Flow](file:///C:/Users/potnu/Documents/antigravity/clever-raman/force-app/main/default/flows/Capture_Agent_Feedback.flow-meta.xml)
- **Execution**: Displays on the [Case Record Page](file:///C:/Users/potnu/Documents/antigravity/clever-raman/force-app/main/default/flexipages/Case_Record_Page.flexipage-meta.xml) as an interactive Screen Flow.
- **Goal**: Support agents can rate (1-5 stars) the accuracy of the AI-predicted categories and log whether the suggested responses were utilized. This logs records on the [Agent_Feedback__c](file:///C:/Users/potnu/Documents/antigravity/clever-raman/force-app/main/default/objects/Agent_Feedback__c/Agent_Feedback__c.object-meta.xml) custom audit object.

### 4. Supervisor Dashboards & Insights
- **Dashboard**: [Support Operations & AI Insights](file:///C:/Users/potnu/Documents/antigravity/clever-raman/force-app/main/default/dashboards/SupportAIAnalytics/Support_AI_Analytics.dashboard-meta.xml)
- compiles visual insights on:
  - **SLA Compliance Rates**: Monitor breached vs. met SLAs.
  - **AI Classification Distribution**: Review ticket volumes broken down by AI-predicted categories.
  - **Agent Feedback Analytics**: Track AI recommendation satisfaction scores.
