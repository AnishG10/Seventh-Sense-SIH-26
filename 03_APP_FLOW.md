# 03 App Flow — Navigation & User Journey

## 1. App Name

**HelmetGuard AI**

**SIH Problem Statement:** **SIH26220 — Student Innovation-Creating intelligent devices to improve commutation sector.**

**Category:** Hardware  
**Theme:** Smart Vehicles

**Core Flow:**

> **Detect → Associate → Verify → Decide → Authorize / Lock**

The application is the human-facing layer of a physical safety interlock. The primary goal is to make the current safety state immediately understandable.

---

## 2. Pages List

### 1. Home / System Status

Shows:

- System health
- Camera connection
- AI model status
- Hardware/controller connection
- Current safety state
- Simulated start/lock state

### 2. Live Verification

Primary screen where:

- Camera feed is displayed
- Target rider is identified/tracked
- Helmet evidence is shown
- Association status is shown
- Verification/evidence progress is shown
- Current safety state is prominent

### 3. Verification Result

Displays:

- **SAFE — START ENABLED**
- **UNSAFE — START LOCKED**
- **VERIFYING**
- **ERROR — SAFE LOCK**

### 4. Safety Logs

Optional MVP page showing minimum event metadata:

- Time
- Result
- Reason
- Evidence/confidence
- Verification duration

### 5. Settings

Optional:

- Camera selection
- Verification region
- Evidence threshold
- Temporal window
- Hardware connection settings

Safety-critical thresholds should not be casually changed during a live demo.

---

## 3. Navigation Type

**Navigation:** Simple top navigation/sidebar on desktop.

```text
Home
 │
 ├── Live Verification
 │
 ├── Safety Logs
 │
 └── Settings
```

For the SIH prototype, navigation remains minimal.

> **Live Verification is the primary screen.**

---

## 4. First Screen

When the application starts:

```text
┌─────────────────────────────────────────┐
│              HELMETGUARD AI             │
│            "No Helmet. No Start."       │
├─────────────────────────────────────────┤
│                                         │
│              SYSTEM READY               │
│                                         │
│ Camera       ✓ Connected                │
│ AI Model     ✓ Loaded                   │
│ Controller   ✓ Connected                │
│ Interlock    ✓ Ready                    │
│                                         │
│       [ START VERIFICATION ]            │
└─────────────────────────────────────────┘
```

The user must immediately understand whether the safety system is operational.

---

## 5. Auth Flow

### MVP

No authentication is required.

```text
Open Application
       ↓
System Initialization
       ↓
Camera Check
       ↓
AI Model Check
       ↓
Controller Check
       ↓
System Ready
```

If any safety-critical component is unavailable:

```text
Initialization Failure
        ↓
SAFE LOCK
        ↓
Show Reason
```

### Future

For fleet/college administration:

```text
Login
  ↓
Authentication
  ↓
Dashboard
  ↓
Device / Fleet Management
```

---

# 6. Core User Journey 1 — Rider Wearing Helmet

### Goal

Allow a rider to start after sufficient helmet evidence is established.

```text
Rider approaches vehicle
        ↓
Camera / image-quality check
        ↓
Target rider detected
        ↓
Helmet candidate detected
        ↓
Helmet associated with rider's head
        ↓
Temporal evidence collected
        ↓
Evidence threshold passed
        ↓
Safety decision = SAFE
        ↓
Time-limited START authorization
        ↓
Motor / ignition simulation ENABLED
```

### User Experience

**Step 1**

> Rider detected

**Step 2**

> Helmet candidate detected

**Step 3**

> Verifying rider–helmet association...

**Step 4**

> ✓ Helmet usage verified

**Final**

> 🟢 **SAFE — START ENABLED**

---

# 7. Core User Journey 2 — Rider Without Helmet

```text
Rider approaches vehicle
        ↓
Camera detects rider
        ↓
No helmet evidence
        ↓
Safety decision = UNSAFE
        ↓
START authorization denied
        ↓
Motor / ignition simulation remains LOCKED
```

### User Experience

> 🔴 **UNSAFE — START LOCKED**

> Please wear your helmet to continue.

The system continues monitoring until sufficient positive evidence is available.

---

# 8. Special Safety Journey — Helmet in Hand

This is a key SIH demonstration case because it shows why simple helmet presence is insufficient.

```text
Rider detected
      ↓
Helmet detected
      ↓
Helmet not associated with rider's head
      ↓
Safety evidence insufficient
      ↓
START LOCKED
```

Display:

> 🔴 **Helmet detected, but rider–helmet association was not verified.**

This demonstrates that the system verifies **rider-specific helmet usage**, not merely object presence.

---

# 9. Multi-Frame / Temporal Verification Flow

The system should use temporal evidence rather than a single-frame decision.

### Stable positive evidence

```text
Frame 1 → PASS
Frame 2 → PASS
Frame 3 → PASS
Frame 4 → PASS
Frame 5 → PASS
          ↓
   Stable evidence
          ↓
         SAFE
```

### Inconsistent evidence

```text
Frame 1 → PASS
Frame 2 → UNKNOWN
Frame 3 → FAIL
Frame 4 → PASS
          ↓
 Insufficient evidence
          ↓
      LOCK / VERIFY
```

The exact temporal window is configurable and must be tuned during testing.

---

# 10. Empty States

### No Rider Detected

> **Waiting for rider...**  
> Position yourself in the verification area.

### Camera Unavailable

> **Camera not connected**  
> Start remains locked until the camera is available.

### Controller Unavailable

> **Safety interlock unavailable**  
> Start remains locked.

### Model Not Loaded

> **AI model unavailable**  
> Start remains locked.

### Poor Image Quality

> **Image quality insufficient**  
> Improve lighting or rider position.

---

# 11. Error States

## Camera Failure

```text
Camera Error
     ↓
Verification stopped
     ↓
START LOCKED
```

Message:

> Camera unavailable. Start remains locked for safety.

## AI Model Failure

```text
AI Error
     ↓
No safety authorization
     ↓
START LOCKED
```

Message:

> Verification unavailable. Start remains locked.

## Hardware Communication Failure

```text
Controller Error
     ↓
Authorization cannot be confirmed
     ↓
START LOCKED
```

Message:

> Controller connection lost. System is in safe-lock mode.

## Low / Uncertain Evidence

```text
Helmet candidate
       ↓
Evidence insufficient
       ↓
VERIFYING
       ↓
If unresolved
       ↓
START LOCKED
```

Message:

> Unable to confidently verify helmet usage.

---

# 12. Redirects

### System Not Ready

```text
Start Verification
        ↓
System Check
        ↓
Not Ready
        ↓
System Status
```

### Camera Disconnects During Verification

```text
Live Verification
       ↓
Camera disconnects
       ↓
Verification stops
       ↓
START LOCKED
       ↓
Show camera error
```

### Controller Disconnects

```text
Verification
     ↓
Controller unavailable
     ↓
START LOCKED
     ↓
Show hardware error
```

---

# 13. Complete Application Flow

```text
                  APPLICATION START
                         │
                         ↓
                 System Initialization
                         │
                         ↓
                  System Health Check
                         │
          ┌──────────────┼──────────────┐
          ↓              ↓              ↓
       Camera           AI          Controller
       Check           Check           Check
          │              │              │
          └──────────────┼──────────────┘
                         ↓
                    System Ready
                         │
                         ↓
                  Live Verification
                         │
                         ↓
                   Image Quality
                         │
                         ↓
                   Rider Detection
                         │
                         ↓
                    Rider Tracking
                         │
                         ↓
                   Helmet Detection
                         │
                         ↓
                Head-aware Association
                         │
                         ↓
                  Temporal Evidence
                         │
                         ↓
                 Safety Evidence Check
                         │
                    ┌────┴────┐
                    ↓         ↓
                  SAFE    UNSAFE/ERROR
                    ↓         ↓
             START ENABLED  START LOCKED
                    ↓         ↓
               Motor ON     Motor OFF
```

---

# 14. Primary UI State Machine

The application should use explicit safety states:

```text
INITIALIZING
     ↓
READY
     ↓
DETECTING
     ↓
VERIFYING
     ↓
 ┌───┴─────────────┐
 ↓                 ↓
SAFE          UNSAFE / UNCERTAIN
 ↓                 ↓
START           LOCK
```

Critical error:

```text
ANY STATE
   ↓
ERROR
   ↓
SAFE LOCK
```

A `SAFE` authorization should expire if the controller loses the valid heartbeat or authorization token.

---

# 15. SIH Demo Flow

### Demo 1 — Correct Helmet

```text
Rider wears helmet
        ↓
Camera detects rider
        ↓
Rider–helmet association succeeds
        ↓
Temporal evidence becomes stable
        ↓
Green SAFE indicator
        ↓
Motor starts
```

### Demo 2 — No Helmet

```text
Rider removes helmet
        ↓
No helmet evidence
        ↓
Red UNSAFE indicator
        ↓
Motor remains OFF
```

### Demo 3 — Helmet in Hand

```text
Helmet visible
        ↓
Association with rider's head fails
        ↓
Verification not accepted
        ↓
Motor remains OFF
```

### Demo 4 — Uncertain Detection

```text
Poor lighting / occlusion / unstable evidence
        ↓
Verification uncertain
        ↓
System remains LOCKED
```

### Demo 5 — Hardware Fault

```text
Disconnect controller/camera
        ↓
System detects fault
        ↓
SAFE LOCK
```

This demonstrates that the project is a **safety interlock**, not just an AI visualizer.

---

# 16. Core UX Principle

> **The rider must always know two things: “Am I verified?” and “Can the vehicle start?”**

Every important state must communicate:

1. Current safety state.
2. Start/lock state.
3. Reason.
4. Recommended action when applicable.

---

# 17. Final User Journey

```text
                RIDER APPROACHES
                       ↓
                 SYSTEM READY?
                  ↙         ↘
                NO           YES
                ↓             ↓
             SAFE LOCK    CAMERA CHECK
                              ↓
                         RIDER FOUND?
                         ↙          ↘
                       NO            YES
                       ↓              ↓
                    WAITING      VERIFYING
                                      ↓
                            RIDER–HELMET CHECK
                                      ↓
                              TEMPORAL EVIDENCE
                                      ↓
                           ┌──────────┴──────────┐
                           ↓                     ↓
                         SAFE             UNSAFE/ERROR
                           ↓                     ↓
                    START ENABLED          START LOCKED
```
