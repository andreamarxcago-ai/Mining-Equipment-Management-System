# API Specification

**Project:** Mining Equipment Management System (MEMS)  
**Backend:** NestJS / Node.js / TypeScript  
**Database:** PostgreSQL  
**Architecture:** RESTful API  
**Testing and documentation tool:** Postman

## 1. Overview

The proposed MEMS API manages user accounts, equipment categories, mining sites, equipment inventory, assignments, maintenance records, maintenance schedules, inspections, and activity logs. It uses JSON requests and responses and standard HTTP methods (`GET`, `POST`, `PATCH`, and `DELETE`). All routes except login require authentication. The routes below are the **planned midterm design**, not endpoints already implemented or tested.

**Base path:** `/api`  
**Authentication header:** `Authorization: Bearer <access_token>`  
**JSON header for requests with bodies:** `Content-Type: application/json`  
**Path parameter:** `:id` represents the record identifier.

## 2. Roles and Access

- **Public:** No login required; limited to authentication login.
- **Admin:** Manages users, categories, sites, and authorized equipment, assignment, maintenance, inspection, and schedule operations; reads activity logs.
- **Equipment Manager:** Creates and updates equipment and assignments; creates, updates, and conditionally deletes maintenance schedules; reads other operational records.
- **Maintenance Staff:** Reads operational records and creates/updates maintenance and inspection records; cannot manage users, categories, sites, assignments, or schedules.
- **All authenticated roles:** Admin, Equipment Manager, and Maintenance Staff.

Permissions below follow the access column in the midterm paper. A `DELETE` route does not necessarily mean unconditional physical deletion; referential integrity and retention rules still apply.

## 3. Planned API Routes

### 3.1 Authentication

| Method | Endpoint | Description | Access |
|---|---|---|---|
| POST | `/api/auth/login` | Authenticates a user and returns an access token. | Public |

### 3.2 Users

| Method | Endpoint | Description | Access |
|---|---|---|---|
| GET | `/api/users` | Retrieves all user accounts. | Admin |
| GET | `/api/users/:id` | Retrieves a specific user account. | Admin |
| POST | `/api/users` | Creates a new user account. | Admin |
| PATCH | `/api/users/:id` | Updates user information or role. | Admin |
| DELETE | `/api/users/:id` | Deactivates a user account while preserving historical records. | Admin |

### 3.3 Equipment Categories

| Method | Endpoint | Description | Access |
|---|---|---|---|
| GET | `/api/equipment-categories` | Retrieves all equipment categories. | All authenticated roles |
| GET | `/api/equipment-categories/:id` | Retrieves a specific equipment category. | All authenticated roles |
| POST | `/api/equipment-categories` | Creates a new equipment category. | Admin |
| PATCH | `/api/equipment-categories/:id` | Updates equipment category information. | Admin |
| DELETE | `/api/equipment-categories/:id` | Deletes a category only if no existing equipment references it. | Admin |

### 3.4 Mining Sites

| Method | Endpoint | Description | Access |
|---|---|---|---|
| GET | `/api/mining-sites` | Retrieves all mining sites. | All authenticated roles |
| GET | `/api/mining-sites/:id` | Retrieves a specific mining site. | All authenticated roles |
| POST | `/api/mining-sites` | Registers a new mining site. | Admin |
| PATCH | `/api/mining-sites/:id` | Updates mining site information or status. | Admin |
| DELETE | `/api/mining-sites/:id` | Deletes a mining site if no existing records depend on it. | Admin |

### 3.5 Equipment

| Method | Endpoint | Description | Access |
|---|---|---|---|
| GET | `/api/equipment` | Retrieves all equipment records. | All authenticated roles |
| GET | `/api/equipment/:id` | Retrieves a specific equipment record. | All authenticated roles |
| POST | `/api/equipment` | Registers new mining equipment. | Admin, Equipment Manager |
| PATCH | `/api/equipment/:id` | Updates equipment information, condition, or operational status. | Admin, Equipment Manager |
| DELETE | `/api/equipment/:id` | Deletes equipment only when removal is permitted and historical dependencies are preserved. | Admin |

### 3.6 Equipment Assignments

| Method | Endpoint | Description | Access |
|---|---|---|---|
| GET | `/api/equipment-assignments` | Retrieves equipment assignment records. | All authenticated roles |
| GET | `/api/equipment-assignments/:id` | Retrieves a specific equipment assignment. | All authenticated roles |
| POST | `/api/equipment-assignments` | Assigns equipment to a mining site and responsible user. | Admin, Equipment Manager |
| PATCH | `/api/equipment-assignments/:id` | Updates assignment information or records an equipment return. | Admin, Equipment Manager |
| DELETE | `/api/equipment-assignments/:id` | Deletes an assignment only when authorized and consistent with constraints and retention rules. | Admin |

### 3.7 Maintenance Records

| Method | Endpoint | Description | Access |
|---|---|---|---|
| GET | `/api/maintenance-records` | Retrieves equipment maintenance history. | All authenticated roles |
| GET | `/api/maintenance-records/:id` | Retrieves a specific maintenance record. | All authenticated roles |
| POST | `/api/maintenance-records` | Records maintenance performed on equipment. | Admin, Maintenance Staff |
| PATCH | `/api/maintenance-records/:id` | Updates maintenance details, cost, or status when authorized. | Admin, Maintenance Staff |
| DELETE | `/api/maintenance-records/:id` | Deletes an incorrect maintenance record only when permitted by constraints and retention rules. | Admin |

### 3.8 Maintenance Schedules

| Method | Endpoint | Description | Access |
|---|---|---|---|
| GET | `/api/maintenance-schedules` | Retrieves planned maintenance schedules. | All authenticated roles |
| GET | `/api/maintenance-schedules/:id` | Retrieves a specific maintenance schedule. | All authenticated roles |
| POST | `/api/maintenance-schedules` | Creates a maintenance schedule for equipment. | Admin, Equipment Manager |
| PATCH | `/api/maintenance-schedules/:id` | Updates a schedule's date, details, or status. | Admin, Equipment Manager |
| DELETE | `/api/maintenance-schedules/:id` | Deletes a schedule only when no maintenance records reference it and retention rules permit. | Admin, Equipment Manager |

### 3.9 Inspection Records

| Method | Endpoint | Description | Access |
|---|---|---|---|
| GET | `/api/inspection-records` | Retrieves equipment inspection records. | All authenticated roles |
| GET | `/api/inspection-records/:id` | Retrieves a specific inspection record. | All authenticated roles |
| POST | `/api/inspection-records` | Records inspection findings and recommendations. | Admin, Maintenance Staff |
| PATCH | `/api/inspection-records/:id` | Updates inspection information when authorized. | Admin, Maintenance Staff |
| DELETE | `/api/inspection-records/:id` | Deletes an incorrect inspection record only when permitted by constraints and retention rules. | Admin |

### 3.10 Activity Logs

| Method | Endpoint | Description | Access |
|---|---|---|---|
| GET | `/api/activity-logs` | Retrieves activity logs for auditing and monitoring user actions. | Admin |
| GET | `/api/activity-logs/:id` | Retrieves a specific activity log. | Admin |

Activity logs are created internally when important system actions occur. The midterm design does **not** expose `POST`, `PATCH`, or `DELETE` routes for activity logs.

## 4. Authentication and Authorization Strategy

MEMS will use **JSON Web Token (JWT)** authentication and **Role-Based Access Control (RBAC)**.

1. The user submits their email and password to `POST /api/auth/login`.
2. The backend checks the account, verifies the password against its stored hash, and confirms the account is active.
3. If valid, the backend issues a JWT access token.
4. The client includes the token as `Authorization: Bearer <access_token>` on protected requests.
5. NestJS authentication guards verify the token; authorization guards check the user's role before allowing the operation.

Requests without valid authentication receive **401 Unauthorized**. Authenticated users without sufficient permissions receive **403 Forbidden**.

## 5. Sample API Request and Responses

These are **planned examples from the midterm document**, not captured results from a running server.

### 5.1 Successful Equipment Registration

**Method:** `POST`  
**Endpoint:** `/api/equipment`  
**Headers:**

```http
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request body:**

```json
{
  "asset_code": "EQ-001",
  "equipment_name": "Excavator 01",
  "category_id": 1,
  "site_id": 1,
  "manufacturer": "Caterpillar",
  "model": "320D",
  "serial_number": "CAT320D001",
  "purchase_date": "2026-01-15",
  "operational_status": "Available",
  "condition_status": "Good"
}
```

**Expected status:** `201 Created`

```json
{
  "message": "Equipment registered successfully",
  "data": {
    "equipment_id": 1,
    "asset_code": "EQ-001",
    "equipment_name": "Excavator 01",
    "category_id": 1,
    "site_id": 1,
    "manufacturer": "Caterpillar",
    "model": "320D",
    "serial_number": "CAT320D001",
    "purchase_date": "2026-01-15",
    "operational_status": "Available",
    "condition_status": "Good"
  }
}
```

The backend validates the equipment information, saves the new record, and returns its generated equipment ID.

### 5.2 Invalid Equipment Registration

**Method:** `POST`  
**Endpoint:** `/api/equipment`

**Request body:**

```json
{
  "asset_code": " ",
  "equipment_name": "Excavator 01",
  "category_id": 999,
  "site_id": 1
}
```

**Expected status:** `400 Bad Request`

```json
{
  "statusCode": 400,
  "message": [
    "asset_code should not be empty",
    "category_id must reference an existing category"
  ],
  "error": "Bad Request"
}
```

The backend rejects the empty asset code and nonexistent equipment category. Exact validation messages may vary in the final NestJS implementation.

## 6. Validation and Data Integrity Rules

The proposed backend validates required fields, data types, allowed values, and references before database changes. Examples from the database design include:

- Equipment must reference existing `equipment_categories` and `mining_sites` records.
- `asset_code` and known `serial_number` values must be unique.
- Equipment operational statuses: `Available`, `In Use`, `Under Maintenance`, `Out of Service`.
- Equipment and inspection condition statuses: `Good`, `Fair`, `Poor`.
- Maintenance types: `Preventive`, `Corrective`; maintenance record statuses: `In Progress`, `Completed`, `Cancelled`.
- Schedule statuses: `Scheduled`, `Completed`, `Cancelled`.
- Inspection results: `Passed`, `Failed`, `Requires Maintenance`.
- Maintenance cost must not be negative when provided.
- Assignment `returned_at` must be absent while active or be at/after `assigned_at`.
- Related records are checked before deletion to preserve database integrity and required history.
- User accounts are deactivated instead of permanently removed when historical records need to be preserved.

When scheduled maintenance is performed, the maintenance record references `maintenance_schedules.schedule_id` through its optional `schedule_id` field. On completion, the corresponding schedule is marked `Completed`. The system will keep equipment's current assignment state and assignment history consistent, using a transaction when assignment starts or ends.

## 7. Planned Error Handling

| HTTP status | Meaning in the planned API |
|---|---|
| `400 Bad Request` | Invalid request data or validation failure (illustrated in the midterm paper). |
| `401 Unauthorized` | Missing or invalid authentication token. |
| `403 Forbidden` | Authenticated user does not have the required role. |


**Implementation note:** Other error cases (such as missing records, duplicate values, or database conflicts) need their exact status codes and response formats finalized during implementation. The midterm paper does not define those response bodies.

## 8. Planned API Testing

Postman will be used during the final phase to test CRUD operations, JWT authentication, role permissions, required fields, valid and invalid inputs, error responses, and foreign-key integrity. Tests will also verify that maintenance records can link to schedules, activity logs are protected from direct modification, and historical records are not removed improperly.

## 9. Design Status and Related Documentation

**Status:** Midterm design completed; API implementation, Postman collection, and actual endpoint testing are planned for the final phase.