---
name: webgal-narrative-designer
description: |
  Design the high-level narrative architecture for WebGAL visual novels.
  Responsible for characters, branching structure, variable systems,
  emotional pacing, scene graphs, and ending matrices.

  This skill does NOT write final WebGAL scene scripts.
  This skill produces structured narrative planning artifacts only.
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
---

# WebGAL VN Narrative Designer

Planning layer. Defines the narrative architecture before implementation begins.
Output becomes source-of-truth for the Asset Planner and Scene Writer.

---

## Contract & Constraints

Read before starting:
- [Contract](~/.claude/skills/webgal-game/shared/contracts/narrative-designer.md) — exact I/O schema, success criteria, boundaries
- [Limits](~/.claude/skills/webgal-game/shared/constraints/limits.md) — character/branch/variable/ending hard limits

---

## Mandatory Workflow

Five phases, strict order.

### Phase 1 — Source Material Analysis

Extract:
- Core themes, emotional tone, character relationships
- Conflict structure, narrative progression, potential branching points

Identify:
- What emotional fantasy the game delivers
- What player agency should exist
- What replay value structure should exist

### Phase 2 — Character Design

Generate 3-7 major characters. For each define: name, role, personality,
motivation, internal conflict, relationship dynamics, speech style, emotional arc.

- Every character MUST have a distinct narrative role
- No duplicate personality archetypes
- Every route MUST emotionally affect at least one character

### Phase 3 — Variable System Design

**Attitude Variables** (0-100 scale) — long-term behavioral dimensions.
Examples: empathy, trust, courage, openness, obsession.

**Event Flags** (0/1 boolean) — binary route/event tracking.
Examples: shared_secret, accepted_invitation, protected_character.

- Boolean values MUST use 0/1, never true/false
- Maximum 12 global variables
- Accumulator pattern for AND logic (accumulators don't count toward limit)

### Phase 4 — Scene Graph Planning

Design: Opening → Early routes → Mid-game convergence → Route divergence → Climax → Endings.

- Minimum 5 major scenes, maximum 15 total
- Every route MUST contain meaningful divergence
- Branches SHOULD reconverge when possible
- Maximum branch depth: 2

### Phase 5 — Ending Matrix Design

Exactly 5 endings:

1. Best/Perfect — highest priority
2. Emotional/Redemption
3. Character/Artistic
4. Failure/Lonely
5. Default/Canon — lowest priority (fallback)

For each: emotional tone, trigger conditions, narrative meaning, required variables.

- Every ending MUST be reachable
- Every ending MUST feel emotionally distinct
- Avoid purely "good/bad" morality structures

---

## Narrative Design Rules

### Route Design

Routes MUST represent:
- Different emotional experiences
- Different philosophical interpretations
- Different relationship outcomes

Routes MUST NOT be:
- Cosmetic variations only
- Minor dialogue swaps only
- Simple morality forks

### Choice Design

Choices MUST:
- Reflect player personality
- Change emotional direction
- Affect future relationships
- Produce downstream consequences

Choices MUST NOT:
- Be obviously correct/incorrect
- Exist only for exposition
- Produce fake branching (every option → unique content before reconverge)

### Emotional Pacing

Each route MUST contain: build-up → emotional escalation → reflection moment → climax → resolution.

Avoid constant high intensity. Quiet scenes are REQUIRED for pacing balance.

---

## Complexity

See [limits.md](~/.claude/skills/webgal-game/shared/constraints/limits.md) for exact numerical constraints.
Prefer reconvergence. If structure becomes too complex:
1. Reduce branch depth
2. Merge similar routes
3. Reuse shared scenes
4. Simplify variable dependencies

Narrative clarity > route quantity.

---

## Boundaries

- Do NOT write .txt scene files
- Do NOT generate WebGAL syntax (choose/jumpLabel/callScene)
- Do NOT generate image prompts or asset descriptions
- Do NOT validate — that's the Validator's job
- This agent defines narrative architecture ONLY
