# Interaction System

> **Status**: Complete
> **Author**: [user + agents]
> **Last Updated**: 2026-04-21
> **Implements Pillar**: Muscle Memory as the Enemy, The "Aha" Moment

## Overview

**Interaction System** is the action execution layer — it bridges player input to world response. When the player presses the interact button while looking at an object, this system determines whether the interaction succeeds, what state change occurs, and triggers the appropriate feedback. It is the "do" in the "look, see, act" loop.

Technically, the system is an event coordinator: it listens for `is_interact_pressed()` from Input Handling, queries `get_hovered_object()` from Hover Detection, determines the intended state change via Object State's schema, and requests the transition through `request_transition()`. The system owns no state itself — it orchestrates the flow between three stateful systems.

From the player's perspective, this system is where their intent becomes action. They see a door, move close, look at it, press the interact button — and something happens. In Mirror World, that "something" is where the game's core twist lives: the interaction may succeed in the expected way (door opens), fail (door is locked), or succeed in an unexpected way (door opens from the wrong side). The Interaction System is the delivery mechanism for the game's muscle memory subversion.

The system exists because the gap between "I pressed the button" and "the object changed" must be bridged with intention. Every interaction passes through this system, making it the touchpoint for the game's core promise: familiar actions with unfamiliar results.

## Player Fantasy

**The Unlearner** — Building New Intuition by Dismantling the Old

The Interaction System is where the game's thesis becomes *felt*. When the player presses interact, their body expects a familiar result — the door opens, the scissors cut, the pen writes. But Mirror World delivers the opposite. The player's hand knows the old way; this system teaches it the new one.

The player fantasy is **bodily relearning** — the visceral sensation of muscle memory failing and new intuition forming. The emotional arc flows: Expectation → Subversion → Unlearning → New Intuition. Each interaction is a small rewiring, a moment where the player's physical instincts are challenged and their brain physically adapts.

**Anchoring moment**: The player approaches a pair of scissors. Their hand wants to press the right handle — muscle memory says "this is how scissors work." But they hesitate. They remember: this world is backwards. They press the *other* way. The scissors cut cleanly. They feel their brain physically rewire. The wrongness becomes rightness.

In Mirror World, the Interaction System serves the player by betraying them — then showing them a new way to be right. The fantasy is not mastery of the system, but mastery of *unlearning* — the quiet satisfaction of watching old instincts dissolve and new ones take their place.

## Detailed Design

### Core Rules

**Rule 1: Interaction Trigger Conditions**

An interaction can only trigger when ALL of the following conditions are met:

- `is_interact_pressed()` returns `true` (player pressed the interact button this frame)
- `is_hovering()` returns `true` (camera is looking at a valid interactive object)
- `get_hovered_object()` returns a valid, non-freed `Node3D` reference
- The hovered object is registered with `StateManager` (has a valid `object_id`)
- `StateManager` is in **Ready** state (not locked)

If any condition fails, the interaction is not attempted and no feedback is emitted.

**Rule 2: Interaction Execution Sequence**

When all trigger conditions are met, the system executes the following sequence:

1. **Acquire target**: Call `get_hovered_object()` to get the hovered `Node3D`
2. **Resolve object_id**: Extract `object_id` from the object's exported property
3. **Query current state**: Call `StateManager.get_state(object_id)` to determine the object's current state
4. **Determine intended transition**: Call `StateManager.get_valid_transitions(object_id)` and select the first (and only, for binary objects) valid transition
5. **Apply inversion logic**: Check the object's `inverted` flag and apply inversion rules (Rule 4)
6. **Emit attempt event**: Emit `interaction_attempted` signal with full context (Rule 6)
7. **Request transition**: Call `StateManager.request_transition(object_id, target_state)` with the calculated target state
8. **Process result**:
   - If `request_transition()` returns `true`: emit `interaction_succeeded`
   - If `request_transition()` returns `false`: emit `interaction_failed` with reason from StateManager

**Rule 3: Single Transition Assumption (MVP)**

For MVP, all interactive objects are **binary** — they have exactly two states with one valid transition from any given state:

- `get_valid_transitions(object_id)` will always return an array with exactly one element
- The Interaction System selects `transitions[0]` as the target state
- No player choice interface is needed (no radial menu, no prompt)

**Example**: A door in state `"closed"` has `get_valid_transitions()` returning `["open"]`. The system automatically selects `"open"` as the target.

**Future consideration (Vertical Slice)**: Objects with multiple valid transitions will require a selection mechanism. This is explicitly out of scope for MVP.

**Rule 4: Inversion Application**

Each interactive object has an `inverted: bool` property (exported in the editor). The Interaction System applies inversion rules during transition determination:

**For non-inverted objects (inverted = false):**
- Execute the standard transition (Rule 3)
- The object behaves as the player expects based on real-world muscle memory

**For inverted objects (inverted = true):**
- Execute the standard transition (Rule 3)
- The object's state schema is designed to behave counter-intuitively
- The transition logic is NOT modified by the Interaction System — inversion is encoded in the object's schema design

**Important**: The Interaction System does NOT invert the transition direction. Instead, the `inverted` flag is metadata that:
- Signals to Visual Feedback system to apply different visual treatment (warm vs. cool tones per art bible)
- Signals to Audio Feedback system to apply different audio cues
- Is logged for analytics (tracking player success rate on inverted vs. non-inverted objects)

The inversion "feeling" comes from the object's schema design, not from the Interaction System flipping transitions.

**Rule 5: Cancel Input Handling**

The `is_cancel_pressed()` input is NOT processed by the Interaction System for MVP:

- Cancel is handled by Input Handling system directly (pauses game, releases mouse capture)
- No interaction-specific cancel behavior exists (no "undo last interaction," no "stop holding object")
- Future: If hold-based interactions are added, cancel would release the held object

**Rule 6: Feedback Event Structure**

The Interaction System emits three rich events with full context for downstream consumers:

**`interaction_attempted(object_id: StringName, current_state: StringName, intended_state: StringName, object_inverted: bool)`**
- Emitted BEFORE `request_transition()` is called
- Allows Visual Feedback to show "attempting" state (e.g., hand reaching animation)
- Allows Audio Feedback to play "attempt" sound (e.g., grunt, effort sound)

**`interaction_succeeded(object_id: StringName, previous_state: StringName, new_state: StringName, object_inverted: bool)`**
- Emitted AFTER `request_transition()` returns `true`
- Signals that the state change completed successfully
- Triggers success visual/audio feedback

**`interaction_failed(object_id: StringName, attempted_state: StringName, reason: String, object_inverted: bool)`**
- Emitted AFTER `request_transition()` returns `false`
- `reason` is copied from StateManager's `transition_rejected` signal
- Triggers failure visual/audio feedback (if any)

**Rule 7: No Owned State**

The Interaction System owns NO persistent state:

- Does not track interaction history (future: Wear Accumulation system)
- Does not track "last interacted object"
- Does not track cooldowns or rate limits (MVP: no cooldown)
- All state lives in StateManager; Interaction System is a pure coordinator

**Rule 8: Frame Timing**

Interaction processing occurs in `_process()` (not `_physics_process()`):

- Input events are discrete (single frame) and processed in `_process()`
- Hover state is updated in `_physics_process()` but remains valid for the frame
- Interaction executes on the same frame the input is detected
- Maximum latency: 1 frame from button press to state change

### States and Transitions

**Interaction System Internal States:**

| State | Description | Accepts Input | Emit Events |
|-------|-------------|---------------|-------------|
| **Ready** | Normal gameplay, waiting for input | Yes | Yes |
| **Processing** | Executing interaction sequence (single-frame) | No (blocked) | Yes |
| **Disabled** | Input blocked (pause, cutscene, loading) | No | No |

**State Properties:**
- `Ready` is the default state during gameplay
- `Processing` exists for exactly one frame (the interaction frame)
- `Disabled` syncs with Input Handling state (Paused, Cutscene, Locked)

**Transitions:**

| From | To | Trigger | On Transition |
|------|----|---------|---------------|
| Ready | Processing | `is_interact_pressed()` AND `is_hovering()` | Begin interaction sequence |
| Processing | Ready | Interaction sequence complete | Emit success/failure event |
| Ready | Disabled | Input state changes to Paused/Cutscene/Locked | Clear any pending state |
| Disabled | Ready | Input state changes to Gameplay | Resume normal operation |

### Interactions with Other Systems

**Input Handling** (upstream, designed)

| Interface | Direction | Purpose |
|-----------|-----------|---------|
| `is_interact_pressed() -> bool` | Query | Detect when player initiates interaction |
| `is_interact_held() -> bool` | Query | (MVP: unused) Future hold-based interactions |
| `is_cancel_pressed() -> bool` | Query | (MVP: unused) Future cancel-based interactions |

**Contract**: Input Handling guarantees discrete action queries return `true` for exactly one frame per press.

**Hover Detection** (upstream, complete)

| Interface | Direction | Purpose |
|-----------|-----------|---------|
| `get_hovered_object() -> Node3D` | Query | Identify which object player is targeting |
| `is_hovering() -> bool` | Query | Fast check before expensive object_id resolution |

**Contract**: Hover Detection guarantees `get_hovered_object()` returns a valid `Node3D` if `is_hovering()` is `true`, or `null` otherwise.

**Object State** (upstream, designed)

| Interface | Direction | Purpose |
|-----------|-----------|---------|
| `get_state(object_id: StringName) -> StringName` | Query | Determine current state of target object |
| `get_valid_transitions(object_id: StringName) -> Array[StringName]` | Query | Determine valid target states |
| `request_transition(object_id: StringName, to_state: StringName) -> bool` | Command | Attempt state change |

**Contract**: Object State guarantees `request_transition()` returns `true` if and only if the transition is valid per the object's schema.

**Visual Feedback** (downstream, not designed)

| Interface | Direction | Purpose |
|-----------|-----------|---------|
| `interaction_attempted` signal | Event | Show "attempting" visual state |
| `interaction_succeeded` signal | Event | Trigger success animation/effect |
| `interaction_failed` signal | Event | Show failure feedback (if any) |

**Contract**: Visual Feedback connects to Interaction System signals during initialization. Signals include full context for rich visual responses.

**Audio Feedback** (downstream, not designed)

| Interface | Direction | Purpose |
|-----------|-----------|---------|
| `interaction_attempted` signal | Event | Play "effort" or "reach" sound |
| `interaction_succeeded` signal | Event | Play success sound (object-specific or generic) |
| `interaction_failed` signal | Event | Play failure sound (if any) |

**Contract**: Audio Feedback connects to Interaction System signals during initialization.

**Wear Accumulation** (downstream, Alpha, not designed)

| Interface | Direction | Purpose |
|-----------|-----------|---------|
| `interaction_succeeded` signal | Event | Track interaction count per object |

## Formulas

**Interaction Success Condition**

```
interaction_allowed = is_interact_pressed() AND is_hovering() AND object_is_valid AND state_manager_ready
```

**Variables:**

| Variable | Type | Range | Description |
|----------|------|-------|-------------|
| `is_interact_pressed()` | bool | true/false | Input Handling discrete action query |
| `is_hovering()` | bool | true/false | Hover Detection gaze query |
| `object_is_valid` | bool | true/false | Hovered object has valid `object_id` and is registered |
| `state_manager_ready` | bool | true/false | StateManager is not in Locked state |

**Output Range:** Boolean — `true` if interaction should proceed, `false` otherwise.

---

**Target State Selection (MVP Binary Objects)**

```
target_state = get_valid_transitions(object_id)[0]
```

**Variables:**

| Variable | Type | Range | Description |
|----------|------|-------|-------------|
| `object_id` | StringName | Any registered ID | Target object identifier |
| `get_valid_transitions()` | Array[StringName] | Length = 1 for MVP | Array of valid target states from current state |
| `target_state` | StringName | Schema-defined | The state to request |

**Output Range:** Single state name (the only valid transition for binary objects).

**Example:** Door in `"closed"` state:
- `get_valid_transitions("door_test")` returns `["open"]`
- `target_state = "open"`

---

**Event Emission Timing**

| Event | Emission Condition | Timing |
|-------|-------------------|--------|
| `interaction_attempted` | `interaction_allowed = true` | Before `request_transition()` call |
| `interaction_succeeded` | `request_transition() = true` | After successful state change |
| `interaction_failed` | `request_transition() = false` | After rejected transition |

## Edge Cases

**EC1: Player Presses Interact While Looking at Nothing**

*Situation*: Player presses interact while `is_hovering()` returns `false`.

*Behavior*: No interaction is attempted. No event is emitted. The input is acknowledged but has no target.

*Rationale*: Silent failure avoids frustration. The player learns "interact only works when looking at something" naturally.

**EC2: Player Presses Interact While Looking at Non-Interactive Object**

*Situation*: Player presses interact while looking at an object that is NOT on the interactive layer (not registered with StateManager).

*Behavior*: No interaction is attempted. No event is emitted. `object_is_valid` check fails because the object has no `object_id`.

*Rationale*: Same as EC1 — silent failure. The visual feedback system may show a subtle "nothing here" cursor, but no interaction event fires.

**EC3: Hovered Object Deleted Between Hover and Interact**

*Situation*: Player is hovering an object, then a script deletes it. Player presses interact before the next frame updates hover state.

*Behavior*: `get_hovered_object()` returns null or freed object. `is_instance_valid()` check fails. Interaction is not attempted.

*Rationale*: Frame-accurate checks prevent crashes. The system validates object existence at interaction time.

**EC4: StateManager Locked During Interaction**

*Situation*: Player presses interact while StateManager is in Locked state (e.g., during a cutscene or loading).

*Behavior*: `interaction_allowed` is `false`. No interaction attempted. No event emitted.

*Rationale*: Locked state is intentional — the game explicitly blocks state changes during these moments.

**EC5: Valid Transition Fails (Object State Rejects)**

*Situation*: Interaction System determines valid transition, but `request_transition()` returns `false` (e.g., conditional transition not met).

*Behavior*: `interaction_failed` event emitted with reason from StateManager. Visual/Audio Feedback can show appropriate failure feedback.

*Rationale*: The Interaction System doesn't know WHY the transition failed — it just reports the failure. Object State owns the validation logic.

**EC6: Player Rapidly Presses Interact Multiple Times**

*Situation*: Player presses interact button repeatedly while hovering the same object.

*Behavior*: Each press triggers a full interaction sequence. If the first interaction changes the object state, the second interaction will trigger the reverse transition (toggle behavior).

*Rationale*: MVP has no cooldown. Rapid pressing toggles the object rapidly. This is acceptable for binary objects.

**EC7: Pause Menu Opens While Interaction Processing**

*Situation*: Player presses interact and pause simultaneously (same frame).

*Behavior*: Input events are processed in order. If `is_interact_pressed()` fires first, interaction executes. If `is_pause_pressed()` fires first, game pauses before interaction.

*Rationale*: Frame order is deterministic. Simultaneous inputs are rare and handled naturally.

**EC8: Object with Multiple Valid Transitions (Future Edge Case)**

*Situation*: A multi-state object returns multiple entries from `get_valid_transitions()`.

*Behavior*: System selects `transitions[0]` by default. May not match player intent.

*Rationale*: This is explicitly out of scope for MVP. Vertical Slice will add schema-defined default interaction or player choice mechanism.

## Dependencies

**Upstream Dependencies**

| System | Status | Interface Used | Purpose |
|--------|--------|----------------|---------|
| **Input Handling** | Designed | `is_interact_pressed()` | Detect interact button press |
| **Hover Detection** | Complete | `get_hovered_object()`, `is_hovering()` | Identify target object |
| **Object State** | Designed | `get_state()`, `get_valid_transitions()`, `request_transition()` | Query and modify state |

**Downstream Dependents**

| System | Status | Interface Provided | Purpose |
|--------|--------|-------------------|---------|
| **Visual Feedback** | Not designed | `interaction_attempted`, `interaction_succeeded`, `interaction_failed` signals | Trigger visual feedback |
| **Audio Feedback** | Not designed | Same signals | Trigger audio feedback |
| **Wear Accumulation** | Not designed (Alpha) | `interaction_succeeded` signal | Track interaction count |

**No Circular Dependencies**

Interaction System has no circular dependencies. It queries upstream systems (Input, Hover, State) and broadcasts to downstream systems (Visual, Audio, Wear).

## Tuning Knobs

| Knob | Type | Default | Safe Range | Affects |
|------|------|---------|------------|---------|
| `enable_interaction_logging` | bool | true | true/false | Whether to log interaction events (attempted, succeeded, failed) for debugging. Disable in release builds. |
| `interaction_cooldown_ms` | int | 0 | 0 – 500 | Minimum milliseconds between interactions on the same object. MVP uses 0 (no cooldown). Adding cooldown prevents rapid toggle spam. |

**Tuning Guidance**

- **enable_interaction_logging**: Keep `true` during development for debugging player behavior. Set to `false` in release builds for performance.
- **interaction_cooldown_ms**: MVP uses `0` (no cooldown). If playtesting reveals rapid toggle spam as a problem, increase to 100-200ms to add a small delay between interactions on the same object.

**Future Tuning Knobs (Vertical Slice)**

| Knob | Type | Purpose |
|------|------|---------|
| `multi_state_selection_method` | enum | How to handle objects with multiple valid transitions: `first_valid`, `last_used`, `player_choice` |
| `hold_interaction_duration_ms` | int | Duration a hold-based interaction must be held before triggering |

## Acceptance Criteria

### Trigger Conditions

1. **GIVEN** the player is NOT hovering any object, **WHEN** the player presses interact, **THEN** no interaction events are emitted (silent failure).

2. **GIVEN** the player is looking at an object NOT registered with StateManager, **WHEN** the player presses interact, **THEN** no interaction events are emitted.

3. **GIVEN** StateManager is in Locked state, **WHEN** the player presses interact while hovering a valid object, **THEN** no interaction events are emitted.

4. **GIVEN** the player is hovering a valid interactive object with StateManager Ready, **WHEN** the player presses interact, **THEN** `interaction_attempted` is emitted, followed by `interaction_succeeded` or `interaction_failed`.

### Execution Sequence

5. **GIVEN** a door is in `"closed"` state, **WHEN** the player interacts with it, **THEN** `interaction_attempted` shows `intended_state = "open"` and `interaction_succeeded` shows `new_state = "open"`.

6. **GIVEN** a successful interaction occurred, **WHEN** querying `StateManager.get_state(object_id)`, **THEN** the returned state matches `interaction_succeeded`'s `new_state`.

7. **GIVEN** an object with `inverted = true`, **WHEN** the player interacts, **THEN** all signals include `object_inverted = true`.

8. **GIVEN** an object with `inverted = false`, **WHEN** the player interacts, **THEN** all signals include `object_inverted = false`.

### Internal State Transitions

9. **GIVEN** the Interaction System is in Ready state, **WHEN** the player presses interact while hovering a valid object, **THEN** the interaction sequence executes.

10. **GIVEN** the Interaction System is in Disabled state, **WHEN** the player presses interact, **THEN** no interaction events are emitted.

11. **GIVEN** the system transitions from Disabled to Ready, **WHEN** the player then interacts with a valid object, **THEN** the interaction executes normally.

### Edge Case Handling

12. **GIVEN** the player presses interact while looking at empty space, **THEN** no crash occurs, no error is logged, and no signals are emitted.

13. **GIVEN** a hovered object is deleted mid-frame, **WHEN** the player presses interact, **THEN** no crash occurs and interaction silently fails.

14. **GIVEN** a transition is valid but `request_transition()` returns false, **WHEN** the interaction fails, **THEN** `interaction_failed` is emitted with a reason string.

15. **GIVEN** the player presses interact 3 times rapidly while hovering the same object, **THEN** 3 interaction sequences execute (object toggles 3 times).

16. **GIVEN** a door in `"closed"` state, **WHEN** the player interacts twice in succession, **THEN** the door opens then closes (toggles correctly).

### Event Emission

17. **GIVEN** a successful interaction, **WHEN** signals are emitted, **THEN** `interaction_attempted` is emitted first, then `interaction_succeeded` (never both succeeded and failed).

18. **GIVEN** a failed interaction, **WHEN** signals are emitted, **THEN** `interaction_attempted` is emitted first, then `interaction_failed` with a non-empty reason string.

19. **GIVEN** any interaction event, **WHEN** the signal is emitted, **THEN** all four parameters (`object_id`, state(s), `object_inverted`) match expected values.

### Frame Timing

20. **GIVEN** the player presses interact, **WHEN** measuring frame latency, **THEN** the state change occurs within 1 frame of input detection.

21. **GIVEN** an interaction is processing, **WHEN** tracking system state, **THEN** Processing state persists for exactly 1 frame.

---

**Total Criteria: 21** | **Story Type**: Integration | **Required Evidence**: Integration test in `tests/integration/interaction_system/` (BLOCKING)
