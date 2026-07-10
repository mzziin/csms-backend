# Construction Site Management System (CSMS)

## 1. Project Overview

### Project Name

Construction Site Management System (CSMS)

### Version

2.0

### Development Status

Backend Development Phase

---

# 2. Project Objective

The Construction Site Management System (CSMS) is a role-based business application designed to digitize and manage the daily operations of construction projects.

The system replaces paper-based and spreadsheet-based workflows by providing a centralized platform for labour management, attendance tracking, site expenses, warehouse inventory, material purchases, driver activity logging, supervisor wallet accounting, and worker salary settlement.

The primary objective of the project is to ensure accurate financial tracking, operational transparency, and data consistency across multiple construction sites.

---

# 3. Primary Business Domains

The application consists of the following business domains.

- Authentication & Authorization
- User Management
- Site Management
- Worker Management
- Attendance Management
- Supervisor Wallet Management
- Expense Management
- Warehouse Management
- Inventory Management
- Material Purchase Management
- Driver Operations
- Worker Salary Management
- Reporting & Dashboard

---

# 4. Technology Stack

## Frontend

Flutter

## Backend

FastAPI

## Database

PostgreSQL

## ORM

SQLAlchemy 2.0

## Database Migration

Alembic

## Validation

Pydantic v2

## Authentication

JWT Access Token

Refresh Token

BCrypt Password Hashing

## API Style

REST API

---

# 5. System Architecture

The backend follows a layered architecture.

```
Flutter Client
        │
        ▼
 REST API (FastAPI)
        │
        ▼
 API Layer
        │
        ▼
 Service Layer
        │
        ▼
 Repository Layer
        │
        ▼
 PostgreSQL Database
```

Each layer has a single responsibility.

---

# 6. Design Principles

The project follows the following engineering principles.

- Clean Architecture
- Separation of Concerns
- Repository Pattern
- Service Layer Pattern
- Dependency Injection
- RESTful API Design
- Atomic Database Transactions
- Soft Delete where applicable
- Role-Based Access Control (RBAC)
- Audit Logging
- Type Safety
- Explicit Error Handling

---

# 7. User Roles

The application supports five operational roles.

- Administrator
- Supervisor
- Ajax Driver
- Hitachi Driver
- Normal Driver

Each role has a dedicated set of permissions.

Role permissions are defined in the Construction_System_Design_v2.md document.

---

# 8. Project Modules

The project is divided into the following modules.

1. Authentication
2. Users
3. Sites
4. Workers
5. Attendance
6. Supervisor Wallet
7. Expenses
8. Warehouse
9. Inventory
10. Purchases
11. Driver Logs
12. Salary Management
13. Reports
14. Dashboard

Each module should remain independent and communicate only through services.

---

# 9. Database

Database Engine

PostgreSQL

Migration Tool

Alembic

ORM

SQLAlchemy 2.0

Database design is completely defined in:

Construction_System_Design_v2.md

The AI agent must never modify the database schema unless explicitly instructed.

---

# 10. Business Rules

All business rules are defined in

Construction_System_Design_v2.md

Examples include

- Supervisor Wallet
- Attendance Verification
- Driver Type Validation
- Warehouse Pricing
- Worker Salary Calculation
- Transaction Integrity

AI agents must not invent new business rules.

---

# 11. Folder Structure

The canonical backend folder structure is defined in

project-structure.txt

AI agents must follow the structure exactly.

No new folders should be introduced without explicit approval.

---

# 12. Dependency Management

All project dependencies are defined in

requirements.txt

AI agents must not replace frameworks or libraries unless explicitly instructed.

---

# 13. Backend Development Rules

The backend follows the following implementation rules.

## API Layer

Responsibilities

- Receive HTTP requests
- Validate input
- Call services
- Return API responses

API routes must not contain business logic.

API routes must not execute SQL queries.

---

## Service Layer

Responsibilities

- Implement business rules
- Coordinate repositories
- Handle transactions
- Perform validations
- Raise business exceptions

Services must never contain HTTP-specific logic.

---

## Repository Layer

Responsibilities

- Execute SQLAlchemy queries
- Perform CRUD operations
- Return ORM models

Repositories must never contain business rules.

---

## Schema Layer

Responsibilities

- Input validation
- Response serialization
- DTO definitions

Only Pydantic schemas should be exposed to the API layer.

---

## Model Layer

Responsibilities

- SQLAlchemy ORM definitions

Models represent database tables only.

---

# 14. Database Transactions

The following operations must always execute inside a single database transaction.

- Expense creation + Wallet deduction
- Worker advance + Wallet deduction
- Warehouse transfer + Expense creation
- Ajax log + Expense creation
- Hitachi log + Expense creation
- Attendance rejection + Expense deletion

Transaction requirements are defined in

Construction_System_Design_v2.md

---

# 15. Coding Standards

The project follows these conventions.

Naming

- snake_case for variables
- PascalCase for classes
- UPPER_CASE for constants

Every function must have type hints.

Every public method should have a docstring.

Keep functions focused on a single responsibility.

Avoid duplicate code.

Write readable code instead of clever code.

---

# 16. Error Handling

Business errors should use custom exceptions.

Unexpected exceptions should be logged.

Internal implementation details must never be exposed through API responses.

---

# 17. Testing Philosophy

Every completed feature should include

- Unit Tests
- Integration Tests
- API Tests

Business-critical workflows must include transaction tests.

---

# 18. AI Agent Instructions

Before implementing any feature, the AI agent must read the following files in order.

1. 01_PROJECT.md
2. Construction_System_Design_v2.md
3. project-structure.txt
4. requirements.txt
5. DEVELOPMENT_LOG.md

The AI agent must:

- Follow the existing architecture.
- Never invent database tables.
- Never invent API endpoints.
- Never change business rules.
- Never change folder structure.
- Never replace project dependencies.
- Keep implementations modular.
- Minimize code duplication.
- Preserve backward compatibility.

If information is missing, the AI agent should stop and request clarification instead of making assumptions.

---

# 19. Current Project Status

| Component             | Status       |
| --------------------- | ------------ |
| Requirements Analysis | ✅ Completed |
| System Design         | ✅ Completed |
| Database Design       | ✅ Completed |
| Backend Architecture  | In Progress  |
| Backend Development   | Not Started  |
| Flutter Development   | Not Started  |
| Testing               | Not Started  |
| Deployment            | Not Started  |

---

# 20. Reference Documents

The following documents are the official source of truth.

| Document                         | Purpose                                                                |
| -------------------------------- | ---------------------------------------------------------------------- |
| Construction_System_Design_v2.md | Functional requirements, workflows, database schema and business rules |
| project-structure.txt            | Backend folder structure                                               |
| requirements.txt                 | Backend dependencies                                                   |
| DEVELOPMENT_LOG.md               | Project implementation history                                         |

No implementation should contradict these documents.

---

# Project Non-Goals

The following features are intentionally out of scope unless explicitly requested.

- Microservices
- GraphQL
- WebSockets
- Event-driven architecture
- Redis caching
- Celery background workers
- Multi-tenancy
- Offline synchronization
- Multi-language support
- Plugin architecture
