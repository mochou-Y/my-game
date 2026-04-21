# Systems Index: 镜像世界 (Mirror World)

> **Status**: Draft
> **Created**: 2026-04-17
> **Last Updated**: 2026-04-21
> **Source Concept**: design/gdd/game-concept.md

---

## Overview

镜像世界 is a first-person puzzle game where everyday objects operate in reverse, forcing players to unlearn muscle memory. The game requires 15 systems across 4 priority tiers. The core technical challenge is clear visual feedback distinguishing "wrong" vs. "right" interactions, while the core design challenge is making failure feel educational rather than frustrating. Systems are organized from Foundation (input, persistence) through Core (camera, state management) to Feature (interaction, inversion logic) and Presentation (rooms, menus).

---

## Systems Enumeration

| # | System Name | Category | Priority | Status | Design Doc | Depends On |
|---|-------------|----------|----------|--------|------------|------------|
| 1 | Input Handling | Core | MVP | Designed | design/gdd/input-handling.md | None |
| 2 | Player Camera | Core | MVP | Approved | design/gdd/player-camera.md | Input Handling |
| 3 | Hover Detection | Core | MVP | Designed | design/gdd/hover-detection.md | Player Camera |
| 4 | Object State | Core | MVP | Designed | design/gdd/object-state.md | Save/Load |
| 5 | Interaction System | Gameplay | MVP | Designed | design/gdd/interaction-system.md | Input Handling, Hover Detection, Object State |
| 6 | Inversion Logic | Gameplay | MVP | Designed | design/gdd/inversion-logic.md | Object State, Interaction System |
| 7 | Visual Feedback | UI | MVP | Not Started | — | Object State, Inversion Logic |
| 8 | Save/Load | Persistence | Vertical Slice | Not Started | — | None |
| 9 | Room Manager | Gameplay | Vertical Slice | Not Started | — | Save/Load, Object State, Wear Accumulation |
| 10 | Audio Feedback | Audio | Vertical Slice | Not Started | — | Object State |
| 11 | Accessibility | Meta | Vertical Slice | Not Started | — | Input Handling, Localization |
| 12 | Room Progression | Gameplay | Alpha | Not Started | — | Room Manager, Save/Load |
| 13 | Wear Accumulation | Gameplay | Alpha | Not Started | — | Object State, Interaction System |
| 14 | Menu System | UI | Alpha | Not Started | — | Save/Load, Localization, Accessibility |
| 15 | Localization | Meta | Full Vision | Not Started | — | None |

---

## Categories

| Category | Description | Systems in This Game |
|----------|-------------|----------------------|
| **Core** | Foundation systems everything depends on | Input Handling, Player Camera, Hover Detection, Object State |
| **Gameplay** | The systems that make the game fun | Interaction System, Inversion Logic, Room Manager, Room Progression, Wear Accumulation |
| **Persistence** | Save state and continuity | Save/Load |
| **UI** | Player-facing information displays | Visual Feedback, Menu System |
| **Audio** | Sound and music systems | Audio Feedback |
| **Meta** | Systems outside the core game loop | Accessibility, Localization |

---

## Priority Tiers

| Tier | Definition | Target Milestone | Design Urgency |
|------|------------|------------------|----------------|
| **MVP** | Required for the core loop to function. One room, 4-5 inverted objects, 10-minute completion. | First playable prototype (2 weeks) | Design FIRST |
| **Vertical Slice** | Required for three complete rooms with progression and polish. Demonstrates the full experience. | Vertical slice / demo (4 weeks) | Design SECOND |
| **Alpha** | All features present. Five rooms, wear system, full menus. | Alpha milestone (6 weeks) | Design THIRD |
| **Full Vision** | Localization and final polish. Eight rooms, 30+ puzzles. | Beta / Release (8 weeks) | Design as needed |

---

## Dependency Map

### Foundation Layer (no dependencies)

1. **Input Handling** — Receives raw input; everything else builds on player action
2. **Save/Load** — Persists data independently; needed by multiple systems
3. **Localization** — String lookup; no game state dependency

### Core Layer (depends on foundation)

1. **Player Camera** — depends on: Input Handling (look direction controlled by mouse)
2. **Object State** — depends on: Save/Load (states must persist)

### Core Extended Layer (depends on core)

1. **Hover Detection** — depends on: Player Camera (raycast origin)
2. **Audio Feedback** — depends on: Object State (sounds triggered by state transitions)

### Feature Layer (depends on core extended)

1. **Interaction System** — depends on: Input Handling, Hover Detection, Object State (input triggers; hover target; state modification)
2. **Inversion Logic** — depends on: Object State, Interaction System (reads current state; triggered by interaction; flips state)
3. **Visual Feedback** — depends on: Object State, Inversion Logic (visual changes based on state and inversion status)
4. **Wear Accumulation** — depends on: Object State, Interaction System (tracks interaction count per object)
5. **Accessibility** — depends on: Input Handling, Localization (remapping controls; text scaling affects localized strings)

### Presentation Layer (depends on features)

1. **Room Manager** — depends on: Save/Load, Object State, Wear Accumulation (loads room; manages object states; persists wear)
2. **Room Progression** — depends on: Room Manager, Save/Load (triggers room change; saves progress)
3. **Menu System** — depends on: Save/Load, Localization, Accessibility (save/load UI; localized text; accessible UI)

---

## Recommended Design Order

| Order | System | Priority | Layer | Agent(s) | Est. Effort |
|-------|--------|----------|-------|----------|-------------|
| 1 | Input Handling | MVP | Foundation | game-designer | S |
| 2 | Object State | MVP | Core | game-designer | M |
| 3 | Player Camera | MVP | Core | game-designer | S |
| 4 | Hover Detection | MVP | Core Extended | game-designer | S |
| 5 | Interaction System | MVP | Feature | game-designer | M |
| 6 | Inversion Logic | MVP | Feature | game-designer | M |
| 7 | Visual Feedback | MVP | Presentation | game-designer, art-director | M |
| 8 | Save/Load | Vertical Slice | Foundation | game-designer | M |
| 9 | Room Manager | Vertical Slice | Presentation | game-designer, level-designer | M |
| 10 | Audio Feedback | Vertical Slice | Core Extended | game-designer, audio-director | S |
| 11 | Accessibility | Vertical Slice | Feature | game-designer, accessibility-specialist | M |
| 12 | Room Progression | Alpha | Presentation | game-designer | S |
| 13 | Wear Accumulation | Alpha | Feature | game-designer | S |
| 14 | Menu System | Alpha | Presentation | game-designer, ux-designer | M |
| 15 | Localization | Full Vision | Foundation | game-designer, localization-lead | M |

---

## Circular Dependencies

- None found

---

## High-Risk Systems

| System | Risk Type | Risk Description | Mitigation |
|--------|-----------|------------------|------------|
| **Object State** | Technical | Central bottleneck — 6 systems depend on it. Changes cascade widely. | Design with stable interface; define state schema early; prototype before other systems |
| **Inversion Logic** | Design | Core novelty — no reference implementations exist for "muscle memory inversion" | Prototype multiple inversion types early; playtest to validate "aha" feeling |
| **Visual Feedback** | Design | Must clearly signal "wrong" vs. "right" without breaking immersion or being heavy-handed | Multiple visual language iterations; art-director collaboration; playtest for clarity |

---

## Progress Tracker

| Metric | Count |
|--------|-------|
| Total systems identified | 15 |
| Design docs started | 6 |
| Design docs reviewed | 1 |
| Design docs approved | 1 |
| MVP systems designed | 6/7 |
| Vertical Slice systems designed | 0/4 |
| Alpha systems designed | 0/3 |
| Full Vision systems designed | 0/1 |

---

## Next Steps

- [ ] Design MVP-tier systems first (use `/design-system [system-name]`)
- [ ] Run `/design-review` on each completed GDD
- [ ] Run `/gate-check pre-production` when MVP systems are designed
- [ ] Prototype the highest-risk system early (`/prototype inversion-logic`)
