# 02 TRD — Technical Requirements Document

## 1. App Name

**HelmetGuard AI**

**System Type:** AI-powered preventive two-wheeler safety interlock

**SIH Problem Statement:** **SIH26220 — Student Innovation-Creating intelligent devices to improve commutation sector.**

**Category:** Hardware  
**Theme:** Smart Vehicles  
**Organization:** AICTE  
**Department:** AICTE, MIC-Student Innovation

**Core Principle:**

> **No Helmet → No Start**

HelmetGuard AI must be engineered as an **intelligent hardware safety device**, not merely a computer-vision dashboard.

---

## 2. Frontend

### Technology

- **React.js** for the web dashboard
- **HTML5 / CSS3 / JavaScript**
- **Vite** for development/build tooling
- A lightweight Python/OpenCV interface is acceptable for the SIH prototype if a full React dashboard delays the safety pipeline.

### Frontend Responsibilities

- Display live camera feed
- Show target-rider detection status
- Show helmet verification status
- Show evidence/confidence information
- Show temporal verification progress/state
- Display system decision:
  - 🟢 **SAFE — START ENABLED**
  - 🔴 **UNSAFE — START LOCKED**
  - 🟡 **VERIFYING**
  - ⚠️ **ERROR — SAFE LOCK**
- Display camera/AI/controller health
- Show simulated motor/interlock state
- Show basic event history if logging is enabled

### Safety rule

The UI must never imply that the vehicle is safe to start merely because a helmet object was detected. The final displayed state must come from the safety decision engine.

---

## 3. Backend / AI Verification Engine

### Primary Technology

**Python**

### AI / Computer Vision

- **OpenCV** — camera capture and image processing
- **YOLO-family detector** — rider/helmet detection
- **PyTorch** — deep-learning inference where applicable
- **NumPy** — numerical processing

### Verification Pipeline

```text
Camera Frame
     ↓
Image Quality Gate
     ↓
Rider / Target Selection
     ↓
Helmet Detection
     ↓
Rider Tracking
     ↓
Head-aware Rider–Helmet Association
     ↓
Temporal Evidence / Hysteresis
     ↓
Evidence / Confidence Fusion
     ↓
Safety State Machine
     ↓
Start Authorization or LOCK
```

### Backend Responsibilities

1. Capture frames.
2. Validate camera/frame health.
3. Detect rider(s).
4. Select/track the target rider.
5. Detect helmet evidence.
6. Associate helmet evidence with the target rider's head region.
7. Track evidence over time.
8. Reject insufficient-quality input.
9. Calculate safety evidence/confidence.
10. Apply deterministic safety-state logic.
11. Issue time-limited START/LOCK command to the hardware controller.
12. Optionally record event metadata.

### Fail-safe rule

> **If the system cannot confidently verify helmet usage, it must not authorize starting.**

---

## 4. Safety Decision Logic

### Baseline

```text
Detector confidence
       ↓
Helmet / No Helmet
       ↓
START / LOCK
```

### Required safety pipeline

```text
             ┌──────────────┐
             │ Camera Health│
             └──────┬───────┘
                    ↓
             Image Quality
                    ↓
              Rider Found?
               ↙       ↘
             NO         YES
             ↓           ↓
           LOCK      Helmet Evidence
                           ↓
                   Head-aware Association
                           ↓
                    Temporal Evidence
                           ↓
                    Evidence Threshold
                       ↙         ↘
                    PASS       FAIL/UNKNOWN
                      ↓             ↓
                   SAFE           LOCK
                      ↓
              START AUTHORIZATION
```

### Critical rule

The following must never be interpreted as permission to start:

- No detection
- Low confidence
- Ambiguous rider identity
- Helmet not associated with rider
- Poor image quality
- Camera failure
- AI/model failure
- Controller communication failure

---

## 5. Database

### MVP

The database is **optional** for the first working safety prototype.

The AI decision and hardware interlock must work without an internet connection, cloud database or remote service.

### Recommended MVP Database

**SQLite**

### Future Deployment

Possible options include:

- PostgreSQL
- Supabase
- Firebase

These are for monitoring/administration, not for the core safety authorization path.

### Possible Stored Data

- Verification event metadata
- Timestamp
- Result/state
- Evidence/confidence
- Reason
- Model version
- Verification duration
- Device status
- Hardware communication status

**Raw camera/video should not be stored by default.**

---

## 6. Authentication

### MVP

**No authentication required** for the local SIH hardware demonstration.

### Future

Authentication may be added for:

- Fleet managers
- College administrators
- Rental operators
- System administrators

Possible approaches:

- Supabase Auth
- Firebase Authentication
- JWT-based authentication

Authentication must never become a dependency for local start/lock safety.

---

## 7. Hosting / Deployment

### Prototype

The safety pipeline should run locally:

```text
Laptop / PC
    +
USB Camera
    +
AI Model
    +
Arduino / ESP32
```

### Production Direction

Inference can eventually move to an edge device installed near the vehicle. Candidate platforms include Raspberry Pi-class or NVIDIA Jetson-class hardware, subject to actual benchmark testing.

**Do not claim that a specific edge device is used unless the team has tested it.**

### Cloud

Cloud hosting is optional for dashboards/logging. It is not required for the core safety function.

---

## 8. Third-Party APIs

### MVP

**No mandatory third-party API.**

The system should be capable of running offline.

### Optional Future Services

- Cloud database API
- Fleet management API
- Notification service
- Analytics service

### Explicitly Not Required

- GPS API
- Maps API
- Police/traffic enforcement API
- Identity verification API

---

## 9. Key Libraries

| Purpose | Technology |
|---|---|
| Object Detection | YOLO-family model |
| Deep Learning | PyTorch |
| Image Processing | OpenCV |
| Numerical Processing | NumPy |
| Backend/API | FastAPI |
| Frontend | React.js |
| Frontend Build | Vite |
| Hardware Communication | PySerial |
| Database | SQLite |
| Data Validation | Pydantic |
| Testing | Pytest |

### Suggested Technical Architecture

```text
┌──────────────┐
│    Camera    │
└──────┬───────┘
       ↓
┌──────────────┐
│ OpenCV/Input │
└──────┬───────┘
       ↓
┌──────────────┐
│ Image Quality│
└──────┬───────┘
       ↓
┌──────────────┐
│ YOLO Detector│
└──────┬───────┘
       ↓
┌───────────────────────┐
│ Rider/Helmet Tracking │
│ + Association         │
└──────────┬────────────┘
           ↓
┌───────────────────────┐
│ Temporal Evidence     │
│ + Safety Decision     │
└──────────┬────────────┘
           ↓
      ┌────┴────┐
      ↓         ↓
    SAFE      LOCK
      ↓         ↓
      └────┬────┘
           ↓
     Serial Controller
           ↓
     Arduino / ESP32
           ↓
      Relay / Motor
```

---

## 10. Environment Variables

Example:

```env
APP_ENV=development
MODEL_PATH=models/helmetguard.pt
CONFIDENCE_THRESHOLD=0.70
TEMPORAL_WINDOW=5
SAFE_EVIDENCE_THRESHOLD=0.80

BACKEND_HOST=127.0.0.1
BACKEND_PORT=8000

SERIAL_PORT=COM3
SERIAL_BAUDRATE=9600

DATABASE_URL=sqlite:///helmetguard.db
```

The values are **configuration examples, not validated final thresholds**. Thresholds and temporal windows must be tuned on validation data and tested in the actual environment.

Never hard-code API keys, passwords or credentials.

---

## 11. Constraints

### Hardware Constraints

- Camera must provide sufficient image quality.
- Camera placement must clearly capture the target rider/head region.
- Prototype uses a small DC motor/LED/relay rather than a real motorcycle ignition.
- Controller communication must fail safely.
- Authorization should expire if the AI heartbeat becomes stale.

### AI Constraints

- Detection depends on training data quality.
- Dataset should cover helmet styles, colours, riders, distances and lighting.
- Occlusion and motion blur can reduce reliability.
- Multiple people can create association errors.
- Helmet detection does not prove certification or fastening.

### Performance Constraints

Target characteristics:

- Near-real-time inference
- Stable frame processing
- Low verification delay
- Low false authorization
- Reliable state transitions

### Safety Constraints

> **Uncertainty must result in LOCK, not START.**

### Privacy Constraints

- Process video locally whenever possible.
- No facial recognition.
- No continuous raw-video storage by default.
- Store minimum event metadata only.

---

## 12. MVP Technical Stack

```text
Frontend:
React + Vite (optional for first demo)

Backend / AI:
Python + FastAPI + OpenCV

AI:
YOLO + PyTorch

Database:
SQLite (optional)

Hardware:
Arduino/ESP32 + Relay/Motor/LED

Communication:
USB Serial

Testing:
Pytest

Development:
Git + GitHub
```

---

## 13. Recommended SIH Prototype Architecture

```text
                 SIH DEMO DEVICE

Webcam
   ↓
Python + OpenCV
   ↓
Image Quality Check
   ↓
YOLO Detection
   ↓
Rider Tracking
   ↓
Head-aware Helmet Association
   ↓
Temporal Evidence
   ↓
Safety Decision Engine
   ↓
Serial + Watchdog/Heartbeat
   ↓
Arduino / ESP32
   ↓
Relay / Motor / LED
```

The physical controller demonstrates that the AI decision is not merely informational: it can drive a **vehicle-start safety simulation**.

---

## 14. Technical Priority

### Priority 1 — Must Work

- Rider detection
- Target rider selection/tracking
- Helmet detection
- Rider–helmet association
- Temporal verification
- Safety state machine
- Fail-safe lock
- Hardware start/lock simulation

### Priority 2 — Important

- Image-quality gate
- Watchdog/heartbeat
- Event logging
- Real-time dashboard
- Performance benchmarking
- Model optimization

### Priority 3 — Future

- Improper helmet verification
- Strap verification
- Liveness/anti-spoofing
- Multiple-rider advanced handling
- Edge hardware deployment
- Fleet management
- Additional rider-safety functions

---

## 15. Technical Success Criteria

The prototype is technically successful when:

- Helmeted riders are consistently verified under tested conditions.
- No-helmet cases remain locked.
- Helmet-in-hand cases do not incorrectly authorize start.
- Ambiguous/multiple-rider cases fail safely.
- Low-confidence cases remain locked.
- Camera/AI/controller failures produce safe lock.
- The simulated motor responds correctly to valid authorization.
- The system operates near real time.
- The safety decision remains independent of the database/internet.
- False authorization and false lock rates are measured.

---

## 16. Core Technical Principle

**HelmetGuard AI is not simply a helmet detector.**

It is a **computer-vision-based safety decision system connected to a physical interlock**.

> **Perceive → Associate → Verify Over Time → Decide → Authorize/Lock**

The architecture therefore prioritizes **false-authorization reduction, fail-safe behaviour, low latency, privacy, robustness and physical integration**.
