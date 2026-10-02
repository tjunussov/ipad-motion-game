# TRD — iPad Motion Game POC

## 1. Goal

Build a native iPad proof-of-concept for a motion-controlled game.

The player stands in front of the iPad front camera. The app detects the player's body pose in real time and converts simple physical actions into game events.

Initial target actions:
- left/right hand punch
- jump
- squat

The game itself should stay deliberately primitive:
- no textures
- no character art
- no complex UI
- basic colored circles / boxes only

The purpose of this POC is to validate:
1. body tracking quality on iPad
2. latency
3. gesture detection reliability
4. simple collision/gameplay loop

---

## 2. Target Device

Primary target:
- iPad Pro with Apple M1
- front camera
- landscape orientation

Development:
- Xcode on macOS
- deploy directly to physical iPad

---

## 3. Technology

Use the simplest native Apple stack.

### Required
- Swift
- SwiftUI
- AVFoundation
- Vision

### Optional rendering
Start with SwiftUI / Canvas or simple CALayer overlays.

Only use SceneKit / RealityKit if basic 2D primitives become limiting.

Do **not** add Unity, ML Kit, YOLO, external ML models, or third-party dependencies for the first POC.

---

## 4. Architecture

```
Front Camera
    ↓
AVFoundation
    ↓
CMSampleBuffer
    ↓
Vision Human Body Pose
    ↓
PoseEstimator
    ↓
GestureDetector
    ↓
GameState
    ↓
Simple Game Renderer
```

Keep these responsibilities separated.

Suggested modules:

```
CameraManager
PoseEstimator
GestureDetector
GameEngine
GameView
DebugPoseOverlay
```

---

## 5. Pose Tracking

Use Apple Vision human body pose detection.

For each processed frame, extract normalized body joint coordinates.

Important joints:
- nose
- shoulders
- elbows
- wrists
- hips
- knees
- ankles

Ignore joints with low confidence.

Start by processing fewer frames if needed, for example 15–30 pose estimations per second.

Real-time responsiveness is more important than maximum camera FPS.

---

## 6. Debug Mode — First Milestone

Before implementing the actual game, show:

1. live front camera preview
2. detected joints as circles
3. skeleton lines between joints
4. joint confidence where useful
5. current detected gesture text

Example:

```
LEFT PUNCH
JUMP
SQUAT
IDLE
```

This debug view is the first acceptance milestone.

---

## 7. Gesture Detection

Do not use an ML classifier initially.

Use simple geometry + velocity thresholds.

### Punch

Detect punch using wrist velocity relative to shoulder/elbow.

Approximate rule:
- wrist moves rapidly away from torso
- movement exceeds configurable velocity threshold
- arm extension increases

Support:
- left punch
- right punch

Add a short cooldown to avoid repeated triggers from one motion.

### Jump

Use vertical body displacement.

Candidate signal:
- average Y of hips / shoulders moves upward rapidly
- both ankles optionally move upward
- compare against recent standing baseline

Use a rolling baseline and threshold.

### Squat

Candidate signal:
- hip Y moves downward
- knee angles close
- torso remains approximately upright

Again, use threshold logic rather than a trained model.

---

## 8. Game POC

After debug pose tracking works, implement one tiny game mode.

### Game: Punch the Balls

Behavior:
- colored balls spawn from screen edges
- balls move toward the player area
- player's left/right wrists are represented as hit points
- if wrist hit-point intersects a ball during a punch, destroy the ball
- increment score

Optional second mechanic:
- horizontal obstacle approaches
- player must jump to avoid it

Optional third mechanic:
- high obstacle
- player must squat

Keep all objects as flat colors and primitive shapes.

---

## 9. Camera / Coordinate Mapping

The front camera preview is mirrored.

Ensure Vision coordinates are mapped correctly to screen coordinates.

Create one reusable coordinate conversion layer instead of spreading mapping logic throughout the app.

Take care with:
- normalized Vision coordinates
- UIKit / SwiftUI coordinate origin differences
- front camera mirroring
- device orientation

---

## 10. Performance

Target:
- visibly smooth camera preview
- pose/game reaction latency low enough to feel immediate
- no requirement for 60 pose inferences per second

Suggested approach:
- camera preview at normal FPS
- Vision inference at ~15–30 FPS
- game renderer at display refresh rate where practical
- drop stale pose frames instead of queueing them

Never allow inference requests to accumulate.

---

## 11. Game State

Minimal state:

```
score
currentGesture
leftWristPosition
rightWristPosition
bodyCenter
isJumping
isSquatting
balls[]
```

No persistence, accounts, backend, analytics, networking, or database.

---

## 12. UI

Single-screen app.

Suggested layout:

```
┌──────────────────────────────┐
│ Score: 12       LEFT PUNCH   │
│                              │
│       ○        ●             │
│                              │
│          skeleton            │
│                              │
│   ●                     ○    │
│                              │
└──────────────────────────────┘
```

Debug controls can include:
- show/hide skeleton
- sensitivity slider
- reset score

---

## 13. Out of Scope

Do not implement in POC:
- avatars
- textures
- 3D characters
- multiplayer
- networking
- cloud backend
- accounts
- AR world anchoring
- ML Kit
- YOLO
- custom Core ML models
- polished graphics
- App Store deployment
- analytics
- sound design

---

## 14. Acceptance Criteria

POC is successful when:

1. app runs natively on the target iPad
2. front camera preview works
3. body skeleton follows one person in real time
4. left and right punches can be detected independently
5. jump can be detected reliably
6. squat can be detected reliably
7. colored balls can be hit using hand movement
8. score updates correctly
9. latency feels interactive
10. no external ML/runtime dependencies are required

---

## 15. Implementation Order

### Phase 1
Create Xcode app and camera preview.

### Phase 2
Add Vision body pose estimation.

### Phase 3
Draw skeleton overlay.

### Phase 4
Implement gesture events:
- leftPunch
- rightPunch
- jump
- squat

### Phase 5
Add primitive game loop with moving balls.

### Phase 6
Tune thresholds on the physical iPad.

---

## 16. Coding Assistant Instructions

Prioritize KISS.

Rules:
- keep files small
- prefer native Apple APIs
- avoid abstractions until they are needed
- avoid third-party dependencies
- keep gesture thresholds configurable
- make every milestone runnable on a physical iPad
- implement one working vertical slice before adding features
- do not spend time on visual polish

First task:

> Create the minimal native iPad app that opens the front camera, runs Vision human-body pose detection, and draws a live skeleton overlay. Do not implement the game until this works reliably on the physical device.
