# Game Concept: 镜像世界 (Mirror World)

*Created: 2026-04-16*
*Status: Draft*

---

## Elevator Pitch

> It's a puzzle game where every familiar object in the world operates in reverse — doors open from the wrong side, scissors cut in the opposite direction, and writing smudges as you create it. Players must unlearn their muscle memory to progress, experiencing the invisible friction that left-handed people face every day.

---

## Core Identity

| Aspect | Detail |
| ---- | ---- |
| **Genre** | Puzzle / Perspective-shifting |
| **Platform** | PC (Steam / Epic) |
| **Target Audience** | Players interested in thoughtful, perspective-challenging experiences; fans of *Superliminal*, *The Witness* |
| **Player Count** | Single-player |
| **Session Length** | 15-45 minutes per session |
| **Monetization** | Premium (one-time purchase) |
| **Estimated Scope** | Small (4-6 weeks, solo) |
| **Comparable Titles** | Superliminal, The Witness, It Takes Two (perspective mechanic) |

---

## Core Fantasy

"Every 'correct' thing I know is wrong here — and that's the point."

The player experiences the disorientation of having their automatic, muscle-memory-driven actions constantly fail. The fantasy is not about "winning" but about the moment of realization: when the player stops fighting the mirror world and starts understanding it. This is a game about learning to see from a different perspective — literally and metaphorically.

---

## Unique Hook

Like *Superliminal*'s perception manipulation, AND ALSO every interaction in the game inverts the player's physical instincts. Unlike narrative games about left-handedness, this game makes you *feel* it in your body through gameplay — your hand wants to move one way, but the world demands the opposite.

---

## Player Experience Analysis (MDA Framework)

### Target Aesthetics (What the player FEELS)

| Aesthetic | Priority | How We Deliver It |
| ---- | ---- | ---- |
| **Sensation** | 1 | The "wrongness" of familiar objects creates cognitive dissonance; visual clarity makes the mirror effect satisfying |
| **Fantasy** | 2 | You are a traveler in a world built for the other hand |
| **Challenge** | 3 | Puzzle difficulty comes from unlearning, not complexity |
| **Discovery** | 4 | Each room reveals new rules of the mirror world |
| **Narrative** | N/A | Minimal explicit storytelling; the experience IS the message |
| **Fellowship** | N/A | Single-player experience |
| **Expression** | N/A | Not a creation game |
| **Submission** | N/A | Not a relaxation game |

### Key Dynamics (Emergent player behaviors)

- Players will naturally try the "normal" way first, fail, then experiment with alternatives
- Players will start predicting how new objects will work based on learned rules
- Players will share "aha" moments with friends — this is a game meant to be discussed

### Core Mechanics (Systems we build)

1. **Muscle Memory Inversion** — The core interaction system where "right-handed" actions fail; player must find the "left-handed" alternative
2. **Progressive Rule Discovery** — Each room introduces new mirror-world rules; earlier rooms teach, later rooms combine
3. **Zero-Penalty Failure** — Failure is the expected state; there's no time limit, health, or punishment for trying the "wrong" way
4. **Visual Consistency** — A coherent visual language tells players when something follows normal rules vs. mirror rules

---

## Player Motivation Profile

### Primary Psychological Needs Served

| Need | How This Game Satisfies It | Strength |
| ---- | ---- | ---- |
| **Autonomy** | Player chooses which puzzles to tackle in what order; multiple valid solutions | Core |
| **Competence** | The "eureka" moment of understanding a new rule is the core satisfaction | Core |
| **Relatedness** | Sharing the experience and realizations with others | Supporting |

### Player Type Appeal (Bartle Taxonomy)

- [x] **Achievers** — Completing each puzzle provides clear satisfaction
- [x] **Explorers** — Understanding the world's rules is the core loop
- [ ] **Socializers** — Not applicable
- [ ] **Killers/Competitors** — Not applicable

### Flow State Design

- **Onboarding curve**: First room is the player's own hand — writing smudges, door handles feel wrong. No tutorial text, just experience.
- **Difficulty scaling**: Starts with single-rule puzzles (door handle), builds to multi-rule combinations (door + scissors + writing in one room)
- **Feedback clarity**: Visual + audio cue when player finds the "mirror" solution — the world "clicks" into place
- **Recovery from failure**: Instant — try the other direction; there's no punishment for wrong attempts

---

## Core Loop

### Moment-to-Moment (30 seconds)

Player enters a new object/room → tries their instinct (fails) → experiments with alternatives → finds the mirror solution → experiences the "click" satisfaction → moves to next interaction.

### Short-Term (5-15 minutes)

Complete one room/area containing 3-5 mirror objects. Each area has a unified theme (kitchen, classroom, office) but each object inverts a different everyday action.

### Session-Level (30-45 minutes)

Complete 3-5 rooms across different environments. Session ends at a natural checkpoint with a moment of reflection (the player's accumulated "aha" moments).

### Long-Term Progression

The deeper the player goes, the more they understand: early puzzles test single rules, later puzzles combine multiple inversions. The progression is from "I can't do anything right" to "I understand how this world works."

### Retention Hooks

- **Curiosity**: What other everyday objects will be inverted?
- **Investment**: Wanting to see all the rooms and environments
- **Mastery**: Wanting to speed-run or find alternative solutions
- **Social**: Wanting to show friends and discuss their own handedness experiences

---

## Game Pillars

### Pillar 1: Muscle Memory as the Enemy

The game leverages the player's automatic, unconscious actions as the primary challenge. Every time the player acts on instinct, they fail. Design test: If we're debating adding a tutorial arrow vs. letting players discover the mirror mechanic through failure, this pillar says let them fail first.

### Pillar 2: Everyday Objects as Puzzles

The focus is on mundane, relatable items — not fantasy weapons or magic. A door handle, a pair of scissors, a pen, a coffee mug. Design test: If we're debating between an "exciting" fantasy item vs. a realistic everyday item for puzzle X, this pillar chooses the everyday item.

### Pillar 3: The "Aha" Moment

Every solved puzzle should deliver a moment of realization — not just completion, but genuine understanding. Design test: If two solutions exist but one feels like a clever shortcut while the other feels like insight, this pillar chooses the insightful solution.

### Pillar 4: Silent Education

The game never tells the player "this is what left-handed people feel." The experience speaks for itself. Design test: If we're debating adding explanatory text vs. letting the experience stand alone, this pillar chooses silence.

### Anti-Pillars (What This Game Is NOT)

- **NOT an educational game**: It doesn't teach facts; it creates an experience
- **NOT frustrating**: The challenge is understanding, not difficulty for its own sake
- **NOT about left-handedness explicitly**: The player discovers the theme through experience, not instruction
- **NOT a narrative game**: No dialogue, characters, or story beats
- **NOT a platformer**: No jumping on heads, no death pits

---

## Inspiration and References

| Reference | What We Take From It | What We Do Differently | Why It Matters |
| ---- | ---- | ---- | ---- |
| *Superliminal* | Perception manipulation as core mechanic | We manipulate physical instinct, not visual perspective | Validates that players accept "seeing differently" as gameplay |
| *The Witness* | Rule-discovery puzzles without text | We use everyday objects, not abstract symbols | Shows players will spend hours learning implicit rules |
| *It Takes Two* | Perspective-shifting creates empathy | We use perspective for one player, not co-op | Proves perspective-shifting can be the entire game |
| *My Left Foot* | Physical difference as daily challenge | We make it interactive, not observational | Proves audiences engage with handedness themes |

**Non-game inspirations**: The user's personal experience of breaking their right hand; discussions with left-handed friends about daily friction; the phrase "right" meaning "correct" in English (and similar linguistic biases in other languages).

---

## Target Player Profile

| Attribute | Detail |
| ---- | ---- |
| **Age range** | 18-45 |
| **Gaming experience** | Mid-core; comfortable with puzzle games |
| **Time availability** | 30-60 minute sessions |
| **Platform preference** | PC (Steam) |
| **Current games they play** | Puzzle games, narrative experiences, *Stardew Valley* |
| **What they're looking for** | A thought-provoking experience that lingers after playing |
| **What would turn them away** | Excessive frustration, "educational" tone, combat or violence |

---

## Technical Considerations

| Consideration | Assessment |
| ---- | ---- |
| **Recommended Engine** | Godot 4.x — lightweight, excellent 2D, quick iteration for puzzle prototyping |
| **Key Technical Challenges** | Designing clear visual feedback for "wrong" vs. "right" interactions; testing player expectations |
| **Art Style** | 2D stylized — clean, warm, slightly isometric or top-down |
| **Art Pipeline Complexity** | Low-Medium; each room needs custom objects but style is consistent |
| **Audio Needs** | Minimal — ambient music, clear "click" sound effects for successful mirror actions |
| **Networking** | None |
| **Content Volume** | ~5-8 rooms, each with 3-5 interactive objects = 20-30 puzzles total |
| **Procedural Systems** | None required |

---

## Risks and Open Questions

### Design Risks
- **Risk 1**: Players may find persistent failure frustrating rather than illuminating
  - *Mitigation*: Clear visual/audio cues when they're "close" to the solution; early rooms are very learnable
- **Risk 2**: The theme may feel heavy-handed if the balance is off
  - *Mitigation*: Let players draw their own conclusions; never explicitly mention handedness

### Technical Risks
- **Risk 1**: Limited — puzzle games are technically straightforward
- **Risk 2**: None identified

### Market Risks
- **Risk 1**: Niche audience — puzzle players who want "message" games are a small segment
  - *Mitigation*: The core hook (muscle memory inversion) is immediately understandable and shareable
- **Risk 2**: Hard to categorize for marketing — not a typical puzzle game
  - *Mitigation*: Lean into the unique hook in all marketing; "the game that makes you left-handed for an hour"

### Scope Risks
- **Risk 1**: Could easily expand scope by adding more rooms
  - *Mitigation*: Start with 5 rooms as MVP; expand only if time allows

### Open Questions
- **Question 1**: How do players from different cultures (different writing directions) experience the inversion?
  - *Prototype plan*: Playtest with left-handed and right-handed players from different backgrounds
- **Question 2**: Is the "aha" moment satisfying enough to carry the whole game?
  - *Prototype plan*: Build one room prototype, test if players want to continue after solving 3-5 puzzles

---

## MVP Definition

**Core hypothesis**: Players find the experience of having their muscle memory inverted creates a memorable, thought-provoking experience that they want to share with others.

**Required for MVP**:
1. One complete room with 4-5 inverted objects
2. Clear visual feedback for "wrong" vs. "right" interactions
3. Player can start, play, and complete the room in under 10 minutes
4. Exit experience that prompts reflection

**Explicitly NOT in MVP**:
- Multiple rooms (MVP tests ONE room)
- Save/load system
- Menu screens beyond "click to start"
- Any narrative elements

### Scope Tiers (if budget/time shrinks)

| Tier | Content | Features | Timeline |
| ---- | ---- | ---- | ---- |
| **MVP** | 1 room, 4 puzzles | Core loop only | 2 weeks |
| **Vertical Slice** | 3 rooms, 12 puzzles | Core + progression + feedback | 4 weeks |
| **Alpha** | 5 rooms, 20 puzzles | All features, placeholder art | 6 weeks |
| **Full Vision** | 8 rooms, 30+ puzzles | All features, polished | 8 weeks |

---

## Visual Identity Anchor

*The following section will be developed in `/art-bible` — this is a seed of the visual direction.*

**Direction**: Warm, everyday realism — the world should look like a place you know, but slightly off

**Core visual rule**: Objects that work "normally" vs. objects that are "inverted" should be immediately distinguishable through subtle visual cues (not text, not obvious indicators)

---

## Next Steps

- [ ] Get concept approval from creative-director
- [ ] Run `/setup-engine` to configure Godot and populate reference docs
- [ ] Create visual identity specification (`/art-bible`)
- [ ] Decompose concept into systems (`/map-systems`)
- [ ] Author per-system GDDs (`/design-system`)
- [ ] Validate readiness with `/gate-check`
- [ ] Prototype core loop (`/prototype mirror-room`)
- [ ] Run playtest report (`/playtest-report`)
- [ ] Plan first sprint (`/sprint-plan new`)