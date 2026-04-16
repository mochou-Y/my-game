# Object State

> **Status**: Designed
> **Author**: user + agents
> **Last Updated**: 2026-04-17
> **Implements Pillar**: Pillar 2 — Everyday Objects as Puzzles

## Overview

The Object State system is the central data store that tracks the current state of every interactive object in the game world. Each object has a defined set of possible states (e.g., a door can be `open` or `closed`, a pair of scissors can be `normal` or `inverted`), and the system manages transitions between those states. It provides a unified interface for querying state, requesting state changes, and broadcasting state change events to dependent systems.

**Player impact**: Players experience this system through the objects they interact with — a door that opens, scissors that cut, a pen that writes. When the player acts on an object, they see the state change. The system's job is to make every object feel consistent: if a door is closed, it can be opened; if it's already open, it can be closed. The player builds a mental model of each object's behavior through repeated interaction.

**Core responsibilities**:

- Define the state schema for each interactive object type (what states exist, what transitions are valid)
- Track current state for every interactive object instance in the scene
- Validate state change requests (can this object transition from state A to state B?)
- Broadcast state change events to dependent systems (Visual Feedback, Audio Feedback, Wear Accumulation)
- Provide queryable state for the Interaction System and Inversion Logic
- Support state persistence interface for Save/Load (Vertical Slice feature)

**MVP scope note**: Full persistence requires Save/Load (Vertical Slice). For MVP, Object State manages runtime state only — all objects reset to initial state when the room restarts.

## Player Fantasy

This system has no direct player fantasy. The Object State layer is invisible infrastructure — players do not engage with "state" as a concept; they engage with objects that have behaviors.

The fantasy is realized downstream:

- **Interaction System** delivers the "I am manipulating this object" feeling
- **Inversion Logic** delivers the "my instincts failed — I must unlearn" feeling
- **Visual Feedback** delivers the "this object changed because of my action" feeling
- **Audio Feedback** delivers the "this object responded to me" feeling

The Object State system's role is to make those downstream fantasies feel consistent and reliable. When a player opens a door, closes it, and opens it again, the behavior is identical each time. The object "remembers" its state between interactions. If the player ever thinks "I don't understand what state this object is in," something has gone wrong.

**Pillar alignment**: This system most directly serves **Pillar 2 (Everyday Objects as Puzzles)**. The state system defines what each object *can* do — what behaviors it supports, what states it can transition between. The puzzle design for each object is encoded in its state schema. A door that can only be `open` or `closed` is a simple puzzle; an object with multiple states and conditional transitions is a more complex puzzle. The state system makes the puzzle rules explicit and enforceable.

## Detailed Design

### Core Rules

**Rule 1: Object Identification**

Every interactive object must have a unique `object_id` assigned in the editor:

- Format: `{object_type}_{room}_{instance}` (e.g., `door_bedroom_main`, `scissors_desk_01`)
- Stored as an exported `StringName` on each interactive object node
- Must be unique within a room — Room Manager validates on load and logs error on duplicates
- This ID is the stable reference for state queries, persistence, and debugging

**Rule 2: State Schema Definition**

Each object type defines its state schema as GDScript constants:

```gdscript
# Example: door_schema.gd
class_name DoorStateSchema

const OBJECT_TYPE := "door"
const STATES := ["closed", "open"]
const INITIAL_STATE := "closed"
const VALID_TRANSITIONS := {
    "closed": ["open"],
    "open": ["closed"]
}
```

Schema components:

- `OBJECT_TYPE`: Human-readable type name for debugging
- `STATES`: Array of all valid state names (strings)
- `INITIAL_STATE`: Default state when object is created
- `VALID_TRANSITIONS`: Dictionary mapping each state to allowed target states

**Rule 3: State Storage**

All object state is stored centrally in the `StateManager` autoload:

- Internal storage: `Dictionary` keyed by `object_id`
- Each entry: `{object_type: StringName, current_state: StringName, schema_ref: Object}`
- State is loaded when Room Manager instantiates objects
- State is destroyed when room is unloaded

**Rule 4: State Queries**

Dependent systems query state through the StateManager API:

| Method | Returns | Description |
|--------|---------|-------------|
| `get_state(object_id: StringName) -> StringName` | Current state name | Returns empty StringName if object not found |
| `has_state(object_id: StringName) -> bool` | bool | True if object exists in state registry |
| `get_object_type(object_id: StringName) -> StringName` | Type name | For schema lookup |
| `can_transition(object_id: StringName, to_state: StringName) -> bool` | bool | Validates transition against schema |
| `get_valid_transitions(object_id: StringName) -> Array[StringName]` | Array of allowed states | For UI hints |

**Rule 5: State Changes**

State changes are requested, not forced:

```gdscript
StateManager.request_transition(object_id: StringName, to_state: StringName) -> bool
```

- If transition is valid: state changes, `state_changed` signal fires, returns `true`
- If transition is invalid: state unchanged, `transition_rejected` signal fires, returns `false`
- State changes are atomic — no partial state exists

**Rule 6: State Change Events**

StateManager emits signals for all state changes:

```gdscript
signal state_changed(object_id: StringName, old_state: StringName, new_state: StringName)
signal transition_rejected(object_id: StringName, from_state: StringName, to_state: StringName, reason: String)
```

Dependent systems connect to these signals during initialization. All events include the `object_id` so listeners can filter.

**Rule 7: Object Registration**

Interactive objects register themselves with StateManager on ready:

1. Object calls `StateManager.register_object(object_id, object_type)`
2. StateManager loads the schema for that object type
3. StateManager sets initial state from schema
4. Object is now queryable by dependent systems

On room unload, Room Manager calls `StateManager.clear_room()` to unregister all objects.

---

### States and Transitions

**StateManager Internal States:**

| State | Description |
|-------|-------------|
| **Uninitialized** | Autoload not yet configured |
| **Ready** | Accepting object registrations |
| **Locked** | Transitions blocked (for cutscenes, loading) |

**Object Lifecycle:**

```
[Unregistered] --register--> [Active] --unregister--> [Removed]
                                  |
                        state_changed events
                                  |
                        transition_rejected events
```

**Example Object State Machine (Door):**

```
                    +-------+
                    | closed|
                    +-------+
                       | ^
         request("open") | | request("close")
                       v |
                    +-------+
                    |  open |
                    +-------+
```

---

### Interactions with Other Systems

**Interaction System** (downstream)

- **Queries**: `get_state(object_id)` — determine current state before triggering interaction
- **Queries**: `can_transition(object_id, to_state)` — validate intended state change
- **Commands**: `request_transition(object_id, to_state)` — attempt state change
- **Signals**: Connects to `state_changed` to update interaction prompts
- **Contract**: Interaction System does not modify state directly; always through StateManager

**Inversion Logic** (downstream)

- **Queries**: `get_state(object_id)` — read current state to determine inversion behavior
- **Queries**: `get_object_type(object_id)` — determine which inversion rule applies
- **Signals**: Connects to `state_changed` to react to state changes
- **Contract**: Inversion Logic observes state; it may request transitions but doesn't own state

**Visual Feedback** (downstream)

- **Queries**: `get_state(object_id)` — determine which visual state to display
- **Signals**: Connects to `state_changed` to trigger visual transitions
- **Contract**: Visual Feedback is read-only; never modifies state

**Audio Feedback** (downstream)

- **Queries**: `get_state(object_id)` — determine which audio to play
- **Signals**: Connects to `state_changed` to trigger audio cues
- **Contract**: Audio Feedback is read-only; never modifies state

**Wear Accumulation** (downstream, Alpha)

- **Queries**: `get_state(object_id)` — correlate wear with object state
- **Signals**: Connects to `state_changed` to track interaction history
- **Contract**: Wear Accumulation tracks metadata, not core state

**Room Manager** (downstream, Vertical Slice)

- **Commands**: `register_object(object_id, object_type)` — register objects on room load
- **Commands**: `clear_room()` — unregister all objects on room unload
- **Commands**: `serialize()` — get all state for Save/Load (Vertical Slice)
- **Commands**: `deserialize(data: Dictionary)` — restore state on load (Vertical Slice)
- **Contract**: Room Manager owns object lifecycle; StateManager owns state data

**Save/Load** (upstream, Vertical Slice — undesigned)

- **Persistence Interface**: StateManager provides `serialize()` and `deserialize(data)` methods
- **Format**: Dictionary of `{object_id: {state: StringName, metadata: Dictionary}}`
- **Contract**: Save/Load calls serialize on save, deserialize on load

## Formulas

### Transition Validation Formula

The `transition_valid` formula is defined as:

```
transition_valid = to_state in VALID_TRANSITIONS[current_state]
```

**Variables:**

| Variable | Symbol | Type | Range | Description |
|----------|--------|------|-------|-------------|
| current_state | s_cur | StringName | schema.STATES | Object's current state |
| to_state | s_to | StringName | schema.STATES | Requested target state |
| VALID_TRANSITIONS | T | Dictionary | schema-defined | Map of state → allowed targets |
| transition_valid | v | bool | true/false | Whether transition is allowed |

**Output Range:** Boolean — `true` if transition is valid, `false` otherwise.

**Example:** Door in `closed` state, player requests `open`:

```
transition_valid = "open" in ["open"] = true
```

**Example:** Door in `closed` state, player requests `closed`:

```
transition_valid = "closed" in ["open"] = false
```

---

### State Lookup Formula

The `get_state` formula is defined as:

```
get_state(object_id) = _object_states[object_id].current_state
```

**Variables:**

| Variable | Symbol | Type | Range | Description |
|----------|--------|------|-------|-------------|
| object_id | id | StringName | Any registered ID | Unique object identifier |
| _object_states | S | Dictionary | N/A | Internal state storage |
| current_state | s | StringName | schema.STATES | Object's current state |

**Output Range:** StringName (state name) or empty StringName if object not found.

---

### Serialization Format (Vertical Slice)

The `serialize` format for persistence:

```
serialized_state = {
    object_id: {
        "state": current_state,
        "object_type": object_type
    },
    ...
}
```

**Variables:**

| Variable | Symbol | Type | Range | Description |
|----------|--------|------|-------|-------------|
| object_id | id | StringName | Any registered ID | Object identifier (key) |
| current_state | s | StringName | schema.STATES | State to restore |
| object_type | t | StringName | Any registered type | For schema lookup on deserialize |

**Output Range:** Dictionary suitable for ConfigFile or JSON serialization.

## Edge Cases

- **If `get_state()` is called with an unregistered object_id**: Returns empty StringName. No error is thrown. Caller should check `has_state()` first if uncertain.

- **If `request_transition()` is called with an unregistered object_id**: Returns `false`. Emits `transition_rejected` signal with reason `"object_not_found"`. State unchanged.

- **If `request_transition()` is called with an invalid target state (not in schema)**: Returns `false`. Emits `transition_rejected` signal with reason `"invalid_state"`. State unchanged.

- **If `request_transition()` is called for a transition not in VALID_TRANSITIONS**: Returns `false`. Emits `transition_rejected` signal with reason `"transition_not_allowed"`. State unchanged.

- **If `request_transition()` is called with the same state as current**: Returns `true` (no-op). Does not emit `state_changed` signal. Object is already in requested state.

- **If an object registers with a duplicate object_id**: StateManager logs error, replaces the previous registration. Last registration wins. Room Manager should validate uniqueness during room load.

- **If StateManager is locked (during cutscene/loading)**: All `request_transition()` calls return `false`. Emits `transition_rejected` signal with reason `"state_locked"`. Existing state is preserved.

- **If an object is unregistered while a dependent system holds a reference**: Dependent system's next query returns empty/invalid. Dependent systems should connect to `object_unregistered` signal (if added) or check `has_state()` before critical operations.

- **If `serialize()` is called with no objects registered**: Returns empty Dictionary `{}`. Valid state for a fresh room.

- **If `deserialize()` is called with data for unknown object_ids**: Those entries are ignored. Only objects currently registered can have state restored. Logs warning for each unknown ID.

- **If `deserialize()` is called with invalid state names**: Those entries are ignored. State is set to schema's INITIAL_STATE instead. Logs warning for each invalid state.

## Dependencies

### Upstream Dependencies (this system depends on)

**Save/Load** (Vertical Slice — undesigned)

- **Hard dependency** for persistence
- **Persistence Interface**: StateManager provides `serialize()` and `deserialize(data)` methods
- **Contract**: Save/Load calls `serialize()` on room save, `deserialize(data)` on room load
- **MVP note**: Object State works without Save/Load — states reset on room restart. Persistence is additive, not blocking.

**Engine dependencies**:

- **Godot Dictionary**: For internal state storage
- **Godot Signal system**: For `state_changed` and `transition_rejected` events

---

### Downstream Dependencies (systems that depend on this one)

| System | Dependency Type | Interface Used | Status |
|--------|-----------------|----------------|--------|
| **Interaction System** | Hard | `get_state()`, `can_transition()`, `request_transition()` | Not designed |
| **Inversion Logic** | Hard | `get_state()`, `get_object_type()`, `state_changed` signal | Not designed |
| **Visual Feedback** | Hard | `get_state()`, `state_changed` signal | Not designed |
| **Audio Feedback** | Hard | `get_state()`, `state_changed` signal | Not designed |
| **Wear Accumulation** | Soft | `get_state()`, `state_changed` signal | Not designed |
| **Room Manager** | Hard | `register_object()`, `clear_room()`, `serialize()`, `deserialize()` | Not designed |

**Hard dependency** = system cannot function without this interface.
**Soft dependency** = system benefits from this interface but can operate with reduced functionality.

---

### Data Flow Diagram

```
                    Save/Load (VS)
                         ↓
                    serialize/deserialize
                         ↓
┌─────────────────────────────────────────────┐
│              StateManager (Autoload)         │
│  - _object_states: Dictionary                │
│  - _state_schemas: Dictionary                │
└─────────────────────────────────────────────┘
          │              │              │
          ↓              ↓              ↓
   state_changed   get_state()   request_transition()
          │              │              │
    ┌─────┴─────┬────────┴──────┬───────┴─────┐
    ↓           ↓               ↓             ↓
 Visual     Audio         Interaction   Inversion
Feedback   Feedback        System         Logic
```

---

### Contracts That Downstream Systems Must Respect

1. **Never modify state directly** — always use `request_transition()`
2. **Check `has_state()` before `get_state()`** if object existence is uncertain
3. **Connect to `state_changed` signal** to react to state changes from other systems
4. **Do not assume state persistence** — MVP resets state on room restart
5. **Use `object_id` from the object's exported variable** — never hardcode IDs

## Tuning Knobs

Object State is a foundational system with minimal tuning knobs — its behavior is defined by object schemas rather than runtime parameters.

| Knob | Type | Default | Range | What It Affects |
|------|------|---------|-------|-----------------|
| **enable_logging** | bool | true | true/false | Whether to log state changes and transition rejections for debugging. Disable in release builds. |
| **lock_state_on_scene_change** | bool | true | true/false | Whether to automatically lock StateManager during scene transitions. Prevents state changes during loading screens. |

---

### Schema-Level Tuning (Per Object Type)

These are defined in each object type's schema, not as global knobs:

| Schema Field | Example Value | What It Affects |
|--------------|---------------|-----------------|
| **INITIAL_STATE** | `"closed"` | Starting state when object is registered |
| **VALID_TRANSITIONS** | `{"closed": ["open"], "open": ["closed"]}` | Which transitions are allowed |

Designers modify these by editing the schema GDScript files. No runtime tuning interface needed for MVP.

---

### Future Tuning Knobs (Vertical Slice)

| Knob | Type | Purpose |
|------|------|---------|
| **max_persisted_objects** | int | Memory limit for serialized state data. Prevents save file bloat. |
| **state_change_history_size** | int | Number of recent state changes to keep for debugging/replay. |

## Acceptance Criteria

### Object Registration

1. **GIVEN** an interactive object with `object_id` "door_test" has not been registered, **WHEN** `has_state("door_test")` is called, **THEN** the result is `false`.

2. **GIVEN** an interactive object calls `register_object("door_test", "door")`, **WHEN** registration completes, **THEN** `has_state("door_test")` returns `true`.

3. **GIVEN** a newly registered object, **WHEN** `get_state("door_test")` is called, **THEN** the result is the schema's INITIAL_STATE (e.g., `"closed"`).

### State Queries

4. **GIVEN** object "door_test" is registered and in state "closed", **WHEN** `get_state("door_test")` is called, **THEN** the result is `"closed"`.

5. **GIVEN** object "door_test" is registered, **WHEN** `get_valid_transitions("door_test")` is called, **THEN** the result matches the schema's VALID_TRANSITIONS for the current state.

6. **GIVEN** object "door_test" is in state "closed", **WHEN** `can_transition("door_test", "open")` is called, **THEN** the result is `true`.

7. **GIVEN** object "door_test" is in state "closed", **WHEN** `can_transition("door_test", "closed")` is called, **THEN** the result is `false`.

### State Changes

8. **GIVEN** object "door_test" is in state "closed", **WHEN** `request_transition("door_test", "open")` is called, **THEN** the result is `true` and `get_state("door_test")` returns `"open"`.

9. **GIVEN** a successful state change from "closed" to "open", **WHEN** the transition completes, **THEN** the `state_changed` signal is emitted with `("door_test", "closed", "open")`.

10. **GIVEN** object "door_test" is in state "closed", **WHEN** `request_transition("door_test", "invalid_state")` is called, **THEN** the result is `false` and state remains "closed".

11. **GIVEN** an invalid transition is attempted, **WHEN** the transition is rejected, **THEN** the `transition_rejected` signal is emitted with a reason string.

12. **GIVEN** object "door_test" is in state "open", **WHEN** `request_transition("door_test", "open")` is called, **THEN** the result is `true` and no `state_changed` signal is emitted (no-op).

### Unregistered Objects

13. **GIVEN** object "nonexistent" is not registered, **WHEN** `get_state("nonexistent")` is called, **THEN** the result is an empty StringName.

14. **GIVEN** object "nonexistent" is not registered, **WHEN** `request_transition("nonexistent", "any_state")` is called, **THEN** the result is `false` and `transition_rejected` is emitted with reason `"object_not_found"`.

### Room Lifecycle

15. **GIVEN** multiple objects are registered, **WHEN** `clear_room()` is called, **THEN** `has_state()` returns `false` for all previously registered objects.

16. **GIVEN** objects are registered, **WHEN** the room is unloaded and reloaded, **THEN** all objects start at their INITIAL_STATE (MVP behavior — no persistence).

## Open Questions

| # | Question | Owner | Resolution Target | Status |
|---|----------|-------|-------------------|--------|
| 1 | Should conditional transitions (e.g., door only opens if player has key) be supported in MVP or deferred to Vertical Slice? | Game Designer | Before implementation | Open |
| 2 | Should `transition_rejected` signal include the object type for easier filtering by listeners? | Lead Programmer | Before implementation | Open |
| 3 | What is the naming convention for schema files? (e.g., `door_state_schema.gd` vs `state_schema_door.gd`) | Lead Programmer | Before implementation | Open |
| 4 | Should duplicate object_id registration log a warning or error? What happens in release builds? | Lead Programmer | Before implementation | Open |
| 5 | Should StateManager support querying all objects of a given type? (e.g., `get_all_by_type("door")`) | Game Designer | During Interaction System design | Open |
