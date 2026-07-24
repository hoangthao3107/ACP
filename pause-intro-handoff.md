# Handoff Spec: "Pause" — Pre-Assessment Intro Screen (short)

**Figma:** node `20252:50012` ("Pause"), file `ORrkWZKXD7aFr12gNQC3lU`
**Not the same screen as** the existing [pause-handoff.md](pause-handoff.md) (that one is the "Hang on, we're calculating…" *results* loading screen). This is a different screen — the "Let's answer a few questions…" intro shown *before* the assessment starts, reusing the same heart illustration and the same two eye shapes (nodes `19966:20914`/`19966:20916`, within the range `19966:20912`–`19966:20917` referenced in the other Pause screen's handoff).

## Overview

An entrance sequence that reveals, in order: the heart icon's eye blink → the heading, word by word → the description card → the subtext/CTA. The heading/card/CTA reveal is one-shot (plays once, holds at its final state) over a **3808ms** timeline. The **eye blink loops forever**, per design — same "still here, thinking" behavior as the other Pause screen, just applied on top of this screen's one-shot content reveal rather than looping alongside a static heading.

## Animation 1 — Heart icon eye blink (loops indefinitely)

Only the two eye shapes animate (nodes `19966:20914` and `19966:20916`); the rest of the heart artwork (outline, face, background cloud/sparkle decoration) is static. Both eyes play identically and simultaneously.

| Property | Value |
|---|---|
| Motion | `scaleY`: 1 → 0 → 1 |
| Timing | 0ms → 180ms → 360ms per blink |
| Easing | `cubic-bezier(0.5,0,0.5,1)` both segments |
| Loop | **Yes — infinite**, per design request. Figma's raw export has this playing once within the 3808ms one-shot timeline (holding open after the first blink); override to loop the blink on its own cycle independent of — and outlasting — the rest of the one-shot reveal below, matching the always-blinking behavior in the other Pause screen's handoff. |

## Animation 2 — Heading word-by-word reveal

11 words ("Let's answer a few questions to understand Paula's current care level."), each independently: `opacity 0→1`, `blur(8px)→0`, `translateY(25px)→0`.

- Each word's reveal window is ~600ms wide (opacity/translate: `ease-out`; blur: `cubic-bezier(0.5,0,0.5,1)`).
- **Stagger: ~100ms between each word's start** (tightened from an earlier ~150ms pass — if re-fetching this node later, re-check this value, it's been adjusted at least once).
- First word ("Let's") starts at 200ms; last word ("level.") finishes revealing at ~1804ms.

## Animation 3 — Secondary content fade-in

Two more reveal waves after the heading, each a plain `opacity 0→1` (no blur/translate):

| Element | Fade window | Easing |
|---|---|---|
| Description card (+ its subtext + disclaimer icon) | ~1300ms → ~2000ms | `cubic-bezier(0.5,0,0.2,1)` (icon: `cubic-bezier(0.5,0,0.201,1)`) |
| "Takes about 2 minutes" subtext + Continue button | ~2000ms → ~2700ms | Subtext: `cubic-bezier(0.5,0,0.2,1)`; button: `cubic-bezier(0.5,0,0.5,1)` |

## Flag before implementing

1. **Eye blink loop is a deliberate override of Figma's raw export**, not a literal port — confirm this reading with design if anyone re-derives this spec from Figma directly, since the file itself only shows one blink.
2. **Word count is hardcoded to English copy** (11 words) — same localization caveat as the other Pause screen: a translated string needs the stagger built to work on however many tokens it splits into, not 11 fixed elements.
3. **All timings above are relative to the 3808ms total** — if that total changes again, everything (word stagger, secondary-content fade windows) needs to scale with it or be re-derived, since Figma's export gives percentages of the timeline, not fixed ms. The eye blink's own 360ms cycle is independent of this total (it loops on its own clock).
4. **Doc-only deliverable, not visually verified in a running prototype** — the individual techniques (blur/opacity/translate reveal, `scaleY` blink) are standard and low-risk, but if the actual pacing needs to *feel* right before shipping, build and eyeball it rather than trusting the derived numbers blind.
