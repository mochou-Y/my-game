# Art Bible: 镜像世界 (Mirror World)

*Created: 2026-04-16*
*Status: In Progress*

---

## 1. Visual Identity Statement

### One-Line Visual Rule

> **"Every element should feel familiar enough to invite instinctive interaction, but carry a subtle visual 'wrongness' (cooler temperature, sharper edges, reversed grain) that the player learns to recognize unconsciously."**

This captures the tension between warmth (invitation) and alienation (uncanny). If an object looks "normal" but functions inverted, the visual distinction hasn't been applied.

### Supporting Visual Principles

**Principle 1: Warm Trust, Cool Dissonance**

*Design test*: When choosing between warm or cool tones for an inverted object, choose cooler — the temperature shift signals "this operates differently."

**Pillar served**: Muscle Memory as the Enemy

The visual temperature primes the player to question their instinct before they act. Warm environments invite habitual action; cool-toned inverted objects create a momentary pause — a split-second of "something feels wrong here." The player learns to read the visual temperature as a warning that their muscle memory will fail.

**Visual application**: Normal objects use warm, inviting tones (amber, cream, soft wood grain). Inverted objects shift cooler (slight blue cast, metallic undertone, reversed wood grain direction). The shift is subtle — not a neon sign, but learnable.

---

**Principle 2: Imperfect Blueprint**

*Design test*: When UI needs to communicate information, choose the aesthetic of a technical manual with subtle errors — slightly misaligned text, ink smudges, reversed diagrams — over polished perfection.

**Pillar served**: Silent Education

The UI's "technically correct but functionally broken" appearance teaches through visual friction. A tutorial arrow that points slightly left of center; button labels where the ink is heavier on one side; diagrams where the flow arrows bend the wrong direction. The player absorbs the theme through the presentation itself — nothing is explicitly stated, but everything is slightly off. The message is in the medium.

**Visual application**: HUD elements use blueprint-style thin lines and technical fonts. But the precision is undermined by organic imperfections — ink bleeds at corners, text stamps sit at micro-rotations, measurement lines don't quite connect. The effect is "official but unreliable."

---

**Principle 3: Tactile Evidence**

*Design test*: When showing player struggle or failure, add accumulated wear — ink stains, worn edges, scuff marks — that emerge from the player's actions rather than being pre-applied.

**Pillar served**: The "Aha" Moment

The tactile evidence builds toward the realization. When the player repeatedly fails an interaction, the object shows the struggle: ink smears from failed writing attempts, scuff marks from pulling a door the wrong way, worn texture where they gripped scissors incorrectly. The "aha" moment is visually earned — the world remembers the player's journey. This also creates a subtle parallel to the daily friction left-handed people experience: the evidence is always there if you look.

**Visual application**: Interactive objects start clean. Failed attempts add subtle wear textures that accumulate (capped at a maximum to avoid visual noise). Successful mirror interactions "clean" the object slightly — the world accepts the correct approach.

---

## 2. Mood & Atmosphere

### Overview

The game's emotional arc mirrors the player's journey from confusion to understanding. Each game state has a distinct emotional target, and visual atmosphere is the primary tool for communicating where the player is in that journey. The mood never punishes the player — even the Struggle State maintains empathy and curiosity.

---

### Game State 1: Puzzle Exploration

**When**: Player enters a room, observes objects, experiments with interactions.

**Primary Emotion**: *Curiosity tinged with gentle unease* — the feeling of walking into a familiar room where someone has moved the furniture slightly.

**Lighting Character**:
- **Time of Day**: Late afternoon (4–5 PM) — warm golden light with long, soft shadows
- **Color Temperature**: Warm base (2700–3200K) with cool accents on inverted objects
- **Contrast Level**: Medium — soft shadows that reveal form without drama
- **Direction**: Side-lit from a single window source, creating gentle modeling

**Atmospheric Adjectives**: Quiet, Inviting, Uncanny, Patient, Observant

**Energy Level**: Low, steady — contemplative engagement

**Mood-Carrying Element**: *Dust motes visible in the window light* — a subtle animation that makes the space feel lived-in and timeless. The particles drift slowly, reinforcing the "no rush" message.

---

### Game State 2: Discovery / "Aha" Moment

**When**: Player correctly identifies and executes a mirror interaction — the door opens the "wrong" way, scissors cut in the reversed direction, writing flows correctly.

**Primary Emotion**: *Delight and validation* — the satisfaction of a lock clicking open, not just solved but *understood*.

**Lighting Character**:
- **Time of Day**: Morning light (9–10 AM) — clear, fresh, optimistic
- **Color Temperature**: Warmer than exploration (3000–3500K), with a brief brightness pulse
- **Contrast Level**: Lower — clarity increases, shadows soften
- **Direction**: More diffuse, as if a second light source activates

**Atmospheric Adjectives**: Clear, Warm, Aligned, Satisfying, Brief

**Energy Level**: Momentary positive spike, then settles back to contemplative

**Mood-Carrying Element**: *A subtle "breath" of light* — the room briefly brightens, as if the space itself exhales. Shadows pull back slightly. This is a 1–2 second visual reward that fades naturally.

---

### Game State 3: Struggle State

**When**: Player has attempted the same interaction 3+ times without success. The game does not punish, but we acknowledge the friction.

**Primary Emotion**: *Empathetic frustration* — the player feels the friction, but the visual tone says "we understand, keep trying" rather than "you are failing."

**Lighting Character**:
- **Time of Day**: Overcast afternoon — flat, even light without the golden warmth
- **Color Temperature**: Neutral to slightly cool (4500–5000K) — the warmth withdraws
- **Contrast Level**: Lower — shadows become softer, flatter
- **Direction**: More diffuse, less directional — a sense of "stuckness"

**Atmospheric Adjectives**: Patient, Neutral, Accumulating, Still, Companionable

**Energy Level**: Stagnant but not oppressive — a plateau

**Mood-Carrying Element**: *Accumulating wear marks* — ink smears from failed writing attempts, scuff marks from wrong-direction door pulls, worn texture on scissors handles. These appear gradually (capped at a maximum) and *remain* until the correct solution is found, then partially fade. The world remembers the struggle.

**Design Test**: If the struggle state ever feels punishing or dark, the lighting has failed. The mood is *empathy*, not judgment — the player should feel seen, not shamed.

---

### Game State 4: Menu / Transition

**When**: Between rooms, game start, game end, pause menu. These are breathing spaces.

**Primary Emotion**: *Reflection and anticipation* — a moment to absorb what was learned and prepare for what's next.

**Lighting Character**:
- **Time of Day**: Dawn or dusk (liminal hours) — the threshold between states
- **Color Temperature**: Neutral with subtle warmth (3500–4000K)
- **Contrast Level**: Low — soft, even, non-dramatic
- **Direction**: Ambient, non-directional — a space outside of "room time"

**Atmospheric Adjectives**: Liminal, Quiet, Technical, Imperfect, Hopeful

**Energy Level**: Low, reset — a mental palate cleanser

**Mood-Carrying Element**: *Blueprint-style UI with subtle imperfections* — thin technical lines, stamped text with ink irregularities, measurement marks that don't quite connect. The UI itself teaches the theme through visual friction.

---

### Transitions Between States

| From | To | Visual Transition | Duration |
|------|-----|-------------------|----------|
| Exploration | Discovery | Light "breathes" brighter, warmth pulse | 1–2 sec |
| Exploration | Struggle | Temperature cools gradually, wear appears | Gradual (after 3rd failed attempt) |
| Struggle | Discovery | Warmth returns, wear marks partially fade | 1–2 sec |
| Any State | Menu | Fade to liminal neutral, UI emerges | 0.5 sec |
| Menu | Exploration | Fade from neutral, room light establishes | 0.5 sec |
| Discovery | Exploration (next room) | Brief fade, new room light establishes | 0.5–1 sec |

**Design Principle**: Transitions should feel organic, not mechanical. No harsh cuts — the world breathes between states.

---

## 3. Shape Language

### Overview

Shape is the second half of the visual "wrongness" signal. Temperature shifts communicate inversion through color; shape communicates it through edge quality. Normal objects use softened geometry that invites touch; inverted objects sharpen into angles that create visual tension. Together, temperature and shape create a consistent, learnable visual language: **warm + rounded = trust your instincts; cool + angular = question everything.**

---

### 3.1 Environment Geometry

**Dominant Character**: Soft geometric — everyday objects simplified into approachable, rounded forms that feel familiar and inviting.

**Shape Strategy by Object Type**:

| Object Category | Normal State | Inverted State | Visual Effect |
|----------------|--------------|----------------|---------------|
| Interactive furniture (doors, drawers) | Rounded rectangles with 8–12px corner radius; subtle edge bevels | Same forms but corners tighten to 2–4px radius; edges sharpen | Player unconsciously registers "something is different" |
| Handheld tools (scissors, pens, cups) | Organic curves where hand meets object; smooth transitions | Angular grips; faceted surfaces; crisp edge lines | The tool looks "manufactured for a different hand" |
| Flat surfaces (tables, desks, floors) | Subtle roundness at edges; soft shadow transitions | Harder edges; cleaner shadow terminators | Surface feels "clinical" or "engineered" |
| Environmental structure (walls, doorframes) | Rounded interior corners (not sharp 90°); beveled baseboards | Sharp corners; precise 90° junctions | The room itself signals its nature through architectural detail |

**Specific Prescriptions**:

1. **Corner Radius Rule**:
   - Normal objects: Minimum 8px corner radius (at 1080p resolution baseline)
   - Inverted objects: Maximum 4px corner radius
   - This creates a visible difference in silhouette when two similar objects appear side-by-side

2. **Edge Treatment**:
   - Normal objects: Beveled edges with 2–3px softness; catch light gently
   - Inverted objects: Crisp edges; sharp shadow terminators; "cut" appearance

3. **Surface Complexity**:
   - Normal objects: Organic imperfections — slight wobble in manufactured items, wood grain that follows natural curves, small surface irregularities
   - Inverted objects: Sterile precision — surfaces too perfect, grain patterns that run counter to natural direction, symmetry that feels "correct" but unsettling

**Pillar Connection**: Serves **Pillar 1: Muscle Memory as the Enemy**. The player's visual instinct says "rounded shapes are safe, sharp shapes warrant caution."

**Design Test**: If an object looks identical in shape whether normal or inverted, the shape language has failed. The player should be able to identify an inverted object from silhouette alone, without color.

---

### 3.2 UI Shape Grammar

**Strategy**: UI exists in a separate visual layer from the game world — it is the "imperfect blueprint" made manifest. Where game objects are soft and approachable, UI is technical and linear.

**Base Shape Vocabulary**:

| Element | Shape Treatment | Imperfection Applied |
|---------|-----------------|---------------------|
| Buttons | Rounded rectangles, 4px radius | Subtle rotation offset (1–2°); ink "heavier" on one side |
| Panels | Rectangular with thin (1px) borders | Border lines don't quite connect at corners |
| Progress indicators | Horizontal bars with square ends | Measurement tick marks slightly irregular in spacing |
| Icons | Simplified geometric forms | Hand-drawn quality: lines slightly wobbly, shapes slightly asymmetrical |
| Text containers | Rectangular stamps with dotted borders | Text sits at micro-rotation; stamp edges have ink irregularities |

**Pillar Connection**: Serves **Pillar 4: Silent Education**. The UI's imperfections teach the theme through visual friction.

**Design Test**: If UI elements look polished and pixel-perfect, the shape grammar has failed. Every UI element should have at least one subtle imperfection visible on close inspection.

---

### 3.3 Hero Shapes vs Supporting Shapes

**Hero Shapes (Interactive Objects)**:
- Clear silhouette from any rotation; minimal internal detail
- Stronger highlight/shadow contrast at edges
- Reduced geometric complexity — the essence of the object
- Each interactive object has a unique silhouette

**Supporting Shapes (Environment)**:
- More internal detail, less clear outline
- Lower contrast; softer transitions to background
- Groups of similar shapes create visual patterns that don't demand attention

**Attention Guidance Through Shape**:
1. Hero objects have higher shape contrast against background
2. Hero objects are simpler (easier to parse quickly)
3. Hero objects have more defined edges

**Pillar Connection**: Serves **Pillar 2: Everyday Objects as Puzzles**. Every interactive object must be instantly recognizable as "something I know how to use."

**Design Test**: In a grayscale screenshot with no color, the player should be able to identify which objects are interactive based on shape hierarchy alone.

---

### 3.4 Perspective-Neutral Shape Principles

Since the final perspective (isometric vs top-down) is undecided, these principles ensure shape language works for both:

1. **Silhouette-First Design**: Every object's silhouette must be readable from 45° elevated (isometric) AND directly above (top-down)

2. **Depth Through Shape**: Objects convey three-dimensional form through silhouette variation, not relying solely on lighting/shading

3. **Planar Clarity**: Objects have distinct top, side, and front silhouettes that are all recognizable

4. **Iconic Readability**: Every interactive object should reduce to a clear icon (32×32px) without losing recognition

5. **Avoid Perspective-Dependent Details**: Don't rely on foreshortening cues; don't hide critical distinctions on faces that disappear in top-down

**Pillar Connection**: Serves **Pillar 3: The "Aha" Moment**. Clear, perspective-neutral shapes ensure recognition is consistent regardless of camera angle.

---

### Shape Language Summary

| Aspect | Normal Objects | Inverted Objects | Visual Signal |
|--------|---------------|------------------|---------------|
| Corner radius | 8–12px | 2–4px | "Softer = safer; sharper = caution" |
| Edge treatment | Beveled, soft shadows | Crisp, sharp shadows | "Organic vs manufactured" |
| Surface complexity | Organic imperfections | Sterile precision | "Lived-in vs too-perfect" |
| Temperature | Warm (2700–3200K) | Cool (shift toward blue) | Combined with shape = complete signal |

---

## 4. Color System

### Design Philosophy

The color system serves the core tension: **warm = trust your instincts, cool = question everything**. Players discover this language gradually through repeated interaction, never through explicit instruction. The shift between normal and inverted states is highly subtle — learnable over time, not immediately obvious.

---

### 4.1 Primary Palette

| Color Name | Hex | Role | Meaning |
|------------|-----|------|---------|
| **Amber Core** | `#E8A84C` | Primary warm accent | Normal-world trust, invitation, "this works as expected" |
| **Cream Base** | `#F5EDE0` | Primary background | Everyday warmth, familiar domesticity |
| **Soft Wood** | `#C4956A` | Secondary warm | Natural materials, organic presence |
| **Steel Drift** | `#7A8B9A` | Primary cool accent | Inverted-world caution, "something operates differently here" |
| **Slate Shadow** | `#4A5568` | Deep neutral | Depth, grounding, cool shadows on inverted objects |
| **Ink Suggestion** | `#2D3748` | Darkest neutral | UI text, technical overlays, blueprint marks |
| **Paper White** | `#FFFEF9` | Lightest neutral | Highlights, paper, clean surfaces |

**Why these colors?**

- **Amber Core** anchors the "normal" world — afternoon light, honey, aged paper. Signals warmth and invitation.
- **Steel Drift** is the inversion signal — cool enough to contrast with amber, desaturated enough to be learnable.
- The palette is warm-dominant (4 warm, 3 cool) because the game world is mostly normal.

---

### 4.2 Semantic Color Usage

**What warm amber communicates:**
- "This object follows your muscle memory"
- "Trust your instincts here"

**What cool steel communicates:**
- "This object inverts your expectations"
- "Pause before acting"

**Normal vs. Inverted — The Subtle Shift:**

| Object State | Base Color | Temperature Shift | Visual Effect |
|--------------|------------|-------------------|---------------|
| Normal door handle | Amber Core dominant | Warm (2700K) | Invites expected action |
| Inverted door handle | Steel Drift undertone | Cool (4500K) | Subtle visual "wrongness" |
| Normal scissors | Soft Wood handles | Warm grain direction | Grips naturally for right hand |
| Inverted scissors | Steel Drift handles | Cool metallic | "Made for the other hand" |

**The Discovery Pattern:**

Players learn through 3-5 interactions:
1. First inverted object: Interaction fails, player experiments, notices "slightly off" in retrospect
2. Second inverted object: Player pauses, notices cooler tone, tries inverted action first
3. Third+ inverted object: Player unconsciously scans for temperature before interacting

---

### 4.3 Per-Room Color Temperature Rules

Each room has a distinct identity while maintaining the warm-dominant / cool-inversion system.

| Room Type | Dominant Temperature | Base Hue | Identity |
|-----------|---------------------|----------|----------|
| **Kitchen** | Warmest | Amber dominant | "The amber kitchen" — honey jars, wooden spoons |
| **Classroom** | Neutral-warm | Cream dominant | "The cream classroom" — chalk dust, aged paper |
| **Office** | Cool-neutral | Steel undertones | "The steel office" — filing cabinets, blue-toned paper |
| **Workshop** | Warm-industrial | Soft Wood dominant | "The wood workshop" — wood shavings, tool handles |
| **Bedroom** | Soft-warm | Cream + amber mix | "The soft bedroom" — pillows, lamp glow |

**Room Identity Rules:**

1. Each room has ONE dominant hue from the primary palette
2. Inverted objects always cool-shift regardless of room
3. Lighting reinforces identity (kitchen = honey light, office = flatter cooler light)

---

### 4.4 UI Palette

The UI exists in deliberate contrast with the warm game world — the "imperfect blueprint" aesthetic.

| UI Element | Color | Hex | Rationale |
|------------|-------|-----|-----------|
| Primary UI background | Cool Slate | `#4A5568` | Technical, detached from warm world |
| Secondary UI surface | Steel Drift | `#7A8B9A` | Blueprint-style cool gray |
| UI text | Ink Suggestion | `#2D3748` | Near-black, stamped appearance |
| UI highlight | Paper White | `#FFFEF9` | Crisp technical marks |
| UI accent | Amber Ghost | `#E8A84C` at 40% opacity | Subtle warmth bridge to world |

**Contrast with Game World:**

| Aspect | Game World | UI |
|--------|------------|-----|
| Dominant temperature | Warm (amber, cream, wood) | Cool (slate, steel, ink) |
| Saturation | Moderate warmth | Desaturated, technical |
| Imperfections | Organic wear | Ink smudges, misaligned stamps |

---

### 4.5 Colorblind Safety

**At-risk semantic pairs:**

| Color Pair | Risk | Affected Vision Types |
|------------|------|----------------------|
| Amber Core vs. Steel Drift | Moderate | Protanopia, Deuteranopia |

**Backup cues required:**

| Backup Cue | Implementation |
|------------|----------------|
| Shape difference | Inverted objects have tighter corners (2-4px) vs normal (8-12px) |
| Grain direction | Wood grain on inverted objects runs counter to natural direction |
| Edge sharpness | Inverted objects have crisper shadow terminators |
| UI icon indicators | Subtle icon differences for inverted objects in menus |

**Design test:** In a grayscale screenshot, a player should identify inverted objects using shape + edge + grain cues alone. Color enhances but never carries the signal exclusively.

---

## 5. Character Design Direction

### Overview

镜像世界 is a first-person puzzle game with no visible avatar. The player exists in the world through camera presence, environmental response, and audio feedback — not through a character model.

---

### 5.1 Player Presence Without Avatar

| Presence Element | Implementation | Emotional Effect |
|------------------|----------------|------------------|
| **Camera Height** | 160–165cm eye level | Grounded, domestic-scale |
| **Camera Movement** | Smooth, weighted acceleration; subtle head bob at 0.3x intensity | Implies physical mass |
| **Idle Subtlety** | After 8 seconds, camera drifts 0.5° downward, then corrects | Mimics natural attention drift |
| **Footstep Audio** | Light footfall sounds on different surfaces; rhythm matches walk speed | Grounding presence |
| **Ambient Breath** | Very subtle breath cycle during idle; faster after rapid movement | Living presence |

**The Responsive World:**

| Trigger | Environmental Response |
|---------|------------------------|
| Player stands still 3+ seconds | Dust motes settle; ambient sounds quiet |
| Player walks near loose objects | Papers flutter; dust swirls in footsteps |
| Player enters a room | Light shifts subtly; sounds reverb briefly |
| Player exits a room | Ambient sounds fade; "empty room" texture returns |

---

### 5.2 Environmental Character

**Objects as Characters:**

| Object Type | Personality | Visual Expression | Audio Signature |
|-------------|-------------|-------------------|-----------------|
| **Doors** | Reluctant gatekeepers | Heavy, settled into frames | Low creak, longer than expected |
| **Scissors** | Precise but judgmental | Angular, sharp edges | Satisfying snip when correct; dull scrape when wrong |
| **Cups** | Passive and waiting | Rounded, open, expectant | Hollow ceramic clink |
| **Pens** | Frustrated collaborators | Ink-stained, slightly chewed | Scratch of paper; wet ink drag |

**Rooms as Moods:**

- **Kitchen**: Nurturing but cluttered — "someone was cooking here recently"
- **Classroom**: Patient and expectant — "waiting for students to return"
- **Office**: Impersonal and cold — "a workspace, not a living space"
- **Workshop**: Creative and focused — "in the middle of a project"

---

### 5.3 Cursor and Hover Language

**Interaction Philosophy**: Click/point to interact. Objects highlight on hover. No visible hand or cursor during interactions.

| Hover State | Visual Signal |
|-------------|---------------|
| **Hover Begin** | Subtle edge highlight (Amber Core at 15% opacity) fades in |
| **Hover Hold** | Highlight intensifies to 25%; object "breathes" (0.02 scale pulse) |
| **Hover Exit** | Highlight fades out |

**What the Highlight Communicates:**
- "You can interact with this"
- Does NOT indicate success or failure
- Does NOT distinguish normal vs. inverted

**Interaction Feedback (Instead of Cursor):**

| Phase | Feedback |
|-------|----------|
| Click | Screen-edge vignette (2% darkening) + click sound |
| Success | Object animation + satisfying audio |
| Failure | Object resistance + friction sound + wear mark appears |
| Discovery | Light "breath" (room warms) + object settles |

---

### 5.4 Contingency: If a Character IS Needed

**Character Design Principles:**

1. **Never show player's reflection**: Mirrors show inverted, empty room
2. **Supporting characters are shapes**: Recognizable from silhouette alone
3. **Voice over visual**: Prefer voice (distorted, distant) over visual presence
4. **Technical aesthetic**: Any character UI uses blueprint/imperfect style

**Design Test**: Before adding character, ask: "Does this enhance the player-world tension, or distract from it?"

---

## 6. Environment Design Language

### 6.1 Architectural Style

**Contemporary Chinese Domestic (2020s)**

| Element | Characteristics |
|---------|-----------------|
| **Floor plans** | Compact, efficient layouts; separate kitchen/dining |
| **Door conventions** | Interior doors open inward; bathroom doors often slide |
| **Light switches** | Toggle style, mounted at ~130cm height |
| **Windows** | Sliding aluminum frames |
| **Ceiling height** | 2.6–2.8m throughout |
| **Baseboards** | White painted wood, 8cm height |

**Why contemporary?** Maximum muscle memory — players have strong, recent experience with these objects and spaces.

---

### 6.2 Texture Philosophy

**Realistic PBR with Inversion-Specific Implementation**

| Object State | Albedo Modification | Roughness | Visual Effect |
|--------------|---------------------|-----------|---------------|
| **Normal** | Base color + warm tint (amber 5%) | Standard | Warm, inviting |
| **Inverted** | Base color + cool tint (steel blue 8%) | +0.05–0.10 | Cooler, slightly flatter |

**Wear Accumulation System:**
- Scuff marks: Darken roughness where player dragged incorrectly
- Ink stains: Darken albedo where writing failed
- Grip wear: Lighten roughness where player repeatedly gripped
- Maximum wear after 5+ failed attempts; 50% removed on successful interaction

---

### 6.3 Prop Density Rules

**Sparse and Minimal — Every Prop Has Purpose**

| Category | Purpose | Density Rule |
|----------|---------|--------------|
| Primary Interactive | Direct puzzle component | 1–3 per puzzle area |
| Secondary Interactive | Context or hint | 2–4 per room |
| Atmospheric Fixed | World-building | Determined by architecture |
| Atmospheric Loose | Lived-in signal | Maximum 2–3 per room |

**Creating "lived-in" feel without clutter:**
- Surface wear (scratches, worn paint at touch points)
- Material memory (aged wood, patinated metal)
- Light evidence (sun-faded patches, dust in corners)

---

### 6.4 Environmental Storytelling

**Story beats told through environment:**

| Story Element | Visual Evidence |
|---------------|-----------------|
| Someone lived here recently | Half-drunk cup, book left open, chair pulled out |
| Mirror logic is systematic | All inverted objects share cool tone and sharp edges |
| Left-handed people exist | Some scissors are warm but left-handed |

**The rule of three:**
- First instance: Player notices something is "off"
- Second: Player forms hypothesis
- Third: Player confirms pattern

---

### 6.5 Per-Room Architecture

**Common Elements (All Rooms):**
- Ceiling height: 2.6–2.8m
- Baseboards: White painted wood, 8cm, beveled
- Doorframes: White painted wood, 90cm wide, 210cm tall
- Wall color: Cream Base (#F5EDE0) or soft white

**Kitchen ("The amber kitchen")**
- U-shaped or galley layout
- Rice cooker, wok, dish drying rack
- Warm overhead lighting (3000K)
- Inversion targets: cabinet doors, faucet handles, rice cooker lid

**Classroom ("The cream classroom")**
- 6–8 student desks in rows
- Chalkboard (not whiteboard)
- Water dispenser
- Inversion targets: desk lid, chalkboard eraser, chair stacking

**Office ("The steel office")**
- Coolest room, impersonal
- Filing cabinet, office chair, desk lamp
- Neutral fluorescent lighting (~4000K)
- Inversion targets: filing drawer, chair lever, blind slats

**Workshop ("The wood workshop")**
- Warm-industrial, creative
- Workbench with vise, pegboard tools
- Bright task lighting (~5000K)
- Inversion targets: vise handle, tool chest drawer, sandpaper

**Bedroom ("The soft bedroom")**
- Warmest, most personal
- Wardrobe (not closet), bedside lamp
- Warm lighting (2700K)
- Inversion targets: wardrobe door, nightstand drawer, lamp toggle

---

### 6.6 Spatial Coherence

**The Single Building Principle:**

| Element | Rule |
|---------|------|
| Baseboards | Same style throughout |
| Doorframes | Same dimensions throughout |
| Floor transitions | Threshold strips at doorways |
| Electrical outlets | Same style and height throughout |
| Window frames | Same aluminum sliding style |

---

## 7. UI/HUD Visual Direction

### Overview

The UI exists as a "blueprint overlay" — a technical layer separate from the warm game world. The UI is a **truth layer** — interaction prompts accurately describe what will happen.

---

### 7.1 Diegetic vs. Screen-Space

**Strategy**: Screen-space overlay with subtle world anchoring.

| Element Type | Implementation |
|--------------|----------------|
| HUD elements | Screen-space, fixed position |
| Interaction prompts | Screen-space near object focus point |
| Menu panels | Screen-space with depth parallax |

---

### 7.2 Typography Direction

**Primary Font**: Monospace Technical

| Use Case | Size (1080p) | Weight |
|----------|--------------|--------|
| Body text | 16–18px | Regular |
| Headers | 24–28px | Bold |
| Interaction prompts | 14px | Regular |
| Labels/captions | 12px | Regular |

**Imperfection Limits**:
- Micro-rotation: Max 1 degree per text block
- Ink weight: Subtle variation in letter thickness
- Baseline wobble: Max 0.5px vertical offset (flavor text only)

**Color Application**:
- Primary text: Ink (#2D3748) on Paper White (#FFFEF9)
- Secondary text: Slate Shadow (#4A5568) on lighter backgrounds

---

### 7.3 Iconography Style

**Thin Line Icons (Blueprint Aesthetic)**

| Specification | Value |
|---------------|-------|
| Stroke weight | 1.5–2px (at 1080p) |
| Corner radius | 2px |
| Grid | 24×24px base |
| Style | Geometric, minimal detail, single-color |

**Imperfection Rules**:
- Lines slightly wobbly (max 0.5px deviation)
- Corners not perfectly sharp
- Smudges on margins only, never on functional center

---

### 7.4 Animation Feel

**Inverted Easing with Hesitation**

| Animation Type | Easing | Duration |
|----------------|--------|----------|
| Panel open | easeOutBack (0.8x) | 300ms |
| Panel close | easeInQuad | 200ms |
| Button hover | easeOutSine | 150ms |
| Fade in/out | easeInOutSine | 200–300ms |

**The "Hesitation" Effect**: 50-100ms pause before UI actions (not on player confirms)

**Motion Options**:
- Camera head bob: Toggle on/off
- Idle drift: Toggle on/off
- UI animations: Toggle on/off

---

### 7.5 Main Menu Design

**Layout**: Blueprint document on cool slate background

- Background: Cool Slate (#4A5568) with subtle blueprint grid
- Title: Ink on Paper White "stamp" rectangle, micro-rotation 0.5-1°
- Menu items: Ink text, Amber Ghost background on hover
- Imperfections: Title stamp ink heavier on one corner; grid lines have intermittent gaps

---

### 7.6 In-Game HUD Elements

**Philosophy**: Minimal HUD. The world communicates; UI is backup.

**Persistent Elements** (only when relevant):
- Room indicator: Top-left, fade after 3s
- Interaction prompt: Bottom-center on object hover
- Click confirmation: 5% screen vignette + ink stamp audio

**Truth Layer Guarantee**: Prompt text ALWAYS describes the literal action

---

### 7.7 Pause/Settings Menus

**Layout**: Reference Appendix style

- Panel: Cool Slate background, 2px border (Steel Drift), 8px corner radius
- Game world behind panel: 15% blur, 30% desaturation
- Border imperfection: Lines don't connect at 2 corners

**Settings Categories**: Audio, Display, Motion, Accessibility

---

### 7.8 Accessibility Mode

**Single Toggle Design**: One checkbox enables all accessibility features.

| Feature | Normal Mode | Accessibility Mode |
|---------|-------------|-------------------|
| Imperfections | Present | All removed |
| Contrast | WCAG AA (4.5:1) | WCAG AAA (7:1) |
| Motion | Animations active | All disabled |
| Text rotation | Max 1° | All horizontal (0°) |

---

### 7.9 Summary

| Category | Specification |
|----------|---------------|
| Typography | Monospace, Ink on Paper White |
| Icons | 1.5-2px stroke, thin line style |
| Animation | Inverted easing, 50-100ms hesitation |
| Contrast | WCAG AA minimum; AAA in Accessibility Mode |
| Click feedback | 5% vignette + ink stamp audio |
| Accessibility | Single toggle |

---

## 8. Asset Standards

### 8.1 File Format Preferences

| Asset Type | Format | Rationale |
|------------|--------|-----------|
| **Models** | `.glb` (GLTF 2.0 binary) | Native Godot support, single-file, preserves PBR |
| **Textures** | PNG (8-bit sRGB, power-of-2) | No compression artifacts on warm/cool signal |
| **Audio source** | WAV (16-bit, 44.1kHz) | Quality for editing |
| **Audio runtime** | OGG Vorbis (128kbps) | Engine compression |

---

### 8.2 Naming Conventions

| Category | Pattern | Example |
|----------|---------|---------|
| Environment props | `env_[object]_[variant]` | `env_door_wooden.glb` |
| Interactive objects | `prop_[object]_[state]_[variant]` | `prop_scissors_clean_normal.glb` |
| Materials | `mat_[surface]_[variant]` | `mat_wood_warm.tres` |
| Textures | `tex_[type]_[object]_[variant]` | `tex_albedo_door_warm.png` |
| Audio | `sfx_[action]_[object]` | `sfx_open_door.wav` |
| UI elements | `ui_[element]_[state]` | `ui_btn_primary_hover.png` |

**State suffixes**: `_normal`, `_inverted`, `_worn`

---

### 8.3 Poly Count Budgets

| Category | Triangle Count |
|----------|---------------|
| Hero Interactive Objects | 500-2,000 tris |
| Secondary Interactive | 200-800 tris |
| Architecture (per room) | 5,000-15,000 tris |
| Atmospheric Props | 50-300 tris each |
| **Total per room** | 15,000-30,000 tris |

---

### 8.4 Texture Resolution Tiers

| Asset Category | Resolution | Texture Budget |
|----------------|------------|----------------|
| Hero props (interactive) | 1,024×1,024 | 3 textures (Albedo, Normal, ORM) |
| Supporting props | 512×512 | 2 textures (Albedo, Normal) |
| Architectural elements | 512×512 or 1,024×512 | Tiling textures |
| UI elements | 256×256 minimum | Per-element |
| Wear overlays | 512×512 | 1 texture (alpha only) |

**ORM Packing**: Red = Occlusion, Green = Roughness, Blue = Metallic

---

### 8.5 Material/LOD Expectations

**Material Slots**: Maximum 2 per object

| Object Type | Slots |
|-------------|-------|
| Simple props (cup, pen) | 1 |
| Complex props (scissors) | 2 (blade + handle) |

**LOD Levels**: LOD0 required, LOD1 optional, LOD2 stretch goal

| Asset Type | LOD0 | LOD1 | Trigger |
|------------|------|------|---------|
| Small props | 200-500 tris | 100-250 tris | 5m |
| Medium props | 500-1000 tris | 250-500 tris | 5m |
| Large props | 1000-2000 tris | 500-1000 tris | 5m |

---

### 8.6 Performance Budgets

| Metric | Target | Ceiling |
|--------|--------|---------|
| Draw calls per frame | 50-80 | 120 |
| Vertex count (visible) | 30,000 | 50,000 |
| Material switches | 20 | 40 |
| Shadow casters | 10 | 25 |
| Max VRAM budget | 512 MB | — |

---

### 8.7 Export Settings

**Blender GLTF Export**:
- Format: glTF Binary (.glb)
- Apply Modifiers: Yes
- Triangulate: Yes
- Export Normals/Tangents: Yes
- Transform: +Y Up, -Z Forward

**Godot Import**:
- Albedo: sRGB ON, Mipmaps ON
- Normal: sRGB OFF, Mipmaps ON
- ORM: sRGB OFF, Mipmaps ON
- UI: sRGB ON, Mipmaps OFF

---

## 9. Reference Direction

### Reference 1: Unpacking (Game, 2021)

**What to Draw From**: Sparse prop placement as environmental storytelling
- Maximum 3-4 meaningful objects per surface, each with clear purpose
- Material warmth: wood, fabric, ceramic dominate — plastic and metal support
- Every prop must earn its place

**What to Avoid**:
- Isometric pixel art style — we use realistic PBR
- "Cozy settlement" narrative arc — our rooms are liminal spaces
- Pastel color palette — our warm/cool contrast is more clinical

---

### Reference 2: Control (Game, 2019)

**What to Draw From**: Clinical institutional atmosphere made uncanny
- Realistic architecture pushed slightly surreal through lighting and geometry
- Material language of "designed for efficiency, not comfort"
- Subtle volumetric lighting that makes spaces feel enclosed

**What to Avoid**:
- Supernatural/action elements — no floating objects, no combat
- Harsh red accent color — our danger signal is cool steel
- Brutalist concrete for environments — use for inverted objects only

---

### Reference 3: Papers, Please (Game, 2013)

**What to Draw From**: The imperfect stamp/document UI aesthetic
- Stamps that don't quite align, ink that bleeds unevenly
- Typewriter font with irregular ink weight
- The visceral "thunk" of a stamp — satisfying despite bureaucratic context

**What to Avoid**:
- Dystopian/authoritarian theme — our UI is flawed, not oppressive
- Top-down desk view — our game is first-person 3D
- Retro-1980s aesthetic — our visual language is contemporary

---

### Reference 4: Jia Zhangke Films (Cinema)

**What to Draw From**: Contemporary Chinese domestic spaces with naturalistic lighting
- Small apartments, natural clutter, lived-in wear
- Soft, diffused daylight as primary source; warm tungsten accents
- Material palette: concrete floors, painted walls, simple wood furniture

**What to Avoid**:
- Social/political commentary — our spaces are neutral
- Melancholy emotional tone — our spaces are liminal
- Handheld camera aesthetics — our camera is stable

---

### Reference 5: P.T. (Game Demo, 2014)

**What to Draw From**: The uncanny domestic corridor — familiar but increasingly wrong
- The same space repeating with subtle variations
- Lighting shifts: warm domestic light gradually cooling
- Object displacement — environmental wrongness without explicit threat

**What to Avoid**:
- Horror elements — no grotesque imagery, no jumpscares
- Aggressive visual distortion — our wrongness is always subtle and learnable
- Radio narrative — our story is environmental, not spoken

---

### Additive Summary

| Reference | Primary Contribution |
|-----------|---------------------|
| Unpacking | Sparse props, environmental storytelling |
| Control | Uncanny clinical atmosphere |
| Papers, Please | Imperfect UI aesthetic |
| Jia Zhangke | Chinese domestic authenticity |
| P.T. | Uncanny domestic mood |
