# 01 — Product Requirements Document (PRD)

## 1. Product Overview

### App / Project Name

**HelmetGuard AI**

### SIH Alignment

| Field | Value |
|---|---|
| Problem Statement ID | **SIH26220** |
| Problem Statement | **Student Innovation-Creating intelligent devices to improve commutation sector.** |
| Category | **Hardware** |
| Theme | **Smart Vehicles** |
| Organization | **AICTE** |
| Department | **AICTE, MIC-Student Innovation** |

### Product Type

**AI-powered preventive two-wheeler safety interlock / intelligent vehicle-start safety device.**

### Tagline

> **“No Helmet. No Start.”**

### One-Line Description

HelmetGuard AI uses camera-based computer vision to verify that the **target two-wheeler rider is wearing a helmet** before issuing a time-limited authorization to a physical vehicle-start simulation.

### SIH Fit

SIH26220 is an open-ended hardware problem statement under Smart Vehicles. HelmetGuard AI fits it by implementing an intelligent device that improves two-wheeler commuting safety through **pre-start safety verification and physical interlocking**.

The SIH statement itself does not specifically prescribe helmet detection; helmet verification is the proposed solution within the stated Smart Vehicles/commutation scope.

---

# 2. Problem

Helmet use is a fundamental two-wheeler safety measure, but compliance often depends on rider behaviour and external enforcement. A conventional camera system may detect or report helmet violations after the vehicle is already in motion.

HelmetGuard AI takes a **preventive** approach:

> **Do not allow vehicle-start authorization unless sufficient visual evidence indicates that the target rider is wearing a helmet.**

A simple object detector is not enough. A helmet may appear elsewhere in the frame, be held in a hand, belong to another person, or be detected incorrectly because of blur, lighting or occlusion.

Therefore, the product requirement is not merely **“detect a helmet.”** It is:

> **“Verify helmet usage for the target rider with sufficient evidence before authorizing start.”**

### Engineering problem

```text
Helmet visible ≠ Helmet worn by target rider

Therefore:
Detection → Association → Temporal Verification → Safety Decision
```

---

# 3. Target Users

### Primary Users

- Two-wheeler riders
- College students using motorcycles/scooters
- Young riders and daily commuters

### Secondary / Deployment Users

- Colleges and educational institutions
- Delivery and corporate fleets
- Rental/shared two-wheeler operators
- Fleet safety managers

### Future Institutional Users

- Vehicle manufacturers
- Transport organizations
- Road-safety organizations

The MVP is designed around the rider and the physical vehicle-start safety decision rather than administrative monitoring.

---

# 4. Product Goal

> **Create a reliable, low-cost intelligent safety device that verifies apparent helmet usage before allowing a two-wheeler to start.**

The system should be:

- Preventive rather than merely reactive
- Fast enough for practical pre-start verification
- Low-cost and prototype-friendly
- Privacy-conscious
- Hardware-integrable
- Robust against obvious false-authorization cases
- Fail-safe
- Capable of local/offline operation

### Safety priority

When the system cannot establish sufficient safety evidence:

> **LOCK. Do not authorize START.**

---

# 5. Core Features — Must Have

## M1. Rider Detection and Target Selection

The camera must determine whether a rider is present and identify the target rider for verification.

If multiple people are visible and the target rider cannot be confidently identified, the system must not authorize start.

```text
No rider
    ↓
WAITING / LOCKED

Target rider detected
    ↓
Begin verification

Ambiguous / multiple riders
    ↓
UNCERTAIN / LOCKED
```

---

## M2. Helmet Detection

The AI system must detect helmet evidence in the relevant rider/head region.

The initial model may support:

- `rider`
- `helmet`
- `no_helmet`

The exact class design can be revised after dataset evaluation.

**Important:** camera-based verification indicates apparent helmet use; it does not prove helmet certification, physical condition or legal compliance in every respect.

---

## M3. Rider–Helmet Association

The system must determine whether the detected helmet belongs to the **target rider's head**, rather than merely detecting a helmet somewhere in the frame.

A preferred approach is:

```text
Rider Detection
      ↓
Rider Tracking
      ↓
Head / upper-body region
      ↓
Helmet candidate
      ↓
Spatial + geometric association
      ↓
Rider-specific helmet evidence
```

### Example

```text
Helmet held in hand
        ↓
Helmet detected
        ↓
Not associated with target rider's head
        ↓
Verification FAIL / LOCK
```

---

## M4. Temporal Verification

The system must not authorize ignition from a single frame.

Instead of treating “5 frames” as a permanent rule, the implementation should support **temporal evidence and hysteresis** so that short detection glitches do not immediately change the safety state.

Example:

```text
Frame evidence:
PASS → PASS → PASS → PASS → PASS
                    ↓
             Stable evidence
                    ↓
                  SAFE
```

For inconsistent evidence:

```text
PASS → UNKNOWN → FAIL → PASS
          ↓
 Insufficient evidence
          ↓
        LOCKED
```

The required window/score should be tuned experimentally.

---

## M5. Confidence / Evidence-Based Decision

The system should combine relevant evidence rather than using a single raw detector confidence.

Possible evidence inputs:

- Rider detection confidence
- Helmet detection confidence
- Rider–helmet association quality
- Temporal consistency
- Image-quality score
- Tracking stability

The implementation may produce an internal safety-evidence score, but the decision threshold must be established using validation/testing data rather than arbitrarily fixed at 70%.

---

## M6. Image Quality Gate

Before accepting a positive safety decision, the system should check whether the frame is sufficiently usable.

Potential checks:

- Excessive blur
- Extremely poor illumination
- Severe occlusion
- Invalid camera frame
- Rider too far from the verification region

```text
Usable image?
   ├── NO → VERIFYING / LOCKED
   └── YES → Continue safety verification
```

This prevents low-quality input from being treated as strong evidence.

---

## M7. Safety Decision Engine

The AI pipeline must ultimately produce a small deterministic state set.

### SAFE

```text
Target rider identified
        +
Helmet associated with rider
        +
Sufficient confidence/evidence
        +
Stable temporal evidence
        +
System healthy
        ↓
START AUTHORIZED
```

### UNSAFE / UNCERTAIN / ERROR

```text
No helmet
OR wrong association
OR insufficient evidence
OR image unusable
OR multiple-rider ambiguity
OR camera/AI/hardware fault
        ↓
START LOCKED
```

---

## M8. Ignition Authorization Interface

The AI system must communicate its safety decision to a controller.

Prototype options:

- Arduino
- ESP32
- Relay
- Motor driver
- LED / small DC motor

The prototype must **simulate vehicle ignition** rather than directly modifying a production motorcycle's safety-critical electronics.

### Authorization principle

A `START` authorization should be **time-limited/expiring**. If the controller stops receiving valid authorization/heartbeat, it should return to the locked state.

---

## M9. User Feedback

The rider must clearly understand the current safety state and the reason for a lock.

### Successful verification

> **Helmet Verified**  
> **SAFE — START ENABLED**

### No helmet

> **Helmet Not Verified**  
> Please wear your helmet.

### Uncertain

> **Unable to Verify Helmet Usage**  
> Adjust your position and try again.

### System fault

> **Safety Interlock Unavailable**  
> Start remains locked.

---

## M10. Near-Real-Time Processing

The system should complete pre-start verification quickly enough to avoid unnecessary rider delay.

Measure:

- Inference FPS
- End-to-end verification latency
- Time to SAFE decision
- Time to LOCK after unsafe evidence

Targets should be finalized after testing on the actual demo hardware rather than claimed in advance.

---

# 6. Nice-to-Have Features

These are **not required for the MVP** and must not distract from the SIH core demonstration.

## N1. Improper Helmet Detection

Detect helmets that appear present but are incorrectly positioned.

## N2. Multiple-Rider Detection

Detect and safely handle multiple people/riders in the verification region.

## N3. Helmet Strap Verification

Explore strap/fastening verification only if reliable visual evidence can be obtained. Do not claim secure fastening from helmet presence alone.

## N4. Safety Event Logging

Store minimum metadata such as:

```text
Timestamp
Device
Decision
Evidence/confidence
Frames/time window
Reason
Model version
Latency
```

No continuous raw video is required.

## N5. Fleet Dashboard

Future organizations could monitor multiple HelmetGuard devices.

## N6. Offline / Edge AI Mode

Run the full safety pipeline locally on an edge platform. This is a deployment direction, not an MVP claim unless tested.

## N7. Additional Rider Safety Features

Future possibilities include drowsiness, distraction and accident detection, but these are outside the current SIH MVP.

## N8. Liveness / Anti-Spoofing

Future enhancement to detect attempts to fool the camera using printed images, screens or pre-recorded video.

---

# 7. Out of Scope

### O1. Automatic Police Reporting

Not part of the MVP.

### O2. Facial Recognition

The system does not need to identify individual riders.

### O3. Continuous Cloud Video Storage

Not required.

### O4. GPS Tracking

Not required for the core safety function.

### O5. Number Plate Recognition

Not required.

### O6. Alcohol Detection

Not part of the MVP.

### O7. Direct Production Motorcycle Modification

The SIH prototype will use a simulated ignition/interlock. Any real vehicle integration would require appropriate electrical, automotive and safety validation.

### O8. Complete Traffic Enforcement Platform

HelmetGuard AI is a **vehicle safety interlock**, not a police or traffic-enforcement system.

### O9. Legal Certification Decision

The camera does not certify helmet standards, authenticity, structural condition or complete legal compliance.

---

# 8. User Stories

## US-01 — Helmeted Rider

**As a rider,** I want the system to recognize sufficient evidence that I am wearing a helmet so that I can start without unnecessary delay.

### Acceptance Criteria

- Target rider is identified.
- Helmet is associated with the rider's head.
- Evidence remains stable for the configured verification window.
- System health is valid.
- Safety state becomes SAFE.
- Start authorization is issued.

## US-02 — Rider Without Helmet

**As a safety system,** I want to prevent start authorization when helmet use is not verified.

### Acceptance Criteria

- Rider is detected.
- Helmet evidence is absent/negative.
- Safety state becomes UNSAFE/LOCKED.
- Start authorization is denied.
- User receives a clear message.

## US-03 — Helmet Held in Hand

**As a safety system,** I want to distinguish a helmet from a helmet worn by the target rider so that showing a helmet to the camera cannot by itself authorize starting.

### Acceptance Criteria

- Helmet may be detected.
- Head-aware association fails.
- Start remains locked.

## US-04 — Uncertain AI Result

**As a safety system,** I want insufficient evidence to fail safely so that uncertainty does not become an unsafe start authorization.

### Acceptance Criteria

- Low/ambiguous evidence does not authorize start.
- System remains locked or verifying.
- User can retry.

## US-05 — Temporary Detection Error

**As a rider,** I want short-lived detection errors to be handled through temporal evidence rather than an immediate state flip.

### Acceptance Criteria

- A single bad frame does not instantly override a stable decision.
- The system uses temporal evidence/hysteresis.
- Critical faults still force a safe lock.

## US-06 — Multiple-Person Ambiguity

**As a safety system,** I want ambiguous rider identity to remain locked so that a helmet worn by one person cannot authorize another person's start.

---

# 9. Core User Journey

```text
Rider approaches vehicle
        ↓
Camera / image-quality check
        ↓
Target rider detected
        ↓
Helmet candidate detected
        ↓
Rider–helmet association
        ↓
Tracking + temporal evidence
        ↓
Safety evidence evaluation
        ↓
      ┌───────────────┐
      ↓               ↓
    SAFE        UNSAFE / UNCERTAIN
      ↓               ↓
START ENABLED      START LOCKED
      ↓               ↓
Simulated motor       Warning / retry
```

---

# 10. Success Metrics

The project must be evaluated at both **model level** and **safety-system level**.

## AI Metrics

- Precision
- Recall
- F1-score
- mAP where appropriate
- Inference latency/FPS

## System Metrics

### 1. False Authorization Rate — Critical

How often an unsafe/uncertain case is incorrectly allowed to start.

> This is the most important safety metric.

### 2. False Lock Rate

How often a genuinely helmeted rider is incorrectly kept locked.

### 3. Verification Time

Time from target-rider detection to stable decision.

### 4. State Transition Latency

Time to lock after unsafe evidence or system fault.

### 5. Robustness Metrics

Evaluate at minimum:

- Helmet in hand
- Different helmet colours/styles
- Different rider clothing
- Different distances
- Camera angles
- Lighting changes
- Partial occlusion
- Multiple people
- Temporary blur

### 6. Hardware Reliability

Measure successful command delivery and safe behaviour after disconnection.

---

# 11. MVP Definition

The first working version must accomplish only this:

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
Serial Communication
  ↓
Microcontroller
  ↓
Motor / LED Ignition Simulation
```

### MVP Success

1. Helmeted rider can be verified consistently.
2. No-helmet rider remains locked.
3. Helmet held in hand does not authorize start.
4. Uncertain detection remains locked.
5. Camera/AI/hardware failure remains locked.
6. End-to-end response is near real time.
7. User receives understandable safety feedback.

---

# 12. Product Principle

## **Safety Before Convenience**

When the system is uncertain:

> **Do not authorize the start.**

The product should optimize for a low **false-authorization rate** while keeping verification latency practical.

---

# 13. Long-Term Vision

HelmetGuard AI can evolve into a broader:

### **AI-Powered Two-Wheeler Safety Platform**

```text
                  AI SAFETY ENGINE
                         ↓
       ┌─────────────────┼─────────────────┐
       ↓                 ↓                 ↓
 Helmet Verification  Rider State      Vehicle State
       ↓                 ↓                 ↓
       └─────────────────┼─────────────────┘
                         ↓
                  Safety Decision
                         ↓
                  Vehicle Authorization
```

The SIH MVP remains intentionally focused on **helmet verification + pre-start authorization**.

---

# 14. Final Product Definition

> **HelmetGuard AI is an intelligent, camera-based pre-start safety interlock for two-wheelers aligned with SIH2026 Problem Statement SIH26220. It verifies whether the target rider is apparently wearing a helmet using rider–helmet association and temporal evidence, then converts the result into a fail-safe physical start/lock decision.**

The project contribution should be demonstrated through **system integration, safety behaviour, measurable false-authorization performance and hardware operation**, not through an unsupported claim that helmet detection itself is novel.
