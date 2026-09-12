# The Long Winter

A first-person, choice-driven narrative game set during Rohan's historical
"Long Winter" (T.A. 2758–2759) — the real, canonical winter described in
Appendix A of *The Lord of the Rings*, in which Rohan is invaded on two
fronts, King Helm Hammerhand is besieged in the Hornburg, and Gondor cannot
send aid.

You play as Ceolwyn, a fence-mender with no soldier's training, from the
first raid on your holding through the siege itself — never as a hero of
legend, always as one of the people history doesn't write songs about.

## Status

Pre-production. Chapter I ("Wídfell") design and technical documentation
complete. No playable build yet.

## Design Pillars

- **Unbroken first person.** No cutscenes — the camera never leaves the
  player character's eyes.
- **No death as a default.** Losing a fight changes the story; it doesn't
  end it — except for a small number of scripted, narratively-signposted
  encounters where the fiction makes the stakes explicit.
- **State, not flags.** The world tracks concrete facts (who's alive, what
  was said, what was carried), and characters reference those facts
  specifically — never an abstracted reputation meter.
- **A guaranteed-reachable best ending.** Nothing critical is locked behind
  a single, easy-to-miss interaction.
- **Miniature-sourced art.** Painted Middle-earth Strategy Battle Game
  miniatures, photogrammetry-scanned, supply the game's visual identity.

## Documentation

| Doc | Description |
|---|---|
| [`/docs/chapter-1-gdd.md`](docs/chapter-1-gdd.md) | Game design document, Chapter I |
| [`/docs/chapter-1-tdd.md`](docs/chapter-1-tdd.md) | Technical design document, Chapter I |

More chapters and a series-wide architecture doc to follow as they're written.

## Tech Stack

- Unreal Engine 5 (Nanite, World Partition)
- ink (inkle) for dialogue/branching narrative
- RealityCapture for photogrammetry reconstruction

## Legal / IP Note

This is a personal, non-commercial project at this stage. It is set in
J.R.R. Tolkien's Middle-earth and references miniatures from Games
Workshop's *Middle-earth Strategy Battle Game* line. It is not affiliated
with, endorsed by, or sponsored by the Tolkien Estate, Middle-earth
Enterprises, HarperCollins, Games Workshop, or any related rights holder.
No copyrighted third-party assets (scanned models, textures, published
text) are included in this repository — only original design writing,
original code, and original art assets are version-controlled here.
