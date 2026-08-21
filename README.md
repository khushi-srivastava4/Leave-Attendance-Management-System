# Leave & Attendance Management System

A serverless HR management web application built for **Jagriti Vidya Mandir** to streamline employee leave requests, attendance tracking, approval workflows, and salary calculations.

The system is designed for real organizational use and currently manages **33+ employees** through role-based access and automated HR workflows.

---

## 🚀 Overview

Managing employee attendance, leave applications, approvals, and salary deductions manually can lead to delayed approvals, inconsistent records, and calculation errors.

This project digitizes the complete workflow through a centralized web application.

The system allows employees to submit leave requests and track their attendance and salary details. Office staff can manage attendance and submit leave requests on behalf of employees, while a Super Admin has complete access to employee records, leave approvals, attendance management, and salary calculations.

The application is built using **Google Apps Script** and uses **Google Sheets as a structured relational datastore**, eliminating the need for maintaining a separate backend server or database.

---

## ✨ Features

### 👤 Employee Portal

Employees can:

* Apply for leave for themselves
* View the status of pending, approved, and rejected leave applications
* View personal attendance records
* View medical and emergency leave records
* View monthly salary details
* View leave balance and unused leave information
* Access only their own data

---

### 📝 Office Staff Portal

Office staff can:

* Apply for leave on behalf of any employee
* Mark and update attendance for employees
* Manage monthly attendance records
* View relevant employee information

Office staff **cannot approve or reject leave applications**.

---

### 👑 Super Admin Portal

The Super Admin has complete system access.

Capabilities include:

* View records of all employees
* Submit leave applications for employees
* Approve or reject leave applications
* Mark and update attendance
* View attendance records of all employees
* View leave history and leave balances
* Access salary calculation details
* Monitor payroll deductions and reconciliation
* Access audit and administrative records

The Super Admin is treated as an administrative account and is not included in employee attendance or leave tracking.

---

## 🏗️ System Architecture

```text
                    ┌──────────────────────┐
                    │       Employee       │
                    │    Office Staff      │
                    │     Super Admin      │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │      Frontend        │
                    │ HTML / CSS / JS      │
                    └──────────┬───────────┘
                               │
                       google.script.run
                               │
                               ▼
                    ┌──────────────────────┐
                    │   Google Apps Script │
                    │       Code.gs        │
                    │                      │
                    │  Authentication      │
                    │  Business Logic      │
                    │  Leave Validation    │
                    │  Attendance Logic    │
                    │  Salary Calculation  │
                    └───────┬────────┬─────┘
                            │        │
                            ▼        ▼
                   ┌────────────┐  ┌───────────┐
                   │ Google     │  │ GmailApp  │
                   │ Sheets     │  │           │
                   │ Database   │  │ Notifications
                   └────────────┘  └───────────┘
```

---

# 🛠️ Tech Stack

### Frontend

* HTML
* CSS
* JavaScript

### Backend

* Google Apps Script

### Data Storage

* Google Sheets

### Communication

* `google.script.run`
* Remote Procedure Calls (RPC)

### Notifications

* GmailApp

---

# 🔐 Role-Based Access Control

The application implements **Role-Based Access Control (RBAC)**.

| Role         | Permissions                                                                              |
| ------------ | ---------------------------------------------------------------------------------------- |
| Employee     | Apply for personal leave, view personal attendance, salary, and leave records            |
| Office Staff | Apply leave for employees and manage attendance                                          |
| Super Admin  | Full access including leave approval/rejection, attendance, salary, and employee records |

Sensitive operations are controlled according to the user's assigned role.

---

# 🗄️ Database Design

Google Sheets are organized as separate logical tables.

### Employees

Stores employee information.

```text
EmployeeID
Name
Email
Department
Role
MonthlySalary
PasswordHash
JoiningDate
IsActive
Phone
```

---

### Attendance

Stores employee attendance records.

```text
AttendanceID
EmployeeID
Month
Year
PresentDays
AbsentDays
HalfDays
ApprovedLeaveDays
SpecialLeaveDays
```

---

### LeaveApplications

Stores employee leave requests and approval status.

```text
AppID
EmployeeID
EmployeeName
AppliedBy
LeaveType
FromDate
ToDate
TotalDays
Reason
AppliedOn
Status
ApprovedBy
ApprovedOn
AdminNote
```

---

### LeaveBalance

Tracks employee leave usage and remaining balance.

```text
EmployeeID
Year
TotalLeaves
UsedLeaves
RemainingLeaves
SpecialLeaves
```

---

### SalaryCalculations

Stores monthly payroll calculations.

```text
CalcID
EmployeeID
Month
Year
BaseSalary
PerDaySalary
AbsentDays
HalfDays
SalaryDeduction
LeaveReward
NetSalary
```

---

### AuditLog

Records important administrative actions for traceability.

```text
LogID
Timestamp
UserID
Action
Entity
EntityID
Details
```

---

## 🔗 Relational Data Model

The application follows relational database principles using `EmployeeID` as the primary identifier.

```text
                 Employees
                     │
                     │ EmployeeID
          ┌──────────┼──────────┐
          │          │          │
          ▼          ▼          ▼
     Attendance   Leave      Salary
                  Balance
```

Although Google Sheets does not automatically enforce foreign key constraints like MySQL or PostgreSQL, the backend maintains these relationships through application logic.

---

# 📊 Data Normalization

The system follows normalization principles to minimize redundant data.

Employee information such as:

* Name
* Department
* Salary
* Contact information

is stored centrally in the **Employees** sheet.

Other tables reference employees through `EmployeeID`.

For example:

```text
Employees
---------------------------
EMP001 | Employee Name | HR
```

```text
Attendance
---------------------------
ATT001 | EMP001 | August
```

```text
LeaveApplications
---------------------------
APP001 | EMP001 | Medical Leave
```

This avoids repeatedly storing employee details in multiple records and makes updates easier and more consistent.

---

# 📅 Leave Management Workflow

```text
Employee / Office Staff
          │
          ▼
Submit Leave Application
          │
          ▼
Backend Validation
          │
          ├── Check Employee
          ├── Check Leave Type
          ├── Check Application Date
          ├── Check Leave Balance
          └── Validate Required Documents
          │
          ▼
     LeaveApplication
          │
          ▼
        Pending
          │
          ▼
      Super Admin
       │        │
       ▼        ▼
    Approved   Rejected
       │        │
       ▼        ▼
 Update Leave  Record Status
 Balance
       │
       ▼
Send Notification
```

---

# 📋 Leave Policies

The application supports configurable leave policies.

### Annual Leave Allowance

Employees receive:

```text
14 leave days per year
```

---

### Unused Leave Reward

If an employee does not use all eligible leave days, the remaining leave can be tracked for year-end compensation.

```text
Unused Leaves = Total Eligible Leaves - Used Leaves
```

The system distinguishes between approved leave and unauthorized absence.

An employee cannot claim unused leave benefits for days when they were absent without approved leave.

---

### Unauthorized Absence

If an employee is absent without informing the organization or without an approved leave request, salary is deducted.

```text
Per Day Salary = Monthly Base Salary / 30
```

```text
Deduction = Unauthorized Absent Days × Per Day Salary
```

---

### Half Day

For half-day attendance:

```text
Half-Day Deduction = Per Day Salary / 2
```

---

### Medical Leave

Medical leave requires supporting documentation such as a doctor's prescription or medical slip.

The application validates the required documentation before the request can be processed according to the configured policy.

---

### Special/Emergency Leave

Special or emergency leave caused by exceptional circumstances can be handled separately.

These leaves do not necessarily consume the employee's standard annual leave balance.

---

# 💰 Salary Calculation

The application automates payroll reconciliation using attendance and leave information.

The system considers:

* Base monthly salary
* Unauthorized absences
* Half days
* Approved leaves
* Special leaves
* Remaining leave balance
* Leave-related rewards

### Example

```text
Monthly Salary = ₹30,000

Per Day Salary = 30,000 / 30
               = ₹1,000
```

If the employee has:

```text
2 unauthorized absences
```

Then:

```text
Salary Deduction = 2 × 1,000
                 = ₹2,000
```

The calculated salary is stored as a monthly salary record.

---

# 🔄 Client–Server Communication

The frontend communicates with the backend using:

```javascript
google.script.run
```

This follows a **Remote Procedure Call (RPC)** model.

Example:

```text
Frontend
   │
   │ google.script.run.login()
   ▼
Backend (Code.gs)
   │
   │ Read Employee Data
   ▼
Google Sheets
   │
   ▼
Backend
   │
   │ Return Result
   ▼
Frontend
```

The communication is asynchronous and uses success and failure handlers.

Conceptually:

```javascript
google.script.run
  .withSuccessHandler(handleSuccess)
  .withFailureHandler(handleError)
  .login(employeeId, password);
```

---

# 📧 Automated Notifications

The application uses `GmailApp` to automate workflow notifications.

Notifications can be sent when:

* A leave application is submitted
* A leave request requires administrative review
* A leave request is approved
* A leave request is rejected

This ensures that employees and administrators are informed without manual follow-up.

---

# ⚡ Performance Optimizations

Google Apps Script interactions with Google Sheets can become slow when sheets are accessed repeatedly.

To reduce unnecessary calls, the application uses batch operations.

Instead of:

```text
Read Sheet
Read Sheet
Read Sheet
Read Sheet
```

the application can load required sheet data once:

```text
Read Employees
Read Attendance
Read Leaves
Read Salary
```

and process the data in memory.

This reduces repeated calls to:

```javascript
sheet.getDataRange().getValues();
```

and improves dashboard and data-processing performance.

---

# 🧩 Backend Design

The backend is structured around reusable functions and separation of responsibilities.

The system separates:

```text
Authentication
     ↓
Data Access
     ↓
Business Logic
     ↓
Leave Policy Validation
     ↓
Attendance Processing
     ↓
Salary Calculation
     ↓
Notifications
```

The implementation follows a **repository-style approach** by centralizing common sheet access and data parsing logic into reusable functions rather than duplicating Google Sheets operations throughout the application.

Different leave and policy rules are also organized to allow modular validation and future extension.

---

# 🔑 Authentication

Users authenticate using:

* Employee/User ID
* Password

The backend verifies:

* Employee/User ID
* Password hash
* Account status
* Assigned role

The authenticated user's role determines the level of access available within the application.

---

# 📁 Project Structure

```text
Leave-Attendance-Management-System/
│
├── Code.gs
│   ├── Authentication
│   ├── Employee Management
│   ├── Leave Management
│   ├── Attendance Management
│   ├── Salary Calculation
│   ├── Notifications
│   └── Audit Logging
│
├── Index.html
│   ├── Login
│   ├── Dashboards
│   ├── Leave Management UI
│   ├── Attendance UI
│   └── Salary UI
│
└── Google Sheets
    ├── Employees
    ├── Attendance
    ├── LeaveApplications
    ├── LeaveBalance
    ├── SalaryCalculations
    ├── AuditLog
    └── Config
```

---

# 🚀 Deployment

The application is deployed as a Google Apps Script Web App.

### Steps

1. Create a Google Spreadsheet.
2. Open:

```text
Extensions → Apps Script
```

3. Add the backend code to `Code.gs`.
4. Add the frontend HTML files.
5. Configure the required Google Sheets.
6. Deploy the project:

```text
Deploy → New Deployment → Web App
```

7. Configure execution and access permissions according to organizational requirements.
8. Share the application with authorized users.


---

# 🧠 Key Concepts Demonstrated

This project demonstrates:

* Serverless Architecture
* Backend Development
* Role-Based Access Control (RBAC)
* Client-Server Architecture
* Remote Procedure Calls (RPC)
* Google Apps Script
* Google Sheets Data Modeling
* Relational Database Principles
* Data Normalization
* Logical Primary and Foreign Keys
* Business Rule Validation
* Attendance Management
* Payroll Calculation
* Workflow Automation
* Asynchronous JavaScript
* Gmail Automation
* Audit Logging
* Batch Data Processing
* Modular Backend Design

---

# 📈 Project Impact

The system was developed as a live project for **Jagriti Vidya Mandir** to digitize and centralize HR operations for **33+ employees**.

It replaces fragmented manual workflows with a single system for:

```text
Leave Applications
        ↓
Approval Workflow
        ↓
Attendance Management
        ↓
Leave Balance Tracking
        ↓
Salary Reconciliation
```

---

# 👩‍💻 Author

**Khushi Srivastava**

Human Resource Manager | Gopali Youth Welfare Society | IIT Kharagpur
Built as a live HR workflow automation project for Jagriti Vidya Mandir.

---

⭐ If you found this project interesting, feel free to explore the code and workflow!


---

## 🔑 Default Credentials (from Dummy Data)
Use these logins to test the interface after deployment:
* **Employee**: ID: `EMP001` | Password: `pass123`
* **Office Staff**: ID: `EMP003` | Password: `hr123`
* **Super Admin**: ID: `GB001` | Password: `admin123`
