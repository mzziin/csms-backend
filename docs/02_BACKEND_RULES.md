# Backend Development Rules

Version: 1.0

This document defines the mandatory backend development standards for the Construction Site Management System (CSMS).

Every AI Agent and developer must follow these rules when implementing backend features.

The Construction_System_Design_v2.md document defines WHAT the system should do.

This document defines HOW it should be implemented.

---

# 1. Technology Stack

The backend must use the following technologies.

| Component        | Technology     |
| ---------------- | -------------- |
| Language         | Python 3.12+   |
| Framework        | FastAPI        |
| Database         | PostgreSQL     |
| ORM              | SQLAlchemy 2.0 |
| Migration        | Alembic        |
| Validation       | Pydantic v2    |
| Authentication   | JWT            |
| Password Hashing | BCrypt         |
| Testing          | Pytest         |

No alternative frameworks or libraries should be introduced without explicit approval.

---

# 2. Architecture

The backend follows a layered architecture.

```
API

↓

Service

↓

Repository

↓

Database
```

Each layer has exactly one responsibility.

Business logic must never bypass the Service Layer.

---

# 3. Layer Responsibilities

## API Layer

Responsibilities

- Receive HTTP requests
- Validate request data
- Authenticate user
- Authorize user
- Call services
- Return response

API routes must never

- execute SQL
- contain business logic
- perform calculations
- update multiple entities

---

## Service Layer

Responsibilities

- Implement business rules
- Validate business conditions
- Coordinate repositories
- Handle transactions
- Raise business exceptions

Services must never

- contain HTTP logic
- return HTTP responses
- directly access request objects

---

## Repository Layer

Responsibilities

- SQLAlchemy queries
- CRUD operations
- Database filtering
- Pagination
- Joins

Repositories must never

- contain business rules
- perform authorization
- call other repositories

---

## Model Layer

Responsibilities

- Database schema
- Relationships
- Constraints

Models must never contain business logic.

---

## Schema Layer

Responsibilities

- Request DTOs
- Response DTOs
- Validation

Only Pydantic schemas should be exposed outside the Service Layer.

---

# 4. Dependency Injection

Use FastAPI dependency injection for

- Database session
- Current user
- Permissions
- Pagination
- Authentication

Do not instantiate dependencies manually.

---

# 5. Repository Pattern

Every business module should have

```
Repository

↓

Service

↓

Endpoint
```

Example

```
WorkerRepository

↓

WorkerService

↓

workers.py
```

Repositories own all SQLAlchemy code.

---

# 6. Transactions

Transactions must always be managed inside the Service Layer.

Repositories must never commit transactions.

Use

```
session.begin()
```

or equivalent transaction management.

Rollback automatically on failure.

---

# 7. Atomic Operations

The following operations must always execute atomically.

- Expense creation + Supervisor wallet deduction
- Worker advance + Wallet deduction
- Wallet credit + Balance Log
- Attendance rejection + Expense deletion
- Warehouse transfer + Expense creation
- Ajax Driver Log + Expense creation
- Hitachi Driver Log + Expense creation
- Stock Movement + Warehouse Stock update

Business rules are defined in Construction_System_Design_v2.md.

---

# 8. Authentication

Authentication uses JWT.

Protected endpoints must always verify

- access token
- active account
- user role

Never trust client-provided role information.

Always read permissions from authenticated user.

---

# 9. Authorization

Authorization must be role-based.

Roles

- Admin
- Supervisor
- Ajax Driver
- Hitachi Driver
- Normal Driver

Every protected endpoint must verify permissions before executing business logic.

---

# 10. Validation

Validation occurs in three stages.

## Request Validation

Handled by Pydantic.

Examples

- Required fields
- String length
- Email
- Numeric ranges

---

## Business Validation

Handled by Services.

Examples

- Wallet balance
- Active worker
- Driver type
- Attendance state

---

## Database Validation

Handled through

- Constraints
- Foreign Keys
- Unique Indexes

---

# 11. Error Handling

Use custom exceptions for business errors.

Examples

```
WorkerNotFoundException

InsufficientWalletBalanceException

AttendanceAlreadyExistsException

UnauthorizedDriverException
```

Avoid raising HTTPException inside services.

Only API layer converts exceptions into HTTP responses.

---

# 12. Response Format

Every endpoint should return consistent responses.

Example

```
{
    "success": true,
    "message": "Worker created successfully.",
    "data": {...}
}
```

Error responses should follow the same structure.

---

# 13. Pagination

All list endpoints must support

- page
- page_size

Optional

- search
- sorting
- filtering

Never return thousands of records in one request.

---

# 14. Logging

Log

- Authentication failures
- Transaction failures
- Unexpected exceptions
- Critical business events

Never log

- Passwords
- JWT tokens
- Sensitive user data

---

# 15. Soft Delete

Entities that maintain historical data should use soft delete.

Examples

- Users
- Workers

Deleted records should remain queryable for historical reports.

---

# 16. Naming Conventions

Database

snake_case

Tables

Plural

Columns

snake_case

Python

Classes

PascalCase

Variables

camel_case

Constants

UPPER_CASE

---

# 17. SQLAlchemy Rules

Use

- SQLAlchemy 2.0 style
- Typed ORM models
- Relationships
- Lazy loading only where appropriate

Avoid

- raw SQL unless necessary
- duplicated queries

---

# 18. API Design

REST conventions

```
GET

POST

PUT

PATCH

DELETE
```

Plural resources

```
/workers

/sites

/expenses
```

Use nouns.

Avoid verbs.

---

# 19. Code Quality

Every function should

- have one responsibility
- include type hints
- be readable
- avoid duplication

Prefer small services over very large classes.

---

# 20. Testing Requirements

Every completed feature should include

- Unit Tests
- Repository Tests
- API Tests

Critical business workflows should include transaction tests.

---

# 21. Performance

Avoid

- N+1 queries
- duplicate database calls
- unnecessary commits

Use eager loading only when beneficial.

---

# 22. AI Agent Rules

The AI Agent must never

- invent database tables
- invent business rules
- invent user roles
- invent workflows
- change folder structure
- change project dependencies
- bypass repositories
- bypass services

When information is missing,

STOP.

Do not guess.

Request clarification.

---

# 23. Source of Truth

The following documents are authoritative.

| Document                         | Purpose                                              |
| -------------------------------- | ---------------------------------------------------- |
| Construction_System_Design_v2.md | Business rules, workflows, entities, database schema |
| 01_PROJECT.md                    | Project overview and architecture                    |
| project-structure.txt            | Folder structure                                     |
| requirements.txt                 | Dependencies                                         |
| DEVELOPMENT_LOG.md               | Project history                                      |

This document defines implementation standards only.

It must never duplicate or override the functional requirements defined in the System Design document.

---

# 24. AI Output Requirements

Every implementation task must produce:

- Updated source code
- Updated unit tests
- Updated integration tests (if applicable)
- Updated API documentation (if endpoints change)
- Updated DEVELOPMENT_LOG.md
- Migration file (if database changes)

An implementation is not complete until all affected artifacts have been updated.
