# THE LONG WINTER
## Technical Design Document — Chapter I: "Wídfell"

**Status:** Living document, v0.1
**Companion document:** Chapter I Game Design Document (v0.1)
**Scope:** Engineering and technical implementation for Chapter I only. Engine-wide architecture is specified here first since Chapter I is the vertical slice that has to prove it.

---

## 1. Engine Recommendation

**Unreal Engine 5.**

Reasoning specific to this project's constraints:

- **Nanite** handles high-poly photogrammetry-derived meshes (armor detail, cloth, weathering from painted minis) without the manual decimation pass that would otherwise eat a huge amount of solo/small-team time.
- **World Partition** is close to a direct answer to "no loading screens, ever" — it streams the Holding/Dooryard/Track continuous space in and out as the player walks, which is exactly the geometry Chapter I needs (Section 3 of the GDD: one continuous walkable space, three zones, no menus).
- **Level Sequencer is explicitly not used for story beats.** This needs to be a standing rule in the engine project, not just a design intention — story beats are driven by gameplay-state triggers and live NPC behavior, not baked sequences, or the "no cutscenes" pillar erodes the first time someone's in a hurry.
- Mature photogrammetry-to-game pipeline (RealityCapture → UE5 is a well-worn path) and strong C++/Blueprint hybrid workflow suits a small team that needs designers iterating without waiting on engineers for every dialogue change.

---

## 2. Core Systems Architecture — Overview

Four systems do almost all the work in Chapter I, and their boundaries matter because they're going to be reused, unchanged in principle, for every later chapter:

```
┌─────────────────┐     ┌──────────────────┐
│   World State    │◄───►│  Dialogue/Event   │
│   (persistent)    │     │  System            │
└─────────────────┘     └──────────────────┘
         ▲                        ▲
         │                        │
         ▼                        ▼
┌─────────────────┐     ┌──────────────────┐
│  Combat System    │     │  Perception/AI     │
│  (encounter-local) │◄───►│  (raiders)          │
└─────────────────┘     └──────────────────┘
```

- **World State** is the single source of truth for every fact tracked in GDD Section 9. Everything else reads from it and writes to it. Nothing else is allowed to hold its own duplicate copy of a game-relevant fact.
- **Dialogue/Event System** reads World State to select lines/behavior and writes to it when a scene resolves. This is where "state, not flags" is actually enforced or violated, so its data model gets its own section below.
- **Combat System** is stateless between encounters except for what it writes back to World State at resolution (who was hurt, who won, what was lost).
- **Perception/AI** governs the raiders specifically for Chapter I: dooryard scouts (Section 5) and the pursuing band (Section 7).

---

## 3. World State — Data Model

Implemented as a single replicated (for future co-op/spectator tooling, even if unused at launch) data asset: `UWorldStateManager`, holding a flat, namespaced dictionary rather than nested objects, specifically so that:

- Writes are simple, auditable, and diff-able in save files (important for QA reproducing bug reports — "what was the exact state when this broke").
- Designers can add new state keys without touching C++ — exposed as a Blueprint-callable `SetState(FName Key, FStateValue Value)` / `GetState(FName Key)` pair.

```cpp
// Simplified structure — full spec in engine repo /Source/WorldState/
USTRUCT(BlueprintType)
struct FStateValue
{
    GENERATED_BODY()
    EStateValueType Type; // Bool, Enum, Int, NameSet
    bool BoolValue;
    FName EnumValue;
    int32 IntValue;
    TSet<FName> NameSetValue; // for things like byre_losses
};
```

**Why not flags/booleans-only:** GDD Section 9 requires enums (`dooryard_outcome`) and sets (`byre_losses`) specifically so dialogue can reference the *specific* thing that happened, not just that "something" happened. The dialogue system (Section 4 below) is built to query these directly and fail a content-lint check (Section 8) if a writer tries to hardcode a generic line where a specific one is available.

**Persistence:** World State serializes to the save file in full at every scene resolution boundary (not on a timer), so a save always corresponds to a coherent, fully-resolved story state — never a mid-fight or mid-dialogue partial state. This matters enormously for a game with a canonical death branch (Section 7 fight): a player who reaches the "caught in the open" epilogue and reloads must reload into a clean, pre-resolution state, never a corrupted mid-combat one.

---

## 4. Dialogue/Event System

**Not a traditional branching dialogue tree tool (e.g., a raw Twine/ink import) used in isolation** — it's a thin authoring layer over World State queries, because the entire design promise ("characters reference specific facts") breaks down if writers can't easily see what state is available at a given node.

- Built on **ink** (inkle's narrative scripting language) embedded via a UE5 plugin, specifically because ink's native conditional syntax (`{hild_chore_helped: ... | ...}`) maps directly onto State queries with minimal custom tooling, and it's proven at scale on far larger branching projects.
- Every ink knot that produces player-facing dialogue is required (via a build-time lint script, not a style guideline) to reference at least one World State variable if one exists for that beat — this is the actual technical enforcement of "state, not flags," not just an instruction to writers.
- Camera and character behavior during dialogue is driven by the **same live animation/behavior system used outside dialogue** — an NPC delivering a line is still a fully simulated character in the world (can be walked around, interrupted by moving away, seen from any angle), never a canned two-shot. This is the direct technical answer to "no cutscenes reserved for movies."

---

## 5. Camera & Movement — Unbroken First Person

- Player capsule and camera are never detached from each other, including during the Section 6 byre sequence (climbing over debris, ducking under a collapsing beam) — these use **procedural camera lean/duck blended with root motion**, not a fixed camera cut to a scripted animation, so the player retains input control (including the ability to hesitate, back out, or fail to move fast enough) throughout.
- **Zone transitions (Holding → Dooryard → Track) are physical, not portals-with-a-black-screen.** World Partition streams the next cell in based on player proximity, at a radius generous enough (target: 60m ahead of typical walk speed) that there is no player-visible pop-in of major set dressing under normal traversal speed.
- **The Section 7 pursuit is simulated, not scripted-distance.** The raiders' band is a set of actual pathfinding agents with a real position on the Track the whole time, visible to the player over their shoulder if they look back — the "gap closing" the GDD describes is a literal, simulated distance value, not a hidden timer with a canned "they caught you" trigger. This is important specifically because the design promise is that the player can *see* the stakes; faking it with a timer would be discoverable by attentive players and undermine trust in every future "the fiction is telling you the truth" moment.

---

## 6. Combat System — Technical Spec

**This is continuous, real-time, embodied combat — not a command or menu system.** "Strike / block / shove" are inputs performed live, at whatever moment the player chooses, against animations that are actually happening in front of them; "flee" is not an input at all, it's the player physically turning and running, which the AI has to genuinely perceive and react to. The closest reference points are *Kingdom Come: Deliverance* and *Dark Messiah of Might and Magic* — grounded, weighty, reactive first-person melee — scoped down deliberately for solo/small-team feasibility per the notes below. This is the direct technical answer to the "Telltale is a walking simulator" problem: combat has to hold up as *game*, moment to moment, not just gate a story beat.

- **Input model:** strike and block are held/pressed inputs read continuously against an animation state machine, not discrete "perform action" commands with a cooldown menu. Shove is a short, high-commitment input (meaningful recovery window if it misses) used to create space or interrupt an enemy's own windup — this is the primary answer to "positioning matters," since a well-timed shove into the dooryard well or cart hitbox should be a genuine tactical option, not a gimmick.
- **Reference target: Skyrim's combat skeleton, modernized — not Kingdom Come's directional precision.** Skyrim's actual combat is mechanically simple (fixed light/heavy attacks, block, stamina-gated power attacks) and that simplicity is the right scope for a solo developer — what makes it feel dated isn't the input model, it's weightless hit feedback, a shallow stamina economy, and AI that mostly just walks at you. "Better" means closing those specific gaps, not adding directional aiming:
  - **Weighted, readable hits:** hit-stop (a few frames of freeze on impact), camera shake tuned to weapon mass, and animation-driven knockback/stagger that reads as a real collision, not a number going down silently.
  - **A stamina economy with real decisions in it**, not just a sprint-and-power-attack meter: blocking drains stamina under pressure, and a raider who breaks the player's stance (via the shove input) gets a genuine opening, not just a stagger animation for its own sake.
  - **A timing-rewarding block**, sitting between Skyrim's "hold to block, no skill involved" and Kingdom Come's precision parries: blocking just before a hit lands staggers the attacker briefly and opens a punish window, without requiring directional input to execute.
  - **AI that behaves like it's actually assessing the fight** (the EQS-driven flanking/positioning from Section 7), instead of Skyrim's straight-line approach-and-swing — this is the single biggest "modernized" delta and it's also the one that's genuinely achievable solo, since it's tuning UE5's existing tools rather than building new animation systems.
  - This can be deepened further in a later chapter once the pipeline is proven — but the Chapter I baseline above is the actual, committed scope.
- **Hitboxes are simple capsule/box primitives per weapon**, not per-bone precise, and combat is tuned around **wind-up/recovery windows** (strike: ~0.4s telegraph, ~0.3s recovery) rather than frame-perfect parries — keeps animation work and tuning cheap, and matches the design intent that skill comes from positioning and decision-making, not twitch precision.
- **Stamina, not health, is the primary resource in most encounters.** Ceolwyn has a real health pool, but it depletes slowly relative to stamina — losing a fight in the "recoverable" category (GDD Section 5) is triggered by stamina depletion plus a scripted "overwhelmed" state, not health hitting zero, which is deliberately reserved for the rare canonical-death encounters (GDD Section 7) so the two failure types are mechanically distinguishable under the hood even though they look the same to the player in the moment.
- **Terrain interaction objects** (the dooryard well, cart, byre door; the Track's chokepoint) are authored as simple tagged volumes the combat AI queries for pathing and positioning preference — this is what makes positioning matter mechanically, mirroring the tabletop skirmish footprint the design explicitly wants, and it's the same tagging system the AI's EQS queries (Section 7) read from.
- **Win/lose resolution writes directly to World State** (Section 3) at the moment an encounter resolves, and nowhere else — combat code has no knowledge of dialogue or narrative consequence; it only ever reports outcome facts.

---

## 7. Perception/AI — Raiders

**Built on Unreal's stock AI stack — Behavior Tree + Environment Query System (EQS) + AI Perception — rather than a custom architecture.** This is the deliberate, solo-dev-feasible answer to "realistic, believable AI": hand-built systems like GOAP or utility AI are a substantial standalone engineering project, while UE5's built-in tools are mature, documented, and specifically designed to produce the kind of situationally-aware behavior ("is there cover here, can I actually see him, should I flank") this game needs, without months of from-scratch AI architecture work.

- **AI Perception component** gives raiders genuine sight/hearing senses — they lose track of the player behind real occlusion (a byre wall, the tree line), not a scripted line-of-sight fake. This is what makes "flee" function as an actual mechanic rather than an animation: the player is really trying to break the AI's sensory contact, and the AI is really trying to re-acquire it.
- **EQS drives positioning decisions** — a raider in the dooryard fight queries nearby points for cover/flanking value against the tagged terrain volumes from Section 6, so two raiders can plausibly split up and approach from different angles without any of that being hand-scripted per-encounter. This is the specific tool that makes "believable" achievable solo: you tune query weights and get emergent-feeling behavior, rather than authoring every possible tactical situation by hand.
- **Behavior Tree layers on top** for the higher-level decision (engage / hold position / retreat / call for help), kept intentionally shallow for Chapter I's two encounter types:
  - Dooryard scouts (Section 5): a simple engage-and-commit tree, since their narrative function is a tutorial fight, not a tactical showcase.
  - Track pursuit band (Section 7): a pursuit-priority tree with **persistent, simulated position** — the raiders are genuinely pathfinding toward the player's last known location the whole sequence, which is what makes the "gap closing" tension real and player-verifiable rather than faked with a hidden timer.
- All raider combat animations are driven off the **shared skeleton** described in the photogrammetry section (Section 9) — a scanned paint variant is a material/texture swap only, never a new skeletal rig or a new Behavior Tree, which is the actual technical mechanism behind "cheap visual and behavioral variety across a warband" for a solo developer.

---

## 8. Content Pipeline & Tooling for Writers/Designers

- **ink files live in version control alongside code**, reviewed the same way (pull requests), specifically so the "living documentation" goal from the GDD extends to narrative content itself — a design decision's history is visible in the same repo as its implementation.
- **A build-time lint pass** checks every dialogue node against the World State schema (Section 3) and fails the build if: a referenced state key doesn't exist, a generic fallback line exists where a specific-state line doesn't (catches accidental flag-like writing), or a state key is written but never read anywhere (catches dead tracking that would silently violate "every choice matters").
- This lint step is the actual engineering enforcement of the "guaranteed-reachable best ending" design contract from GDD Section 8 — it can't verify narrative *quality*, but it can mechanically verify that every state key the design doc says should have two routes to it does, in fact, have two writer locations in the ink files, flagged for manual QA sign-off if only one exists.

---

## 9. Photogrammetry Technical Pipeline

1. **Capture:** turntable rig, fixed lighting (polarized to reduce specular highlights on painted miniature varnish), ~150–200 photos per model at 28mm scale.
2. **Reconstruction:** RealityCapture, exported as high-poly mesh + 4K–8K texture bake.
3. **Retopology:** manual/ZRemesher pass to a game-ready poly count with clean UVs — the scanned high-poly is retained only as a **bake source** for normal/albedo/roughness maps, never shipped as the render mesh directly, per the reproportioning note from the earlier discussion (miniature sculpting conventions read as "toy" at 1:1 in dynamic lighting).
4. **Reproportion pass:** the retopologized mesh is conformed to a standardized humanoid proportion template before texture bake, correcting the chunky-weapon/exaggerated-detail issue.
5. **Rigging:** conformed to the project's shared humanoid skeleton (one for player-scale humans, no separate rig per raider) so animations and the combat system's hitbox authoring (Section 6) work identically across every reskinned enemy.
6. **Material library:** each painted scheme becomes a reusable material instance, letting a single Dunlending Warrior sculpt supply several visually distinct raiders in the Section 7 band at zero additional modeling cost.

**Chapter I specific asset list** (from the GDD, restated here as production tickets):
- Dunlending Warrior — 2 paint variants minimum (dooryard scouts, Section 5)
- Dunlending Warrior — 2–3 additional paint variants (Track pursuit band, Section 7)
- Rohan Warrior (civilian-converted) — Aldric, unarmoured state
- Custom sculpt or heavy conversion — Ceolwyn, Hild (no MESBG equivalent; flagged as an out-of-pipeline art task, not a scan)

---

## 10. Save System & the Canonical Death Branch

- Autosave triggers only at scene-resolution boundaries (Section 3), never mid-encounter.
- The Section 7 death branch resolves to a **distinct, authored epilogue scene** (per GDD) rather than a "Game Over" screen — technically, this is just another World State terminal write (`track_outcome = caught_in_open_DEATH`) that routes the *same* scene-loading system into a short alternate scene instead of Chapter II, keeping the engineering uniform: there is no special-cased "death handler," just a state value that happens to route to a different next-scene.
- On reload, the player returns to the last clean resolution boundary before the Track sequence began — per the GDD's open question, we're recommending **no separate punitive walk-back**: reload drops the player at the tree line's approach, preserving tension without adding sunk-cost frustration that would get misread as systemic punishment rather than narrative stakes.

---

## 11. Performance Targets (Chapter I Vertical Slice)

- 60fps target on mid-range current-gen hardware (Series S / equivalent PC) with Nanite/Lumen at standard settings, given the photogrammetry-sourced asset load is concentrated in a handful of hero characters, not an entire battlefield of unique models, in this chapter.
- Streaming budget: World Partition cell sizes tuned so the largest single load-in (Dooryard → Track) completes within the ~600m walk described in the GDD, well inside a comfortable streaming window at walking pace.

---

## 12. Open Technical Risks

- Ink-to-UE5 plugin maturity for the specific conditional complexity this project needs (nested state queries referencing sets, not just booleans) should be spiked early — if it can't cleanly express `byre_losses` set-membership queries, a custom lightweight scripting layer may be needed instead.
- Photogrammetry reproportioning (Section 9, step 4) is a manual art task with no fully automated solution — needs a time-boxed pilot on two or three models before committing the full Dunlending roster to the pipeline.
- Simulated pursuit AI (Section 7) needs early playtesting specifically to confirm players correctly read "the gap is closing" from the sim alone, without a HUD element — if it doesn't read clearly, the fallback is a subtle diegetic audio cue (raiders' shouting growing louder) rather than any UI, to preserve the "the fiction tells you" principle.

---

*End of Chapter I technical design document, v0.1.*
