# Input Handling

> **Status**: Designed
> **Author**: user + agents
> **Last Updated**: 2026-04-17
> **Implements Pillar**: Pillar 1 — Muscle Memory as the Enemy

## Overview

The Input Handling system is the foundation layer that receives all player input (mouse, keyboard, gamepad) and translates it into game actions. It provides a unified interface for other systems to query input state without needing to know the source device. The system manages input actions (move, look, interact), handles mouse capture for first-person camera control, and exposes a remapping interface for accessibility.

**Player impact**: Players experience this system through its absence — if input lagged, felt unresponsive, or failed to register, the game would feel broken. But when working correctly, the input layer is invisible: the player thinks "I want to look left" and the camera moves. The system's primary job is to get out of the way.

**Core responsibilities**:

- Capture and route all input events (mouse, keyboard, gamepad)
- Map raw input to named actions (`look`, `move`, `interact`, `cancel`)
- Provide queryable input state for downstream systems (Player Camera, Interaction System)
- Expose action remapping for Accessibility system
- Handle mouse capture mode for first-person camera control

## Player Fantasy

This system has no direct player fantasy. The Input Handling layer is invisible infrastructure — players do not engage with "input" as a concept; they engage with the world through it.

The fantasy is realized downstream:

- **Player Camera** delivers the "I am looking around" feeling
- **Interaction System** delivers the "I am touching and manipulating objects" feeling
- **Accessibility** delivers the "this game adapts to how I play" feeling

The Input Handling system's role is to make those downstream fantasies feel instantaneous and reliable. If the player ever thinks about the input layer itself, something has gone wrong.

**Pillar alignment**: While this system serves all pillars indirectly, its closest connection is to **Pillar 1 (Muscle Memory as the Enemy)**. The input system is where the player's physical instincts enter the game world. When a right-handed player instinctively moves the mouse right, the input layer faithfully delivers that action — and the inversion logic elsewhere in the game makes it fail. The input system does not judge or filter; it translates.

## Detailed Design

### Core Rules

**Rule 1: Input Actions**

The game defines the following named input actions:

| Action | Default Binding | Description |
|--------|-----------------|-------------|
| `move_forward` | W | Move player forward |
| `move_back` | S | Move player backward |
| `move_left` | A | Strafe left |
| `move_right` | D | Strafe right |
| `look` | Mouse motion / Right stick | Camera rotation |
| `interact` | Left click / E / Gamepad A | Interact with hovered object |
| `cancel` | Escape | Release mouse capture, open pause menu |
| `pause` | Escape / P / Gamepad Start | Toggle pause menu |

**Rule 2: Mouse Capture Mode**

The mouse is captured during gameplay for first-person camera control:

- **On game start**: Mouse is captured immediately after "click to start"
- **On pause**: Mouse is released, becomes visible for menu navigation
- **On unpause**: Mouse is re-captured automatically
- **On window focus loss**: Godot auto-releases capture; game should pause automatically
- **On game completion**: Mouse is released for end-screen interaction

**Rule 3: Look Input Processing**

Mouse look uses `InputEventMouseMotion.relative` for camera rotation:

1. Accumulate relative motion each frame
2. Apply sensitivity multiplier (default 0.25)
3. Scale by delta time for frame-rate independence
4. Pass to Player Camera system via `get_look_delta()` query

Analog stick look uses `Input.get_vector()` with deadzone applied.

**Rule 4: Interaction Input**

Interact is a discrete action (click-to-interact, not hold):

- `is_interact_pressed()` returns true only on the frame the button is pressed
- `is_interact_held()` returns true while the button is held (for future hold-actions)
- Interaction System receives the event and determines the target

**Rule 5: Multi-Input Device Switching**

When multiple input devices are active (mouse + gamepad):

- The device that provided the most recent input becomes the "active" input method
- Visual feedback (crosshair style, button prompts) updates to match active device
- No explicit "switch input" action required — switching is automatic

**Rule 6: Accessibility Features (MVP)**

The following accessibility features are required in MVP:

| Feature | Behavior |
|---------|----------|
| **Key remapping** | All actions except `look` can be rebound via menu |
| **Hold vs. Toggle** | Toggle option for interact (press once to activate, press again to release) |
| **Sensitivity range** | Mouse sensitivity adjustable from 0.1 to 2.0 (20x range) |
| **Invert Y-axis** | Toggle for inverted vertical camera look |
| **FOV slider** | Adjustable from 60° to 110°, default 85° |

**Rule 7: Input Remapping Persistence**

Remapped bindings are saved to `user://input_remaps.cfg` (ConfigFile format):

- Loaded on game start, applied before input is processed
- Saved immediately when player changes a binding
- Reset to defaults option available

---

### States and Transitions

| State | Mouse Mode | Input Processing |
|-------|------------|------------------|
| **Gameplay** | Captured | All actions active |
| **Paused** | Visible | Only UI navigation actions |
| **Cutscene** | Visible | No gameplay input, skip prompt only |
| **Menu** | Visible | UI navigation only |
| **Locked** | Captured | No input processed (for scripted moments) |

**Transitions**:

| From | To | Trigger |
|------|----|---------|
| Gameplay | Paused | `pause` action pressed |
| Paused | Gameplay | `pause` action pressed, or "Resume" selected |
| Gameplay | Cutscene | Scripted event starts |
| Cutscene | Gameplay | Cutscene ends or `cancel` pressed to skip |
| Gameplay | Menu | Terminal/computer object interacted (future) |
| Any | Locked | Script locks input (rare, for effect) |
| Locked | Gameplay | Script unlocks input |

---

### Interactions with Other Systems

**Player Camera** (downstream)

- **Queries**: `get_look_delta() -> Vector2` — returns accumulated mouse/stick motion
- **Queries**: `get_move_vector() -> Vector2` — returns normalized movement input
- **Contract**: Look delta is reset each frame after being queried

**Interaction System** (downstream)

- **Queries**: `is_interact_pressed() -> bool` — true on frame interact is pressed
- **Queries**: `is_interact_held() -> bool` — true while interact is held
- **Queries**: `is_cancel_pressed() -> bool` — true on frame cancel is pressed
- **Contract**: Discrete actions return true only once per press

**Accessibility** (downstream)

- **Queries**: `get_available_actions() -> Array[StringName]` — list of remappable actions
- **Queries**: `get_action_bindings(action: StringName) -> Array[InputEvent]` — current bindings
- **Commands**: `set_action_binding(action: StringName, index: int, event: InputEvent) -> bool` — rebind action
- **Commands**: `reset_to_defaults() -> void` — clear all remaps

**Room Manager** (downstream, future)

- **Commands**: `set_input_state(state: String) -> void` — transition input state (gameplay/paused/locked)

## Formulas

### Look Sensitivity Formula

The `look_sensitivity` formula is defined as:

```
look_delta = raw_input * sensitivity * invert_y
```

**Variables:**

| Variable | Symbol | Type | Range | Description |
|----------|--------|------|-------|-------------|
| raw_input | r | Vector2 | -∞ to +∞ | Raw mouse motion or stick position |
| sensitivity | s | float | 0.1 to 2.0 | User-configurable sensitivity multiplier |
| invert_y | i_y | int | 1 or -1 | Y-axis inversion (1 = normal, -1 = inverted) |
| look_delta | d | Vector2 | -∞ to +∞ | Processed camera rotation delta |

**Output Range:** Depends on input device. Mouse: typically 0–200 pixels/frame at normal speed. Stick: -1.0 to 1.0 after deadzone.

**Example:** Player moves mouse 50 pixels right with sensitivity 0.25:

```
look_delta_x = 50 * 0.25 * 1 = 12.5 (degrees of rotation, approximately)
```

---

### Gamepad Deadzone Formula

The `deadzone` formula is defined as:

```
if |raw_axis| < deadzone:
    processed_axis = 0
else:
    processed_axis = sign(raw_axis) * (|raw_axis| - deadzone) / (1 - deadzone)
```

**Variables:**

| Variable | Symbol | Type | Range | Description |
|----------|--------|------|-------|-------------|
| raw_axis | a | float | -1.0 to 1.0 | Raw analog stick position |
| deadzone | dz | float | 0.1 to 0.3 | Deadzone threshold (default 0.15) |
| processed_axis | p | float | -1.0 to 1.0 | Stick position after deadzone applied |

**Output Range:** 0 (within deadzone) or rescaled value from (deadzone, 1.0] mapped to (0, 1.0].

**Example:** Stick at 0.3 with deadzone 0.15:

```
processed_axis = sign(0.3) * (0.3 - 0.15) / (1 - 0.15) = 1 * 0.15 / 0.85 ≈ 0.176
```

---

### Multiplier Bounds Formula

Sensitivity and other multipliers are clamped to valid ranges:

```
clamped_value = clamp(raw_value, min_value, max_value)
```

**Variables:**

| Variable | Symbol | Type | Range | Description |
|----------|--------|------|-------|-------------|
| raw_value | v | float | Any | User-provided or calculated value |
| min_value | min | float | Per-setting | Minimum allowed (e.g., 0.1 for sensitivity) |
| max_value | max | float | Per-setting | Maximum allowed (e.g., 2.0 for sensitivity) |
| clamped_value | c | float | [min, max] | Value after clamping |

**Tuning knob defaults:**

| Setting | Min | Default | Max |
|---------|-----|---------|-----|
| Mouse sensitivity | 0.1 | 0.25 | 2.0 |
| Gamepad sensitivity | 0.1 | 1.0 | 2.0 |
| Gamepad deadzone | 0.05 | 0.15 | 0.30 |
| FOV | 60° | 85° | 110° |

## Edge Cases

- **If window loses focus while mouse is captured**: Godot auto-releases mouse capture. The game should transition to Paused state automatically to prevent unintended gameplay during alt-tab.

- **If player presses interact while looking at nothing**: No interaction occurs. No feedback is shown (or very subtle "nothing to interact with" sound at low volume). The input is acknowledged but has no target.

- **If player presses interact while looking at non-interactive object**: No interaction occurs. Hover Detection system determines target; if target has no interaction component, nothing happens.

- **If player presses pause during a cutscene**: Cutscene skips if skippable, otherwise ignored. Mouse capture is not released until cutscene ends.

- **If player reconnects a gamepad mid-session**: The new device is detected automatically. No player action required. The "most recent wins" rule applies — if the player was using mouse and picks up the gamepad, gamepad becomes active.

- **If player removes the only input device (unplugs keyboard/mouse/gamepad)**: Game pauses automatically. On-screen message prompts player to reconnect device. Input resumes when device is detected.

- **If player remaps an action to a key already bound to another action**: The new binding replaces the old one. The previously bound action shows "No binding" in the UI and must be reassigned.

- **If player sets sensitivity to minimum (0.1)**: Camera still moves, but very slowly. This is valid for players who need very precise control.

- **If player sets sensitivity to maximum (2.0)**: Camera moves very fast. This is valid for players who prefer high sensitivity.

- **If two actions are pressed simultaneously (e.g., move_forward + move_back)**: Movement cancels out — player does not move. This is standard behavior from `Input.get_vector()`.

- **If player presses escape during mouse capture for the first time**: Mouse is released and pause menu opens. This is the intended way to exit gameplay.

## Dependencies

### Upstream Dependencies (this system depends on)

**None.** Input Handling is a Foundation layer system with no dependencies on other game systems.

**Engine dependencies:**

- **Godot Input system**: `Input` class for action queries, `InputMap` for action definitions, `InputEvent` types for raw input processing
- **Godot MainLoop**: `_input()` and `_unhandled_input()` callbacks for event processing

---

### Downstream Dependencies (systems that depend on this one)

| System | Dependency Type | Interface Used |
|--------|-----------------|----------------|
| **Player Camera** | Hard | `get_look_delta()`, `get_move_vector()` |
| **Interaction System** | Hard | `is_interact_pressed()`, `is_interact_held()`, `is_cancel_pressed()` |
| **Accessibility** | Hard | `get_available_actions()`, `get_action_bindings()`, `set_action_binding()`, `reset_to_defaults()` |
| **Room Manager** | Soft | `set_input_state()` (used for state transitions) |

**Hard dependency** = system cannot function without this interface.
**Soft dependency** = system benefits from this interface but can operate with reduced functionality.

---

### Data Flow Diagram

```
Raw Input (mouse/keyboard/gamepad)
         ↓
    Input Handling
         ↓
    ┌────┴────┬──────────┐
    ↓         ↓          ↓
Camera    Interaction  Accessibility
          System       (remap UI)
```

**Contracts that downstream systems must respect:**

- `get_look_delta()` returns accumulated motion and resets the accumulator each frame. Call once per frame.
- Discrete action queries (`is_*_pressed()`) return true only on the frame the action started. Do not rely on them across frames.

## Tuning Knobs

| Knob | Type | Default | Range | What It Affects |
|------|------|---------|-------|-----------------|
| **mouse_sensitivity** | float | 0.25 | 0.1 – 2.0 | Camera rotation speed for mouse input. Higher = faster camera turn. |
| **gamepad_sensitivity** | float | 1.0 | 0.1 – 2.0 | Camera rotation speed for gamepad stick. Higher = faster camera turn. |
| **gamepad_deadzone** | float | 0.15 | 0.05 – 0.30 | Analog stick deadzone. Higher = more stick drift ignored, but less precision near center. |
| **invert_y** | bool | false | true/false | Inverts vertical camera look. Some players require this as their default mental model. |
| **fov** | float | 85 | 60 – 110 | Field of view in degrees. Higher = wider view, may cause motion sickness in some players. |
| **interact_toggle_mode** | bool | false | true/false | If true, interact toggles (press once to activate, press again to release). If false, interact is momentary (hold to maintain). |

---

### Knob Interactions

| If you change... | It may affect... | Why |
|------------------|------------------|-----|
| `mouse_sensitivity` to very high | Player comfort, precision | Fast camera may cause disorientation; precision interactions become harder |
| `gamepad_deadzone` to very low | Stick drift | Players may see camera drift when hands are off the stick |
| `fov` to very low (< 70) | Motion sickness risk | Narrow FOV can cause discomfort in some players |
| `fov` to very high (> 100) | Performance, motion sickness | Wider FOV renders more; some players experience nausea |

---

### What Breaks at Extremes

| Knob | Too Low | Too High |
|------|---------|----------|
| `mouse_sensitivity` | Camera barely moves; frustrating for fast players | Camera whips around; precision impossible |
| `gamepad_deadzone` | Stick drift causes unintended movement | Small stick movements are ignored; precision lost |
| `fov` | Tunnel vision; motion sickness for some | Fish-eye effect; motion sickness for others |

## Acceptance Criteria

### Core Input Actions

1. **GIVEN** the game has started, **WHEN** the player clicks "Start", **THEN** the mouse cursor disappears and camera rotation follows mouse movement.

2. **GIVEN** the mouse is captured, **WHEN** the player presses the Escape key, **THEN** the mouse cursor appears and the pause menu is displayed.

3. **GIVEN** the pause menu is open, **WHEN** the player presses Escape or clicks "Resume", **THEN** the mouse cursor disappears and gameplay resumes.

4. **GIVEN** a menu is open, **WHEN** the player presses the cancel key (Escape), **THEN** the menu closes.

5. **GIVEN** the player presses W, **WHEN** observing the player movement, **THEN** the player moves forward relative to camera facing.

6. **GIVEN** the player presses W and S simultaneously, **WHEN** observing movement, **THEN** the player does not move (inputs cancel).

### Camera Look

7. **GIVEN** sensitivity is at default (0.25), **WHEN** the player moves the mouse 100 pixels right, **THEN** the camera rotates approximately 25 degrees right.

8. **GIVEN** invert_y is enabled, **WHEN** the player moves the mouse forward, **THEN** the camera looks down.

9. **GIVEN** sensitivity is changed from default to maximum (2.0), **WHEN** the player moves the mouse, **THEN** camera rotation is noticeably faster than at default.

10. **GIVEN** sensitivity is changed from default to minimum (0.1), **WHEN** the player moves the mouse, **THEN** camera rotation is noticeably slower than at default.

### Interaction

11. **GIVEN** the player is facing an interactable object, **WHEN** the player presses the interact key, **THEN** the interaction begins.

12. **GIVEN** interaction mode is set to "toggle", **WHEN** the player presses interact once, **THEN** the interaction begins and continues without holding the key.

13. **GIVEN** the player is NOT facing any interactable object, **WHEN** the player presses interact, **THEN** no visible or audible feedback occurs and no error is shown.

### Accessibility and Remapping

14. **GIVEN** the player remaps "interact" from E to F, **WHEN** the player presses F, **THEN** the interaction triggers.

15. **GIVEN** the player has remapped bindings, **WHEN** the game is restarted, **THEN** the remapped bindings are preserved.

16. **GIVEN** the player resets input to defaults, **WHEN** the reset is confirmed, **THEN** all bindings return to their original values.

### Gamepad Support

17. **GIVEN** a gamepad is connected with default sensitivity, **WHEN** the right stick is fully deflected right, **THEN** the camera rotates right at approximately 90 degrees per second.

18. **GIVEN** gamepad deadzone is set to default (0.15), **WHEN** the stick is deflected slightly (within deadzone), **THEN** no camera movement occurs.

19. **GIVEN** gamepad deadzone is applied, **WHEN** the stick is fully deflected past the deadzone, **THEN** camera rotation reaches maximum speed.

20. **GIVEN** the player is using mouse, **WHEN** the player moves the gamepad stick, **THEN** on-screen button prompts switch to gamepad icons.

### System Behavior

21. **GIVEN** the game is running, **WHEN** the window loses focus (alt-tab), **THEN** the game pauses automatically.

22. **GIVEN** a gamepad is disconnected during gameplay, **WHEN** the gamepad is reconnected, **THEN** input resumes on the gamepad without manual configuration.

23. **GIVEN** sensitivity is set to minimum or maximum, **WHEN** the player moves the mouse, **THEN** camera rotation functions correctly without jitter or freezing.

24. **GIVEN** FOV is set to 90, **WHEN** the player observes the game view, **THEN** the field of view appears wider than at default (85).

## Open Questions

| # | Question | Owner | Resolution Target | Status |
|---|----------|-------|-------------------|--------|
| 1 | Should the Input Handling system be an Autoload singleton or a scene node? | Lead Programmer | Before implementation | Open |
| 2 | Should "look" (mouse motion) be remappable, or is it always mouse/right-stick? | Game Designer | Before implementation | Open |
| 3 | What is the exact rotation speed (degrees per second) for gamepad at full deflection? | Game Designer | During tuning phase | Open |
| 4 | Should there be a visual FOV test scene for QA verification? | QA Lead | Before Alpha | Open |
| 5 | Should the game support multiple bindings per action (e.g., both E and left-click for interact)? | Game Designer | Before implementation | Open |
