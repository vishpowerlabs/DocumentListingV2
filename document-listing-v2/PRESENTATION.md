---
marp: true
theme: default
paginate: true
backgroundColor: #fff
style: |
  section {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  }
  h1 {
    color: #0078d4;
  }
  h2 {
    color: #2b88d8;
  }
---

# Document Listing V2
## Architecture & Design Overview

---

# Agenda

1. Project Overview & Use Cases
2. High-Level Architecture
3. Technical Stack
4. UX Layer (SharePoint Framework)
5. Data Layer (SharePoint Lists)
6. Automation Layer (Power Automate)
7. Security Model
8. Code Quality
9. Future Roadmap

---

# 1. Project Overview

**Document Listing V2** is a persistent, theme-aware SPFx web part for displaying curated document catalogs.

### Key Use Cases
- **Curated Experience**: Policies, Procedures, Standard Operating Procedures (SOPs).
- **Access Control**: Users can see documents exist but must **request access** to download sensitive files.
- **Theme Consistency**: Automatically adapts to any SharePoint site branding.

---

# 2. Architecture Diagram

![w:900](docs/images/architecture_overview.png)

---

# 3. Technical Stack

- **Framework**: SharePoint Framework (SPFx) 1.18+
- **Frontend**: React (Functional Components) + Hooks
- **UI System**: Fluent UI (Office UI Fabric)
- **Data Access**: SPHttpClient (REST API)
- **Automation**: Power Automate (Flows)
- **Messaging**: Exchange Online

---

# 4. UX Layer (SPFx)

### Dynamic Catalog
- Fetches real-time data from SharePoint Libraries.
- **Filtering**: Side Nav (Category), Top Tabs (SubCategory), Search Bar.

### Access Request Workflow
- Dedicated "Action" column for restricted files.
- Visual feedback on request status.

### Premium UI/UX
- **Theme Aware**: Dynamic contrast calculation.
- **Responsive**: Mobile-friendly layout.

---

# 5. Data Layer

### A. Source Document Library
- **Role**: Content Storage (PDFs, Docs).
- **Permissions**: **Read-Only** for all users.
- **Key Metadata**: Category, SubCategory, Description, IsPinned.

### B. Request Access List
- **Role**: Transactional Logging.
- **Permissions**: **Contribute** (Create/Edit Own).
- **Key Metadata**: RequestID, Requester Email, FileID, Feedback Status.

---

# 6. Automation Layer

![w:900](docs/images/power_automate_structure.png)

---

# 6. Automation Layer (Details)

1. **Send Download Link**
   - Trigger: Item Created.
   - Action: Generates secure link.
   - Output: Emails user the download button.

2. **Send Feedback Link**
   - Trigger: Scheduled (e.g., +14 days).
   - Logic: If no feedback provided, ask for it.

3. **Send Feedback Reminder**
   - Trigger: Scheduled check.
   - Logic: Nudge user if feedback is overdue.

---

# 7. Security Model

### Web Part Configuration
- **Site Selection**: Cross-site capability (Source Site vs. Request Site).
- **Mapping**: Dynamic field mapping in Property Pane.

### Permissions
| Component | Access Level | Rationale |
| :--- | :--- | :--- |
| **Doc Library** | Read-Only | Protects "Gold Copy" integrity. |
| **Request List** | Contribute* | *Item-level security:* Users only see their own requests. |

---

# 8. Code Quality & Security

**SonarQube Analysis Result: PASSED**

![w:800](docs/images/sonarqube_results.png)

- **Security**: **A** (0 Issues)
- **Reliability**: **A**
- **Maintainability**: **A**

---

# 9. Future Roadmap

- **Feedback Collection**: In-app rating dialog (5-star + comments).
- **Usage Analytics**: Dashboard for "Top Requested Documents".
- **Auto-Approval**: Immediate access for "Public" category documents.

---

# Thank You

**Questions?**
