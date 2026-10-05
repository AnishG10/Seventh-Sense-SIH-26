# 05 Backend Schema — HelmetGuard AI

## 1. Backend Overview

HelmetGuard AI requires only a small amount of persistent data for the MVP.

The **AI verification and physical safety decision must run locally** and must not depend on the database.

The backend/database is primarily used for:

- Safety verification events
- Device/system information
- Configuration metadata
- Optional future user/admin accounts
- Diagnostics

### SIH Role

The backend supports the **SIH26220 Hardware / Smart Vehicles** prototype, but it is not the safety authority. The local safety decision engine is the authority for START/LOCK.

### Architecture

```text
Camera
   ↓
AI Verification Engine
   ↓
Safety Decision Engine
   ├──────────────→ Hardware Controller
   │
   └──────────────→ Backend API (optional logging)
                         ↓
                      Database
                         ↓
                 Dashboard / Logs
```

---

# 2. Tables

## Table 1 — `users`

Stores optional application users for a future administrative dashboard.

| Column | Data Type | Constraints |
|---|---|---|
| `id` | UUID | Primary Key |
| `name` | VARCHAR(100) | Required |
| `email` | VARCHAR(255) | Unique, Required |
| `role` | VARCHAR(30) | Required |
| `created_at` | TIMESTAMP | Required |
| `updated_at` | TIMESTAMP | Required |

### Roles

```text
admin
operator
viewer
```

For the MVP, this table can be omitted.

---

# 3. Table — `devices`

Represents a HelmetGuard AI installation/prototype.

| Column | Data Type | Constraints |
|---|---|---|
| `id` | UUID | Primary Key |
| `device_name` | VARCHAR(100) | Required |
| `device_type` | VARCHAR(50) | Required |
| `status` | VARCHAR(30) | Required |
| `camera_status` | VARCHAR(30) | Required |
| `hardware_status` | VARCHAR(30) | Required |
| `model_version` | VARCHAR(50) | Optional |
| `created_at` | TIMESTAMP | Required |
| `updated_at` | TIMESTAMP | Required |

### Example

```text
device_name: Demo Bike 01
device_type: prototype
status: online
camera_status: connected
hardware_status: connected
model_version: helmetguard-v1
```

No vehicle identity/number plate is required for the MVP.

---

# 4. Table — `verification_events`

This is the **main database table** for the MVP.

It records the outcome of a safety verification without storing continuous camera footage.

| Column | Data Type | Constraints |
|---|---|---|
| `id` | UUID | Primary Key |
| `device_id` | UUID | Foreign Key |
| `result` | VARCHAR(20) | Required |
| `helmet_detected` | BOOLEAN | Required |
| `rider_detected` | BOOLEAN | Required |
| `association_verified` | BOOLEAN | Required |
| `evidence_score` | DECIMAL(5,4) | Required |
| `frames_verified` | INTEGER | Required |
| `verification_ms` | INTEGER | Optional |
| `model_version` | VARCHAR(50) | Optional |
| `reason` | VARCHAR(255) | Optional |
| `created_at` | TIMESTAMP | Required |

### Possible `result` values

```text
PASS
FAIL
UNCERTAIN
ERROR
```

### Example PASS Event

```text
result: PASS
helmet_detected: true
rider_detected: true
association_verified: true
evidence_score: 0.94
frames_verified: 5
verification_ms: 620
model_version: helmetguard-v1
reason: Rider-specific helmet evidence verified
```

### Example FAIL Event

```text
result: FAIL
helmet_detected: false
rider_detected: true
association_verified: false
evidence_score: 0.91
frames_verified: 5
reason: No helmet evidence
```

### Important

The database record is an **audit/logging record**. It must never be queried synchronously to decide whether the motor should start.

---

# 5. Table — `system_logs`

Stores important technical/system events.

| Column | Data Type | Constraints |
|---|---|---|
| `id` | UUID | Primary Key |
| `device_id` | UUID | Foreign Key |
| `level` | VARCHAR(20) | Required |
| `event_type` | VARCHAR(50) | Required |
| `message` | TEXT | Required |
| `created_at` | TIMESTAMP | Required |

### Log Levels

```text
INFO
WARNING
ERROR
```

### Example Events

```text
INFO → Camera connected
INFO → AI model loaded
WARNING → Evidence below threshold
WARNING → Multiple rider ambiguity
ERROR → Hardware connection lost
ERROR → Camera unavailable
```

---

# 6. Table — `device_settings`

Stores configurable AI/system parameters.

| Column | Data Type | Constraints |
|---|---|---|
| `id` | UUID | Primary Key |
| `device_id` | UUID | Foreign Key, Unique |
| `confidence_threshold` | DECIMAL(5,4) | Required |
| `required_frames` | INTEGER | Required |
| `camera_id` | VARCHAR(50) | Required |
| `updated_at` | TIMESTAMP | Required |

Optional future settings:

- Image-quality threshold
- Temporal evidence threshold
- Verification region
- Heartbeat timeout

### Example

```text
confidence_threshold: 0.70
required_frames: 5
camera_id: 0
```

These are configuration examples. Final thresholds must be determined through validation/testing.

---

# 7. Relationships

```text
USERS
  │
  │ optional
  ↓
DEVICES
  │
  ├───────────────┬────────────────┐
  ↓               ↓                ↓
VERIFICATION   SYSTEM          DEVICE_SETTINGS
EVENTS         LOGS
```

### Relationship Details

```text
devices 1 ──── N verification_events

devices 1 ──── N system_logs

devices 1 ──── 1 device_settings
```

A device can generate many verification events and system logs.

---

# 8. Auth Provider

## MVP

**No authentication required.**

The local prototype can operate without user accounts.

## Future

Recommended:

**Supabase Auth**

Alternatives:

- Firebase Authentication
- JWT authentication with FastAPI

Authentication is an administrative feature and must not become a dependency for the local safety interlock.

---

# 9. Row Level Security

If PostgreSQL/Supabase is used in the future, enable Row Level Security (RLS).

### Admin

Can:

- View devices
- View verification events
- Modify configuration
- View system logs

### Operator

Can:

- View assigned devices
- View verification events
- View system status

### Viewer

Can:

- View dashboard
- View non-sensitive logs

### MVP

RLS is unnecessary if the database remains local and inaccessible externally.

---

# 10. User Roles

| Role | Purpose |
|---|---|
| `admin` | Full system management |
| `operator` | Monitor/operate system |
| `viewer` | Read-only monitoring |

### MVP

The prototype effectively operates as:

```text
SYSTEM / DEMO USER
```

No complex permission system is required.

---

# 11. File Storage

### MVP

**No file storage required.**

The system should process camera frames in memory.

```text
Camera
   ↓
Frame
   ↓
AI Processing
   ↓
Safety Result
   ↓
Frame Discarded
```

Do not continuously upload/store camera footage.

Benefits:

- Privacy-conscious
- Lightweight
- Faster
- Less cloud dependency

---

# 12. Optional Evidence Storage

A future version may store a limited verification snapshot only when there is a clear operational purpose.

If implemented:

- Require explicit purpose
- Avoid facial identification
- Restrict access
- Define retention limits
- Never store continuous video

For the SIH MVP, **do not implement evidence-image storage unless required for a clearly justified test/diagnostic use case.**

---

# 13. Sensitive Fields

Potentially sensitive:

- Camera frames
- Verification images
- User email addresses
- Device identifiers
- Logs containing identifiable information

### Data that should NOT be collected for the MVP

- Facial recognition data
- Face embeddings
- Rider identity
- Aadhaar/ID information
- Phone contacts
- GPS location
- Vehicle number plates
- Continuous video recordings

The system should focus on the safety decision.

---

# 14. Data Retention

### Verification Events

For prototype/testing, retain only as long as useful for evaluation; a short configurable retention period is preferred.

### System Logs

Use a short configurable retention period for diagnostics.

### Raw Video

**No retention by default.**

### Future Fleet Deployment

Retention policies should be explicitly defined by the deploying organization.

---

# 15. Example Database Record

### Verification Event

```json
{
  "id": "event-001",
  "device_id": "device-001",
  "result": "PASS",
  "helmet_detected": true,
  "rider_detected": true,
  "association_verified": true,
  "evidence_score": 0.94,
  "frames_verified": 5,
  "verification_ms": 620,
  "model_version": "helmetguard-v1",
  "reason": "Rider-specific helmet evidence verified"
}
```

---

# 16. API Data Flow

```text
AI Engine
    ↓
Safety Decision
    ├──────────────→ Hardware Controller
    │
    └──────────────→ POST /api/verification (optional)
                            ↓
                         Backend
                            ↓
                    verification_events
                            ↓
                         Dashboard
```

### Safety rule

The hardware path must not wait for:

- Database response
- Cloud response
- Dashboard response
- Internet connectivity

The database is for logging/monitoring only.

---

# 17. Minimum MVP Database

If development time is limited, only these tables are required:

```text
verification_events
system_logs (optional)
```

`users` and advanced device management can be added later.

---

# 18. Database Design Principle

> **The database observes the safety system; it does not authorize the safety system.**

The local AI decision engine must remain operational even when:

```text
Database = OFF
Internet = OFF
Backend = OFF
```

---

# 19. Final Backend Principle

HelmetGuard AI follows a **local-first, privacy-conscious, safety-independent backend architecture**.

```text
AI + Safety Engine
        ↓
Physical Interlock     ← critical path

        +

Backend + Database     ← optional monitoring path
```

This separation prevents a web/database failure from becoming an unsafe authorization condition and supports the project's SIH26220 hardware-device positioning.
