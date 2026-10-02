# Import Data using Transform Maps (ServiceNow)

**Team ID:** SWTID-2026-4498  
**Platform:** ServiceNow (Personal Developer Instance)  
**Date:** 01 October 2026

Import structured data from Excel / CSV files into ServiceNow tables using **Import Sets** and **Transform Maps**, avoid duplicate records with **Coalesce**, and present the data through **Reports** and a **Dashboard**.

---

## Table of Contents
1. [Project Description](#project-description)
2. [Team Members](#team-members)
3. [Key Features](#key-features)
4. [Architecture](#architecture)
5. [Tables Used](#tables-used)
6. [Prerequisites](#prerequisites)
7. [Setup](#setup)
8. [How It Works (Step by Step)](#how-it-works-step-by-step)
9. [Results](#results)
10. [Testing](#testing)
11. [Known Issues](#known-issues)
12. [Future Enhancements](#future-enhancements)
13. [Demo](#demo)

---

## Project Description
Employee data often arrives as spreadsheets. Entering it manually into ServiceNow is slow and causes errors and duplicate records. This project loads the spreadsheet into a **staging (Import Set) table**, maps the source fields to the target table using a **Transform Map**, and runs the transform to create or update records.

**Coalesce on Employee ID** makes repeated or updated files *update* existing employees instead of creating duplicates. Three reports and an **Employee Analytics Dashboard** give HR teams a quick view of the imported data.

## Team Members
| S.No | Team Member | Role |
|------|-------------|------|
| 1 | Sarah Christa T | Team Leader & Project Coordination Lead |
| 2 | Saravanan K | Quality Assurance & Presentation Support |
| 3 | Sarudharshini B | Project Developer & Technical Documentation Lead |
| 4 | Sathya Priya R | Data Validation & Testing Lead |
| 5 | Selva Karthiga B | Reports, Dashboard & Documentation Support |

## Key Features
- Custom tables: **Employee Test** and **Employee Training Records**
- Spreadsheet load through **Load Data** into a staging table (`u_employee_import`)
- **Transform Map** with field maps (`NMEmp_details Import`, `Employee Transform`)
- Transform execution with **Transform History** and **Import Log**
- **Coalesce** on Employee ID (update existing, insert new, no duplicates)
- Reports: pie chart (by department), bar chart (by location), list report
- **Employee Analytics Dashboard** and the **Hr Manager** role for access

## Architecture
```
Excel / CSV file
      │  Load Data
      ▼
Import Set (staging) table  ──►  Transform Map (field maps + Coalesce)  ──►  Target table
  u_employee_import                 NMEmp_details Import                      u_employee_test
                                                                                  │
                                                         Transform History / Import Log
                                                                                  │
                                                                  Reports  ──►  Dashboard
```

## Tables Used
| Table (Label) | Table Name | Purpose |
|---------------|------------|---------|
| Employee Import | `u_employee_import` | Staging table created by Load Data |
| Employee Test | `u_employee_test` | Target table for employee data; used for reports |
| Employee Training | `u_employee_training` | Staging table for training data |
| Employee Training Records | `u_employee_training_records` | Target table for training records (reference to `sys_user`) |
| User | `sys_user` | Out-of-the-box ServiceNow users table |

## Prerequisites
- ServiceNow Developer account and a **Personal Developer Instance (PDI)** – https://developer.servicenow.com
- Admin access to the instance
- Microsoft Excel or Google Sheets to prepare `.xlsx` files
- Google Chrome (or any current browser)

No software installation or environment variables are needed. Everything runs in the ServiceNow instance.

## Setup
1. Sign up on the ServiceNow Developer site and request a PDI.
2. Log in to your instance as an administrator.
3. Prepare the source files:
   - `NMEmp_details.xlsx` – original employee data (Employee ID, Employee Name, Email, Department, Location)
   - `Employee_Update.xlsx` – changed and new employees
4. Create the custom tables (see below) and follow the steps in the next section.

## How It Works (Step by Step)

### 1. Create the custom tables
`All > System Definition > Tables > New`
- **Employee Test** (`u_employee_test`): Employee ID, Employee Name, Email, Department, Location (String)
- **Employee Training Records** (`u_employee_training_records`): Training Name (String), Completion Date (Date), Employee (Reference – User), Department (String), Status (Choice)
- Configure the form layout with split sections (`Form Context Menu > Configure > Form Layout`).

### 2. Load data into the Import Set table
`All > System Import Sets > Load Data`
- Import set table: **Create table**, Label: `Employee Import`
- Source: **File** → `NMEmp_details.xlsx`, Sheet number `1`, Header row `1` → **Submit**
- Result: State *Complete*, Completion code *Success*, **5 processed, 5 inserts, 0 errors**

### 3. Create the Transform Map
`System Import Sets > Administration > Transform Maps > New`
- Name: `NMEmp_details Import`
- Source table: `Employee Import [u_employee_import]`
- Target table: `Employee Test [u_employee_test]`
- Keep **Active** and **Run business rules** selected, **Enforce mandatory fields:** No, **Order:** 100
- Use **Auto Map Matching Fields** and map the remaining fields (for example `u_name` → `u_employee_name`)

### 4. Run the transform
- On the Transform Map, click **Transform**, select the import set and click **Transform**
- Check **Transform History**: Total 5, Inserts 5, Errors 0

### 5. Enable Coalesce and load updates
- Open the field map for `u_employee_id` and set **Coalesce = true**
- Load `Employee_Update.xlsx` into the **existing** `Employee Import` table
- Run the transform with **only** `NMEmp_details Import` selected (selecting both maps would process every row twice)
- Check Transform History for **Inserts** (new employees) and **Updates** (existing employees)

### 6. Create reports (on Employee Test)
| Report | Type | Group by | Aggregation |
|--------|------|----------|-------------|
| Employees by Department | Pie | Department | Count |
| Employees by Location | Bar | Location | Count |
| Employee List Report | List | – | Columns: Employee ID, Name, Department, Email, Location |

### 7. Create the dashboard
- `All > Dashboards > Create new dashboard` → **In-line editor**
- Name: `Employee Analytics Dashboard`
- **Add new element > Data visualization > Saved Visualization**, add the three reports, arrange them and **Save**

## Results
- First load: 5 rows processed, 5 inserted, 0 errors
- Before Coalesce: repeated loads created duplicate Employee IDs (S104082, S104085, S104086)
- After Coalesce: **Employee Test shows 7 of 7 records**, each Employee ID exactly once
  - S104082 updated from *developer1 / ServiceNow / TamilNadu* to *developer12 / Salesforce / Delhi*
  - New employees S104088 and S104089 inserted
- Pie chart: ServiceNow 3, AgentBlazer 2, Salesforce 2
- Bar chart: TamilNadu 3, Assam 2, Delhi 2

## Testing
| S.No | Test Case | Expected | Result |
|------|-----------|----------|--------|
| 1 | Load spreadsheet into staging table | Complete, Success | 5 processed, 5 inserts, 0 errors – Pass |
| 2 | Run the transform | Transformation complete | Pass |
| 3 | Check Transform History | Rows inserted, no errors | Total 5, Inserts 5, Errors 0 – Pass |
| 4 | Repeated load without Coalesce | No duplicates | Duplicates found – Issue fixed |
| 5 | Update load with Coalesce on Employee ID | Existing updated, new inserted | Pass |
| 6 | Verify Employee Test after update | Each ID once | 7 of 7, no duplicates – Pass |
| 7 | Reports and dashboard | Data displayed | Pass |

## Known Issues
- Two transform maps exist for the same staging table. Run only the Coalesce-enabled map (`NMEmp_details Import`).
- `Share > Add to Dashboard` showed "no writable dashboards" for the in-line dashboard. Add visualizations from inside the dashboard using **Add new element**.
- `Employee Transform` (training data) has Coalesce false on all field maps.
- A PDI may hibernate after inactivity. Wake it from the developer portal.

## Future Enhancements
- Scheduled Data Import for recurring loads
- Other data sources (REST, JDBC)
- onBefore / field-level transform scripts for data cleansing
- Email notification when an import has errors
- Dashboard for import metrics (inserted, updated, ignored, errors)

## Demo
Demo video: (https://drive.google.com/file/d/1NfjB0lsE0yE642P0S1gS06VRfcII_tXi/view?usp=sharing)

## References
- [ServiceNow Developer Program](https://developer.servicenow.com)
- [ServiceNow Documentation](https://www.servicenow.com/docs)
