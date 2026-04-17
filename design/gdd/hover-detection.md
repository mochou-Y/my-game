# Hover Detection

> **Status**: Complete
> **Author**: [user + agents]
> **Last Updated**: 2026-04-17
> **Implements Pillar**: Muscle Memory as the Enemy, Silent Education

## Overview

**Hover Detection** is the player's gaze translation layer — it converts camera orientation into actionable interaction data by casting a ray from the player's viewpoint into the scene every frame. When the ray intersects a valid interactive object, the system publishes the target to downstream consumers (Visual Feedback, Interaction System), enabling the core "look, see, act" loop.

Technically, the system performs a physics raycast each frame using `PhysicsDirectSpaceState3D.intersect_ray()`, filtered to the "interactive" collision layer. It exposes a clean interface — `get_hovered_object() -> Node3D` and `is_hovering() -> bool` — that hides the implementation complexity from consuming systems.

From the player's perspective, this system is invisible until it works: looking at an object creates an immediate visual response (highlight, cursor change), establishing the causal link between gaze and potential action. In a game built on subverted muscle memory, this visual feedback must be instantaneous and unambiguous — any lag or flicker breaks the trust that "looking" equals "targeting."

The system exists because every interaction in Mirror World begins with looking. Without hover detection, players cannot know which objects are interactive or whether their inversion attempt will affect the intended target. It is the silent foundation of the game's core loop.

## Player Fantasy

**The Silent Guide** — Teaching Through Presence

Hover Detection embodies Silent Education in its purest form. In a game without tutorials or explicit instruction, the hover cursor is the first teacher players encounter. It says, quietly and without words: *"This object matters. You can touch this."*

The player fantasy is **clarity through attention** — the feeling that the world responds to their gaze, that looking is the first step toward understanding. When the player scans a room and the cursor flickers across objects, each flash is a whispered hint: *here is a puzzle piece, here is something to investigate.*

In Mirror World, where every interaction subverts expectation, the hover cursor must be the one trustworthy element. It is not the source of surprise — it is the anchor. The fantasy is not about the cursor itself, but about what it enables: the confidence to try, the safety to experiment, the knowledge that the game will always tell you when you're looking at something that matters.

**Anchoring moment**: A player enters a new room, scans the environment, and watches the highlight dance across a lamp, a book, a light switch. Each flicker is a quiet invitation. The player feels guided without being told what to do.

## Detailed Design

### Core Rules

**Rule 1: Raycast Execution**

Each frame while in Gameplay state, the system performs a physics raycast:

1. Query `PlayerCamera.get_camera_position()` for raycast origin
2. Query `PlayerCamera.get_camera_forward()` for raycast direction
3. Construct `PhysicsRayQueryParameters3D` with:
   - `from = camera_position`
   - `to = camera_position + camera_forward * max_hover_distance`
   - `collision_mask = hover_collision_mask` (layer for interactive objects)
4. Call `PhysicsDirectSpaceState3D.intersect_ray(parameters)`
5. Process result to update hovered object state

**Rule 2: Collision Layer Configuration**

- Interactive objects MUST be assigned to a dedicated collision layer (default: layer 4)
- The `hover_collision_mask` targets ONLY this layer
- Non-interactive objects (walls, floors, props) are NOT on this layer and are ignored by hover detection
- Multiple interactive objects in a line: only the nearest is considered hovered

**Rule 3: Hovered Object Tracking**

The system maintains hover state across frames:

- `current_hovered_object: Node3D` — object being looked at this frame, or null
- `previous_hovered_object: Node3D` — object looked at last frame, or null
- `hover_entered: bool` — true only on the frame an object is first looked at
- `hover_exited: bool` — true only on the frame the player stops looking at an object
- `hover_changed: bool` — true whenever current differs from previous (enter or exit)

**Rule 4: Object Validity Check**

When a raycast hits an object:

1. Verify the collision result contains a valid collider
2. Verify the collider is a `CollisionObject3D` or subclass
3. If the object has been freed or is invalid, treat as no hit
4. Update `current_hovered_object` with the valid collider

**Rule 5: Hover State Comparison**

At the end of each frame, compare current and previous:

```
if current_hovered_object != previous_hovered_object:
    hover_changed = true
    if current_hovered_object != null:
        hover_entered = true
        hover_exited = false
    else:
        hover_entered = false
        hover_exited = true
else:
    hover_changed = false
    hover_entered = false
    hover_exited = false

previous_hovered_object = current_hovered_object
```

**Rule 6: Distance Limit**

Hover detection has a maximum range:

- `max_hover_distance: float` — default 10.0 meters
- Objects beyond this distance are not detected even if on the correct layer
- The raycast endpoint is `camera_position + camera_forward * max_hover_distance`

**Rule 7: State-Dependent Behavior**

| Hover System State | Raycast Active | Output |
|--------------------|----------------|--------|
| **Enabled** | Yes | Normal operation |
| **Disabled** | No | All queries return null/false |

The system state syncs with Input Handling state:
- Input state = Gameplay → Hover system = Enabled
- Input state = Paused/Cutscene/Locked → Hover system = Disabled

**Rule 8: Interface Contracts**

The system exposes these methods:

- `get_hovered_object() -> Node3D` — returns current hovered object or null
- `is_hovering() -> bool` — returns true if an object is currently hovered
- `is_hover_entered() -> bool` — returns true only on first frame of hover
- `is_hover_exited() -> bool` — returns true only on frame hover ends
- `is_hover_changed() -> bool` — returns true if hover state changed this frame

**Rule 9: Frame Timing**

All queries return values from the current frame's raycast:

- Raycast executes in `_physics_process()` for physics synchronization
- Queries can be called from any process but return the most recent physics frame result
- `hover_entered` and `hover_exited` are true for exactly one frame each

### States and Transitions

| State | Raycast Active | Hovered Object | Use Case |
|-------|----------------|----------------|----------|
| **Enabled** | Yes | Updated each frame | Normal gameplay |
| **Disabled** | No | Always null | Pause menu, cutscenes, locked input |

**Transitions:**

| From | To | Trigger | Behavior |
|------|----|---------|----------|
| Enabled | Disabled | Input state changes to Paused/Cutscene/Locked | Clear current hovered object; emit hover_exited if needed |
| Disabled | Enabled | Input state changes to Gameplay | Resume raycast; establish fresh hover state |

**Transition Edge Cases:**

- **Enabled → Disabled with active hover**: Emit `hover_exited` signal immediately. Downstream systems must handle the exit even though the player didn't look away naturally.
- **Disabled → Enabled**: First frame may have `hover_entered = true` if the player was looking at an interactive object when the system re-enabled.

### Interactions with Other Systems

**Player Camera** (upstream, approved)
- **Queries**: `get_camera_position() -> Vector3`, `get_camera_forward() -> Vector3`
- **Contract**: Call once per frame in `_physics_process()`. Camera transform is guaranteed valid during Gameplay state.

**Input Handling** (upstream, designed)
- **Signals**: Input state changed (gameplay/paused/cutscene/locked)
- **Contract**: Hover Detection syncs its state to Input Handling state automatically.

**Visual Feedback** (downstream, not designed)
- **Queries**: `get_hovered_object()`, `is_hover_entered()`, `is_hover_exited()`
- **Contract**: Visual Feedback consumes hover state to show/hide object highlights. Does NOT modify hover state.

**Interaction System** (downstream, not designed)
- **Queries**: `get_hovered_object()`, `is_hovering()`
- **Contract**: Uses hover target when player presses interact. Does NOT modify hover state.

**Object State** (downstream, designed)
- **Queries**: `get_hovered_object()`
- **Contract**: May query hover to determine interaction eligibility.

## Formulas

**Raycast Endpoint Calculation**

```
raycast_origin = camera_position
raycast_endpoint = camera_position + (camera_forward × max_hover_distance)
```

Where:
- `camera_position`: Vector3 — from `PlayerCamera.get_camera_position()`
- `camera_forward`: Vector3 — normalized direction from `PlayerCamera.get_camera_forward()`
- `max_hover_distance`: float — default 10.0 meters (tuning knob)

**Hover State Change Detection**

```
hover_changed = (current_hovered_object ≠ previous_hovered_object)
hover_entered = hover_changed AND (current_hovered_object ≠ null)
hover_exited = hover_changed AND (previous_hovered_object ≠ null)
```

**Expected Value Ranges**

| Variable | Type | Range | Description |
|----------|------|-------|-------------|
| `max_hover_distance` | float | 1.0 – 50.0 meters | Maximum raycast distance; default 10.0m |
| `camera_forward` | Vector3 | Unit vector (magnitude = 1.0) | Must be normalized |
| `hover_collision_mask` | int | Bit flag | Layer 4 default (value = 8) |

**Example Calculation**

Given:
- `camera_position = (0, 1.7, 0)` — player eye height 1.7m
- `camera_forward = (0, 0, 1)` — looking straight ahead
- `max_hover_distance = 10.0`

Result:
- `raycast_endpoint = (0, 1.7, 0) + (0, 0, 1) × 10.0 = (0, 1.7, 10.0)`

The raycast travels 10 meters forward from the camera.

## Edge Cases

**EC1: Two Interactive Objects Aligned**

*Situation*: Player looks at two interactive objects in a line (e.g., a lamp in front of a bookshelf).

*Behavior*: Only the nearest object is hovered. The raycast stops at the first hit.

*Rationale*: Prevents ambiguity. Visual Feedback highlights one object, not a stack.

**EC2: Object Deleted While Hovered**

*Situation*: A script deletes or frees the hovered object while the player is looking at it.

*Behavior*: `is_hover_exited()` returns true on the next frame. `get_hovered_object()` returns null. No crash.

*Implementation*: Check `is_instance_valid(hovered_object)` before returning from queries.

**EC3: Object Moves Out of Range**

*Situation*: Hovered object is pushed or falls beyond `max_hover_distance`.

*Behavior*: Hover is lost. `is_hover_exited()` returns true on the frame the object leaves range.

*Rationale*: The raycast endpoint doesn't follow moving objects; they exit the detection volume naturally.

**EC4: Camera Disabled During Hover**

*Situation*: Player Camera transitions to Cutscene or Locked state while an object is hovered.

*Behavior*: Hover system transitions to Disabled state. `is_hover_exited()` returns true. Visual Feedback must hide the highlight.

*Rationale*: Prevents stale hover state when the player loses camera control.

**EC5: Player Teleports (Spawns)**

*Situation*: Player teleports to a new position while looking toward an interactive object.

*Behavior*: First frame at new position recalculates raycast normally. If still looking at an interactive object, `is_hover_entered()` returns true immediately.

*Rationale*: No special handling needed — the raycast is frame-accurate to camera position.

**EC6: Object Behind the Player**

*Situation*: Player rotates 180° quickly; an interactive object is now behind them.

*Behavior*: Hover is lost for the previously looked-at object. No object behind is hovered because the raycast only travels forward.

*Rationale*: Raycast direction is always `camera_forward`, which points where the camera faces.

**EC7: Overlapping Collision Shapes**

*Situation*: Two interactive objects have overlapping collision shapes (e.g., a book on a desk, both interactive).

*Behavior*: Physics engine returns the nearest hit. Only one object is hovered.

*Rationale*: Godot's `intersect_ray` returns the closest collision. This is deterministic.

**EC8: Pause Menu Opens During Hover**

*Situation*: Player pauses the game while hovering an object.

*Behavior*: Input Handling transitions to Paused state. Hover system transitions to Disabled. `is_hover_exited()` returns true. Highlight disappears.

*Rationale*: Pause menu should not show stale hover state.

**EC9: Extremely Close Object**

*Situation*: Interactive object is 0.1m from camera (player's nose).

*Behavior*: Object is hovered normally, assuming it's on the interactive layer.

*Rationale*: Raycast starts at camera position; no minimum distance.

## Dependencies

**Upstream Dependencies**

| System | Status | Interface Used | Purpose |
|--------|--------|----------------|---------|
| **Player Camera** | Approved | `get_camera_position() -> Vector3` | Raycast origin |
| **Player Camera** | Approved | `get_camera_forward() -> Vector3` | Raycast direction |
| **Input Handling** | Designed | Input state signals | Enable/disable hover system |

**Downstream Dependents**

| System | Status | Interface Provided | Purpose |
|--------|--------|-------------------|---------|
| **Visual Feedback** | Not designed | `get_hovered_object()`, `is_hover_entered()`, `is_hover_exited()` | Highlight rendering |
| **Interaction System** | Not designed | `get_hovered_object()`, `is_hovering()` | Target for interact action |
| **Object State** | Designed | `get_hovered_object()` | Query gaze target |

**No Circular Dependencies**

Hover Detection has no circular dependencies. It reads from Player Camera and Input Handling (upstream), publishes to Visual Feedback and Interaction System (downstream).

## Tuning Knobs

| Knob | Type | Default | Safe Range | Affects |
|------|------|---------|------------|---------|
| `max_hover_distance` | float | 10.0m | 1.0m – 50.0m | How far the player can "see" interactive objects. Shorter = more intimate, longer = more exploration. Affects perceived object density. |
| `hover_collision_layer` | int | 4 | 1–32 | Which collision layer marks interactive objects. Must match the layer assigned to interactive object prefabs. Changing this requires re-tagging all interactive objects. |
| `raycast_update_rate` | int | 1 | 1–3 | How many physics frames between raycasts. 1 = every frame (most responsive), 2 = every other frame (slight lag, better perf). Affects cursor "snappiness." |

**Tuning Guidance**

- **max_hover_distance**: For MVP's single room, 10m is generous. Consider reducing to 5m to create a "nearby" feel and force closer examination.
- **hover_collision_layer**: Layer 4 is arbitrary but consistent. Document this in the Object State GDD so prefab creators know which layer to use.
- **raycast_update_rate**: Keep at 1 for MVP. Only increase if profiling shows raycast as a bottleneck (unlikely with 4-5 objects).

## Acceptance Criteria

### Basic Raycast Operation

1. **GIVEN** the Hover Detection system is in Enabled state with max_hover_distance = 10.0m, **WHEN** the camera faces an interactive object at 5.0m distance on collision layer 4, **THEN** `get_hovered_object()` returns that object's Node3D reference.

2. **GIVEN** the Hover Detection system is in Enabled state, **WHEN** the camera faces a non-interactive object (not on layer 4), **THEN** `get_hovered_object()` returns null.

3. **GIVEN** the Hover Detection system is in Enabled state, **WHEN** the camera faces empty space with no objects within 10.0m, **THEN** `is_hovering()` returns false.

4. **GIVEN** the Hover Detection system is in Enabled state with max_hover_distance = 10.0m, **WHEN** the camera faces an interactive object at 12.0m distance (beyond range), **THEN** `get_hovered_object()` returns null.

### Hover State Queries

5. **GIVEN** the camera is looking at an interactive object, **WHEN** `is_hovering()` is called, **THEN** it returns true.

6. **GIVEN** the camera is NOT looking at any interactive object, **WHEN** `is_hovering()` is called, **THEN** it returns false.

7. **GIVEN** the camera was NOT looking at an interactive object on frame N-1, **WHEN** on frame N the camera begins looking at an interactive object, **THEN** `is_hover_entered()` returns true on frame N only.

8. **GIVEN** the camera WAS looking at an interactive object on frame N-1, **WHEN** on frame N the camera looks away from all interactive objects, **THEN** `is_hover_exited()` returns true on frame N only.

9. **GIVEN** the hover state changed on frame N (either entered or exited), **WHEN** `is_hover_changed()` is called on frame N, **THEN** it returns true.

10. **GIVEN** the hover state did NOT change on frame N (same object or no object), **WHEN** `is_hover_changed()` is called on frame N, **THEN** it returns false.

### Single-Frame Signal Behavior

11. **GIVEN** the camera begins looking at an interactive object on frame N, **WHEN** `is_hover_entered()` is called on frames N, N+1, and N+2 (while still looking at the same object), **THEN** `is_hover_entered()` returns true on frame N only, false on N+1 and N+2.

12. **GIVEN** the camera stops looking at an interactive object on frame N, **WHEN** `is_hover_exited()` is called on frames N, N+1, and N+2, **THEN** `is_hover_exited()` returns true on frame N only, false on N+1 and N+2.

13. **GIVEN** the camera looks at Object A on frame N and Object B on frame N+1, **WHEN** `is_hover_entered()` and `is_hover_exited()` are called on frame N+1, **THEN** both return true (exited A, entered B in the same frame).

### State Transitions

14. **GIVEN** the Hover Detection system is in Disabled state, **WHEN** `get_hovered_object()` is called, **THEN** it returns null regardless of what the camera faces.

15. **GIVEN** the Hover Detection system is in Disabled state, **WHEN** `is_hovering()` is called, **THEN** it returns false.

16. **GIVEN** the Hover Detection system is in Disabled state, **WHEN** `is_hover_entered()`, `is_hover_exited()`, or `is_hover_changed()` are called, **THEN** they all return false.

17. **GIVEN** the Hover Detection system transitions from Enabled to Disabled while an object is hovered, **WHEN** the transition occurs, **THEN** `is_hover_exited()` returns true on the transition frame.

18. **GIVEN** the Hover Detection system transitions from Disabled to Enabled while the camera faces an interactive object, **WHEN** the first Enabled frame processes, **THEN** `is_hover_entered()` returns true if an object is immediately detected.

19. **GIVEN** Input Handling state changes to Paused/Cutscene/Locked, **WHEN** the state change signal is received, **THEN** the Hover Detection system transitions to Disabled state within the same frame.

### Edge Cases — Multiple Objects

20. **GIVEN** two interactive objects are aligned in the camera's view (Object A at 3.0m, Object B at 6.0m), **WHEN** the raycast executes, **THEN** `get_hovered_object()` returns Object A only (nearest object).

21. **GIVEN** two interactive objects have overlapping collision shapes, **WHEN** the raycast executes, **THEN** `get_hovered_object()` returns exactly one object (deterministic by physics engine nearest-hit rule).

### Edge Cases — Object Lifecycle

22. **GIVEN** an object is currently hovered, **WHEN** the object is freed/deleted via script on frame N, **THEN** `is_hover_exited()` returns true on frame N+1 and `get_hovered_object()` returns null on frame N+1 without crashing.

23. **GIVEN** an object is currently hovered at 4.0m distance, **WHEN** the object moves beyond max_hover_distance (10.0m) on frame N, **THEN** `is_hover_exited()` returns true on frame N.

24. **GIVEN** an object is currently hovered, **WHEN** the object's collision layer is changed from layer 4 to layer 1 on frame N, **THEN** `is_hover_exited()` returns true on frame N (object no longer detected by hover mask).

### Edge Cases — Camera State

25. **GIVEN** the camera is looking at an interactive object, **WHEN** the Player Camera transitions to Cutscene or Locked state, **THEN** the Hover Detection system transitions to Disabled and `is_hover_exited()` returns true.

26. **GIVEN** the player teleports to a new position while looking toward an interactive object, **WHEN** the first frame at the new position processes, **THEN** the raycast uses the new camera position and `is_hover_entered()` returns true if an interactive object is in view.

27. **GIVEN** the player rotates 180 degrees quickly, **WHEN** the previously hovered object is now behind the camera, **THEN** `is_hover_exited()` returns true on that frame (raycast only travels forward).

### Edge Cases — Pause

28. **GIVEN** an interactive object is currently hovered, **WHEN** the player opens the pause menu (Input Handling → Paused), **THEN** `is_hover_exited()` returns true and `get_hovered_object()` returns null while paused.

29. **GIVEN** the game is paused while looking at an interactive object, **WHEN** the player unpauses (Input Handling → Gameplay), **THEN** `is_hover_entered()` returns true on the first gameplay frame if still looking at the object.

### Edge Cases — Extreme Distances

30. **GIVEN** an interactive object is 0.1m from the camera (extremely close), **WHEN** the camera faces the object, **THEN** the object is hovered normally (no minimum distance enforced).

31. **GIVEN** max_hover_distance is set to 1.0m (minimum), **WHEN** the camera faces an interactive object at 1.5m, **THEN** `get_hovered_object()` returns null (object beyond reduced range).

### Performance Requirements

32. **GIVEN** the Hover Detection system performs raycast each frame, **WHEN** profiling with 5 interactive objects in the scene, **THEN** the raycast execution time does not exceed 0.1ms per frame on target hardware.

33. **GIVEN** the Hover Detection system runs for 60 seconds continuously, **WHEN** memory profiling is performed, **THEN** no memory leak is detected (hover state does not accumulate garbage).

34. **GIVEN** raycast_update_rate is set to 1 (default), **WHEN** the game runs at 60 FPS, **THEN** the hover state updates 60 times per second (every physics frame).

### Integration with Upstream Systems

35. **GIVEN** Player Camera is at position (0, 1.7, 0) facing forward (0, 0, 1), **WHEN** the raycast executes with max_hover_distance = 10.0m, **THEN** the raycast endpoint is (0, 1.7, 10.0).

36. **GIVEN** Player Camera's `get_camera_position()` returns Vector3(100, 50, 200), **WHEN** the Hover Detection system queries the camera, **THEN** the raycast origin is (100, 50, 200).

---

**Total Criteria: 36** | **Story Type**: Logic | **Required Evidence**: Automated unit test in `tests/unit/hover_detection/` (BLOCKING)
