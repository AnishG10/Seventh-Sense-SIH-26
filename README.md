# HelmetGuard AI

> **No Helmet. No Start.**

HelmetGuard AI is an **AI-powered preventive two-wheeler safety interlock** that uses camera-based computer vision to verify helmet usage **before vehicle-start authorization**. The project is designed as an intelligent hardware device for the Smart India Hackathon (SIH) 2026 Smart Vehicles theme.

---

## SIH 2026 Problem Statement Alignment

| Field | Confirmed value |
|---|---|
| **Problem Statement ID** | **SIH26220** |
| **Problem Statement** | **Student Innovation-Creating intelligent devices to improve commutation sector.** |
| **Category** | **Hardware** |
| **Theme** | **Smart Vehicles** |
| **Organization** | **AICTE** |
| **Department** | **AICTE, MIC-Student Innovation** |

### Why HelmetGuard AI fits SIH26220

SIH26220 is an open-ended Student Innovation problem statement under **Hardware → Smart Vehicles**. Its stated objective is to create intelligent devices that improve the commutation sector. HelmetGuard AI addresses this by turning a two-wheeler into a **safety-aware vehicle-start system**: the device observes the rider, verifies helmet usage, and physically controls a start/lock simulation.

The project should therefore be presented as:

> **An intelligent pre-start safety verification and interlock device for two-wheelers.**

It should **not** be presented merely as a helmet-detection application or traffic-monitoring dashboard.

### Important scope clarification

The SIH problem statement is broad; it does **not** specifically prescribe helmet detection. Helmet safety is the team's proposed solution within the Smart Vehicles/commutation scope.

**SIH26203** is the software counterpart with the same Student Innovation wording. This project is aligned to **SIH26220 because the proposed solution includes a physical interlock/controller and therefore belongs to the Hardware category.**

---

## What the System Does

```text
                 HELMETGUARD AI
                       │
                    CAMERA
                       ↓
               Image Quality Check
                       ↓
              Rider / Helmet Detection
                    ↙       ↘
                 Rider     Helmet
                    \       /
                 Head-aware Association
                       ↓
                  Rider Tracking
                       ↓
               Temporal Verification
                       ↓
              Safety Evidence Fusion
                       ↓
              Safety Decision Engine
                 ↙             ↘
              SAFE        UNSAFE / ERROR
                ↓               ↓
          START ENABLED     START LOCKED
                ↓               ↓
              MCU / Interlock Controller
                       ↓
             Relay / Motor / LED
```

### Core principle

> **Uncertainty must result in LOCK, not START.**

The system does not authorize a start merely because a helmet-shaped object appears in a frame. It attempts to establish that the **target rider is actually wearing the helmet**, using spatial association and temporal evidence.

---

## Core MVP

The SIH prototype focuses on one clear end-to-end chain:

```text
Camera
  ↓
Rider Detection
  ↓
Helmet Detection
  ↓
Rider–Helmet Association
  ↓
Temporal / Multi-Frame Verification
  ↓
Safety Decision
  ↓
Serial Command
  ↓
Arduino / ESP32
  ↓
Relay / Motor / LED
```

### Demonstration cases

1. **Helmet worn** → verification succeeds → simulated start enabled.
2. **No helmet** → verification fails → simulated start remains locked.
3. **Helmet held in hand** → helmet presence alone is insufficient → start remains locked.
4. **Poor/uncertain detection** → system stays locked until sufficient evidence is available.
5. **Camera/AI/hardware fault** → safe-lock state.

---

## What Makes the Project Stronger Than Simple Helmet Detection

HelmetGuard AI should be evaluated as a **safety decision pipeline**, not only as an object detector.

### Baseline approach

```text
YOLO confidence
      ↓
Helmet / No Helmet
      ↓
START / LOCK
```

### Proposed safety-oriented approach

```text
Detection
   ↓
Image Quality Gate
   ↓
Rider Identification / Tracking
   ↓
Head-aware Rider–Helmet Association
   ↓
Temporal Evidence
   ↓
Confidence / Evidence Fusion
   ↓
Safety State Machine
   ↓
Expiring Start Authorization
   ↓
Physical Interlock
```

This reduces the risk of unsafe authorization caused by a single false detection, a helmet elsewhere in the frame, temporary blur, occlusion, or system faults.

---

## Key Safety Requirements

- **False authorization is the critical failure mode.**
- A low-confidence or uncertain state must not authorize start.
- Camera failure must not authorize start.
- AI/model failure must not authorize start.
- Hardware communication failure must not authorize start.
- Start authorization should be temporary/expiring rather than permanent.
- The backend/database must never be required for the core safety decision.
- The prototype must use a simulated ignition, not a real motorcycle ignition circuit.
- No facial recognition is required.
- No continuous cloud video storage is required.

---

## Privacy by Design

The MVP processes camera frames locally and discards them after inference. Event logging stores only the minimum information needed for testing and system monitoring. HelmetGuard AI does not require:

- Facial recognition or face embeddings
- Aadhaar/identity information
- GPS tracking
- Number-plate recognition
- Continuous video recording
- Automatic police reporting

---

## Repository Documentation

The project documentation is intentionally ordered as follows:

```text
README.md
01_PRD.md
02_TRD.md
03_APP_FLOW.md
04_UI_UX_DESIGN.md
05_BACKEND_SCHEMA.md
06_IMPLEMENTATION_PLAN.md
```

These files form the current **source-of-truth product and engineering documentation**. Repository/code structure can evolve later without changing the SIH problem-statement alignment.

---

## Technology Direction

| Layer | Current direction |
|---|---|
| Computer Vision | OpenCV |
| Detection | YOLO-family detector |
| Deep Learning | PyTorch |
| Backend/API | Python + FastAPI |
| Dashboard | React + Vite, or lightweight Python UI for MVP |
| Hardware | Arduino/ESP32 + relay/motor/LED |
| Communication | USB Serial |
| MVP Database | SQLite, optional |
| Deployment direction | Local/edge inference |

The current SIH prototype should **not claim an edge device such as Jetson or Raspberry Pi unless it has actually been tested**. Those are future deployment options.

---

## Evaluation Metrics

### AI/model metrics

- Precision
- Recall
- F1-score
- mAP where applicable
- Inference latency / FPS

### Safety/system metrics

- **False Authorization Rate** — unsafe state incorrectly allowed to start.
- **False Lock Rate** — safe helmeted state incorrectly kept locked.
- Verification time
- Decision latency
- Hardware command reliability
- Recovery time after camera/AI/hardware faults
- Helmet-in-hand rejection rate
- Performance under lighting, occlusion, distance and multi-person conditions

The most important system metric is **false authorization**, because the device is a preventive safety interlock.

---

## Important Positioning

Do not claim that HelmetGuard AI is the first AI helmet detector or the first helmet-detection ignition system. Existing research, patents and commercial systems demonstrate that these ideas already exist.

The defensible project contribution is the **system-level integration and safety engineering**:

> **camera perception → rider-specific helmet verification → temporal evidence → uncertainty-aware safety state → physical pre-start interlock**

The project should demonstrate and measure this pipeline rather than rely on novelty claims that cannot be substantiated.

---

## Current Project Definition

> **HelmetGuard AI is an intelligent, camera-based pre-start safety interlock for two-wheelers that verifies rider helmet usage before authorizing vehicle operation. It combines computer vision, rider–helmet association, temporal verification, uncertainty-aware safety logic and physical hardware control to move from merely detecting helmet violations toward preventing helmet-less starts.**

---

## SIH Reference

The confirmed SIH 2026 record for **SIH26220** identifies it as a **Hardware / Smart Vehicles / AICTE** Student Innovation problem statement. The wording is broad, so the team's solution defines the specific safety application within that scope.
