**Construction Site Management System**

System Design Document — Version 2

Database Schema • Entity Relationships • Workflows • User Roles • Changelog

# **Table of Contents**

1\. Overview3

2\. Version 2 — What Changed3

3\. User Roles5

4\. Database Tables6

5\. Entity Relationships14

6\. System Workflow15

7\. Role Functions & Responsibilities18

8\. Key Business Rules22

# **1\. Overview**

The Construction Site Management System is a data-entry and reporting platform designed to manage labour, attendance, expenses, warehouse inventory, and purchase activity across one or more active construction sites.

The platform supports user roles — Admin, Supervisor, and three types of Drivers (Ajax, Hitachi, Normal) — each with specific responsibilities. All site financial activity flows through a supervisor's wallet, topped up by admin and debited automatically as expenses are recorded.

| Primary focus areas                                                      |
| :----------------------------------------------------------------------- |
| Labour & Attendance Management (with admin verification for supervisors) |
| Site Expenses & Supervisor Wallet                                        |
| Warehouse Inventory & Material Transfers                                 |
| Ajax Driver — mix-based site expense logging                             |
| Hitachi Driver — hour-based site expense logging                         |
| Normal Driver — material purchases & vehicle cost tracking               |
| Worker Salary Tracking & Admin-Managed Settlements                       |

# **2\. Version 2 — What Changed**

The following tables and features were modified or added in this version of the system. All other tables remain unchanged.

## **2.1 Users — Updated**

A new driver_type column differentiates the three driver subtypes. This is null for admin and supervisor roles and only meaningful when role \= driver.

| Column      | Change | Detail        |
| :---------- | :----- | :------------ | ---- | ------ | -------------------------------- |
| driver_type | Added  | enum: hitachi | ajax | normal | null — null for non-driver roles |

## **2.2 Workers — Updated**

Worker registration was previously limited to supervisors. Admin can now also create, edit, activate, and deactivate workers.

| Column     | Change  | Detail                                                         |
| :--------- | :------ | :------------------------------------------------------------- |
| created_by | Updated | Now accepts admin or supervisor user_id (no structural change) |

## **2.3 Attendance — Updated**

Three new columns support admin verification of supervisor attendance. Workers and drivers are unaffected by this feature. A critical business rule applies: if admin marks a supervisor's attendance as not verified, all expense records by that supervisor for that date are deleted.

| Column      | Change | Detail                                                               |
| :---------- | :----- | :------------------------------------------------------------------- |
| is_verified | Added  | boolean — null for workers/drivers; false by default for supervisors |
| verified_by | Added  | int FK → Users.user_id — admin who performed verification; nullable  |
| verified_at | Added  | datetime — timestamp of verification; nullable                       |

## **2.4 WorkerPayment — Updated**

Weekend salary settlement has been moved from supervisors to admin. The supervisor_id column has been renamed to paid_by to reflect that either a supervisor (advance) or admin (settlement) can record a payment.

| Column        | Change  | Detail                                                                  |
| :------------ | :------ | :---------------------------------------------------------------------- |
| supervisor_id | Removed | Column dropped                                                          |
| paid_by       | Added   | int FK → Users.user_id — supervisor for advances, admin for settlements |

## **2.5 Expense — Updated**

Two new expense types support the Ajax and Hitachi driver workflows. Two new reference types allow expenses to link back to their source log records.

| Column         | Change  | Detail                               |
| :------------- | :------ | :----------------------------------- |
| expense_type   | Updated | Added: ajax_service, hitachi_service |
| reference_type | Updated | Added: ajax_log, hitachi_log         |

## **2.6 AjaxDriverLog — New Table**

Captures the daily mix count for Ajax drivers per site. On save, the system auto-creates an Expense entry of type ajax_service. The rate_per_mix is a snapshot from the app configuration at the time of entry, ensuring historical records are preserved if the rate is later changed.

| Column       | Type    | Description                                            |
| :----------- | :------ | :----------------------------------------------------- |
| log_id       | int PK  | Unique identifier                                      |
| driver_id    | int FK  | References Users.user_id — must be driver_type \= ajax |
| site_id      | int FK  | References Site.site_id — site the mixes were done for |
| date         | date    | Date of work                                           |
| num_mixes    | int     | Number of mixes completed                              |
| rate_per_mix | decimal | Snapshot of rate at time of entry                      |
| total_amount | decimal | num_mixes × rate_per_mix — auto-calculated             |
| expense_id   | int FK  | References Expense.expense_id — auto-created on save   |
| note         | varchar | Optional remarks                                       |

## **2.7 HitachiDriverLog — New Table**

Captures daily hours worked by Hitachi drivers per site. On save, the system auto-creates an Expense entry of type hitachi_service. The hourly_rate is snapshotted at entry time.

| Column       | Type    | Description                                               |
| :----------- | :------ | :-------------------------------------------------------- |
| log_id       | int PK  | Unique identifier                                         |
| driver_id    | int FK  | References Users.user_id — must be driver_type \= hitachi |
| site_id      | int FK  | References Site.site_id — site the hours were billed to   |
| date         | date    | Date of work                                              |
| hours_worked | decimal | Number of hours worked                                    |
| hourly_rate  | decimal | Snapshot of rate at time of entry                         |
| total_amount | decimal | hours_worked × hourly_rate — auto-calculated              |
| expense_id   | int FK  | References Expense.expense_id — auto-created on save      |
| note         | varchar | Optional remarks                                          |

## **2.8 Purchase — Updated**

This table is now exclusively for Normal drivers. The distance_km column has been removed. Bata is now calculated as a fixed 30% of vehicle_rent when the driver uses their own vehicle — no km tracking is needed.

| Column       | Change  | Detail                                                            |
| :----------- | :------ | :---------------------------------------------------------------- |
| distance_km  | Removed | No longer tracked — bata is 30% of vehicle_rent                   |
| vehicle_rent | Updated | Now used for both: outer vehicle rent AND bata base (own vehicle) |
| purchased_by | Updated | Must be driver_type \= normal (Ajax and Hitachi excluded)         |

## **2.9 SystemConfig — Removed**

The SystemConfig table has been removed. Ajax and Hitachi rates are managed as application-level configuration rather than database records. The rate_per_mix and hourly_rate values are still snapshotted into AjaxDriverLog and HitachiDriverLog at entry time, preserving historical accuracy.

# **3\. User Roles**

The system has five distinct operational roles. Admin and Supervisor are unchanged from v1 in terms of structure. Drivers are now split into three subtypes stored via the driver_type column in the Users table.

| ADMIN | Full system access. Manages accounts, sites, warehouse, workers, attendance verification, and weekend salary settlements. |
| :---: | :------------------------------------------------------------------------------------------------------------------------ |

| SUPERVISOR | Manages workers, marks attendance, records site expenses, handles material transfers, and pays daily advances to workers. |
| :--------: | :------------------------------------------------------------------------------------------------------------------------ |

| AJAX DRIVER | Records number of mixes completed per site per day. System auto-calculates expense and links it to the site. |
| :---------: | :----------------------------------------------------------------------------------------------------------- |

| HITACHI DRIVER | Records hours worked per site per day. System auto-calculates expense and links it to the site. |
| :------------: | :---------------------------------------------------------------------------------------------- |

| NORMAL DRIVER | Records material purchases for sites or warehouse. Logs vehicle rent. Bata is auto-calculated at 30% of rent for own-vehicle trips. |
| :-----------: | :---------------------------------------------------------------------------------------------------------------------------------- |

# **4\. Database Tables**

The system consists of 15 tables. Tables marked with a version badge have been added or changed in v2.

## **4.1 Users \[UPDATED\]**

Central authentication and identity table. All user types — admin, supervisor, and all driver subtypes — are stored in a single table. driver_type is only meaningful when role \= driver.

| Column        | Type    | Description                                                |
| :------------ | :------ | :--------------------------------------------------------- | ---------- | ------ | ------------------------------------ |
| user_id       | int PK  | Unique identifier                                          |
| role          | enum    | admin                                                      | supervisor | driver |
| driver_type   | enum    | hitachi                                                    | ajax       | normal | null — null for admin and supervisor |
| full_name     | varchar | Full name of the user                                      |
| phone         | varchar | Login identifier, must be unique                           |
| password_hash | varchar | Hashed password                                            |
| is_active     | boolean | Soft delete — inactive users cannot log in                 |
| acc_balance   | decimal | Running wallet balance — supervisors only; null for others |

## **4.2 Site**

Represents a construction site. A site can have multiple supervisors assigned to it via the SiteSupervisor junction table. created_by links to the admin who created the record.

| Column     | Type     | Description                              |
| :--------- | :------- | :--------------------------------------- | --------- | ------- |
| site_id    | int PK   | Unique identifier                        |
| site_name  | varchar  | Name of the construction site            |
| location   | varchar  | Physical address or area                 |
| status     | enum     | active                                   | completed | on_hold |
| created_at | datetime | When the site was created                |
| created_by | int FK   | References Users.user_id — must be admin |

## **4.3 SiteSupervisor**

Junction table enabling many-to-many assignment between supervisors and sites. A supervisor can be active on multiple sites simultaneously. The is_active flag allows reassignment without deleting history.

| Column        | Type     | Description                                     |
| :------------ | :------- | :---------------------------------------------- |
| id            | int PK   | Unique identifier                               |
| site_id       | int FK   | References Site.site_id                         |
| supervisor_id | int FK   | References Users.user_id — must be a supervisor |
| assigned_at   | datetime | Date of assignment                              |
| is_active     | boolean  | Whether this assignment is currently active     |

## **4.4 Workers \[UPDATED\]**

Labour records managed by admin or supervisors. Workers do not log into the system. They are not tied to a fixed site — their site association is determined through attendance records.

| Column     | Type    | Description                                                              |
| :--------- | :------ | :----------------------------------------------------------------------- |
| worker_id  | int PK  | Unique identifier                                                        |
| full_name  | varchar | Worker's full name                                                       |
| daily_wage | decimal | Standard daily wage rate used for salary calculation                     |
| is_active  | boolean | Soft delete flag                                                         |
| created_by | int FK  | References Users.user_id — admin or supervisor who registered the worker |

## **4.5 Attendance \[UPDATED\]**

Polymorphic attendance table covering Users (supervisors, drivers) and Workers via labour_type \+ labour_id. Unique index is on (labour_type, labour_id, site_id, date), allowing a worker to be marked half-day on two different sites on the same date. site_id is nullable for drivers. Supervisor attendance is subject to admin verification via three added columns.

| Column        | Type     | Description                                                            |
| :------------ | :------- | :--------------------------------------------------------------------- | ---------------------------------------------------- | -------- |
| attendance_id | int PK   | Unique identifier                                                      |
| date          | date     | Date of attendance                                                     |
| labour_type   | enum     | USER                                                                   | WORKER — determines which table labour_id references |
| labour_id     | int      | References Users.user_id or Workers.worker_id depending on labour_type |
| site_id       | int FK   | References Site.site_id — nullable for drivers                         |
| status        | enum     | present                                                                | absent                                               | half_day |
| is_verified   | boolean  | Supervisor verification status — null for workers/drivers              |
| verified_by   | int FK   | References Users.user_id (admin) — nullable                            |
| verified_at   | datetime | Timestamp of admin verification — nullable                             |

| Verification rule                                                                                                                                                              |
| :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| If admin sets is_verified \= false on a supervisor attendance record, the app must delete all Expense records created by that supervisor on that date in the same transaction. |
| Supervisors can still record expenses before their attendance is verified.                                                                                                     |

## **4.6 SupervisorBalanceLog**

Full audit ledger of every credit and debit to a supervisor's wallet. Every update to acc_balance on Users must produce a corresponding entry here.

| Column        | Type     | Description                                    |
| :------------ | :------- | :--------------------------------------------- | ----- |
| log_id        | int PK   | Unique identifier                              |
| supervisor_id | int FK   | References Users.user_id                       |
| txn_type      | enum     | credit                                         | debit |
| amount        | decimal  | Transaction amount                             |
| note          | varchar  | e.g. Admin cash received, Expense: cement bags |
| created_at    | datetime | Timestamp of transaction                       |

## **4.7 WorkerPayment \[UPDATED\]**

Tracks all payments to workers. Advances are entered by supervisors and deducted from the supervisor wallet. Settlements are now handled by admin — the supervisor_id column has been replaced by the more general paid_by column.

| Column       | Type     | Description                                                               |
| :----------- | :------- | :------------------------------------------------------------------------ | ---------- |
| payment_id   | int PK   | Unique identifier                                                         |
| worker_id    | int FK   | References Workers.worker_id                                              |
| paid_by      | int FK   | References Users.user_id — supervisor for advances, admin for settlements |
| amount       | decimal  | Amount paid                                                               |
| payment_type | enum     | advance                                                                   | settlement |
| paid_at      | datetime | Timestamp of payment                                                      |
| note         | varchar  | Optional note                                                             |

| Pending balance formula                                                                                                             |
| :---------------------------------------------------------------------------------------------------------------------------------- |
| Pending \= (days worked × daily_wage) − sum of all advance payments                                                                 |
| Admin views this per worker during settlement. Settlement cash is given by admin outside the app; only the Paid action is recorded. |

## **4.8 Expense \[UPDATED\]**

Central table recording every cost incurred at a site. Auto-generated expenses from Ajax logs, Hitachi logs, material transfers, and normal driver purchases all write here. The expense_type and reference_type columns let admin filter and trace each cost to its origin.

| Column         | Type     | Description                                                                     |
| :------------- | :------- | :------------------------------------------------------------------------------ | ------------- | --------------- | ----------- | ------------------- | ------------ | --------------- | ---- |
| expense_id     | int PK   | Unique identifier                                                               |
| date           | date     | Date the expense occurred                                                       |
| recorded_at    | datetime | Timestamp of entry                                                              |
| amount         | decimal  | Expense amount                                                                  |
| site_id        | int FK   | References Site.site_id                                                         |
| recorded_by    | int FK   | References Users.user_id                                                        |
| expense_type   | enum     | material_transfer                                                               | cash_purchase | driver_material | driver_bata | driver_vehicle_rent | ajax_service | hitachi_service | misc |
| reference_id   | int      | Optional — links to StockMovement, Purchase, AjaxDriverLog, or HitachiDriverLog |
| reference_type | enum     | stock_movement                                                                  | purchase      | ajax_log        | hitachi_log | null                |
| description    | text     | Free-text description                                                           |

## **4.9 AjaxDriverLog \[NEW\]**

Records daily mix count per Ajax driver per site. On save, the system auto-calculates total_amount and creates an Expense of type ajax_service. The rate_per_mix is snapshotted at entry time so historical records are unaffected by future rate changes.

| Column       | Type    | Description                                            |
| :----------- | :------ | :----------------------------------------------------- |
| log_id       | int PK  | Unique identifier                                      |
| driver_id    | int FK  | References Users.user_id — must be driver_type \= ajax |
| site_id      | int FK  | References Site.site_id                                |
| date         | date    | Date of work                                           |
| num_mixes    | int     | Number of mixes completed                              |
| rate_per_mix | decimal | Rate snapshot from app config at time of entry         |
| total_amount | decimal | num_mixes × rate_per_mix — auto-calculated on save     |
| expense_id   | int FK  | References Expense.expense_id — auto-created on save   |
| note         | varchar | Optional remarks                                       |

## **4.10 HitachiDriverLog \[NEW\]**

Records daily hours worked per Hitachi driver per site. On save, total_amount is auto-calculated and an Expense of type hitachi_service is created. The hourly_rate is snapshotted at entry time.

| Column       | Type    | Description                                               |
| :----------- | :------ | :-------------------------------------------------------- |
| log_id       | int PK  | Unique identifier                                         |
| driver_id    | int FK  | References Users.user_id — must be driver_type \= hitachi |
| site_id      | int FK  | References Site.site_id                                   |
| date         | date    | Date of work                                              |
| hours_worked | decimal | Number of hours worked                                    |
| hourly_rate  | decimal | Rate snapshot from app config at time of entry            |
| total_amount | decimal | hours_worked × hourly_rate — auto-calculated on save      |
| expense_id   | int FK  | References Expense.expense_id — auto-created on save      |
| note         | varchar | Optional remarks                                          |

## **4.11 Warehouse**

Represents a physical storage location. Supports multiple warehouses. Items move only from warehouse to site — warehouse-to-warehouse transfers are not supported.

| Column       | Type    | Description       |
| :----------- | :------ | :---------------- |
| warehouse_id | int PK  | Unique identifier |
| name         | varchar | Warehouse name    |
| location     | varchar | Physical location |

## **4.12 WarehouseItem**

Global catalog of all material types. Not tied to a specific warehouse. last_unit_price is updated on every purchase IN and is used to calculate expense when materials are transferred to a site.

| Column          | Type    | Description                                                          |
| :-------------- | :------ | :------------------------------------------------------------------- | ----- | ---- | ----------- | ----- | ----- |
| item_id         | int PK  | Unique identifier                                                    |
| name            | varchar | e.g. Cement, GSP Sand, Steel Rod                                     |
| description     | text    | Optional details such as grade or specification                      |
| category        | enum    | cement                                                               | metal | sand | aggregate   | wood  | other |
| unit            | varchar | kg                                                                   | tonne | bag  | cubic_meter | piece |
| last_unit_price | decimal | Price per unit from most recent purchase — used for transfer costing |

## **4.13 WarehouseStock**

Tracks current available quantity of each item at each warehouse. Updated on every StockMovement IN (increase) or OUT (decrease). One row per item per warehouse.

| Column       | Type     | Description                       |
| :----------- | :------- | :-------------------------------- |
| stock_id     | int PK   | Unique identifier                 |
| warehouse_id | int FK   | References Warehouse.warehouse_id |
| item_id      | int FK   | References WarehouseItem.item_id  |
| quantity     | decimal  | Current available quantity        |
| last_updated | datetime | Timestamp of last stock change    |

## **4.14 StockMovement**

Records every inventory movement. IN movements are purchase deliveries to warehouse. OUT movements are materials issued to a site, which auto-create an Expense of type material_transfer using last_unit_price.

| Column        | Type     | Description                                                     |
| :------------ | :------- | :-------------------------------------------------------------- | --- |
| movement_id   | int PK   | Unique identifier                                               |
| warehouse_id  | int FK   | References Warehouse.warehouse_id                               |
| item_id       | int FK   | References WarehouseItem.item_id                                |
| movement_type | enum     | IN                                                              | OUT |
| quantity      | decimal  | Quantity moved                                                  |
| unit_price    | decimal  | Price per unit at time of movement (snapshot)                   |
| total_amount  | decimal  | quantity × unit_price                                           |
| movement_date | datetime | When the movement occurred                                      |
| site_id       | int FK   | References Site.site_id — nullable, populated for OUT movements |
| reference     | varchar  | PO number, delivery note, or any reference                      |
| created_by    | int FK   | References Users.user_id                                        |

## **4.15 Purchase \[UPDATED\]**

Records all Normal driver purchase transactions. Ajax and Hitachi drivers do not use this table. Bata is now a fixed 30% of vehicle_rent when using own vehicle — the distance_km column has been removed. When destination is a site, three Expense entries are auto-created: driver_material, driver_bata (if own vehicle), and driver_vehicle_rent (if outer vehicle).

| Column           | Type     | Description                                                                |
| :--------------- | :------- | :------------------------------------------------------------------------- | --------- | ---- |
| purchase_id      | int PK   | Unique identifier                                                          |
| purchased_by     | int FK   | References Users.user_id — must be driver_type \= normal                   |
| item_id          | int FK   | References WarehouseItem.item_id                                           |
| site_id          | int FK   | References Site.site_id — nullable when destination_type \= warehouse      |
| warehouse_id     | int FK   | References Warehouse.warehouse_id — nullable when destination_type \= site |
| destination_type | enum     | site                                                                       | warehouse |
| quantity         | decimal  | Quantity purchased                                                         |
| unit_price       | decimal  | Price per unit                                                             |
| total_amount     | decimal  | quantity × unit_price                                                      |
| unit             | varchar  | Unit of measurement                                                        |
| purchased_from   | varchar  | Vendor name or source location                                             |
| vehicle_type     | enum     | own                                                                        | outer     | none |
| vehicle_rent     | decimal  | Rent amount — bata \= vehicle_rent × 0.30 if vehicle_type \= own           |
| purchase_date    | datetime | Date and time of purchase                                                  |
| note             | text     | Optional remarks                                                           |

# **5\. Entity Relationships**

## **5.1 Users**

- One User (supervisor) can be assigned to many Sites through SiteSupervisor

- One User (supervisor) has many SupervisorBalanceLog entries

- One User (supervisor) makes advance WorkerPayments

- One User (admin) makes settlement WorkerPayments

- One User records many Expenses

- One User (ajax driver) has many AjaxDriverLog entries

- One User (hitachi driver) has many HitachiDriverLog entries

- One User (normal driver) makes many Purchases

- One User (admin or supervisor) creates many Workers

- One User (admin) verifies many Attendance records

## **5.2 Site**

- One Site has many Supervisors through SiteSupervisor

- One Site has many Attendance records

- One Site has many Expense records

- One Site receives many StockMovements (OUT)

- One Site is referenced by many AjaxDriverLog entries

- One Site is referenced by many HitachiDriverLog entries

- One Site can be the destination for many Purchases

## **5.3 Workers**

- Workers have no fixed site — association is through Attendance only

- One Worker receives many WorkerPayments (advances and settlements)

- One Worker has many Attendance records across multiple sites on the same or different days

## **5.4 Driver Logs & Expenses**

- Each AjaxDriverLog entry creates exactly one Expense (ajax_service) via expense_id FK

- Each HitachiDriverLog entry creates exactly one Expense (hitachi_service) via expense_id FK

- Each Purchase to site creates 1 to 3 Expense entries (material, bata, vehicle rent)

- Expense.reference_id \+ reference_type traces any expense back to its source record

## **5.5 Warehouse & Inventory**

- One Warehouse has many WarehouseStock entries (one per item it holds)

- One WarehouseItem exists across many Warehouses via WarehouseStock

- StockMovement OUT links to a site and auto-creates a material_transfer Expense

# **6\. System Workflow**

## **6.1 System Setup (Admin)**

|  1  | Create user accounts Admin creates supervisor, ajax driver, hitachi driver, and normal driver accounts with the appropriate role and driver_type. |
| :-: | :------------------------------------------------------------------------------------------------------------------------------------------------ |

|  2  | Create sites Admin creates construction site records with name, location, and status. |
| :-: | :------------------------------------------------------------------------------------ |

|  3  | Assign supervisors to sites Admin assigns one or more supervisors to each site via SiteSupervisor. |
| :-: | :------------------------------------------------------------------------------------------------- |

|  4  | Set up warehouse and item catalog Admin adds warehouse records and creates WarehouseItem entries with category, unit, and initial pricing. |
| :-: | :----------------------------------------------------------------------------------------------------------------------------------------- |

|  5  | Register workers Admin or supervisor registers workers with name and daily wage. |
| :-: | :------------------------------------------------------------------------------- |

## **6.2 Daily Operations (Supervisor)**

|  1  | Top up wallet Supervisor credits their acc_balance. A SupervisorBalanceLog credit entry is created. |
| :-: | :-------------------------------------------------------------------------------------------------- |

|  2  | Mark attendance Supervisor marks attendance for workers and themselves. Workers can be half-day on Site A and half-day on Site B on the same day. |
| :-: | :------------------------------------------------------------------------------------------------------------------------------------------------ |

|  3  | Admin verifies supervisor attendance Admin reviews and marks supervisor attendance as verified. If marked not verified, all expenses by that supervisor for that date are deleted. |
| :-: | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

|  4  | Record expenses Supervisor records each site expense. Each entry debits the supervisor wallet and creates a SupervisorBalanceLog entry. |
| :-: | :-------------------------------------------------------------------------------------------------------------------------------------- |

|  5  | Transfer materials from warehouse Supervisor selects item and quantity. System creates StockMovement OUT, reduces stock, and auto-creates a material_transfer Expense using last_unit_price. |
| :-: | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

|  6  | Pay worker advances Supervisor enters advance amount per worker. WorkerPayment (advance) and SupervisorBalanceLog debit created atomically. |
| :-: | :------------------------------------------------------------------------------------------------------------------------------------------ |

## **6.3 Ajax Driver Daily Flow \[NEW\]**

|  1  | Driver opens site log Ajax driver selects the site they worked on and enters the date. |
| :-: | :------------------------------------------------------------------------------------- |

|  2  | Enter number of mixes Driver inputs the number of mixes completed for the day. |
| :-: | :----------------------------------------------------------------------------- |

|  3  | System calculates and saves App reads rate_per_mix from config, calculates total_amount \= num_mixes × rate, saves AjaxDriverLog, and auto-creates an ajax_service Expense for the site. |
| :-: | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

|  4  | Mark own attendance Driver records attendance for the day (not site-specific). |
| :-: | :----------------------------------------------------------------------------- |

## **6.4 Hitachi Driver Daily Flow \[NEW\]**

|  1  | Driver opens site log Hitachi driver selects the site and enters the date. |
| :-: | :------------------------------------------------------------------------- |

|  2  | Enter hours worked Driver inputs total hours worked for the day. |
| :-: | :--------------------------------------------------------------- |

|  3  | System calculates and saves App reads hourly_rate from config, calculates total_amount \= hours_worked × rate, saves HitachiDriverLog, and auto-creates a hitachi_service Expense for the site. |
| :-: | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

|  4  | Mark own attendance Driver records attendance for the day (not site-specific). |
| :-: | :----------------------------------------------------------------------------- |

## **6.5 Normal Driver Purchase Flow \[UPDATED\]**

|  1  | Driver records purchase Normal driver enters: item, quantity, price, vendor, destination (site or warehouse), and vehicle details. |
| :-: | :--------------------------------------------------------------------------------------------------------------------------------- |

|  2  | Destination \= Site System auto-creates: driver_material Expense (purchase cost), driver_bata Expense (vehicle_rent × 0.30 if own vehicle), driver_vehicle_rent Expense (if outer vehicle). No stock added to warehouse. |
| :-: | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

|  3  | Destination \= Warehouse System creates StockMovement IN, increases WarehouseStock, and updates WarehouseItem.last_unit_price. |
| :-: | :----------------------------------------------------------------------------------------------------------------------------- |

|  4  | Mark own attendance Driver records attendance for the day. |
| :-: | :--------------------------------------------------------- |

## **6.6 Weekend Salary Settlement (Admin) \[UPDATED\]**

|  1  | Admin views pending balances System shows each worker's pending balance: total earned (attendance × daily_wage) minus total advances paid. |
| :-: | :----------------------------------------------------------------------------------------------------------------------------------------- |

|  2  | Admin receives settlement cash from admin funds Cash is given to supervisor outside the app. |
| :-: | :------------------------------------------------------------------------------------------- |

|  3  | Admin marks workers as paid Admin clicks Paid per worker. A WorkerPayment settlement record is created. No wallet deduction occurs for settlements. |
| :-: | :-------------------------------------------------------------------------------------------------------------------------------------------------- |

# **7\. Role Functions & Responsibilities**

## **7.1 Admin**

### **Account Management**

- Create supervisor and driver accounts (with driver_type for drivers)

- Activate or deactivate any user account

- Edit user details

### **Worker Management \[NEW\]**

- Register new workers with name and daily wage

- Edit worker details

- Activate or deactivate workers

### **Site Management**

- Create and edit construction sites

- Assign and remove supervisors from sites

### **Attendance Verification \[NEW\]**

- View all supervisor attendance records

- Mark supervisor attendance as verified or not verified

- On not-verified: all expenses by that supervisor for that date are deleted

- Supervisors may still record expenses before verification

### **Warehouse Management**

- Add and edit warehouse locations

- Maintain the global item catalog

- Record large purchases to warehouse

- View current stock levels per warehouse

### **Weekend Salary Settlement \[UPDATED\]**

- View pending salary balance per worker (total earned minus advances)

- Mark workers as paid after cash is distributed

- Settlement recorded as WorkerPayment with payment_type \= settlement

### **Reporting & Oversight**

- View all expenses per site broken down by expense_type

- View supervisor wallet balance and full transaction history

- View attendance across all workers, supervisors, and drivers

- View Ajax driver logs and associated site expenses

- View Hitachi driver logs and associated site expenses

- View normal driver purchases and bata/vehicle rent costs

- View all stock movement history

## **7.2 Supervisor**

### **Worker Management**

- Register new workers with name and daily wage

- Edit worker details and activate or deactivate workers

### **Attendance**

- Mark attendance for workers — present, absent, or half-day

- Mark a worker half-day on Site A and half-day on Site B on the same day

- Mark own attendance (subject to admin verification)

### **Wallet Management**

- Credit own wallet when cash is received from admin

- View current wallet balance and full credit/debit history

### **Expense Recording**

- Record any site expense against a specific site

- Each expense auto-debits the supervisor wallet

### **Material Transfers**

- Transfer items from warehouse to site

- System auto-calculates expense using last_unit_price and creates material_transfer Expense

### **Worker Advances**

- Pay daily advances to workers — amount entered manually

- Each advance debits the supervisor wallet atomically with a SupervisorBalanceLog entry

## **7.3 Ajax Driver \[NEW\]**

### **Daily Log**

- Select site and date

- Enter number of mixes completed

- System auto-calculates expense at rate_per_mix and creates an ajax_service Expense for the site

### **Attendance**

- Mark own attendance (not site-specific)

## **7.4 Hitachi Driver \[NEW\]**

### **Daily Log**

- Select site and date

- Enter hours worked

- System auto-calculates expense at hourly_rate and creates a hitachi_service Expense for the site

### **Attendance**

- Mark own attendance (not site-specific)

## **7.5 Normal Driver \[UPDATED\]**

### **Purchase Recording**

- Record material purchases: item, quantity, unit price, vendor, date

- Specify destination: site or warehouse

- Specify vehicle type: own, outer, or none

- Enter vehicle_rent — bata is auto-calculated as 30% if vehicle_type \= own

### **Attendance**

- Mark own attendance (not site-specific)

# **8\. Key Business Rules**

## **Supervisor Wallet**

- One shared wallet per supervisor covers all their assigned sites

- Every expense and worker advance debits the wallet

- acc_balance on Users is the running total; SupervisorBalanceLog is the full audit trail

- Both must be written atomically — if either write fails, both roll back

- Weekend salary settlements do NOT deduct from the supervisor wallet

## **Attendance Verification**

- Supervisor attendance defaults to is_verified \= false

- Admin can verify or invalidate any supervisor attendance record

- Invalidating (not verified) triggers deletion of all expenses by that supervisor for that date

- This deletion must be atomic with the verification update

- Supervisors can record expenses regardless of verification status

## **Driver Type Enforcement**

- Only ajax drivers (driver_type \= ajax) can submit AjaxDriverLog entries

- Only hitachi drivers (driver_type \= hitachi) can submit HitachiDriverLog entries

- Only normal drivers (driver_type \= normal) can submit Purchase entries

- The app must enforce this at the UI and API level

## **Ajax & Hitachi Rate Snapshots**

- rate_per_mix and hourly_rate are snapshotted from app config at the time of entry

- Changing the rate in app config does not affect historical log records

- total_amount is calculated and stored at save time — never re-calculated

## **Normal Driver Bata**

- Bata applies only when vehicle_type \= own

- Bata \= vehicle_rent × 0.30 — fixed percentage, no km tracking

- Bata is stored as a separate driver_bata Expense entry, distinct from material cost

- Outer vehicle rent is stored as a separate driver_vehicle_rent Expense entry

## **Material Pricing**

- Warehouse-to-site transfers use quantity × WarehouseItem.last_unit_price

- last_unit_price is updated on every Purchase IN to warehouse

- StockMovement stores a unit_price snapshot for historical accuracy

## **Worker Salary**

- Pending \= (attendance days × daily_wage) − sum of all advances

- Workers have no fixed site — salary is computed across all attendance records

- Admin views pending balance per worker and marks as paid during settlement

- Settlement cash is distributed outside the app; only the Paid action is recorded

## **Transaction Integrity**

- WorkerPayment (advance) \+ SupervisorBalanceLog debit — must be atomic

- Expense creation \+ SupervisorBalanceLog debit — must be atomic

- StockMovement OUT \+ Expense (material_transfer) — must be atomic

- AjaxDriverLog \+ Expense (ajax_service) — must be atomic

- HitachiDriverLog \+ Expense (hitachi_service) — must be atomic

- Attendance verification \= false \+ Expense deletion — must be atomic

_End of Document — Version 2_
