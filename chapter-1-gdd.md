# THE LONG WINTER
## Game Design Document — Chapter I: "Wídfell"

**Status:** Living document, v0.1
**Scope of this document:** Chapter I only. Series-wide pillars are summarized where they constrain this chapter; full series bible is a separate document.

---

## 1. Series Pillars (context for this chapter)

These are non-negotiable constraints every chapter is designed against. Chapter I is where the player *learns* them, so each one gets an explicit teaching moment below.

1. **Unbroken first person.** No cutscenes. Camera never leaves Ceolwyn's eyes. Transitions between scenes are walked, not cut.
2. **No death as a systemic concept — except where the story says otherwise.** Losing a fight is a narrative fork by default. A small number of encounters per game are true failure states, and they are always sold by the fiction, never by a meter or warning.
3. **State, not flags.** The game tracks concrete world facts (who is alive, what was said, what was carried, what was seen) and characters reference those facts specifically. No abstracted "reputation" values.
4. **Guaranteed-reachable best ending.** The best outcome is never lost to an easy-to-miss interaction. Every thread that feeds it has at least two independent ways to secure it, and the fiction always signals when a door is closing.
5. **Miniature-sourced art.** All painted MESBG models are scanned for texture/material fidelity; skeletal meshes are separate and shared across reskins.

---

## 2. Chapter I Premise

Wídfell, a fence-mending holding on the western Wold, in the first snowfall of what will become the Long Winter. Ceolwyn (player character) is alone at the holding with younger sister Hild when Wulf's Dunlending raiders reach the Wold ahead of the main war-band. The chapter runs from the first sign of smoke on the horizon to the survivors' first night on the road east.

**Design goal:** teach the player, through play rather than text, that (a) they are physically weak in a fight, (b) losing doesn't always end the game but always costs something, (c) exactly one fight this chapter is different from that rule, and (d) their choices are being tracked as specific remembered facts, not points.

**Runtime target:** 45–70 minutes depending on player choices and exploration.

---

## 3. Playable Space

One continuous, walkable space, no loading transitions. Built as three connected zones:

| Zone | Function |
|---|---|
| **The Holding** (house, byre, yard, well) | Opening exploration, low stakes, establishes Ceolwyn's daily life and relationship to Hild before the raid |
| **The Dooryard & Byre Fire** | Raid begins here. First combat encounter. First "what/who do you save" choice |
| **The Wold Track** (holding to the tree line, ~600m of walked road) | Escape. Second combat encounter. First rendezvous with other refugees |

No zone is ever presented as a menu or level-select. The player walks between them at their own pace; the raid begins based on in-world triggers (Ceolwyn finishing a task, a dog barking, smoke becoming visible) rather than a timer.

---

## 4. Opening Sequence (pre-raid)

Purpose: bond the player to Hild and the holding *through interaction*, not dialogue exposition, before either is threatened.

- Player wakes inside the house. No HUD, no prompt overlay beyond minimal context-sensitive interaction icons.
- **Optional, non-critical tasks** available in any order: mend a fence post (teaches the "hold/strike" input in a completely safe context), feed the livestock with Hild, overhear Hild worrying about the cold, help her with a small chore she's bad at.
- None of these are required to proceed, but **all of them are quietly logged as state**: `hild_bond_established = true/false per specific interaction`, not a single meter. This matters later — see Section 7.
- The player decides when to end this sequence by walking toward the yard gate, at which point smoke is first visible on the horizon. This is the only "trigger" in the whole sequence, and it's diegetic — the player caused it by choosing to go outside.

---

## 5. First Combat Encounter — The Dooryard

**Context:** Two Dunlending scouts, ahead of the main raid, are already at the byre when Ceolwyn and Hild come around from the house. This is the player's first fight, and it is designed to be lost more often than won on a first playthrough — that's intentional.

**Mechanics used (per prior combat spec):** strike, block, shove, flee. Positioning matters — the dooryard has a well, a cart, and the byre door as usable terrain, mirroring tabletop skirmish footprint.

**Outcomes:**

- **Win both:** Rare on a first attempt. Ceolwyn kills or drives off both scouts before the main band arrives. Byre survives. This is tracked as `dooryard_won = true`, feeding directly into the guaranteed-best-ending path (see Section 8).
- **Lose, but survive:** The far more common and fully intended outcome. Ceolwyn is knocked down, disarmed, or driven back — the scene does not end, the camera does not cut. Hild, or a neighbor (Aldric, introduced here for the first time, fleeing his own burning croft) intervenes to break the fight, at a cost: Aldric takes a wound here if he's the one who intervenes, tracked as `aldric_wounded_ch1 = true`, which changes his physical capability in Acts II–IV.
- **This encounter cannot end the game.** It's the tutorial for "you can lose and the story continues." No lethal fight happens until the player has been taught this rule once.

---

## 6. The Byre Fire — First Major Choice

Immediately following the dooryard fight (won or lost), the byre is burning and Hild is not where the player left her — she went back in for the family's last milk cow, against instruction, because of an earlier optional interaction (if `hild_bond_established` includes the byre chore, she explicitly says why on the way in — otherwise she's simply gone, and the player has to *find out* why from Aldric afterward, which is a colder, harder version of the same beat).

**This is a real choice with no correct answer**, presented entirely through the space, not a menu:

- Go into the byre after Hild. The smoke and structural collapse are a skill-and-timing challenge, not a QTE — pure movement and readable environmental cause-and-effect (support beams visibly failing, a widening gap in the roof).
- Go for the horse and cart at the yard's edge instead, and get Aldric (if wounded) and any surviving livestock out onto the road before the raiders' main body arrives — a real, competing use of the same window of time.

**State recorded, not a flag:** `wídfell_byre_outcome` records the *specific* combination of who went where, who got hurt doing it, and what was lost, and future dialogue refers to these specifics by name and event, never as a score.

---

## 7. Second Combat Encounter — The Track (Chapter I's Canonical-Death Fight)

**This is the encounter that establishes death is real.**

**Diegetic signaling, not a UI warning:** as the survivors flee down the Wold Track toward the tree line, the raiders' main body is now visible behind them — not an ambush, not a surprise, but a pursuit the player can see coming for a sustained stretch of the walk. Aldric (or whoever is present) says, plainly, in-fiction: *"If they catch us in the open, that's it. Get to the trees."* There is no ambiguity about the stakes because the fiction states them outright — this is the load-bearing line the whole "no death" trust is built on, and it must land as a character being honest, not a game being helpful.

- If the player is caught in the open (failed to reach the tree line before the raiders close the distance — determined by how much time was spent in Sections 5–6, not a hidden clock the player can't perceive: the raiders are *visibly, physically* closing the gap the whole time), the resulting fight is winnable only under specific terrain conditions (the tree line's narrow deer path, usable exactly the way a tabletop skirmish uses a chokepoint). Losing this fight **is a game over** — the only one in Chapter I — and it is followed immediately by a distinct, authored epilogue scene from another surviving character's perspective finding the aftermath, not a "you died" screen. It functions as a canonical bad ending, not a failstate to retry blindly (though the player can, of course, load back and try again).
- If the player reaches the tree line in time, the fight (if it happens at all) is against one or two stragglers only, using the same mechanics but firmly in "lose and the story continues" territory.

This single encounter is what teaches the player, once, early, and unambiguously: *most fights are survivable losses, but the fiction will always tell you, out loud, through a person, when that stops being true.* No other encounter in the game needs to re-teach this.

---

## 8. Guaranteed-Best-Ending Design for Chapter I

Chapter I contributes exactly three durable facts to the eventual best ending. Each has two independent routes to secure it, per the redundancy principle:

| Best-ending contribution | Route A | Route B |
|---|---|---|
| Hild survives Chapter I in good standing with Ceolwyn | Complete the byre-chore interaction in Section 4, *and* choose to go in after her in Section 6 | Miss the chore interaction, but choose correctly in the Section 7 dialogue that follows (Aldric explains why she ran back in; player has a chance to reassure her afterward on the Track) |
| Aldric survives Chapter I able-bodied | Win the dooryard fight outright (Section 5) | Lose the dooryard fight, but succeed at the Section 7 Track chokepoint fight cleanly, which the narrative treats as Ceolwyn "making it up" to him |
| Player reaches Act II with a working weapon and a workable route east | Recover the seax during the byre sequence (it's visibly on the wall where Ceolwyn's father kept it, discoverable in Section 4's free exploration) | If missed, Aldric explicitly offers his own blade on the Track in Section 7 — reframed in dialogue as a debt, not a do-over |

**No thread in Chapter I is single-point-of-failure.** If the player misses the "soft" route to something, the game does not silently note a failure — a specific character says or does something that closes the loop the other way, so the player is never wondering after the fact whether they missed an ending purely by not clicking on something in a bedroom.

---

## 9. State Variables Introduced This Chapter

Representative, not exhaustive — full state dictionary lives in the technical design doc:

```
hild_chore_helped: bool
hild_knows_ceolwyn_cares: bool  (derived from combination of interactions, not a single toggle)
dooryard_outcome: enum [won_clean, lost_aldric_saved, lost_hild_saved]
aldric_wounded_ch1: bool
aldric_debt_to_ceolwyn: bool
seax_recovered: bool
track_outcome: enum [reached_treeline_safe, reached_treeline_fight, caught_in_open_DEATH]
byre_losses: set  (specific named livestock/items lost, referenced by name later)
```

---

## 10. Photogrammetry Assets Needed for Chapter I

- 2x Dunlending Warrior (dooryard scouts) — existing MESBG sculpts, minimum two paint variants for reuse across the pursuing band in Section 7
- 1x Dunlending Chieftain or unique conversion, held in reserve for Wulf's eventual reveal later in the game (do not use his likeness here — first sighting should be deferred)
- Ceolwyn and Hild have no direct MESBG equivalents — plan for custom sculpts or heavily converted Rohan civilian proxies, scanned the same way for pipeline consistency
- Aldric — Rohan Warrior model, unarmoured/civilian-converted for this chapter, re-scanned in armor for his Act IV appearance if he survives

---

## 11. Open Questions

- Does the Section 7 chokepoint fight need a distinct, easier "coached" version if the player has never fought before, without undermining the stakes of the scene? (Leaning: no — difficulty should come from the terrain puzzle, not raw execution skill, so it's fair without being softened.)
- How much should Hild's dialogue differ between `hild_chore_helped` true/false — full alternate lines, or the same lines with different framing? (Leaning: full alternate lines for the two or three moments that matter most; framing-only elsewhere, to keep VO scope sane.)
- Confirm whether the Section 7 death scene needs its own short save-slot/checkpoint messaging so players don't feel punished for a systemic reason (long walk back) rather than the intended narrative reason.

---

*End of Chapter I document, v0.1.*
