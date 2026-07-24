# Handoff Spec: "Pause" — Calculating Result Loading Screen

**Prototype:** [`pause.html`](pause.html) (clickable, self-contained — open directly or run `npx serve .`)
**Figma:** [Alzheimer's Care Partner x ILABS • Discovery](https://www.figma.com/design/ORrkWZKXD7aFr12gNQC3lU/Alzheimer-s-Care-Partner-x-ILABS-%E2%80%A2-Discovery?node-id=19948-21917), text nodes `19948:21921`–`19948:21926`, heart icon nodes `19966:20912`–`19966:20917`
**Source Figma Motion JSON:** 2 exports (word-reveal timeline for the text nodes, blink timeline for the eye nodes) — see "Motion JSON — what it did and didn't cover" below before treating this repo as the only source of truth.

## Overview

A single, full-screen loading state shown while the app calculates an assessment result for a named person ("Paula" in the design). Built and pixel-checked against Figma in HTML/CSS as an interaction and motion spec — **it is not production code** (see "Prototype-only, do not port" at the bottom before estimating).

---

## Design Tokens Used

| Token | Value | Usage |
|---|---|---|
| `green-400` | `#DFEBE8` | Screen background |
| `green-700` | `#1C4A43` | Loading text color |
| `neutral-500` (darker text) | `#11302A` | Status bar icons/time, home indicator |
| `pink-500` | `#CC6691` | Heart icon outline, eyes, eyebrows, mouth |
| `pink-200` | `#FBEAEF` | Heart icon inner fill |
| `white` / `#F6F9F8` | — | Heart icon top highlight + decorative motion-trail/cloud shapes behind it |
| `heading-m` | LT Soul SemiBold, 25px/32px, -0.5px | Loading sentence |

---

## Layout

| Element | Spec |
|---|---|
| Viewport | iPhone mock, 375×812 |
| Status bar | 44px, standard time/cellular/wifi/battery |
| Home indicator | 21px band, pill `139×5px`, `#11302A` |
| Content area | Everything between status bar and home indicator, laid out as a centered flex column, `20px` gap |
| Heart icon | `300px` wide (`max-width: 100%`), aspect ratio `320:157` (≈147px tall), centered |
| Loading text | `280px` max-width, centered, wraps naturally (2 lines at this copy length) |

---

## Animation 1 — Word-by-word text reveal

Copy: **"Hang on, we're calculating Paula's result..."** — 6 words, each on its own reveal timeline, staggered ~150ms apart.

| Word | Stagger start | Blur | Rise (translateY) | Opacity |
|---|---|---|---|---|
| Hang | 200ms | 8px → 0 | +25px → 0 | 0 → 1 |
| on, | 350ms | 8px → 0 | +25px → 0 | 0 → 1 |
| we're | 500ms | 8px → 0 | +25px → 0 | 0 → 1 |
| calculating | 650ms | 0 (briefly blips to 8px at 660ms, per source keyframes) → 0 | +25px → 0 | 0 → 1 |
| Paula's | 800ms | 8px → 0 | +25px → 0 | 0 → 1 |
| result... | 950ms | 8px → 0 | +25px → 0 | 0 → 1 |

- Blur easing: `cubic-bezier(.5, 0, .5, 1)`
- Opacity + rise easing: `cubic-bezier(0, 0, .58, 1)`
- Each word's reveal takes ~600ms once it starts
- **Plays once, then holds fully revealed** — the source Figma Motion JSON marked this `"playbackStyle": "loop"`, but looping a "hang on" sentence forever reads as broken/stuck, so this was overridden in the prototype to play once. **Confirm with design/product before shipping** — this is a judgment call made during the build, not a spec'd decision.

---

## Animation 2 — Heart icon blink (loops indefinitely)

Only the two eye shapes animate; the heart body, outline, mouth, eyebrows, and background cloud/sparkle decoration are static.

| Property | Value |
|---|---|
| Trigger | Automatic, on screen load |
| Animation | `scaleY`: 1 → 0 → 1 (both eyes simultaneously) |
| Timing | 0ms → 180ms → 360ms, then holds open until 2000ms |
| Easing | `cubic-bezier(.5, 0, .5, 1)`, both segments |
| Loop | Infinite — this is the "still alive/thinking" indicator for however long the real wait takes |
| Transform origin | Each eye's own bounding-box center (`transform-box: fill-box`) |

---

## Motion Summary Table

| Element | Trigger | Animation | Duration | Easing | Loops? |
|---|---|---|---|---|---|
| Loading text (per word) | Screen load | blur + opacity + translateY | ~600ms, staggered 150ms/word | see above (2 curves) | No — plays once, holds |
| Heart icon eyes | Screen load | scaleY blink | 360ms per blink, 2000ms cycle | `cubic-bezier(.5,0,.5,1)` | Yes, infinite |

---

## Edge Cases (need confirmation)

- **Real wait time vs. fixed 2s animation:** the text reveal is a hardcoded 2000ms CSS animation, not tied to any actual calculation. If the real backend call finishes in 400ms or takes 15s, this prototype doesn't define what happens — see "Prototype-only" section below.
- **Name interpolation:** "Paula's result" — the apostrophe-s possessive, capitalization, and word count all depend on the real name. A name ending in "s" (e.g. "Chris") changes to "Chris'" grammatically — needs a copy rule, not a hardcoded string.
- **Localization:** the word-by-word reveal is built as 6 individually-tuned `<span>`s. A translated string will have a different word count/order — the animation approach needs to work on however many tokens the localized string splits into, not on 6 fixed elements.
- **No error/timeout state:** nothing in Figma or the motion JSON defines what this screen does if the calculation fails or times out.
- **No progress or cancel affordance:** for a long wait, there's no indication of expected duration and no way to back out.

---

## Accessibility Notes (not implemented — flagging for engineering)

- No `aria-live`/`role="status"` on the loading text — a screen reader user gets no announcement that a calculation is in progress or has completed.
- Both animations are purely decorative motion and currently ignore `prefers-reduced-motion` — should jump straight to end states (text fully revealed, eyes open) when reduced motion is requested.
- No focus management on entering/leaving this screen.
- Long unexplained waits are a specific concern for this user base (caregivers under stress) — consider whether a "this may take up to Xs" message is warranted, beyond what Figma currently specifies.

---

## Motion JSON — what it did and didn't cover

Both animations started from a Figma Motion JSON export. Worth calling out for future handoffs like this one:

- The JSON gives exact keyframe timing/values/easing per node ID — that part ported directly into the CSS above.
- It does **not** include content, layout, or composition — which node is which word, how the icon and text are positioned relative to each other, screen chrome — all of that required separate `get_design_context`/`get_metadata` calls.
- For the heart icon specifically, the motion JSON only referenced the two eye "Vector" nodes (the animated parts). The full artwork (heart outline, two-tone fill, sparkles, cloud trail) wasn't in the JSON or the initial Figma fetch — it had to be supplied separately as a complete SVG before the icon looked right. **If handing off a Motion JSON for a similar animated icon, attach the full source SVG alongside it**, not just the exported node IDs.

---

## Prototype-only — do NOT port as-is

This is a **static HTML/CSS spec**, not the app's tech stack. Treat the following as "build this properly," not "copy this code":

1. **No real backend / no real timing.** The "calculating" duration is a fixed 2000ms CSS animation. Production should reveal the text once (as designed), then keep the heart-blink loop running for however long the actual calculation takes, and transition away only when the real result arrives — not on a timer.
2. **Fixed English copy.** Rework as data-driven copy (name interpolation, localization) with a reveal animation that works for an arbitrary number of words/tokens, not 6 hardcoded spans.
3. **Hand-rolled CSS keyframes for the icon.** The project already uses Rive elsewhere (`acp_confetti.riv`) — this animated icon is a strong candidate for a proper Rive/Lottie asset in production rather than inline SVG + CSS `scaleY`.
4. **Not framework-native.** The timing/easing/layout ratios above are the portable part; don't port the DOM structure line-by-line if the shipping app isn't a web view.
5. **Single viewport tested** (iPhone-width mock at 375px). No tablet, dynamic type, or landscape behavior defined.
6. **No navigation wiring.** This is a standalone screen in the prototype — not connected to what triggers "calculating" or what screen follows once the result is ready.
