# Handoff Spec: "Care Level 1 — Loading" — Result-Generation Skeleton Screen

**Prototype:** [`care-level-loading.html`](care-level-loading.html) (clickable, self-contained — open directly or run `npx serve .`)
**Figma:** [Alzheimer's Care Partner x ILABS • Discovery](https://www.figma.com/design/ORrkWZKXD7aFr12gNQC3lU/Alzheimer-s-Care-Partner-x-ILABS-%E2%80%A2-Discovery?node-id=19940-18011), root frame `19940:18011`
**Source Figma Motion JSON:** 1 export covering the gauge arc's animated ellipse (`19948:21378`) and the four "Care Level" digit nodes (`19966:15610/15604/15609/15611`) — see "Motion JSON — what it did and didn't cover" below.

## Overview

A full-screen loading state shown while the app calculates a person's Care Level and generates their alerts/recommendations. Unlike [`pause.html`](pause.html) (a pre-result "calculating" interstitial), this screen shows the **destination layout already in place** — gauge card, alert cards, recommended actions, and the full Care Level Results breakdown — with the not-yet-available pieces (gauge fill, Care Level number, alert summaries) rendered as shimmering skeletons. Built and pixel-checked against Figma in HTML/CSS as an interaction and motion spec — **it is not production code** (see "Prototype-only, do not port" below).

The prototype also plays the Figma-specified navigation: after a fixed 4000ms delay it dissolves (300ms, `cubic-bezier(0.63, 0, 0.28, 1)`) into the resolved destination screen, Figma node [`19966:16011`](https://www.figma.com/design/ORrkWZKXD7aFr12gNQC3lU/Alzheimer-s-Care-Partner-x-ILABS-%E2%80%A2-Discovery?node-id=19966-16011) ("Care Level 3 - High Needs") — see "Screen transition" below.

---

## Design Tokens Used

| Token | Value | Usage |
|---|---|---|
| `green-400` | `#DFEBE8` | Screen background, section backgrounds |
| `green-500` (BG 1) | `#C3D9D6` | Band behind sections (shows as the 8px divider strip) |
| `green-100` | `#F6F9F8` | Card backgrounds |
| `green-600` | `#347F71` | CTA text/border, footer links |
| `neutral-500` (darker text) | `#11302A` | Headings, status bar |
| `neutral-400` | `#3E4141` | Body copy, action item text |
| `neutral-300` | `#656766` | Secondary/disclaimer text |
| `level/green` | `#69C975` | Score bar — no decline |
| `level/yellow` | `#FDC500` | Score bar — mild |
| `level/orange` | `#FD8C00` | Score bar — moderate; gauge fill |
| `level/red` | `#D43611` | Score bar — major support |
| `heading-l` | LT Soul Bold, 32px/1.1, -0.32px | "Care Level" number |
| `heading-m` | LT Soul SemiBold, 25px/32px, -0.5px | Section titles |
| skeleton base | `#E0EBE8` | Shimmer placeholder background |

---

## Layout

| Element | Spec |
|---|---|
| Viewport | iPhone mock, 375×812 |
| Status bar | 44px, standard time/cellular/wifi/battery |
| Structure | 3 stacked sections on a `#C3D9D6` band, each `#DFEBE8` with `24px 16px` padding, `8px` gap exposing the band as thin dividers |
| Section 1 | Stages card (gauge + Care Level heading + skeleton + disclaimer) + Share PDF CTA |
| Section 2 | "Care Alerts & Recommendations" title, 3 alert cards (icon + label + skeleton), Recommended actions list (4 static items) |
| Section 3 | "Care Level Results" title, score bar + breakdown list + footer links (static, already resolved) |

Note: sections 2's action list and section 3's results are shown **already resolved** in the source Figma frame — only the gauge/number and the 3 alert-card summaries are in a loading state. This is a deliberate split in the design (ported as-is); flagged as an edge case below.

---

## Animation 1 — Care Level gauge arc

Source: node `20063:16097`, Figma's native ellipse Arc properties — confirmed directly from the Figma inspector: **`Start -3°` (fixed) and `Sweep 0% → 38.61%` of 360°** (i.e. `0° → 138.996°`) over 2500ms, easing `cubic-bezier(0.5, 0, 0.2098, 1)`, `playbackStyle: "loop"`. This superseded an earlier estimate (`≈0.47° → ≈135.96°`) reverse-engineered from the raw motion JSON's radian value mod 2π — close, but the inspector's `Start`/`Sweep` reading is the precise source of truth and is what's implemented now.

**This is a wedge that grows from nothing to ~139° every cycle, then resets** (a classic "radar sweep that regrows" loader), not a rotating crescent. Getting this right took a few passes — the exact SVG source (`gauge-track.svg`, Figma path `Union`; a matching `_4` fill path was also fetched but is no longer used, see below) plus the Figma-confirmed angle math got the geometry right:

- `Start -3°` is the ellipse's **local** angle, before the 180° rotation wrapper visible in the original Figma export (`rotate-180` on the containing group) is applied. Converting to our angle axis: local angle → `+180°` (the wrapper rotation) → `+90°` (Figma's 0°=3-o'clock/clockwise axis vs. our 0°=12-o'clock/clockwise axis) = `-3 + 180 + 90 = 267°`. This lines up with (and confirms) an earlier pixel-traced estimate of `267.9°` from the exported path coordinates — the two independent methods agree within 1°.
- The wedge grows **clockwise** from ~9 o'clock, up over the top, ending near 1–2 o'clock at full sweep (~139°).
- **Implemented as a real stroked SVG arc**, not a mask over raster artwork: a `<circle r="150.5">` with `fill: none`, `stroke: #FD8C00`, `stroke-width: 12px`, `stroke-linecap: round`, revealed via `stroke-dasharray: 365.1 945.62` (target arc length, then a gap ≥ the full circumference so the pattern never wraps into a second visible dash) animating `stroke-dashoffset` from `365.1` (nothing visible) to `0` (fully revealed) — the standard "animated line draw" SVG technique. This replaced an earlier `conic-gradient()` `mask-image` approach over pre-rendered fill artwork (`gauge-fill.svg`, now unused/deleted) — the raster artwork's own edges didn't match the track's rounded caps, and a `conic-gradient` mask can only produce a hard radial cut, not a rounded one.
- **Rotation gotcha:** a native `<circle>`'s stroke-dash pattern starts at 3 o'clock (our `267°` target's reference point is `0°`/12 o'clock), so the needed `transform: rotate()` is **not** `267deg` directly — it's offset by the circle's own 3-o'clock start (`90°` in our axis): `267 - 90 = 177deg`. Using `267deg` directly (an easy mistake carried over from the old `conic-gradient`'s `from` angle, which uses a different reference) draws the arc on the wrong side entirely, mirrored.
- **Rounded caps on both ends, for free.** `stroke-linecap: round` caps the fixed start point and the animated growing tip identically and automatically, integrated into the stroke itself — no separate dot/cap element needed (an earlier pass tried layering a small circle at the tip to fake this; removed as it read as a disconnected floating dot rather than part of the bar, per design feedback).
- **Masked to the track's exact silhouette**, matching what Figma actually does (visible when scrubbing this node's keyframes in Figma — the fill layer sits inside a mask matching the track shape, not a plain circle). A single-radius circle is only an approximation of the track's true (non-circular) band — the true outer radius ranges from ~146px near the fixed end up to ~155.5px at the very top of the arch — so the stroke is drawn wider than the visually-intended ~9px and clipped with an SVG `<mask>` built from the same path as `gauge-track.svg`, guaranteeing the visible fill never spills past the gray track regardless of the circle approximation's radius error.
- **Mask + CSS `transform` gotcha:** applying `mask="url(#...)"` directly to the `<circle>` that also has `transform: rotate(177deg)` clipped nearly everything except a sliver near the fixed end, in every browser tested — a real cross-browser quirk in how SVG masking interacts with an element's own CSS transform, not just a coordinate mismatch. Fixed by moving the `mask` to a non-rotated wrapping `<g>`, with the rotated `<circle>` as its child: `<g mask="url(#...)"><circle transform="rotate(177deg)" .../></g>`. Keep this structure — reapplying `mask` straight to the transformed circle silently breaks the fill down to a tiny corner.
- **Track moved from `<img src="gauge-track.svg">` to an inline `<path>`** in the same `<svg>` as the fill/mask, so track and fill render through the same vector pipeline in one shared coordinate space (no raster-vs-vector registration risk between an `<img>`-rasterized track and an inline-SVG-clipped fill).
- **The track's 4-subpath gaps are real and intentional — do not dilate/bridge them.** The Figma `Union` track shape is built from 4 separate subpaths (left leg / right leg / top-arch-left / top-arch-right), each with a small real gap to its neighbor. An earlier pass mistook this for an export/rendering bug and "fixed" it with an SVG `feMorphology` dilate filter (`radius="4"`) on the mask path (and later, on the track path too, once it became clear the gap was visible in the plain track as well). **This was wrong and has been reverted.** Fetching this exact node fresh from Figma (`get_design_context` on node `19966:19849`, the `Union` track alone) shows the same gaps directly in Figma's own rendering — they are part of the design, not an artifact of this HTML/CSS port. The mask and track now use the plain, unmodified path data with no filter.
- **What actually needs fixing instead:** since the fill is still a single-radius circle approximating the track's true (non-circular, ~146–155.5px-radius) band, its edge can sit slightly off from a real gap boundary, which is a separate, legitimate alignment problem — not something to solve by erasing the gap. If a visible mismatch shows up at a gap edge, the fix is tuning the circle's radius/stroke-width to hug the track more precisely at that specific angle, not dilating the mask.
- The resolved screen's gauge (Section 1's static "Care Level 3" display) uses the **same** `<circle>`/`<g mask>` markup with `stroke-dashoffset: 0` (no animation, fully revealed) instead of the animated variant — so the loading and resolved gauges are pixel-identical, making the dissolve transition seamless.

| Property | Value |
|---|---|
| Track | Inline `<path>` (same `<svg>` as the fill), Figma `Union` path data, `#C3D9D6` |
| Fill | `<g mask="url(#cl-gauge-mask-*)"><circle cx="155.5" cy="155.5" r="150.5"></circle></g>`, `stroke: #FD8C00`, `stroke-width: 12px` (over-wide, trimmed by the mask), `stroke-linecap: round`, `transform: rotate(177deg)` on the circle |
| Mask | Inline SVG `<mask style="mask-type:alpha">`, same path data as `gauge-track.svg`'s `Union` path, one instance per gauge (`cl-gauge-mask-loading` / `cl-gauge-mask-resolved` — must be unique per `<svg>` since both are inline in the same document) |
| Fill reveal | `stroke-dasharray: 365.1 945.62`; `stroke-dashoffset` animates `365.1 → 0` |
| Duration | 2.5s, plays **once** (`animation-fill-mode: forwards`, no `infinite`) and holds at full sweep |
| Easing | `cubic-bezier(0.5, 0, 0.2098, 1)` (ported directly from the motion JSON) |

Note: the source Motion JSON's `playbackStyle: "loop"` describes the raw Figma prototype asset in isolation. In context, this screen only shows the gauge once while the real result is being generated, then the whole screen dissolves away (see "Screen transition" below) — so a single play-once-and-hold, not an infinite loop, is what's implemented and what design confirmed.

---

## Animation 2 — "Care Level" rolling digit

Source: 4 `TEXT` nodes ("1","2","4","3" — note the source order), each with **the same** CSS export: `translate` from `0` to `-100px` (`-98px` for the "3" node) over 4s linear, holding `62.5%→100%`, `cubic-bezier(0.5,0,0.21,1)` on the rise. Ported literally at first (matching pause-handoff.md's precedent for porting motion JSON as-is), but the literal keyframes only produced a two-state flicker (digit "1" → digit "3", with "2" and "4" never landing in the visible window since their static per-digit offsets weren't evenly spaced) — flagged as needing a design decision.

**Rebuilt as a genuine odometer/slot-reel effect per design's request**: a full first lap through 1→2→3→4, then a second lap that stops early at the real result — for this frame ("Care Level 3"), that's `1→2→3→4→1→2→3`. General rule as described by design: complete lap 1 (1-2-3-4) regardless of the answer, then lap 2 counts `1, 2, …` up to and stopping at the actual level (a level of 2 would be `1→2→3→4→1→2`, just five transitions instead of six). Implemented as a single strip of 7 stacked digits — `1,2,3,4,1,2,3` — that slides up together as one unit (rather than independently-animated spans), so the motion reads as one continuous reel.

| Property | Value |
|---|---|
| Container | `18×35px`, `overflow: hidden` |
| Strip | `.cl-number-strip`, 7 stacked `<span>`s (`1,2,3,4,1,2,3`) in a flex column, each evenly `35px` tall |
| Motion | Single `translateY`, `0 → -35 → -70 → -105 → -140 → -175 → -210px` (6 even steps of 35px — one per digit transition) |
| Duration | 6s linear overall, plays **once** (`animation-fill-mode: forwards`, no `infinite`) and holds at `-210px` (final "3") |
| Easing | `cubic-bezier(0.5,0,0.21,1)` on each of the 6 rises, `62.5%`-into-segment rise / `37.5%`-into-segment hold per step (same rise/hold ratio as the original single-digit keyframe, repeated across 6 evenly-spaced segments) |

Verified geometrically (seeking the strip's `Animation.currentTime` directly, since this session's browser preview had an intermittent paint-refresh issue with wall-clock playback): `t=0→"1"`, `t=900ms→"2"`, `t=1900ms→"3"`, `t=2900ms→"4"` (lap 1 complete), `t=3900ms→"1"` (lap 2 begins), `t=4900ms→"2"`, `t=5900ms→"3"` (final), holding at `"3"` at `t=6000ms` and beyond — exactly matching the intended two-lap sequence.

**Hardcoded for this frame's answer (3).** The strip's digit list and step count are only correct for a Care Level of 3 — see "Prototype-only" below.

---

## Animation 3 — Skeleton shimmer (not in Figma motion export; standard pattern)

The design includes `SkeletonWave` component instances (`19966:16003` etc.) with a "Demo"/"Start" property but no exported keyframe data for the shine sweep — implemented as a conventional shimmer: a `160px`-wide white gradient band sweeping left→right, `1.6s ease-in-out infinite`, staggered `0.3s`/`0.6s` across the three alert-card skeletons and the two gauge-area skeletons for visual variety (not spec'd — a judgment call).

---

## Screen transition — dissolve to resolved result

Source: Figma prototype interaction on this frame — `Trigger: After delay 4000ms`, `Action: Navigate to "Care Level 3 - High Needs"` (node `19966:16011`), `Animation: Dissolve`, `Duration: 300ms`, `Easing: Custom bezier (0.63, 0, 0.28, 1)`.

**Delay extended to 6000ms** (from Figma's spec'd 4000ms) to stay in sync with the "Care Level" digit reel above, which now takes a full 6s to complete its two-lap cycle (see Animation 2) — the delay was originally matched 1:1 to the (then 4s) digit animation so the dissolve fires right as the number finishes settling, and that relationship was kept when the digit animation's duration changed. If the digit animation's duration changes again, update this delay to match.

Implemented as two full-screen siblings (`#screen-loading`, `#screen-resolved`) inside `.cl-screens`, cross-faded via a single `cl-dissolve` class toggled by a `setTimeout(6000)`:

- Both screens transition `opacity` over 300ms with the exact Figma easing curve — a true dissolve (simultaneous fade out/in), not a fade-through-black or slide.
- The resolved screen starts `position: absolute` (stacked on top, `opacity: 0`, `pointer-events: none`) so it doesn't affect scroll height before the transition. After the fade completes (`transitionend` on `opacity`), the loading screen is set to `display: none` and the resolved screen is promoted to `position: static`, handing scroll layout back to normal flow.
- `aria-hidden` is swapped between the two screens at the moment the dissolve starts (not after it finishes), so assistive tech isn't left announcing stale "Generating…" content for the extra 300ms.
- The resolved screen's gauge reuses the same `.cl-gauge-fill-circle` markup as the loading screen (same `<circle>`, same `stroke-dasharray`), just with `.cl-gauge-fill-circle--static` (`stroke-dashoffset: 0`, no animation) instead of `--animated` — so loading and resolved are pixel-identical and the dissolve is seamless (see "Animation 1" above).
- Recommended Actions and Care Level Results are identical between the two Figma frames (see "Edge Cases" above), but are still duplicated as separate markup in each screen rather than shared — the Figma interaction is a full-frame navigation, and duplicating keeps the prototype's two screens independent and simple to reason about, at the cost of some repeated HTML.

**Not implemented / needs a product decision:** the 6000ms delay is a fixed timer, not tied to when the real Care Level calculation actually finishes (same caveat as the individual loading animations — see "Prototype-only" below).

---

## Motion Summary Table

| Element | Trigger | Animation | Duration | Easing | Loops? |
|---|---|---|---|---|---|
| Gauge fill | Screen load | SVG stroke `stroke-dashoffset` reveal, ported from `arcDataEndingAngle` | 2.5s | `cubic-bezier(0.5,0,0.2098,1)` | No, plays once and holds |
| Care Level digit | Screen load | single-strip translateY, two-lap cycle 1→2→3→4→1→2→3 | 6s | `cubic-bezier(0.5,0,0.21,1)` per step | No, plays once and holds |
| Skeleton shimmer (×5) | Screen load | translateX sweep | 1.6s | `ease-in-out` | Yes, staggered |
| Screen dissolve | 6000ms after load | opacity cross-fade, loading → resolved screen | 300ms | `cubic-bezier(0.63,0,0.28,1)` | No, plays once |

---

## Edge Cases (need confirmation)

- **Mixed resolved/unresolved content:** Recommended Actions and the full Care Level Results breakdown appear fully resolved even while the gauge/alerts above are still "Generating…" — confirm this is intentional (e.g., a static previous result staying visible while a *new* Care Level recalculates) rather than a Figma authoring gap.
- **No real backend / no real timing:** all animations loop indefinitely on a fixed CSS timer, not tied to when the actual calculation finishes. Production needs to swap skeletons for real content and stop the loops on data arrival.
- **No error/timeout state:** nothing in Figma defines what happens if generation fails.

---

## Accessibility Notes (not implemented — flagging for engineering)

- No `aria-live`/`role="status"` on the "Generating…" labels or skeletons — screen reader users get no announcement of loading state or completion.
- All three animations ignore `prefers-reduced-motion` — should freeze to a static state when reduced motion is requested.
- Score-bar and breakdown-dot colors (green/yellow/orange/red) are the only encoding of severity in the Results section — needs a text/icon redundancy check for color-blind users (the breakdown headings do provide text labels, which helps, but confirm this meets contrast/redundancy requirements).

---

## Motion JSON — what it did and didn't cover

- The JSON gave exact keyframe timing/values/easing for the digit nodes — ported directly.
- For the gauge, the first pass at this handoff used `get_motion_context`'s summary output, which reported the animated field as an anonymous float on an `ELLIPSE` node — that framing obscured that it was actually `arcDataEndingAngle`, Figma's native arc-sweep property, and led to an incorrect rotation-based implementation. The corrected build used the **exact exported SVG** (Figma paths `Union` for the track and `_4` for the fill, both supplied directly rather than re-derived) plus the raw JSON's field name (`arcDataEndingAngle@-1:-1`) to work out the real geometry: both keyframe values reduce mod 2π to a sweep growing from ~0.5° to ~136°. **Lesson for future arc/gauge handoffs: get the raw field name from the motion JSON (not just the summarized value), and get the exact source SVG paths, not a re-export** — reconstructing gauge/donut motion from a lossy summary produces plausible-looking but wrong results.
- Skeleton shimmer had no motion export at all — built as a standard loading pattern instead.

---

## Prototype-only — do NOT port as-is

1. **No real backend / no real timing.** All three animations loop on fixed CSS timers. Production should drive the gauge from a real (possibly still-unknown) value and stop all loops once the actual result/alerts arrive.
2. **Gauge fill's target angle (138.996°) is hardcoded to this one Figma frame's example value**, not driven by an actual Care Level number — production should compute the sweep angle from the real result (the `stroke-dasharray`/`stroke-dashoffset` SVG technique already used here is the right approach to build on, just needs a real `value` prop instead of the fixed constant).
3. **Digit reel's entire strip is hardcoded for a result of 3**, same caveat as the gauge — the `1,2,3,4,1,2,3` sequence and its 6-step timing only produce the correct two-lap effect (full lap, then a second lap stopping at the answer) for a Care Level of 3. Production needs to generate the strip's digit list, step count, and animation duration (currently a fixed 6s split into N even segments) from the real computed level, per the general rule design specified: lap 1 is always `1,2,3,4`; lap 2 is `1,2,…` up to and including the real level. The dissolve delay (see "Screen transition") is currently hand-synced to this fixed 6s; a dynamic version needs to keep that relationship (or switch the dissolve trigger to an `animationend` listener instead of a fixed timer, which would be more robust than keeping two durations manually in sync).
4. **Fixed English copy**, hardcoded alert/action copy — not data-driven.
5. **Single viewport tested** (iPhone-width mock at 375px). No tablet, dynamic type, or landscape behavior defined.
6. **No navigation wiring.** Standalone screen — not connected to what triggers generation or what screen follows when data arrives.
