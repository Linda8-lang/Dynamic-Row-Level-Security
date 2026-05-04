# Dynamic Row-Level Security (RLS) with Organizational Hierarchy

This repository demonstrates a professional implementation of **Dynamic Row-Level Security** in Power BI. The project solves a common business challenge: ensuring users only see their own data, while managers can see a consolidated view of their entire reporting line.

## Concept Overview
In many organizations, data privacy is a top priority. Standard (Static) RLS is difficult to maintain as teams grow. This project uses **Dynamic RLS**, which automatically identifies the logged-in user and filters the report based on their position in the company hierarchy.

##  Technical Implementation

The solution relies on three core DAX functions working together:

### 1. The Identity (`USERPRINCIPALNAME`)

`StaffAccess=USERPRINCIPALNAME()`

Captures the email address of the user currently viewing the report. This is the "Key" that unlocks the specific data rows.
You may create a measure  for the Identity as shown below:

<img width="1310" height="290" alt="image" src="https://github.com/user-attachments/assets/c97a7616-c43b-485c-a8f1-1e3a494f3198" />

Visualize the currently logged in user using a card.
<img width="758" height="510" alt="image" src="https://github.com/user-attachments/assets/b7aef247-fb12-451a-a332-3d9e599e4c28" />

### 2. The Map (`PATH`)
A calculated column that flattens the parent-child relationship (Employee -> Manager) into a single string.
dax

`OrgPath = PATH(Employees[EmployeeID], Employees[ManagerID])`

### This creates a breadcrumb trail, such as 101 | 102 | 104, representing the full reporting line.
<img width="593" height="131" alt="image" src="https://github.com/user-attachments/assets/b1f7970b-4b00-4b74-81bf-27ed621e02bd" />

3. The Security Rule (PATHCONTAINS + LOOKUPVALUE)
   
Applied within the Manage Roles feature, this rule acts as a filter on the Employee table. It checks if the logged-in user's ID exists anywhere within the OrgPath of a record.

DAX Filter Expression:

`PATHCONTAINS(
    Employees[OrgPath], 
    LOOKUPVALUE(Employees[EmployeeID], Employees[Email], USERPRINCIPALNAME())
)`

<img width="1029" height="539" alt="image" src="https://github.com/user-attachments/assets/9aecd5e6-7ebe-4490-b565-f62a21121547" />


### Testing & Validation
To ensure zero data leakage, the security model was tested using the "View As" feature in Power BI Desktop.

Sales Rep View: Only shows individual sales performance.

Manager View: Shows the manager's performance plus all subordinates.

Director View: Full access to the entire organizational tree.
<img width="851" height="280" alt="image" src="https://github.com/user-attachments/assets/3c5d0749-dd25-4c09-aa1b-51901df7c1f3" />

Best Practices Followed (Center Of Excellence, COE,  Methodology)

Single Source of Truth: Centralized security logic in the dimension table.

Scalability: The model automatically handles new hires without requiring manual role updates.

Performance: Leverages the Star Schema to allow filters to ripple from the Employee table to the Fact (Sales) table.
