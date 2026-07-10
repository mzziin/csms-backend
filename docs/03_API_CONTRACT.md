# API Contract

Version: 1.0

Status: Living Document

This document defines the REST API contract for the Construction Site Management System (CSMS).

Business requirements are defined in:

Construction_System_Design_v2.md

This document defines how those requirements are exposed through REST APIs.

---

# 1. API Overview

Protocol

HTTPS

Architecture

REST

Data Format

JSON

Authentication

JWT Bearer Token

API Version

v1

Base URL

/api/v1

---

# 2. API Design Principles

The API follows RESTful conventions.

Resources are nouns.

Examples

/users

/workers

/sites

/attendance

/expenses

Avoid verbs.

Incorrect

/createWorker

/getUsers

/updateAttendance

Correct

POST /workers

GET /workers

PATCH /attendance/{id}

---

# 3. Standard Response Format

## Success

```json
{
  "success": true,
  "message": "Operation completed successfully.",
  "data": {}
}
```

---

## Error

```json
{
  "success": false,
  "message": "Validation failed.",
  "errors": {}
}
```

---

# 4. Authentication

Protected endpoints require

Authorization

```
Bearer <JWT_TOKEN>
```

Public endpoints

POST /auth/login

POST /auth/refresh

---

# 5. HTTP Status Codes

| Code | Meaning               |
| ---- | --------------------- |
| 200  | Success               |
| 201  | Created               |
| 204  | Deleted               |
| 400  | Bad Request           |
| 401  | Unauthorized          |
| 403  | Forbidden             |
| 404  | Not Found             |
| 409  | Conflict              |
| 422  | Validation Error      |
| 500  | Internal Server Error |

---

# 6. Pagination

All list endpoints should support

?page=1

&page_size=20

Optional

search

sort

order

filters

Example

GET

```
/workers?page=1&page_size=20
```

---

# 7. API Modules

The backend is divided into the following API modules.

## Authentication

Authentication and authorization.

## Users

User account management.

## Sites

Construction site management.

## Workers

Worker management.

## Attendance

Attendance management.

## Wallet

Supervisor wallet operations.

## Expenses

Site expenses.

## Warehouse

Warehouse management.

## Inventory

Warehouse stock.

## Purchases

Normal driver purchases.

## Driver Logs

Ajax Driver Logs

Hitachi Driver Logs

## Salary

Worker payment and settlements.

## Reports

Reporting.

## Dashboard

Dashboard statistics.

---

# 8. Endpoint Index

Status

🟡 Planned

🟢 Implemented

🔴 Deprecated

---

## Authentication

| Status | Method | Endpoint      |
| ------ | ------ | ------------- |
| 🟡     | POST   | /auth/login   |
| 🟡     | POST   | /auth/refresh |
| 🟡     | POST   | /auth/logout  |
| 🟡     | GET    | /auth/me      |

---

## Users

| Status | Method | Endpoint    |
| ------ | ------ | ----------- |
| 🟡     | POST   | /users      |
| 🟡     | GET    | /users      |
| 🟡     | GET    | /users/{id} |
| 🟡     | PATCH  | /users/{id} |
| 🟡     | DELETE | /users/{id} |

---

## Sites

| Status | Method | Endpoint    |
| ------ | ------ | ----------- |
| 🟡     | POST   | /sites      |
| 🟡     | GET    | /sites      |
| 🟡     | GET    | /sites/{id} |
| 🟡     | PATCH  | /sites/{id} |
| 🟡     | DELETE | /sites/{id} |

---

## Workers

| Status | Method | Endpoint      |
| ------ | ------ | ------------- |
| 🟡     | POST   | /workers      |
| 🟡     | GET    | /workers      |
| 🟡     | GET    | /workers/{id} |
| 🟡     | PATCH  | /workers/{id} |
| 🟡     | DELETE | /workers/{id} |

---

## Attendance

| Status | Method | Endpoint         |
| ------ | ------ | ---------------- |
| 🟡     | POST   | /attendance      |
| 🟡     | GET    | /attendance      |
| 🟡     | PATCH  | /attendance/{id} |

---

## Expenses

| Status | Method | Endpoint       |
| ------ | ------ | -------------- |
| 🟡     | POST   | /expenses      |
| 🟡     | GET    | /expenses      |
| 🟡     | GET    | /expenses/{id} |

---

## Wallet

| Status | Method | Endpoint        |
| ------ | ------ | --------------- |
| 🟡     | POST   | /wallet/credit  |
| 🟡     | GET    | /wallet/balance |
| 🟡     | GET    | /wallet/history |

---

## Warehouse

| Status | Method | Endpoint         |
| ------ | ------ | ---------------- |
| 🟡     | POST   | /warehouses      |
| 🟡     | GET    | /warehouses      |
| 🟡     | PATCH  | /warehouses/{id} |

---

## Warehouse Items

| Status | Method | Endpoint              |
| ------ | ------ | --------------------- |
| 🟡     | POST   | /warehouse-items      |
| 🟡     | GET    | /warehouse-items      |
| 🟡     | PATCH  | /warehouse-items/{id} |

---

## Warehouse Stock

| Status | Method | Endpoint                        |
| ------ | ------ | ------------------------------- |
| 🟡     | GET    | /warehouse-stock                |
| 🟡     | GET    | /warehouse-stock/{warehouse_id} |

---

## Stock Movement

| Status | Method | Endpoint         |
| ------ | ------ | ---------------- |
| 🟡     | POST   | /stock-movements |
| 🟡     | GET    | /stock-movements |

---

## Purchases

| Status | Method | Endpoint        |
| ------ | ------ | --------------- |
| 🟡     | POST   | /purchases      |
| 🟡     | GET    | /purchases      |
| 🟡     | GET    | /purchases/{id} |

---

## Ajax Driver Logs

| Status | Method | Endpoint   |
| ------ | ------ | ---------- |
| 🟡     | POST   | /ajax-logs |
| 🟡     | GET    | /ajax-logs |

---

## Hitachi Driver Logs

| Status | Method | Endpoint      |
| ------ | ------ | ------------- |
| 🟡     | POST   | /hitachi-logs |
| 🟡     | GET    | /hitachi-logs |

---

## Worker Payments

| Status | Method | Endpoint         |
| ------ | ------ | ---------------- |
| 🟡     | POST   | /worker-payments |
| 🟡     | GET    | /worker-payments |

---

## Reports

| Status | Method | Endpoint            |
| ------ | ------ | ------------------- |
| 🟡     | GET    | /reports/attendance |
| 🟡     | GET    | /reports/expenses   |
| 🟡     | GET    | /reports/salary     |
| 🟡     | GET    | /reports/wallet     |
| 🟡     | GET    | /reports/warehouse  |

---

## Dashboard

| Status | Method | Endpoint   |
| ------ | ------ | ---------- |
| 🟡     | GET    | /dashboard |

---

# 9. API Documentation Rules

Every completed endpoint must include

- Request DTO
- Response DTO
- Validation Rules
- Authorization Requirements
- Possible Error Responses
- Example Request
- Example Response

---

# 10. Change Log

Initially all endpoints are marked as Planned.

Whenever an endpoint is implemented

- Update status
- Add request schema
- Add response schema
- Add validation
- Add examples

The API contract evolves alongside the codebase.
