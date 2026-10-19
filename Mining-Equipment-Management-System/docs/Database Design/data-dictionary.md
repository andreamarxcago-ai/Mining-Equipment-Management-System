# Mining Equipment Management System (MEMS)

## Data Dictionary

**Notation:** PK = primary key, FK = foreign key, UQ = unique. Identifier columns use `INTEGER GENERATED ALWAYS AS IDENTITY`. Timestamp fields that are automatically populated with the current date and time use `DEFAULT CURRENT_TIMESTAMP`.

### 1. users

**Purpose:** Stores the accounts of users who can access the system and perform authorized operations.

| Field Name | Data Type | Key | Description |
|---|---|---|---|
| user_id | INTEGER | PK | Unique identifier for each user |
| full_name | VARCHAR(120) | | Full name of the user |
| email | VARCHAR(150) | UQ | Unique email address used for login |
| password_hash | VARCHAR(250) | | Secure hash of the user’s password |
| role | VARCHAR(30) | | Role of the user: Admin, Maintenance Staff, or Equipment Manager |
| is_active | BOOLEAN | | Whether the account is active (default TRUE) |
| created_at | TIMESTAMP | | Date and time the account was created |
| updated_at | TIMESTAMP | | Date and time the account was last updated |

The role constraint will allow only Admin, Maintenance Staff, and Equipment Manager. The `updated_at` field must be updated when the corresponding record changes; its default alone does not update it automatically.

### 2. equipment_categories

**Purpose:** Classifies mining equipment into groups (e.g., excavators, bulldozers, dump trucks, drilling machines).

| Field Name | Data Type | Key | Description |
|---|---|---|---|
| category_id | INTEGER | PK | Unique identifier for each category |
| category_name | VARCHAR(100) | UQ | Name of the equipment category |
| description | TEXT | | Explanation of the category |
| created_at | TIMESTAMP | | Date and time the category was created |

### 3. mining_sites

**Purpose:** Stores the mining locations where equipment is assigned or operated.

| Field Name | Data Type | Key | Description |
|---|---|---|---|
| site_id | INTEGER | PK | Unique identifier for each mining site |
| site_name | VARCHAR(150) | | Name of the mining site |
| location_description | TEXT | | Geographical or operational location of the site |
| site_status | VARCHAR(20) | | Active or Inactive |
| created_at | TIMESTAMP | | Date and time the site record was created |

The `site_status` field will allow only Active or Inactive.

### 4. equipment

**Purpose:** Stores the main information and current status of every registered equipment item.

| Field Name | Data Type | Key | Description |
|---|---|---|---|
| equipment_id | INTEGER | PK | Unique identifier for each equipment item |
| asset_code | VARCHAR(50) | UQ | Unique internal equipment identification code |
| equipment_name | VARCHAR(150) | | Name or designation of the equipment |
| category_id | INTEGER | FK | References `equipment_categories.category_id` |
| site_id | INTEGER | FK | References `mining_sites.site_id` (current site) |
| manufacturer | VARCHAR(100) | | Name of the manufacturer |
| model | VARCHAR(100) | | Model of the equipment |
| serial_number | VARCHAR(100) | UQ | Manufacturer serial number, when available |
| purchase_date | DATE | | Date the equipment was purchased |
| operational_status | VARCHAR(30) | | Available, In Use, Under Maintenance, or Out of Service |
| condition_status | VARCHAR(30) | | Current physical condition: Good, Fair, or Poor |
| created_at | TIMESTAMP | | Date and time the record was created |
| updated_at | TIMESTAMP | | Date and time the record was last updated |

Allowed operational statuses will include Available, In Use, Under Maintenance, and Out of Service. Condition statuses will include Good, Fair, and Poor.

The serial number may be left empty when unknown. PostgreSQL allows multiple `NULL` values under an ordinary unique constraint, so this design can enforce uniqueness for known serial numbers.

The `site_id` and `operational_status` fields hold only the equipment’s current state; the full history is kept in `equipment_assignments`. The application updates both in one transaction when an assignment starts or ends, so the active assignment (`returned_at IS NULL`) is the authoritative record.

### 5. equipment_assignments

**Purpose:** Records equipment assignments and preserves the history of who was responsible for equipment and where it was assigned.

| Field Name | Data Type | Key | Description |
|---|---|---|---|
| assignment_id | INTEGER | PK | Unique identifier for each assignment |
| equipment_id | INTEGER | FK | References `equipment.equipment_id` |
| site_id | INTEGER | FK | References `mining_sites.site_id` |
| assigned_by | INTEGER | FK | References `users.user_id` for the user who created the assignment |
| assigned_to_user_id | INTEGER | FK | References `users.user_id` for the responsible user |
| assigned_at | TIMESTAMP | | Date and time the assignment began |
| returned_at | TIMESTAMP | | Date and time the assignment ended; NULL while still active |
| notes | TEXT | | Additional assignment details |

The `returned_at` field must be `NULL` while the assignment is active or contain a timestamp that is equal to or later than `assigned_at`.

This design assumes the responsible person has a system account. If the project later needs to manage operators who do not log in, a separate employee table can be added.

### 6. maintenance_records

**Purpose:** Stores actual maintenance work performed on equipment.

| Field Name | Data Type | Key | Description |
|---|---|---|---|
| maintenance_id | INTEGER | PK | Unique identifier for each maintenance record |
| equipment_id | INTEGER | FK | References `equipment.equipment_id` |
| schedule_id | INTEGER | FK | References `maintenance_schedules.schedule_id`; NULL for unscheduled work |
| performed_by | INTEGER | FK | References `users.user_id` for the staff member who performed the work |
| maintenance_type | VARCHAR(30) | | Preventive or Corrective |
| description | TEXT | | Description of the maintenance work |
| maintenance_date | DATE | | Date the maintenance took place |
| cost | DECIMAL(12,2) | | Maintenance cost, if known (must be 0 or higher) |
| status | VARCHAR(30) | | In Progress, Completed, or Cancelled |
| created_at | TIMESTAMP | | Date and time the record was created |

Maintenance types will include Preventive and Corrective. Maintenance status values will include In Progress, Completed, and Cancelled; planned work is tracked in `maintenance_schedules`. The cost must be non-negative when provided.

### 7. maintenance_schedules

**Purpose:** Stores planned maintenance activities to monitor upcoming and overdue services.

| Field Name | Data Type | Key | Description |
|---|---|---|---|
| schedule_id | INTEGER | PK | Unique identifier for each schedule |
| equipment_id | INTEGER | FK | References `equipment.equipment_id` |
| schedule_date | DATE | | Planned maintenance date |
| maintenance_type | VARCHAR(30) | | Type of planned maintenance (Preventive or Corrective) |
| description | TEXT | | Description of the planned service |
| status | VARCHAR(30) | | Scheduled, Completed, or Cancelled |
| created_by | INTEGER | FK | References `users.user_id` for the user who created the schedule |
| created_at | TIMESTAMP | | Date and time the schedule was created |

Schedule status values will include Scheduled, Completed, and Cancelled. Maintenance types are Preventive or Corrective, the same as in `maintenance_records`. When a scheduled service is performed, the system records the actual work in `maintenance_records` with `schedule_id` pointing to the schedule, and updates the schedule status to Completed.

### 8. inspection_records

**Purpose:** Stores inspection results to monitor equipment condition and identify maintenance needs.

| Field Name | Data Type | Key | Description |
|---|---|---|---|
| inspection_id | INTEGER | PK | Unique identifier for each inspection |
| equipment_id | INTEGER | FK | References `equipment.equipment_id` |
| inspected_by | INTEGER | FK | References `users.user_id` for the person who performed the inspection |
| inspection_date | DATE | | Date of the inspection |
| condition_status | VARCHAR(30) | | Condition observed: Good, Fair, or Poor |
| findings | TEXT | | Problems or observations identified |
| recommendations | TEXT | | Recommended corrective actions |
| result_status | VARCHAR(30) | | Overall result: Passed, Failed, or Requires Maintenance |
| created_at | TIMESTAMP | | Date and time the record was created |

Condition values will include Good, Fair, and Poor. Result values will include Passed, Failed, and Requires Maintenance.

Inspection results will help inform maintenance decisions, but the system will not automatically certify that equipment is safe to operate. Such decisions remain subject to appropriate mining safety procedures.

### 9. activity_logs

**Purpose:** Records important system actions to support accountability and auditing.

| Field Name | Data Type | Key | Description |
|---|---|---|---|
| log_id | INTEGER | PK | Unique identifier for each log entry |
| user_id | INTEGER | FK | References `users.user_id` for the user who performed the action |
| action | VARCHAR(100) | | Type of action performed |
| entity_type | VARCHAR(80) | | Type of record affected, such as equipment or maintenance |
| entity_id | INTEGER | | Identifier of the affected record (no FK, polymorphic reference) |
| details | TEXT | | Additional information about the action |
| created_at | TIMESTAMP | | Date and time the action was recorded |

The combination of `entity_type` and `entity_id` identifies the affected record across different entity types. Since this is a polymorphic reference, the database cannot enforce a conventional foreign key from `entity_id` to all possible tables. The application must validate the reference, and audit records should be protected from unauthorized alteration or deletion.
