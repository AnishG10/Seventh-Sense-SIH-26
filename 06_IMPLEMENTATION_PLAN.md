# 06 Implementation Plan — HelmetGuard AI

## 1. Project Name

**HelmetGuard AI**

**SIH 2026 Problem Statement:** **SIH26220 — Student Innovation-Creating intelligent devices to improve commutation sector.**

**Category:** Hardware  
**Theme:** Smart Vehicles  
**Organization:** AICTE

**Tagline:**

> **No Helmet. No Start.**

---

# 2. Implementation Strategy

Build the project incrementally. The priority is an end-to-end **AI → safety decision → physical interlock** prototype before adding dashboards or advanced features.

### Development Order

```text
Project Setup
      ↓
Dataset + Baseline Detector
      ↓
Rider / Helmet Verification
      ↓
Association + Tracking
      ↓
Temporal Evidence + Safety State Machine
      ↓
Hardware Interlock
      ↓
Failure Handling / Watchdog
      ↓
Backend & Logging
      ↓
Dashboard UI
      ↓
Testing & Benchmarking
      ↓
SIH Demo / Deployment Prototype
```

### Core MVP

> **Camera → Detect Rider → Verify Helmet → Decide SAFE/LOCK → Control Motor Simulation**

Everything else is secondary.

---

# PHASE 1 — Project Setup

## Objective

Create a reproducible development environment and establish the basic hardware/software connection.

### Tasks

- Create GitHub repository
- Set up Python environment
- Install required libraries
- Set up React + Vite if dashboard is included
- Set up Arduino/ESP32 environment
- Connect webcam
- Test camera input
- Establish Git workflow
- Create `.gitignore`
- Create `.env.example`
- Record actual hardware/software versions used for the SIH prototype

### Suggested Structure

```text
helmetguard-ai/
│
├── ai/
│   ├── models/
│   ├── datasets/
│   ├── inference/
│   ├── tracking/
│   └── training/
│
├── backend/
│   ├── api/
│   ├── services/
│   ├── database/
│   └── main.py
│
├── frontend/
│   └── ...
│
├── hardware/
│   └── ...
│
├── tests/
├── docs/
├── .env.example
├── README.md
└── requirements.txt
```

### Done Criteria

- [ ] Repository created
- [ ] Python environment works
- [ ] Dependencies installed
- [ ] Camera feed captured
- [ ] Microcontroller programmed
- [ ] START/LOCK output hardware identified
- [ ] Team can clone and run the project

---

# PHASE 2 — AI Model & Dataset

## Objective

Build a detector that can provide useful rider and helmet evidence under realistic demo conditions.

### Dataset Requirements

Include variation in:

- Helmet types
- Helmet colours
- Rider clothing
- Rider distances
- Camera angles
- Lighting conditions
- Backgrounds
- Occlusion
- Motion blur
- Multiple people
- Helmet held in hand

### Dataset Split

```text
Training
Validation
Testing
```

Do not use the same scenes/frames across train and test in a way that inflates performance.

### Initial Classes

```text
rider
helmet
no_helmet
```

Class design can be changed after dataset analysis.

### Baseline Model

First implement a simple baseline:

```text
YOLO Detection
     ↓
Helmet Confidence
     ↓
PASS / FAIL
```

This baseline is important because later improvements can be measured against it.

### Done Criteria

- [ ] Dataset prepared
- [ ] Annotations validated
- [ ] Train/validation/test split created
- [ ] Baseline model trained/fine-tuned
- [ ] Precision/recall/F1/mAP measured
- [ ] Live camera inference works
- [ ] Failure examples documented

---

# PHASE 3 — Helmet Verification Engine

## Objective

Convert object detection into **rider-specific helmet verification**.

### 3.1 Rider Detection

```text
Camera
  ↓
Detector
  ↓
Rider Bounding Box
```

### 3.2 Target Rider Selection

If only one rider is present, use the valid rider candidate.

If multiple people are present:

```text
Multiple candidates
       ↓
Can target rider be identified?
    ↙              ↘
  YES               NO
   ↓                 ↓
Track target      LOCK / UNCERTAIN
```

Do not allow a helmet worn by another person to authorize the target vehicle.

### 3.3 Helmet Detection

Detect helmet candidates and obtain confidence/evidence.

### 3.4 Head-Aware Rider–Helmet Association

Do not use only whole-box overlap.

Use available geometric cues such as:

- Helmet centre relative to rider/head region
- Distance from head/upper-body ROI
- Relative bounding-box geometry
- Temporal consistency

Conceptually:

```text
Rider Box
   ↓
Head / Upper-body ROI
   ↓
Helmet Candidate
   ↓
Association Score
```

### 3.5 Tracking

Track the target rider across frames so that evidence is attached to the same rider.

### 3.6 Temporal Evidence

Replace a rigid “5 consecutive positive frames” assumption with a configurable temporal evidence mechanism.

Example:

```text
PASS  PASS  UNKNOWN  PASS  PASS
  ↓      ↓      ↓       ↓     ↓
      Temporal Evidence Score
                ↓
        Threshold / Hysteresis
                ↓
             SAFE/LOCK
```

### 3.7 Image Quality Gate

Check for:

- Excessive blur
- Extremely dark/bright image
- Invalid frame
- Severe occlusion
- Rider outside verification region

### Done Criteria

- [ ] Rider detected
- [ ] Target rider selected
- [ ] Helmet detected
- [ ] Rider–helmet association implemented
- [ ] Rider tracking implemented
- [ ] Temporal verification implemented
- [ ] Image-quality gate implemented or justified as unnecessary
- [ ] Helmet-in-hand scenario rejected
- [ ] Multiple-person ambiguity fails safely

---

# PHASE 4 — Safety Decision Engine

## Objective

Convert AI evidence into a deterministic, fail-safe safety state.

### Decision States

```text
INITIALIZING
READY
DETECTING
VERIFYING
SAFE
UNSAFE
ERROR
SAFE_LOCK
```

### Core Logic

```text
IF system health is invalid
    → SAFE_LOCK

IF target rider cannot be identified
    → VERIFYING / LOCK

IF image quality is insufficient
    → VERIFYING / LOCK

IF helmet evidence is absent
    → UNSAFE / LOCK

IF rider–helmet association fails
    → UNSAFE / LOCK

IF temporal evidence is insufficient
    → VERIFYING / LOCK

IF sufficient safety evidence exists
    → SAFE
    → issue temporary START authorization
```

### Safety State Machine

```text
                ┌──────────────┐
                │ INITIALIZING │
                └──────┬───────┘
                       ↓
                     READY
                       ↓
                   DETECTING
                       ↓
                   VERIFYING
                    ↙       ↘
                 SAFE       UNSAFE
                  ↓            ↓
            START ENABLED    LOCK
                  │            │
                  └─────┬──────┘
                        ↓
                 CONTINUOUS CHECK
                        ↓
             Any critical fault?
                   ↙          ↘
                 YES           NO
                  ↓             ↓
             SAFE_LOCK       Continue
```

### Temporary Authorization

A successful verification should produce a time-limited authorization/heartbeat rather than an indefinite `START=true` flag.

```text
SAFE
 ↓
START authorization issued
 ↓
Controller heartbeat valid?
 ↙                  ↘
NO                   YES
↓                     ↓
LOCK                Maintain
```

### Output Example

```json
{
  "status": "SAFE",
  "evidence_score": 0.94,
  "frames_verified": 5,
  "start_authorized": true,
  "authorization_ttl_ms": 1000
}
```

### Done Criteria

- [ ] State machine implemented
- [ ] Evidence threshold implemented
- [ ] Temporal logic works
- [ ] Fail-safe behavior implemented
- [ ] AI failure → LOCK
- [ ] Camera failure → LOCK
- [ ] Controller failure → LOCK
- [ ] No-helmet → LOCK
- [ ] Successful verification → temporary START authorization
- [ ] Decision latency measured

---

# PHASE 5 — Hardware Interlock

## Objective

Demonstrate that the AI decision can physically control a vehicle-start simulation.

### Recommended Hardware

```text
Webcam
   ↓
Computer
   ↓
AI + Safety Decision
   ↓
USB Serial
   ↓
Arduino / ESP32
   ↓
Relay / Motor Driver
   ↓
Small DC Motor / LED
```

### Hardware States

```text
SAFE
 ↓
START ENABLED
 ↓
Motor ON
```

and:

```text
UNSAFE / ERROR
 ↓
START LOCKED
 ↓
Motor OFF
```

### Serial Communication

Example:

```text
AI → Arduino
"START"
```

or:

```text
AI → Arduino
"LOCK"
```

Prefer a heartbeat/watchdog in the final prototype:

```text
AI → Controller
HEARTBEAT + AUTHORIZATION

No valid heartbeat
       ↓
     LOCK
```

### Important Safety Rule

Do **not** connect the prototype directly to a real motorcycle ignition system during development.

Use:

- LED
- Buzzer
- Relay
- Small DC motor

to represent ignition.

### Done Criteria

- [ ] Arduino/ESP32 communicates with computer
- [ ] START command works
- [ ] LOCK command works
- [ ] Motor/LED turns ON after valid authorization
- [ ] Motor/LED remains OFF without helmet
- [ ] Authorization expires correctly
- [ ] Controller disconnect results in safe lock
- [ ] Complete AI → hardware chain works

---

# PHASE 6 — Backend & Database

## Objective

Add monitoring and event logging **without making the safety system dependent on the backend**.

### Tasks

- Create FastAPI backend
- Create SQLite database if required
- Create verification-events table
- Create system logs if required
- Create status endpoint
- Record safety results
- Record system faults
- Connect dashboard to event/status data

### Example Flow

```text
AI
 ↓
Safety Decision
 ├──────────────→ Hardware Controller
 │
 └──────────────→ FastAPI
                    ↓
                 Database
                    ↓
                Dashboard
```

### Critical requirement

The safety decision must continue working if:

```text
Database = OFF
Internet = OFF
Backend = OFF
```

The database is for logging/monitoring, **not motor authorization**.

### Done Criteria

- [ ] Backend runs
- [ ] Database created if needed
- [ ] Verification events saved
- [ ] System status available
- [ ] API responds correctly
- [ ] AI-to-backend logging works
- [ ] Safety path works with backend/database disabled

---

# PHASE 7 — Frontend Dashboard

## Objective

Create a clear SIH demonstration interface.

### Main Screen

Display:

```text
┌───────────────────────────────────────────┐
│              HELMETGUARD AI               │
│           "No Helmet. No Start."          │
├─────────────────────────┬─────────────────┤
│                         │ SYSTEM HEALTH    │
│      LIVE CAMERA        │                 │
│                         │ Camera      ✓   │
│     [Rider + Head]      │ AI Model    ✓   │
│     [Helmet ROI]        │ Controller  ✓   │
│                         │ Interlock   ✓   │
├─────────────────────────┴─────────────────┤
│                                           │
│        🟢 SAFE — START ENABLED           │
│        Helmet Usage Verified              │
│        Evidence: 94%                     │
│                                           │
├───────────────────────────────────────────┤
│        MOTOR / IGNITION: ON              │
└───────────────────────────────────────────┘
```

### Done Criteria

- [ ] Live camera displayed
- [ ] Rider/helmet evidence displayed
- [ ] Safety state prominent
- [ ] Hardware state displayed
- [ ] Error state clearly visible
- [ ] UI remains responsive during inference

---

# PHASE 8 — UI Polish & Demo Mode

## Objective

Make the system easy for an SIH jury to understand within seconds.

### Tasks

- Add SAFE/LOCK/VERIFYING state visuals
- Add clear reason messages
- Add presentation mode
- Remove unnecessary cards
- Improve typography/contrast
- Add hardware status
- Show verification latency/evidence only where useful

### SIH Demo Banner

```text
SIH26220 • SMART VEHICLES • HARDWARE
```

### Done Criteria

- [ ] Jury can understand the safety state immediately
- [ ] SAFE and LOCKED states are visually distinct
- [ ] Error state clearly says START LOCKED
- [ ] No decorative element distracts from safety decision

---

# PHASE 9 — Testing

## Objective

Measure the project as a **safety system**, not only as an object detector.

## 9.1 AI Testing

Test:

- Helmeted rider
- No helmet
- Helmet held in hand
- Different helmet colours/styles
- Different rider clothing
- Different distances
- Camera angles
- Bright lighting
- Low lighting
- Partial occlusion
- Motion blur
- Multiple people

Record:

- Precision
- Recall
- F1-score
- mAP where appropriate
- FPS
- Inference latency

## 9.2 System Testing

### Critical tests

| Scenario | Expected Result |
|---|---|
| Helmet worn | START ENABLED after stable verification |
| No helmet | START LOCKED |
| Helmet in hand | START LOCKED |
| Poor image quality | VERIFY/LOCK |
| Multiple-rider ambiguity | LOCK |
| Camera failure | LOCK |
| AI failure | LOCK |
| Controller failure | LOCK |
| Backend failure | Safety decision continues locally |
| Internet failure | Safety decision continues locally |
| Lost heartbeat | LOCK |

## 9.3 Performance Testing

Measure:

- End-to-end verification time
- Time to SAFE
- Time to LOCK
- FPS
- CPU/GPU usage where useful
- Memory usage
- Serial command latency

### Most important system metric

> **False Authorization Rate**

Track false locks separately so that safety and usability can be balanced deliberately.

---

# PHASE 10 — Integration Testing

## Objective

Verify the complete chain.

```text
Camera
  ↓
AI
  ↓
Association
  ↓
Temporal Verification
  ↓
Safety Decision
  ↓
Serial
  ↓
MCU
  ↓
Relay
  ↓
Motor
```

### Integration scenarios

1. Helmet → motor ON.
2. Remove helmet → motor OFF.
3. Helmet in hand → motor OFF.
4. Camera disconnected → motor OFF.
5. AI stopped → motor OFF.
6. Serial disconnected → motor OFF.
7. Backend stopped → motor behaviour unchanged.
8. Temporary bad frame → stable state should not oscillate unnecessarily.

### Done Criteria

- [ ] End-to-end chain validated
- [ ] Failure states validated
- [ ] No backend dependency
- [ ] No unsafe failure path identified in prototype tests

---

# PHASE 11 — Deployment

## Objective

Prepare a repeatable SIH demonstration setup.

### Prototype Deployment

```text
Laptop / Edge Computer
        +
Webcam
        +
Arduino/ESP32
        +
Relay / Motor / LED
```

### Future Edge Direction

Possible platforms:

- Raspberry Pi-class edge hardware
- NVIDIA Jetson-class edge hardware
- Other embedded AI hardware

These must be benchmarked before being claimed as deployment hardware.

### Deployment Checklist

- [ ] Camera mounted at validated position
- [ ] Lighting checked
- [ ] Model loaded locally
- [ ] Serial port configured
- [ ] Controller watchdog tested
- [ ] Motor/LED wiring checked
- [ ] Demo cases rehearsed
- [ ] Backup camera/model available if possible
- [ ] Offline operation verified

---

# 12. Team Division

A six-person team can divide work as follows:

### Member 1 — AI / Dataset

- Dataset
- Annotation
- Model training
- Evaluation

### Member 2 — Verification / Computer Vision

- Rider tracking
- Helmet association
- Temporal evidence
- Image-quality gate

### Member 3 — Safety Decision / Backend

- State machine
- FastAPI
- Event logging
- Configuration

### Member 4 — Hardware / Embedded

- Arduino/ESP32
- Relay/motor
- Serial protocol
- Watchdog/heartbeat

### Member 5 — Frontend / UI-UX

- Dashboard
- Live verification UI
- Demo mode
- Visual state system

### Member 6 — Integration / Testing / SIH Documentation

- Integration testing
- Metrics
- Failure scenarios
- Demo script
- PPT/documentation

All members should understand the complete system flow for the jury presentation.

---

# 13. Suggested Development Order

The recommended order is:

```text
1. Camera + baseline detector
2. Dataset/model evaluation
3. Rider + helmet detection
4. Rider tracking
5. Rider–helmet association
6. Temporal evidence
7. Safety state machine
8. Hardware interlock
9. Watchdog/heartbeat
10. Testing + metrics
11. Backend/logging
12. Dashboard
13. Demo polish
```

Do not spend significant time on UI before the AI-to-hardware chain works.

---

# 14. MVP Definition

The MVP is complete when this chain works reliably:

```text
Camera
  ↓
Rider Detection
  ↓
Helmet Detection
  ↓
Rider–Helmet Association
  ↓
Temporal Verification
  ↓
Safety Decision
  ↓
Serial Authorization
  ↓
Arduino/ESP32
  ↓
Motor / LED
```

### MVP must demonstrate

1. Helmeted rider → safe authorization.
2. No helmet → lock.
3. Helmet in hand → lock.
4. Uncertain evidence → lock.
5. Camera/AI/controller failure → lock.
6. Backend/database failure does not break safety.
7. Jury can see the decision and physical result.

---

# 15. Final Done Criteria

### AI

- [ ] Dataset evaluated
- [ ] Model metrics recorded
- [ ] Failure cases documented

### Verification

- [ ] Rider tracking works
- [ ] Helmet association works
- [ ] Temporal evidence works
- [ ] Image-quality handling works

### Safety

- [ ] State machine works
- [ ] False authorization measured
- [ ] False lock measured
- [ ] All critical faults lock safely

### Hardware

- [ ] START command works
- [ ] LOCK command works
- [ ] Watchdog/heartbeat works
- [ ] Physical motor/LED follows decision

### Software

- [ ] Backend optional path works
- [ ] UI displays correct state
- [ ] Local/offline mode works

### SIH

- [ ] Project explicitly mapped to SIH26220
- [ ] Hardware nature is obvious
- [ ] Smart Vehicles/commutation impact is explained
- [ ] No unsupported novelty claim is used

---

# 16. SIH Final Demo Sequence

### Step 1 — Introduce the problem

```text
Helmet compliance is often checked after unsafe riding has already begun.
```

### Step 2 — Show the device

```text
Camera → AI → Safety Decision → Controller → Motor
```

### Step 3 — Helmeted rider

```text
Helmet
 ↓
Verification
 ↓
SAFE
 ↓
Motor ON
```

### Step 4 — Remove helmet

```text
No Helmet
 ↓
UNSAFE
 ↓
START LOCKED
 ↓
Motor OFF
```

### Step 5 — Helmet in hand

```text
Helmet visible
 ↓
Association fails
 ↓
START LOCKED
```

### Step 6 — Failure case

```text
Camera/AI/controller fault
 ↓
SAFE LOCK
```

### Step 7 — Explain the contribution

> **The system does not simply detect a helmet. It converts rider-specific visual evidence into a fail-safe physical pre-start authorization.**

---

# 17. Long-Term Implementation Roadmap

### Version 1 — SIH MVP

- Helmet verification
- Rider association
- Temporal evidence
- Safety state machine
- Physical interlock simulation

### Version 2 — Edge Prototype

- Embedded inference
- Better low-light robustness
- Improved enclosure
- Controller watchdog
- Tamper detection

### Version 3 — Production-Oriented Research

- Liveness/anti-spoofing
- More robust multi-rider handling
- Improper helmet/strap research
- Real-vehicle integration studies
- Automotive-grade hardware/safety validation

### Version 4 — Broader Safety Platform

- Additional rider-state checks
- Fleet management
- Safety analytics

These future capabilities must not dilute the SIH MVP.

---

# 18. Final Development Principle

> **Build the safety path first. Make the AI measurable. Make uncertainty fail safely. Then add the dashboard.**

HelmetGuard AI should be implemented and demonstrated as a **hardware safety device aligned with SIH26220**, with computer vision acting as the perception layer and the interlock/controller acting as the physical safety intervention.
