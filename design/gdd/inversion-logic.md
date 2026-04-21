# Inversion Logic

> **Status**: Complete
> **Author**: [user + agents]
> **Last Updated**: 2026-04-21
> **Implements Pillar**: Pillar 1 — Muscle Memory as the Enemy

## Overview

**Inversion Logic** is the design framework that defines what "inverted behavior" means for each object type in Mirror World. It provides a catalog of inversion rules — patterns like "reverse the expected action direction," "swap cause and effect," or "mirror the interaction axis" — that designers apply when creating object state schemas. The system exists as a reference and validation tool, not as runtime logic; when a door is marked `inverted: true`, its state schema is already designed to behave counter-intuitively (e.g., pulling when you expect to push), and Inversion Logic documents why that inversion works and ensures consistency across similar objects.

The player experiences this system as the core novelty of Mirror World — every inverted object subverts their muscle memory in a learnable, consistent way. A door that pulls open when every door in their life pushes. Scissors that cut when squeezed the "wrong" way. The system's job is to make inversions feel like a coherent world rule, not random gotchas. Without Inversion Logic, each inverted object would be an arbitrary exception; with it, players can learn the mirror world's logic and predict how new objects will behave.

Player impact: Every time a player realizes "of course it works backwards here," that's Inversion Logic doing its job — creating a consistent grammar of inversion that transforms frustration into discovery.

## Player Fantasy

**The Law-Finder** — Discovering the Mirror World's Hidden Grammar

The Inversion Logic system delivers the satisfaction of discovering that the mirror world operates by **consistent laws**, not arbitrary chaos. The player's journey flows from confusion ("this object is weird") to recognition ("this type of object always works backwards") to mastery ("I can predict how NEW objects will behave").

The emotional arc: **Encounter → Struggle → Pattern Recognition → Prediction → Mastery**

The core fantasy is not solving individual puzzles — that's the Interaction System's domain. Inversion Logic owns the **pattern-level** satisfaction. When a player encounters their fourth door and *knows* to pull instead of push without trial-and-error, that's Inversion Logic working. The scattered experiences coalesce into understanding: "There are laws here. I can learn them. I can predict."

**Anchoring moment**: The player has struggled through three inverted doors — each failure felt like a random trick. They enter a new room and see a fourth door. Instead of trying the "normal" way first, they hesitate. They remember the pattern. They deliberately try the inverted action. It works. They exhale: "I understand how this world works now." The chaos has become order.

**Pillar alignment**:
- **Pillar 1 (Muscle Memory as the Enemy)**: Every failure teaches a learnable rule, not a random gotcha
- **Pillar 3 (The "Aha" Moment)**: Pattern-discovery is the core satisfaction — understanding the system, not just solving the puzzle
- **Pillar 4 (Silent Education)**: The grammar is never stated; the player discovers it through consistent application

**Design test**: If we're debating whether an inversion should be "clever and unique" vs. "consistent with established patterns," this fantasy says choose consistency — the player's joy comes from finding laws, not solving exceptions.

## Detailed Design

### Core Rules

**Rule 1: Inversion Rule Definition**

An **Inversion Rule** is a formal specification that defines how an object type's behavior differs from the player's muscle-memory expectation. Each rule has seven components:

| Component | Description | Example |
|-----------|-------------|---------|
| **Rule Name** | Unique identifier | `SPATIAL_AXIS_INVERSION` |
| **Object Types** | Which object types this applies to | `door`, `drawer` |
| **Normal Behavior** | What muscle memory expects | Push forward to open door |
| **Inverted Behavior** | What the mirror world delivers | Pull backward to open door |
| **Schema Implementation** | How the object's schema encodes the inversion | `trigger_direction: "backward"` |
| **Player Learning Arc** | Expected discovery → mastery progression | Fail push → try pull → succeed → predict pull for all doors |
| **Consistency Check** | Validation criteria for designer use | All inverted doors must use `trigger_direction: "backward"` |

**Rule 2: Inversion Rule Catalog**

The Inversion Rule Catalog is a design-time reference document maintained in this GDD. It is not implemented as runtime code — it guides schema authoring and design review.

**Catalog Organization:**

| Pattern ID | Pattern Name | Description | Applies To |
|------------|--------------|-------------|------------|
| `PATTERN_A` | Spatial Axis Inversion | Reverse the physical direction of interaction | Door, Drawer |
| `PATTERN_B` | Temporal Inversion | Effect occurs at unexpected time (delayed or early) | Scissors, Light Switch |
| `PATTERN_C` | Directional Inversion | Input axis mirrored | Pen (writing direction) |

**Rule 3: Schema Encoding Principle**

Inverted objects MUST have different schemas than normal objects. The `inverted: bool` flag is metadata for Visual/Audio Feedback — it signals "show cool tones" — but the behavioral difference is encoded in the schema itself.

**Example: Door Schema Comparison**

```gdscript
# Normal door schema
const TRIGGER_CONDITION := "player_pushes_forward"
const ANIMATION := "door_swing_outward"

# Inverted door schema (same states, different triggers/animation)
const TRIGGER_CONDITION := "player_pulls_backward"
const ANIMATION := "door_swing_inward"
```

Both schemas have the same Object State structure (`closed` ↔ `open`), but differ in:

- Trigger condition (what player action initiates the transition)
- Animation (visual feedback for the transition)

**Rule 4: Pattern Consistency Per Object Type**

All inverted instances of the same object type MUST use the same inversion pattern.

- ✅ Correct: All inverted doors use Pattern A (Spatial Axis: pull instead of push)
- ❌ Incorrect: Door A uses Pattern A, Door B uses Pattern B (delayed open)

**Rationale**: The "Law-Finder" fantasy requires consistent laws. If inverted doors behave differently from each other, players cannot predict — they must trial-and-error each door individually.

**Exception**: Different object subtypes (hinged door vs sliding door) may use different patterns IF they are visually distinct so players know which law applies.

**Rule 5: One Pattern Per Object (MVP)**

Each object applies exactly ONE inversion pattern. Pattern combinations (e.g., a door that pulls AND opens with a delay) are excluded from MVP scope.

**Rationale**: Multiple inversions per object increase cognitive load and risk frustration. The player should unlearn ONE instinct per interaction.

**Future (Vertical Slice)**: Pattern combinations may be introduced for advanced puzzles, requiring design review.

**Rule 6: MVP Inversion Pattern Assignments**

| Object Type | Pattern | Normal Behavior | Inverted Behavior |
|-------------|---------|-----------------|-------------------|
| **Door** | `PATTERN_A` (Spatial Axis) | Push forward to open | Pull backward to open |
| **Drawer** | `PATTERN_A` (Spatial Axis) | Pull toward self to open | Push away to open |
| **Scissors** | `PATTERN_B` (Temporal) | Cut happens during squeeze | Cut happens on release |
| **Pen** | `PATTERN_C` (Directional) | Marks form in motion direction | Marks form in reverse direction (or fade instead of form) |

**Rule 7: Designer Validation Checklist**

Before marking an object as `inverted: true`, the designer must verify:

1. ☐ The object's schema differs from the normal schema (Rule 3)
2. ☐ The inversion follows the assigned pattern for this object type (Rule 4)
3. ☐ Only one inversion pattern is applied (Rule 5)
4. ☐ Visual Feedback documentation specifies the cool-tone treatment for this object
5. ☐ The player learning arc is documented (what the player discovers)

**Rule 8: No Runtime Inversion Logic**

The Inversion Logic system has NO runtime code. It exists as:

- This design document (reference for designers)
- The schema differences in each inverted object (actual behavior)
- The `inverted: bool` flag (metadata for Visual/Audio Feedback)

There is no `InversionLogic` autoload or manager class. The "system" is the design pattern, not a code module.

### States and Transitions

The Inversion Logic system has no runtime states — it is a design-time framework, not an active game system.

**Design-Time States** (metaphorical):

| State | Description | Owner |
|-------|-------------|-------|
| **Undefined** | No inversion pattern assigned to object type | Designer decision pending |
| **Defined** | Inversion pattern assigned, schema authored | Designer complete |
| **Validated** | Schema passes consistency check | Design review complete |

These states describe the design workflow, not game runtime.

### Interactions with Other Systems

**Object State** (upstream, designed)

| Interface | Direction | Purpose |
|-----------|-----------|---------|
| Schema structure | Reference | Inversion Logic defines HOW schemas differ for inverted objects; Object State stores the schemas |
| `get_object_type()` | Query | Determine which inversion pattern applies to an object |

**Contract**: Object State owns schema storage; Inversion Logic owns schema difference patterns. An inverted door's schema is stored in Object State, but Inversion Logic documented that the schema should differ by trigger direction.

**Interaction System** (upstream, designed)

| Interface | Direction | Purpose |
|-----------|-----------|---------|
| `object_inverted: bool` in signals | Reference | Interaction System passes the flag to Visual/Audio Feedback; Inversion Logic explains what the flag means |
| Schema differences | Alignment | Interaction System processes all objects uniformly; inversion is in schema, not in interaction logic |

**Contract**: Interaction System does NOT treat inverted objects differently at runtime. The schema difference creates the behavioral inversion; the Interaction System just executes transitions.

**Visual Feedback** (downstream, not designed)

| Interface | Direction | Purpose |
|-----------|-----------|---------|
| `inverted: bool` flag | Signal | Visual Feedback applies cool tones when `true`, warm tones when `false` |
| Pattern-to-visual mapping | Reference | Inversion Logic may specify that Pattern A objects have specific visual cues (e.g., handle on opposite side) |

**Contract**: Visual Feedback does NOT determine inversion behavior — it only signals it visually. The behavior comes from schema design.

**Audio Feedback** (downstream, not designed)

| Interface | Direction | Purpose |
|-----------|-----------|---------|
| `inverted: bool` flag | Signal | Audio Feedback may apply different sound cues for inverted objects |

**Room Manager** (downstream, Vertical Slice, not designed)

| Interface | Direction | Purpose |
|-----------|-----------|---------|
| Schema validation on load | Query | Room Manager could validate that inverted objects have different schemas than normal ones (future enhancement) |

## Formulas

The Inversion Logic system has no runtime formulas or mathematical calculations. It is a design-time rule library that defines patterns through prose specifications and lookup tables.

**Why no formulas exist:**

- Pattern assignments are declarative mappings (object type → pattern), not calculations
- Consistency validation is a logical rule (same type → same pattern), not a numeric formula
- Schema differences are authored by designers, not computed at runtime

**Validation rules** (logical, not mathematical):

| Rule | Condition |
|------|-----------|
| Pattern Consistency | For all objects A, B: if `type(A) == type(B)` AND `inverted(A) == inverted(B) == true`, then `pattern(A) == pattern(B)` |
| Single Pattern | For any object O: `count_patterns_applied(O) == 1` |
| Schema Difference | For any object O marked `inverted: true`: `schema(O) != schema_normal(type(O))` |

These are design-time validation checks, not runtime formulas.

## Edge Cases

**EC1: Designer Marks Object as Inverted But Schema Is Identical to Normal**

*Situation*: A designer sets `inverted: true` on a door, but the schema is identical to a normal door (same trigger condition, same animation).

*Behavior*: This is a design-time error. During design review, the consistency check (Rule 7) should catch this. At runtime, the door behaves normally despite the `inverted: true` flag, which violates the core promise.

*Resolution*: The designer must modify the schema to differ from the normal schema (different trigger condition, animation, or both). The validation checklist (Rule 7) should be completed before marking an object as inverted.

*Rationale*: An inverted object that behaves normally breaks the "Law-Finder" fantasy — the visual signal says "this is backwards" but the behavior says "this is normal."

---

**EC2: Two Objects of Same Type Use Different Inversion Patterns**

*Situation*: Door A uses Pattern A (Spatial Axis: pull to open), Door B uses Pattern B (Temporal: delayed open). Both are marked `inverted: true`.

*Behavior*: This violates Rule 4 (Pattern Consistency Per Object Type). During design review, this should be flagged as an error.

*Resolution*: Choose one pattern for all inverted doors. Either redesign Door B to use Pattern A, or justify why Door B is a different subtype (e.g., "sliding door") with clear visual differentiation.

*Rationale*: Players learn "inverted doors pull." If one door breaks this pattern, prediction fails and the "Law-Finder" fantasy is compromised.

---

**EC3: Object with Multiple Valid Inversion Patterns**

*Situation*: A designer wants a door that both pulls to open AND opens with a delay (Pattern A + Pattern B combined).

*Behavior*: This violates Rule 5 (One Pattern Per Object) for MVP. The design is rejected for MVP scope.

*Resolution for MVP*: Choose ONE pattern. A door that pulls is sufficient inversion; adding a delay is excessive for a single object.

*Future (Vertical Slice)*: Pattern combinations may be allowed with design review. Document the combined pattern and its player learning arc.

*Rationale*: Multiple inversions per object create frustration, not discovery. The player should unlearn ONE instinct per interaction.

---

**EC4: Object Type with No Assigned Inversion Pattern**

*Situation*: A new object type (e.g., "lamp") is added to the game, but no inversion pattern is assigned in Rule 6.

*Behavior*: Designers cannot create inverted versions of this object type until a pattern is assigned.

*Resolution*: Before adding inverted lamps, the designer must:

1. Determine which existing pattern applies (if any)
2. OR define a new pattern (Pattern D) and add it to the catalog
3. Document the pattern's schema implementation

*Rationale*: Inversion patterns are the "laws" players discover. Ad-hoc inversions without pattern documentation create inconsistencies.

---

**EC5: Player Encounters Object Type for the First Time**

*Situation*: The player has never seen a pen before. They don't know whether it's normal or inverted.

*Behavior*: The Visual Feedback system signals the object's status: warm tones for normal, cool tones for inverted. The player can attempt an interaction and observe the result.

*Resolution*: This is the intended discovery loop. The player tries their instinct (normal behavior), fails (if inverted), then experiments. The visual signal gives a hint, but the player must still discover the specific inversion.

*Rationale*: Silent Education (Pillar 4) — the game never tells the player the rule; they discover it through consistent feedback and experimentation.

---

**EC6: Inverted Object That Behaves "More Conveniently" Than Normal**

*Situation*: An inverted door opens automatically when approached (no button press needed), which is actually easier than the normal door.

*Behavior*: This is a valid inversion IF it subverts muscle memory. The player expects to press a button; the door opens without input. The instinct ("I need to interact") is inverted.

*Resolution*: This inversion is acceptable because it violates expectation, even if the result is "better." The player's muscle memory was wrong — that's the core experience.

*Rationale*: Inversion is about subverting instinct, not about making things harder. A "convenient" inversion still delivers the "wrongness" feeling.

---

**EC7: All Objects in a Room Are Normal (No Inversions)**

*Situation*: A room contains 4 objects, all with `inverted: false`.

*Behavior*: This is valid. Not every room needs inverted objects. The room may serve as a "control" experience or a pacing break.

*Resolution*: No action needed. The player experiences "normal" interactions, which may provide relief before the next inverted room.

*Rationale*: Contrast enhances the inversion experience. A room of normal objects makes the next inverted room more impactful.

---

**EC8: All Objects in a Room Are Inverted**

*Situation*: A room contains 4 objects, all with `inverted: true`.

*Behavior*: This is valid but demanding. The player must unlearn for every interaction. May be appropriate for late-game rooms.

*Resolution*: Consider pacing. Early rooms should have a mix of normal and inverted objects to allow learning. Late-game rooms may be fully inverted for challenge.

*Rationale*: Full inversion rooms are high cognitive load. Use sparingly and escalate through the game.

## Dependencies

**Upstream Dependencies** (systems this one depends on)

| System | Status | Interface Used | Purpose | Hard/Soft |
|--------|--------|----------------|---------|-----------|
| **Object State** | Designed | Schema structure, `get_object_type()` | Inversion Logic defines HOW schemas differ; Object State stores them | Soft — Inversion Logic is design-time documentation, not runtime code |
| **Interaction System** | Designed | `object_inverted: bool` in signals | Alignment — confirms inversion is in schema, not runtime logic | Soft — documentation alignment only |

**Downstream Dependents** (systems that depend on this one)

| System | Status | Interface Provided | Purpose | Hard/Soft |
|--------|--------|-------------------|---------|-----------|
| **Visual Feedback** | Not designed | Inversion pattern definitions, `inverted: bool` meaning | Visual Feedback signals inversion through cool tones; Inversion Logic defines what inversion means | Soft — reference documentation |
| **Audio Feedback** | Not designed | `inverted: bool` meaning | Audio Feedback may apply different cues for inverted objects | Soft — reference documentation |

**No Circular Dependencies**

Inversion Logic is a design-time framework with no runtime code. It cannot have circular dependencies because it has no runtime dependencies — it only documents patterns that other systems reference.

**Dependency Nature**

All dependencies for Inversion Logic are **soft** because this system is design-time documentation, not runtime code. Other systems reference this GDD to understand what "inverted" means, but they do not call into an Inversion Logic module at runtime.

## Tuning Knobs

Inversion Logic is a design-time rule library with no runtime code. It has no traditional tuning knobs — the "knobs" are design decisions documented in this GDD, not runtime variables.

**Design-Time Parameters** (not runtime knobs):

| Parameter | Description | Where It's Defined | Who Adjusts It |
|-----------|-------------|-------------------|----------------|
| **Inversion pattern catalog** | The set of available patterns (A, B, C, etc.) | Rule 2 in this GDD | Designer (via GDD update) |
| **Pattern-to-object-type mapping** | Which pattern applies to each object type | Rule 6 in this GDD | Designer (via GDD update) |
| **Schema implementation rules** | How each pattern modifies schemas | Per-pattern documentation | Designer (when creating schemas) |

**Why no runtime knobs exist:**

- Inversion patterns are declarative mappings, not calculations that could be tuned
- Schema differences are authored by designers, not computed from parameters
- Consistency rules are binary (pass/fail), not tunable thresholds
- The `inverted: bool` flag is metadata for Visual/Audio Feedback, not a tuning knob for this system

**Future Consideration (Vertical Slice)**:

If a validation tool is built to check inversion consistency at design time, it might have these knobs:

| Knob | Type | Default | Purpose |
|------|------|---------|---------|
| `warn_on_pattern_mismatch` | bool | true | Emit warning if two objects of same type use different patterns |
| `error_on_identical_schema` | bool | true | Emit error if inverted object schema matches normal schema |

These would be editor-time validation settings, not runtime game parameters.

## Acceptance Criteria

### Pattern Definition

1. **GIVEN** an inversion pattern is defined in Rule 2, **WHEN** a designer reviews the pattern, **THEN** the pattern has all seven required components (Rule Name, Object Types, Normal Behavior, Inverted Behavior, Schema Implementation, Player Learning Arc, Consistency Check).

2. **GIVEN** a new object type is added to the game, **WHEN** the designer wants to create inverted versions, **THEN** an inversion pattern is assigned in Rule 6 for that object type.

3. **GIVEN** the pattern catalog (Rule 2), **WHEN** reviewed, **THEN** at least three patterns exist for MVP (Spatial Axis, Temporal, Directional).

### Schema Encoding

4. **GIVEN** an object marked `inverted: true`, **WHEN** its schema is compared to the normal schema for that object type, **THEN** at least one schema field differs (trigger condition, animation, or other behavioral property).

5. **GIVEN** an inverted door, **WHEN** the player interacts with it, **THEN** the trigger condition is "pull backward" instead of "push forward" (Pattern A: Spatial Axis).

6. **GIVEN** an inverted scissors, **WHEN** the player squeezes, **THEN** the cut effect occurs on release instead of during squeeze (Pattern B: Temporal).

### Pattern Consistency

7. **GIVEN** two inverted doors (same object type), **WHEN** reviewing their schemas, **THEN** both use the same inversion pattern (Pattern A: Spatial Axis).

8. **GIVEN** an object type with an assigned inversion pattern, **WHEN** a designer creates a new inverted instance of that type, **THEN** the instance uses the assigned pattern (not a different one).

9. **GIVEN** the MVP object type list (Rule 6), **WHEN** reviewed, **THEN** each object type has exactly one assigned inversion pattern.

### Single Pattern Per Object

10. **GIVEN** an inverted object, **WHEN** reviewing its schema, **THEN** only one inversion pattern is applied (no pattern combinations in MVP).

### Designer Validation

11. **GIVEN** the designer validation checklist (Rule 7), **WHEN** a designer marks an object as `inverted: true`, **THEN** all five checklist items are completed and documented.

12. **GIVEN** a design review, **WHEN** an inverted object is presented, **THEN** the reviewer can verify the schema differs from the normal schema.

### No Runtime Code

13. **GIVEN** the Inversion Logic system, **WHEN** searching for runtime code files, **THEN** no `InversionLogic` autoload or manager class exists.

14. **GIVEN** the `inverted: bool` flag on an object, **WHEN** tracing its usage, **THEN** it is only used by Visual/Audio Feedback systems for signaling — never by game logic to modify behavior.

### Edge Case Handling

15. **GIVEN** a designer creates an object with `inverted: true` but an identical schema to normal, **WHEN** design review occurs, **THEN** the error is caught and the schema must be modified.

16. **GIVEN** two inverted objects of the same type use different patterns, **WHEN** design review occurs, **THEN** the inconsistency is flagged and resolved.

17. **GIVEN** a room with all normal objects, **WHEN** playtesting, **THEN** the experience is valid (no requirement for inversions per room).

18. **GIVEN** a room with all inverted objects, **WHEN** playtesting, **THEN** the cognitive load is evaluated for appropriateness (may be suitable for late-game).

### Player Experience

19. **GIVEN** a player encounters an inverted door for the first time, **WHEN** they try the "normal" interaction, **THEN** it fails (or produces unexpected result), and the alternative succeeds.

20. **GIVEN** a player has encountered three inverted doors, **WHEN** they encounter a fourth inverted door, **THEN** they can predict the correct interaction without trial-and-error (if they've learned the pattern).

---

**Total Criteria: 20** | **Story Type**: Logic (design validation) | **Required Evidence**: Design review checklist in `production/design-review/` (BLOCKING for implementation sign-off)
