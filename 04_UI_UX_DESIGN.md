# 04 UI/UX Design Brief — HelmetGuard AI

## 1. App Name

**HelmetGuard AI**

**SIH 2026:** **SIH26220 — Student Innovation-Creating intelligent devices to improve commutation sector.**

**Theme:** Smart Vehicles  
**Category:** Hardware

**Tagline:**

> **No Helmet. No Start.**

---

## 2. Design Goal

The interface should feel like a **modern vehicle safety/control system**, not a generic AI dashboard.

The most important information must always be immediately visible:

> **Is the rider verified, and is the vehicle allowed to start?**

The UI should be:

- Simple
- Professional
- Safety-focused
- Easy to understand during an SIH demonstration
- Responsive
- Suitable for desktop/tablet
- Readable from a distance
- Clear even when viewed briefly by a jury member

The UI must reflect the physical safety state; it must not become the source of truth for authorization.

---

## 3. Aesthetic

### Overall Style

**Modern • Minimal • Technical • Automotive • Safety-oriented**

Visual inspiration:

- Automotive dashboards
- EV control systems
- Industrial monitoring interfaces
- Modern AI monitoring tools

Avoid:

- Excessive animations
- Crowded dashboards
- Too many cards
- Decorative elements that distract from safety
- Generic “AI” visual effects that do not communicate system state

---

## 4. Primary Color

**Deep Navy / Dark Blue**

Suggested:

```text
Primary: #0F172A
```

Purpose:

- Technology
- Trust
- Professional appearance

---

## 5. Background Color

**Very Light Gray / Off White**

Suggested:

```text
Background: #F8FAFC
```

A darker live-verification surface may be used to make the camera/safety state visually prominent during SIH demonstrations.

---

## 6. Text Color

```text
Text: #1E293B
Secondary: #64748B
```

Text must maintain strong contrast and remain readable from a distance.

---

## 7. Accent / CTA Colors

Status colours must have text/icons in addition to colour so the state is never colour-dependent.

### Safe / Start Enabled

```text
Green: #16A34A
```

Use for:

- Helmet verified
- SAFE
- START ENABLED
- Healthy interlock

### Unsafe / Start Locked

```text
Red: #DC2626
```

Use for:

- No helmet
- Unsafe condition
- START LOCKED
- Critical safety errors

### Verifying / Uncertain

```text
Amber: #F59E0B
```

Use for:

- Verification in progress
- Insufficient evidence
- Waiting for temporal confirmation

### Normal CTA

Use the primary navy/blue theme for normal actions.

---

# 8. Font

### Recommended

**Inter**

Why:

- Modern
- Highly readable
- Good dashboard typography
- Clear numerical/status display

Alternative:

**Roboto**

---

# 9. Border Radius

Use moderately rounded components.

```text
Cards: 12px
Buttons: 8px
Inputs: 8px
Status panels: 12px
```

Avoid excessive pill-shaped components because the product should feel like a serious safety system.

---

# 10. Shadows

Use subtle shadows only.

```text
0 2px 8px rgba(...)
```

The dashboard should feel controlled and industrial rather than decorative.

---

# 11. Dark / Light Mode

### MVP

**Light mode by default.**

Reason:

- Easy to read
- Good for SIH exhibition/classroom lighting
- Professional dashboard appearance

### Optional

Dark mode can be added later.

A dark verification screen may be used for a high-contrast demo state:

```text
🟢 SAFE — START ENABLED
```

or

```text
🔴 UNSAFE — START LOCKED
```

---

# 12. Main Dashboard Layout

The safety decision should dominate the page.

```text
┌──────────────────────────────────────────────────────┐
│ HelmetGuard AI                         SYSTEM ●       │
├──────────────────────────────────────────────────────┤
│                                                      │
│ ┌────────────────────────────┐ ┌───────────────────┐ │
│ │                            │ │ SYSTEM HEALTH     │ │
│ │        LIVE CAMERA         │ │                   │ │
│ │                            │ │ Camera       ✓    │ │
│ │   [Rider + Helmet ROI]     │ │ AI Model     ✓    │ │
│ │                            │ │ Controller   ✓    │ │
│ └────────────────────────────┘ │ Interlock    ✓    │ │
│                                └───────────────────┘ │
│                                                      │
│ ┌──────────────────────────────────────────────────┐ │
│ │       🟢 SAFE — START ENABLED                   │ │
│ │       Helmet usage verified                      │ │
│ │       Evidence: 94% | State: SAFE               │ │
│ └──────────────────────────────────────────────────┘ │
│                                                      │
│       MOTOR / IGNITION SIMULATION: ON              │
└──────────────────────────────────────────────────────┘
```

### Information hierarchy

1. Safety state
2. Start/lock state
3. Live verification evidence
4. Hardware/interlock state
5. Supporting metrics

---

# 13. Live Camera Component

The camera feed should be the largest visual element.

It may display:

- Rider bounding box
- Helmet bounding box
- Target rider indicator
- Head/association region
- Detection labels
- Confidence/evidence
- Verification state

Example:

```text
┌───────────────────────────────────┐
│                                   │
│       ┌───────────────┐           │
│       │   HELMET      │  94%      │
│       └───────────────┘           │
│       │     HEAD ROI  │           │
│       │               │           │
│       │     RIDER     │           │
│       │               │           │
│       └───────────────┘           │
│                                   │
│  ASSOCIATION: VERIFIED            │
│  EVIDENCE: 4 / 5                  │
└───────────────────────────────────┘
```

Do not overload the feed with technical overlays that make the safety state harder to see.

---

# 14. Safety Status Card

This is the **most important UI component**.

### Safe

```text
┌──────────────────────────────────┐
│ ✓ SAFE                            │
│                                  │
│ START ENABLED                    │
│ Helmet Usage Verified             │
│ Evidence: 94%                    │
└──────────────────────────────────┘
```

### Unsafe

```text
┌──────────────────────────────────┐
│ ✕ UNSAFE                          │
│                                  │
│ START LOCKED                     │
│ Helmet Not Verified              │
└──────────────────────────────────┘
```

### Verifying

```text
┌──────────────────────────────────┐
│ ◌ VERIFYING                      │
│                                  │
│ Checking Rider–Helmet Association│
│ Evidence: 3 / 5                 │
└──────────────────────────────────┘
```

### Error

```text
┌──────────────────────────────────┐
│ ⚠ SAFE LOCK                      │
│                                  │
│ CAMERA / AI / CONTROLLER ERROR   │
│ START REMAINS LOCKED             │
└──────────────────────────────────┘
```

---

# 15. Hardware Status

Display the physical safety chain separately.

```text
Hardware
──────────────
Controller      ✓ Connected
Interlock       ✓ Ready
Authorization   ✓ Valid
Motor           OFF
```

When authorized:

```text
Controller      ✓ Connected
Interlock       ✓ Enabled
Authorization   ✓ Active
Motor           ON
```

The motor represents vehicle ignition only in the prototype.

### Watchdog state

If the controller loses the AI heartbeat:

```text
Heartbeat       ✕ Lost
Interlock       🔒 SAFE LOCK
Motor           OFF
```

---

# 16. Navigation

Keep navigation simple.

```text
┌─────────────────────────────────────────┐
│ HelmetGuard AI                           │
│                                         │
│ Dashboard | Verification | Logs | Setup │
└─────────────────────────────────────────┘
```

Do not create unnecessary pages for the MVP.

---

# 17. Empty States

### Waiting for Rider

> **Waiting for rider**  
> Position yourself in the verification area.

### No Events

> **No safety events yet**  
> Verification events will appear here.

### Camera Unavailable

> **Camera not connected**  
> Start remains locked.

### Controller Unavailable

> **Safety interlock unavailable**  
> Start remains locked.

### Poor Image Quality

> **Image quality insufficient**  
> Improve lighting or rider position.

---

# 18. Error UI

Errors must clearly communicate the safety consequence.

Example:

```text
⚠ CAMERA ERROR

Camera connection lost.

🔒 START LOCKED

Reconnect the camera to continue.
```

Avoid generic messages such as:

> “Something went wrong.”

The user should immediately understand why the vehicle remains locked.

---

# 19. Animations

Animations should be minimal and functional.

Recommended:

- Detection-box updates
- Verification progress
- Smooth safety-state transition
- Hardware connection indicator
- Short lock/enable transition

Avoid:

- Large page transitions
- Constant moving backgrounds
- Decorative particles
- Excessive loading animations

Safety-state changes must be noticeable but not distracting.

---

# 20. Mobile Constraints

The system is primarily intended for **desktop/tablet SIH demonstration**.

### Desktop

```text
Camera Feed | System Status
             ↓
       Safety Status
```

### Tablet

```text
Camera
  ↓
Safety Status
  ↓
System Health
```

### Mobile

Stack components vertically.

The **Safety Status** must remain near the top and never be hidden below large content blocks.

---

# 21. Accessibility

The interface must not depend only on colour.

Good:

> 🟢 ✓ **SAFE — START ENABLED**

Instead of displaying only a green box.

Similarly:

> 🔴 ✕ **UNSAFE — START LOCKED**

Use clear icons, text, sufficient contrast and readable font sizes.

---

# 22. SIH Presentation Mode

A dedicated presentation/demo mode can simplify the UI for jury viewing.

### Always visible

```text
SIH26220
HelmetGuard AI

CURRENT STATE: SAFE / LOCKED / VERIFYING
START STATUS: ENABLED / LOCKED
RIDER: DETECTED / NOT DETECTED
HELMET: VERIFIED / NOT VERIFIED / UNKNOWN
CONTROLLER: CONNECTED / ERROR
```

### Recommended demo emphasis

The jury should be able to understand the complete chain in seconds:

```text
AI VERIFICATION
      ↓
SAFETY DECISION
      ↓
PHYSICAL INTERLOCK
      ↓
MOTOR ON / OFF
```

---

# 23. Reference Apps

Use automotive dashboards, industrial safety interfaces and EV control systems as inspiration.

Do not copy generic AI dashboard layouts where the safety decision becomes secondary to analytics cards.

---

# 24. Design Priorities

### Priority 1

- Safety state visibility
- Start/lock visibility
- Live rider/helmet verification
- Clear fault state

### Priority 2

- Evidence/confidence information
- Hardware state
- Verification progress

### Priority 3

- Logs
- Settings
- Analytics
- Fleet features

---

# 25. Final Design Principle

> **The interface is not an AI showcase; it is the visual control surface for a safety interlock.**

Every design decision should answer one question:

> **Can a rider and an SIH jury member immediately understand why the system is allowing or preventing the vehicle from starting?**
