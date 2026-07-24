# Handoff Spec: "Paula's Results" — Care Level Loading (short)

**Figma:** node `20764:50745`, file `ORrkWZKXD7aFr12gNQC3lU`
**Reference implementation:** [care-level-loading.html](care-level-loading.html) / [care-level-loading-handoff.md](care-level-loading-handoff.md) — this screen uses the **exact same** gauge-arc and digit-reel techniques already built and verified there (including the mask/rotation math and cross-browser gotchas). This doc only covers what's different on this specific node; read the reference doc for full implementation detail.

## What's different from the reference screen

- **Title** "Paula's results" (LT Soul SemiBold, 25px, `#1C4A43`) above the Stages card — not present on the reference screen.
- **No Share PDF button.** Replaced by a static info card below the Stages card: icon + "This is your quick-start result." heading, plus body copy — "Inside the app, the full assessment goes deeper — and taking it each month is how you'll see what's really changing over time for [person living with dementia]." Same card styling as the alert containers (`#F6F9F8` bg, 12px radius, 16px padding).
- Everything else — gauge, digit reel, Care Alerts & Recommendations, Recommended actions, Care Level Results — is structurally identical to the reference screen.

## Motion — confirmed via this node's own Figma export

**Gauge arc** (`ELLIPSE` node `20764:50766`):
- `6.2308 → 8.6570` rad over a 4000ms timeline; rise completes at 3000ms (75%), holds 3000–4000ms (25%).
- Easing: `cubic-bezier(0.5,0,0.5,1) → cubic-bezier(0.5,0,0.2,1)`.
- Same native Figma `arcData` technique as the reference screen — convert via Start/Sweep %, not a raw rotation value (see reference handoff's "Rotation gotcha" — same trap applies here).

**Digit reel** (8 stacked `TEXT` nodes, evenly spaced `47px` apart: `0,47,94,141,188,235,282,329`):
- Every digit node carries the *identical* animation: `translate 0 → -282px`, rise 0–75% (3000ms), hold 75–100% (1000ms), easing `cubic-bezier(0.497,0,0.201,1)`, total 4000ms.
- Working through the math (`top_i + translateY` swept against the 35px visible window) confirms this smoothly sweeps `1→2→3→4→1→2→3→4` in order before holding — a full first lap plus a full second lap, ending on "4". This matches the reference screen's design intent (full first lap, second lap stops at the real result), **except** Figma's static template always completes the second lap to 4, since it doesn't know the real answer at design time. The code needs to truncate the strip / stop the reel early at whatever the actual Care Level result is (e.g. drop the last digit and stop at "3" for a level-3 result).

## Flag before implementing — do not port literally

1. **Figma's export loops both animations forever** (`animation-iteration-count: infinite`). Correct for previewing inside Figma; wrong for production — both should play once, hold on the final frame, then the screen transitions to the resolved result (same dissolve pattern as the reference screen).
2. **Gauge angle needs the Start/Sweep → CSS conversion**, not a literal rotation value — see the reference handoff for the exact math and the `mask` + `transform` cross-browser bug it documents (applying `mask` directly to a rotated element silently breaks the fill).
3. **Digit strip length is answer-dependent** — build it to stop at the real Care Level, not always run the full 8-digit template.
