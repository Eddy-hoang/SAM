# 13 - Data Model & Schema Specification

> **Document Status:** `[DECISION]` Relational Database Blueprint  
> **Database Target:** SQLite (Embedded Edge) / PostgreSQL with TimescaleDB extension  

---

## 1. Entity-Relationship Diagram (ERD)

```mermaid
erDiagram
    LOCATION ||--o{ DEVICE : houses
    DEVICE ||--o{ SENSOR : contains
    DEVICE ||--o{ CAMERA : incorporates
    DEVICE ||--o{ EVENT : generates
    DEVICE ||--o{ DEVICE_HEALTH : reports
    EVENT ||--o{ DETECTION : contains
    EVENT ||--o{ ALERT : triggers
    ALERT ||--o{ RISK_ASSESSMENT : evaluates
    ALERT ||--o{ NOTIFICATION : dispatches
    USER ||--o{ AUDIT_LOG : executes

    LOCATION {
        string location_id PK
        string name
        string description
    }

    DEVICE {
        string device_id PK
        string location_id FK
        string mac_address UK
        string device_type
        string firmware_version
        string status
        datetime created_at
    }

    SENSOR {
        string sensor_id PK
        string device_id FK
        string sensor_type
        string pin_assignment
    }

    CAMERA {
        string camera_id PK
        string device_id FK
        string sensor_model
        string resolution_default
    }

    EVENT {
        string event_id PK
        string device_id FK
        string event_type
        string severity
        float confidence
        integer sequence_number
        datetime timestamp_ms
    }

    DETECTION {
        string detection_id PK
        string event_id FK
        string label
        float score
        string bounding_box_json
    }

    ALERT {
        string alert_id PK
        string event_id FK
        string risk_level
        integer risk_score
        string status
        datetime created_at
    }

    RISK_ASSESSMENT {
        string assessment_id PK
        string alert_id FK
        string calculated_rules_json
        float final_score
    }

    NOTIFICATION {
        string notification_id PK
        string alert_id FK
        string channel
        string status
        datetime sent_at
    }

    DEVICE_HEALTH {
        string health_id PK
        string device_id FK
        float battery_v
        integer free_heap_b
        integer wifi_rssi
        datetime recorded_at
    }

    USER {
        string user_id PK
        string username UK
        string password_hash
        string role
    }

    AUDIT_LOG {
        string log_id PK
        string user_id FK
        string action
        string target_entity
        datetime timestamp
    }
```

---

## 2. Table Specifications & Indexes

### 2.1 Table: `events`
* Primary Key: `event_id` (TEXT / UUID)
* Foreign Key: `device_id` $\rightarrow$ `devices(device_id)`
* Fields:
  * `event_id` TEXT NOT NULL PRIMARY KEY
  * `device_id` TEXT NOT NULL
  * `event_type` TEXT NOT NULL
  * `severity` TEXT NOT NULL
  * `confidence` REAL NOT NULL
  * `sequence_number` INTEGER NOT NULL
  * `timestamp_ms` TIMESTAMP NOT NULL
* Indexes:
  * `idx_events_device_time` ON (`device_id`, `timestamp_ms` DESC)
  * `idx_events_type_sev` ON (`event_type`, `severity`)

### 2.2 Table: `alerts`
* Primary Key: `alert_id` (TEXT / UUID)
* Foreign Key: `event_id` $\rightarrow$ `events(event_id)`
* Fields:
  * `alert_id` TEXT NOT NULL PRIMARY KEY
  * `event_id` TEXT NOT NULL
  * `risk_level` TEXT NOT NULL
  * `risk_score` INTEGER NOT NULL
  * `status` TEXT NOT NULL -- ('ACTIVE', 'ACKNOWLEDGED', 'RESOLVED')
  * `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
* Indexes:
  * `idx_alerts_status_created` ON (`status`, `created_at` DESC)
