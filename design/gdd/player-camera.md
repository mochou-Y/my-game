# Player Camera

> **Status**: Approved
> **Author**: user + agents
> **Last Updated**: 2026-04-17
> **Implements Pillar**: Pillar 1 — Muscle Memory as the Enemy

## Overview

The Player Camera system is the player's eyes in the world — a first-person camera that translates look input (mouse or gamepad) into camera rotation. It receives processed look deltas from the Input Handling system and applies them to the camera's orientation, maintaining smooth, frame-rate-independent rotation. The camera follows the player's movement through the scene (position is driven by the player character, not the camera system itself) and provides configuration for field of view and Y-axis inversion.

**Player impact**: Players experience this system as the direct connection between their physical input and their view into the mirror world. When they move the mouse right, they expect the camera to turn right — and it does. The camera system's job is to make looking feel invisible: responsive enough that players never think about lag, smooth enough that they never feel jerkiness, and configurable enough that players who need inverted controls or different sensitivity can make it work for them. If the player ever notices the camera itself rather than what they're looking at, something has gone wrong.

**Core responsibilities**:

- Apply look deltas to camera rotation (yaw and pitch)
- Maintain player-forward movement direction relative to camera facing
- Provide FOV configuration (vertical FOV, Godot standard)
- Support Y-axis inversion for players who need it
- Clamp pitch to prevent over-rotation (looking too far up/down)
- Expose camera transform for Hover Detection (raycast origin)

## Player Fantasy

In a world where every object fights the player's instincts, the camera is their one reliable ally. It responds to input without resistance, without inversion, without surprise. This consistency does more than feel good—it teaches. By holding the camera steady while objects twist, we create a clear boundary: "This is me. That is the puzzle." The player learns through contrast, not instruction.

**Emotional target**: Relief amidst disorientation. The player feels that at least their view into the world is under their control. When they frantically search a room for a solution, the camera responds instantly—no lag, no drift, no fighting their own eyes. It's the one system that says "you're not broken, the world is."

**Player moment**: The player has just failed to open a door the "normal" way for the third time. They step back, frustrated, and look around the room for another approach. The camera swivels smoothly, responsively—the one thing that works. In that moment, the contrast clarifies: "The door is the puzzle. My eyes are still mine."

**Pillar alignment**:
- **Serves Pillar 1 (Muscle Memory as the Enemy)**: By keeping the camera "normal," we create contrast. The player feels the inversion more acutely because looking around works perfectly—the dissonance is isolated to objects, not the entire interface.
- **Serves Pillar 4 (Silent Education)**: We never tell the player "the camera works normally, objects are the problem." They feel it. The consistency teaches through contrast.

**Reference touchstones**: *Superliminal*'s camera is invisible and reliable—the focus is on perception puzzles, not camera fighting. *The Witness* similarly uses an unremarkable first-person camera that never calls attention to itself. Both games understand: the camera is the player's window, not a puzzle itself.

## Detailed Design

### Core Rules

**Rule 1: Look Input Processing**

Each frame, the camera applies look deltas directly to rotation with no smoothing or interpolation:

1. Query `InputHandling.get_look_delta()` to receive accumulated mouse/stick motion **already scaled by sensitivity and inverted if enabled**
2. Apply directly to camera rotation (yaw for X delta, pitch for Y delta)
3. No smoothing buffer, no interpolation — input is 1:1 with rotation

**Note:** Sensitivity and Y-axis inversion are applied by Input Handling, not by Player Camera. This prevents double-application bugs.

**Rule 2: Pitch Clamping**

Camera pitch (vertical rotation) is clamped to prevent over-rotation:

- **Upper limit**: +85 degrees (looking nearly straight up)
- **Lower limit**: -85 degrees (looking nearly straight down)
- **Soft stop**: Starting at 75 degrees, pitch sensitivity is reduced (multiplied by 0.5 at 75°, 0.25 at 80°)
- **Hard stop**: At 85 degrees, no further pitch rotation is applied

The soft stop prevents jarring "hitting a wall" feel at the limits.

**Rule 3: Yaw Handling**

Camera yaw (horizontal rotation) has no limits — the player can rotate 360° freely:

- Yaw is applied directly from look_delta.x
- No smoothing, no acceleration curve
- Mouse sensitivity (0.1–2.0) and gamepad sensitivity (0.1–2.0) are applied by Input Handling

**Rule 4: Frame-Rate Independence**

Camera rotation is frame-rate independent:

- Look deltas from Input Handling are already scaled by delta time
- Camera applies the delta directly — no additional delta multiplication needed
- This ensures the same mouse movement produces the same rotation at any frame rate

**Rule 5: Movement Direction**

Player movement is relative to camera facing:

- Input Handling provides `get_move_vector()` — a normalized Vector2 of movement input
- The camera's forward vector (transform.basis.z) defines "forward" for movement
- Movement code (outside this system) uses camera basis to translate input to world direction

**Rule 6: FOV Configuration**

Field of view is configurable:

- Range: 60° to 110° (vertical FOV, Godot standard)
- Default: 85°
- Applied directly to Camera3D.fov property
- Takes effect immediately when changed — no fade or transition

**Rule 7: Camera States**

The camera has explicit states matching the Input Handling system:

| State | Look Input | Position Updates | Description |
|-------|------------|------------------|-------------|
| **Gameplay** | Active | Active | Normal gameplay |
| **Paused** | Frozen | Frozen | Pause menu open |
| **Cutscene** | Script-controlled | Script-controlled | Cinematic moments |
| **Locked** | Frozen | Frozen | Scripted moments (rare) |

State transitions are triggered by Input Handling state changes or script commands.

**Rule 8: Camera Transform Exposure**

The camera exposes its transform for dependent systems:

- `get_camera_position() -> Vector3` — world position of the camera
- `get_camera_forward() -> Vector3` — normalized forward vector (basis.z)
- `get_camera_transform() -> Transform3D` — full transform for raycast calculations

Hover Detection uses these for raycast origin and direction.

---

### States and Transitions

```
                    ┌──────────────┐
                    │   Gameplay   │←─────────────────┐
                    └──────────────┘                  │
                          │   ↑                       │
          pause action    │   │    unpause            │
                          ↓   │                       │
                    ┌──────────────┐                  │
                    │    Paused    │──────────────────┘
                    └──────────────┘
                          ↑   │
              cutscene    │   │  cutscene ends
              starts      │   ↓
                    ┌──────────────┐
                    │   Cutscene   │
                    └──────────────┘
                          ↑   │
              script      │   │  script
              locks       │   ↓  unlocks
                    ┌──────────────┐
                    │    Locked    │
                    └──────────────┘
```

| Transition | Trigger | Behavior |
|------------|---------|----------|
| Gameplay → Paused | Pause action pressed | Freeze camera at current rotation |
| Paused → Gameplay | Unpause action or menu selection | Resume camera from frozen state |
| Gameplay → Cutscene | Script triggers cutscene | Camera control transferred to script |
| Cutscene → Gameplay | Cutscene ends or is skipped | Camera control returned to player |
| Any → Locked | Script locks input | Camera frozen; no input processed |
| Locked → Gameplay | Script unlocks input | Camera control returned to player |

---

### Interactions with Other Systems

**Input Handling** (upstream, designed)

- **Queries**: `get_look_delta() -> Vector2` — processed look delta for this frame (already scaled by sensitivity and inverted if enabled)
- **Signals**: Connects to input state changes (gameplay/paused/locked) to sync camera state
- **Contract**: Look delta is reset each frame after being queried; call once per frame. Input Handling owns sensitivity and inversion — Player Camera applies values directly.

**Hover Detection** (downstream, not designed)

- **Queries**: `get_camera_position() -> Vector3` — raycast origin for hover checks
- **Queries**: `get_camera_forward() -> Vector3` — raycast direction
- **Queries**: `get_camera_transform() -> Transform3D` — full transform if needed
- **Contract**: Hover Detection queries this every frame to determine what object the player is looking at

**Player Movement** (parallel, not designed)

- **Provides**: Camera forward vector for movement direction
- **Contract**: Movement system queries camera basis to translate movement input to world space
- **Note**: Camera does not control movement; it only provides the facing direction

**Visual Feedback** (downstream, not designed)

- **Queries**: Camera FOV for potential FOV-based effects (e.g., zoom on focus)
- **Contract**: Visual Feedback reads FOV; does not modify it directly

**Room Manager** (downstream, not designed)

- **Commands**: `set_camera_state(state: StringName)` — transition camera state
- **Commands**: `set_camera_transform(transform: Transform3D)` — for room transitions (fade handled separately)
- **Contract**: Room Manager controls camera during room load; camera resumes player control after fade-in

## Formulas

### Camera Rotation Delta Formula

The `raw_rotation_delta` formula applies look deltas received from Input Handling:

`raw_rotation_delta = (yaw_delta, pitch_delta)`

Where:
- `yaw_delta = look_delta.x`
- `pitch_delta = look_delta.y`

**Variables:**

| Variable | Symbol | Type | Range | Description |
|----------|--------|------|-------|-------------|
| look_delta | ΔL | Vector2 | [-100,000, +100,000] | Processed input motion from `get_look_delta()`. Already scaled by sensitivity and inverted by Input Handling. x = horizontal (yaw), y = vertical (pitch) |
| raw_rotation_delta | ΔR | Vector2 | [-100,000, +100,000] | Rotation deltas in degrees. x = yaw, y = pitch |

**Output Range:** Bounded by look_delta clamping from Input Handling.

**Note on sensitivity ownership:** Sensitivity and Y-axis inversion are owned by Input Handling. Player Camera receives `look_delta` already scaled and inverted. This prevents double-application bugs.

**Example:**
```
Input: look_delta = (2.0, -1.44) from Input Handling (already scaled at sensitivity 0.8)
yaw_delta = 2.0°
pitch_delta = -1.44° (looking down)
raw_rotation_delta = (2.0°, -1.44°)
```

---

### Pitch Soft Stop Formula

The `soft_stop_multiplier` formula reduces sensitivity within the soft stop zone **only when moving toward the limit**:

```
if is_moving_toward_limit(pitch_current, pitch_delta):
    M_soft = lerp(1.0, 0.0, (|pitch_current| - θ_soft) / (θ_max - θ_soft))
else:
    M_soft = 1.0
```

Where `is_moving_toward_limit(pitch_current, pitch_delta)` is true when:
- `pitch_current > 0 AND pitch_delta > 0` (positive pitch, looking further up)
- `pitch_current < 0 AND pitch_delta < 0` (negative pitch, looking further down)

**Variables:**

| Variable | Symbol | Type | Range | Description |
|----------|--------|------|-------|-------------|
| pitch_current | θ<sub>cur</sub> | float | -85° to +85° | Current pitch angle in degrees |
| pitch_delta | Δθ | float | [-100,000, +100,000] | Raw pitch rotation from rotation delta formula, in degrees |
| soft_stop_start | θ<sub>soft</sub> | float | 75° | Angle where soft stop begins (constant) |
| pitch_limit | θ<sub>max</sub> | float | 85° | Hard pitch limit in degrees (constant) |
| soft_stop_multiplier | M<sub>soft</sub> | float | 0.0–1.0 | Output: sensitivity multiplier for pitch |

**Precondition:** `soft_stop_start < pitch_limit` — if violated, formula behavior is undefined.

**Output Range:**

- 1.0 when |θ<sub>cur</sub>| ≤ 75°
- 1.0 when moving **away** from limit (exiting soft stop zone)
- 0.5 at |θ<sub>cur</sub>| = 80° when moving toward limit
- Approaches 0.0 as |θ<sub>cur</sub>| approaches 85° when moving toward limit

**Example:**
```
Input: pitch_current = 78°, pitch_delta = +5° (moving toward +85° limit)
is_moving_toward_limit = true (78 > 0 AND +5 > 0)
t = (78 - 75) / 10 = 0.3
M_soft = lerp(1.0, 0.0, 0.3) = 0.7

Input: pitch_current = 78°, pitch_delta = -5° (moving away from limit toward center)
is_moving_toward_limit = false (78 > 0 BUT -5 < 0)
M_soft = 1.0 (full sensitivity when exiting soft stop)

Input: pitch_current = 60°
|θ_cur| < 75°, so M_soft = 1.0 (not in soft stop zone)
```

---

### Final Pitch Application Formula

The `pitch_final` formula applies soft stop and clamping:

`pitch_final = clamp(pitch_current + pitch_delta * M_soft, -θ_max, θ_max)`

**Variables:**

| Variable | Symbol | Type | Range | Description |
|----------|--------|------|-------|-------------|
| pitch_current | θ<sub>cur</sub> | float | -85° to +85° | Current pitch angle before this frame |
| pitch_delta | Δθ | float | (unbounded) | Raw pitch rotation from rotation delta formula, in degrees |
| soft_stop_multiplier | M<sub>soft</sub> | float | 0.0–1.0 | From soft stop formula |
| pitch_limit | θ<sub>max</sub> | float | 85° | Hard pitch limit (constant) |
| pitch_final | θ<sub>final</sub> | float | -85° to +85° | Output: new pitch angle after this frame |

**Output Range:** Strictly clamped to [-85°, +85°]. Player cannot exceed limits.

**Example:**
```
Input: pitch_current = 84°, pitch_delta = 15.0°, M_soft = 0.1
pitch_new = 84 + 15.0 * 0.1 = 85.5°
pitch_final = clamp(85.5, -85, 85) = 85° (hard stop)
```

---

### Yaw Rotation Formula

The `yaw_final` formula applies horizontal rotation:

`yaw_final = yaw_current + yaw_delta`

**Variables:**

| Variable | Symbol | Type | Range | Description |
|----------|--------|------|-------|-------------|
| yaw_current | ψ<sub>cur</sub> | float | (unbounded) | Current yaw angle in degrees |
| yaw_delta | Δψ | float | (unbounded) | Raw yaw rotation from rotation delta formula, in degrees |
| yaw_final | ψ<sub>final</sub> | float | (unbounded) | Output: new yaw angle after this frame |

**Output Range:** Unbounded. Full 360° rotation supported.

**Example:**
```
Input: yaw_current = 170°, yaw_delta = 25°
yaw_final = 170 + 25 = 195°
```

## Edge Cases

### Pitch Boundary Values

- **If pitch_current is exactly 75°**: `M_soft = 1.0`. The soft stop formula uses lerp with t = 0, so sensitivity is unchanged. The soft stop zone is defined as "starting at 75°," meaning 75° itself is still full sensitivity — the reduction applies to angles *greater than* 75°.

- **If pitch_current is exactly 80°**: `M_soft = 0.5`. The lerp t = (80-75)/(85-75) = 0.5, giving half sensitivity. This is the midpoint of the soft stop zone.

- **If pitch_current is exactly 85°**: `M_soft = 0.0` and no further positive pitch delta is applied. The lerp t = (85-75)/(85-75) = 1.0, giving zero sensitivity. Negative pitch delta (looking back down) is still applied normally since the player is moving *away* from the limit.

### Very Fast Mouse Movement

- **If look_delta.x or look_delta.y exceeds 1000 pixels**: Apply the full delta normally. No clamping, no smoothing. The player can flick the mouse rapidly and the camera responds proportionally. This is intentional — 1:1 input response means large motions produce large rotations.

- **If look_delta.y causes pitch_delta that would exceed 85° by a large margin**: Calculate M_soft using pitch_current (before delta), apply reduced delta, then clamp to 85°. Player never exceeds the limit regardless of input magnitude.

### Sensitivity Boundary Values

- **If sensitivity is 0.1 (minimum)**: Rotation deltas are multiplied by 0.1, producing slow camera movement. One pixel of mouse motion ≈ 0.1° of rotation.

- **If sensitivity is 2.0 (maximum)**: Rotation deltas are doubled. One pixel of mouse motion ≈ 2.0° of rotation.

### State Transition During Active Camera Rotation

- **If Gameplay → Paused occurs mid-frame**: Freeze immediately. Do not apply the pending pitch_delta or yaw_delta. The camera rotation stays at values from the previous frame. Pause should freeze game state instantly.

- **If Paused → Gameplay occurs**: Resume from the frozen rotation. `get_look_delta()` returns (0, 0) on the first frame back since no input accumulated during pause.

- **If Any → Cutscene occurs**: Discard any queued look_delta. Camera rotation is set by the cutscene script. The player's last input direction is not preserved.

- **If Cutscene → Gameplay occurs**: Camera starts from whatever rotation the cutscene ended on. Normal input resumes immediately.

### Device Switching Mid-Rotation

- **If player moves mouse while holding gamepad stick, then releases stick**: Both inputs are accumulated into the same `look_delta` by Input Handling. No device conflict — input is aggregated.

- **If player switches from mouse to gamepad or vice versa mid-turn**: No special handling needed. `get_look_delta()` returns accumulated motion from whichever device(s) provided input this frame.

### FOV Boundary Values

- **If FOV is set to 60° (minimum)**: Narrow field of view, "zoomed in" effect. Acceptable for players who prefer focused view or experience motion sickness at wider FOVs.

- **If FOV is set to 110° (maximum)**: Wide field of view, peripheral awareness. Acceptable for players who prefer situational awareness.

- **If FOV is changed during gameplay**: Applied immediately to Camera3D.fov. No interpolation — the view snaps to the new FOV on the next frame.

### Pause During Pitch Soft Stop Zone

- **If camera is paused while pitch_current is in the soft stop zone**: Rotation freezes at current angle. When gameplay resumes, M_soft is recalculated using the frozen pitch_current — soft stop continues from where it left off.

### Yaw Accumulation

- **If yaw_current exceeds 3600° (10 full rotations) or drops below -3600°**: No special handling. Yaw is unbounded. Godot's rotation quaternions handle large angles correctly.

- **If yaw_current reaches extreme values (±100,000°)**: Godot's quaternions do not suffer from gimbal lock. Precision loss only matters if code explicitly reads yaw_current for comparison.

### Invalid or Extreme look_delta Values

- **If look_delta.x or look_delta.y is NaN**: Input Handling should have already handled this. If received, return early without applying rotation. Log a warning: "Invalid look_delta (NaN) received — frame skipped."

- **If look_delta.x or look_delta.y exceeds ±100,000**: Input Handling should have already clamped this. If received, apply directly (trust Input Handling's preprocessing).

- **If look_delta contains INF or -INF**: Input Handling should have already handled this. If received, return early and log a warning.

### Hover Detection Query During Paused State

- **If Hover Detection calls `get_camera_position()` or `get_camera_forward()` while camera is Paused**: Return the frozen camera position and forward vector. These are valid even during pause — the player is still looking at something.

- **If Hover Detection calls `get_camera_transform()` while camera is Paused**: Return the frozen Transform3D. The transform remains valid during pause.

## Dependencies

### Upstream Dependencies (this system depends on)

**Input Handling** (designed)

- **Hard dependency** — Camera cannot function without look input
- **Interface**: `get_look_delta() -> Vector2` — returns processed look delta (already scaled by sensitivity and inverted if enabled)
- **Signals**: Input state changes (gameplay/paused/locked) to sync camera state
- **Contract**: Look delta is reset each frame after being queried; call once per frame. Values are pre-processed (sensitivity applied, inversion applied, NaN/extreme values handled).

**Engine dependencies:**

- **Godot Camera3D** — Node for first-person camera
- **Godot Node3D** — Rotation properties (rotation_degrees)
- **Godot @export** — For exposing configuration in editor

---

### Downstream Dependencies (systems that depend on this one)

| System | Dependency Type | Interface Used | Status |
|--------|-----------------|----------------|--------|
| **Hover Detection** | Hard | `get_camera_position()`, `get_camera_forward()`, `get_camera_transform()` | Not designed |

**Hard dependency** = system cannot function without this interface.

---

### Data Flow Diagram

```
                Input Handling
                     ↓
              get_look_delta()
                     ↓
            ┌────────────────┐
            │ Player Camera  │
            │  - yaw: float  │
            │  - pitch: float│
            │  - fov: float  │
            └────────────────┘
                     ↓
         get_camera_position()
         get_camera_forward()
         get_camera_transform()
                     ↓
             Hover Detection
```

---

### Contracts That Downstream Systems Must Respect

1. **Call `get_camera_*` methods once per frame** — multiple calls return the same cached value
2. **Do not modify camera rotation directly** — always through the Player Camera system
3. **Camera transform is read-only** — downstream systems observe but do not modify

## Tuning Knobs

| Knob | Type | Default | Range | What It Affects |
|------|------|---------|-------|-----------------|
| **fov** | float | 85 | 60 – 110 | Vertical field of view in degrees. Higher = wider view, more peripheral awareness. Lower = narrower view, "zoomed in" feel. |
| **pitch_limit** | float | 85 | 60 – 89 | Maximum pitch angle in degrees. Prevents camera from flipping upside down. Lower values restrict how far the player can look up/down. |
| **soft_stop_start** | float | 75 | 0 – pitch_limit | Angle in degrees where pitch sensitivity begins to reduce. Must be less than pitch_limit. |
| **enable_logging** | bool | true | true/false | Whether to log camera state changes and edge case warnings for debugging. Disable in release builds. |

---

### Knob Interactions

| If you change... | It may affect... | Why |
|------------------|------------------|-----|
| `fov` to very low (< 70) | Motion sickness risk | Narrow FOV can cause discomfort in some players |
| `fov` to very high (> 100) | Performance, motion sickness | Wider FOV renders more; some players experience nausea |
| `pitch_limit` to very low (< 60) | Player agency | Players cannot look far up/down, may miss puzzle elements |
| `soft_stop_start` close to `pitch_limit` | Soft stop effectiveness | Less room for gradual slowdown; feels more like hard stop |

---

### What Breaks at Extremes

| Knob | Too Low | Too High |
|------|---------|----------|
| `fov` | Tunnel vision; motion sickness for some | Fish-eye effect; motion sickness for others |
| `pitch_limit` | Player cannot look up/down enough | Camera can flip (if > 89°); disorientation |
| `soft_stop_start` | Always in soft stop; camera feels sluggish | No soft stop; jarring hard stop at limit |

---

### Notes

- **Sensitivity** and **invert_y** are owned by Input Handling, not this system. Input Handling applies these settings before passing `look_delta` to Player Camera. This prevents double-application bugs.
- **Camera position** is not a tuning knob — it is driven by player character position, not the camera system itself.

## Visual/Audio Requirements

[To be designed]

## UI Requirements

[To be designed]

## Acceptance Criteria

### Camera Rotation — Yaw

1. **GIVEN** the camera is in Gameplay state with yaw at 0°, **WHEN** Input Handling returns `get_look_delta() = (50.0, 0.0)` (already scaled by sensitivity), **THEN** the camera yaw becomes exactly 50° after the frame processes.

### Camera Rotation — Pitch

2. **GIVEN** the camera is in Gameplay state with pitch at 0°, **WHEN** Input Handling returns `get_look_delta() = (0.0, 25.0)` (already scaled by sensitivity, no inversion), **THEN** the camera pitch becomes exactly 25° after the frame processes.

### Pitch Clamping — Hard Limits

3. **GIVEN** the camera pitch is at 80°, **WHEN** the player inputs a pitch increase of +10°, **THEN** the camera pitch becomes exactly 85° (clamped, not 90°).

4. **GIVEN** the camera pitch is at −80°, **WHEN** the player inputs a pitch decrease of −10°, **THEN** the camera pitch becomes exactly −85° (clamped, not −90°).

### Pitch Clamping — Soft Stop

5. **GIVEN** the camera pitch is at 70°, **WHEN** Input Handling returns `get_look_delta() = (0.0, 5.0)`, **THEN** the camera pitch becomes exactly 75° with no soft stop applied (M_soft = 1.0 because |70°| < 75°).

6. **GIVEN** the camera pitch is at 78°, **WHEN** Input Handling returns `get_look_delta() = (0.0, 10.0)`, **THEN** the camera pitch becomes exactly 85° (M_soft = 0.7 at 78°, reducing the 10° delta to 7°, and the remaining movement applies M_soft progressively until reaching 85°).

7. **GIVEN** the camera pitch is at exactly 80°, **WHEN** the formula `M_soft = lerp(1.0, 0.0, (|80| - 75) / 10)` is evaluated with pitch_delta moving toward limit, **THEN** M_soft equals exactly 0.5, meaning input is reduced by half.

8. **GIVEN** the camera pitch is at 80°, **WHEN** Input Handling returns `get_look_delta() = (0.0, -10.0)` (moving away from limit toward center), **THEN** M_soft equals 1.0 (full sensitivity when exiting soft stop zone).

### Y-Axis Inversion

9. **GIVEN** invert_y is disabled in Input Handling, **WHEN** the player moves the mouse/stick upward (negative Y in screen coordinates for mouse, positive Y for gamepad), **THEN** Input Handling returns positive pitch_delta and the camera pitch increases (looks up).

10. **GIVEN** invert_y is enabled in Input Handling, **WHEN** the player moves the mouse/stick upward, **THEN** Input Handling returns negative pitch_delta and the camera pitch decreases (looks down).

### FOV Configuration

11. **GIVEN** the game launches with default settings, **WHEN** the camera initializes, **THEN** the field of view is exactly 85°.

12. **GIVEN** the player attempts to set FOV below the minimum, **WHEN** FOV is set to 50°, **THEN** the field of view is clamped to exactly 60°.

13. **GIVEN** the player attempts to set FOV above the maximum, **WHEN** FOV is set to 120°, **THEN** the field of view is clamped to exactly 110°.

### Camera States

14. **GIVEN** the camera is in Gameplay state, **WHEN** the player provides look input, **THEN** the camera rotates according to input.

15. **GIVEN** the camera is in Paused state, **WHEN** the player provides look input, **THEN** the camera does not rotate.

16. **GIVEN** the camera is in Cutscene state, **WHEN** the player provides look input, **THEN** the camera does not rotate.

17. **GIVEN** the camera is in Locked state, **WHEN** the player provides look input, **THEN** the camera does not rotate.

18. **GIVEN** the camera is in Paused state with pitch at 30° and yaw at 90°, **WHEN** the camera transitions to Gameplay state, **THEN** the camera retains pitch 30° and yaw 90° (no reset on state change).

### Camera Transform Interfaces

19. **GIVEN** the camera is positioned at world coordinates (100, 50, 200), **WHEN** get_camera_position() is called, **THEN** the function returns Vector3(100, 50, 200).

20. **GIVEN** the camera has yaw 0° and pitch 0° (facing -Z in Godot's coordinate system), **WHEN** get_camera_forward() is called, **THEN** the function returns a normalized Vector3 approximately (0, 0, -1) within tolerance of 0.001.

21. **GIVEN** the camera has yaw 0° and pitch 45° (looking up), **WHEN** get_camera_forward() is called, **THEN** the function returns a normalized Vector3 with a positive Y component and negative Z component, magnitude exactly 1.0 within tolerance of 0.001.

22. **GIVEN** the camera exists in the scene, **WHEN** get_camera_transform() is called, **THEN** the function returns a Transform3D where origin equals camera world position and basis columns have magnitude 1.0 ± 0.001 and are orthogonal.

### Edge Cases

23. **GIVEN** the camera pitch is at 0°, **WHEN** Input Handling returns `get_look_delta() = (0.0, 45.0)`, **THEN** the camera pitch becomes exactly 45° after the frame (no interpolation over multiple frames).

24. **GIVEN** the camera pitch is at exactly 85°, **WHEN** Input Handling returns `get_look_delta() = (0.0, 10.0)`, **THEN** the camera pitch remains at 85° (hard clamp prevents further movement).

25. **GIVEN** the camera yaw is at 350°, **WHEN** Input Handling returns `get_look_delta() = (20.0, 0.0)`, **THEN** the camera yaw becomes 370° (unlimited, no wrapping to 10°).

26. **GIVEN** the camera is in Gameplay state at any orientation, **WHEN** the player provides no input for 60 seconds, **THEN** the camera orientation remains unchanged (no drift, no auto-centering).

### Formula Verification

27. **GIVEN** Input Handling returns `get_look_delta() = (60.0, 30.0)` (already scaled), **WHEN** raw_rotation_delta is calculated, **THEN** raw_rotation_delta = (60°, 30°).

28. **GIVEN** pitch_current = 50°, pitch_delta = +5°, M_soft = 1.0 (not in soft stop zone), **WHEN** pitch_final is calculated, **THEN** pitch_final = 55°.

29. **GIVEN** pitch_current = 78°, pitch_delta = +5°, M_soft = 0.7 (in soft stop zone, moving toward limit), **WHEN** pitch_final is calculated, **THEN** pitch_final = 81.5°.
