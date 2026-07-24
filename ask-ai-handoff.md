# Handoff Spec: "Ask AI Care Partner" Chat Flow

**Prototype:** [`ask-ai.html`](ask-ai.html) (clickable, self-contained — open directly or run `npx serve .`)
**Figma:** [Alzheimer's Care Partner x ILABS • Discovery](https://www.figma.com/design/ORrkWZKXD7aFr12gNQC3lU/Alzheimer-s-Care-Partner-x-ILABS-%E2%80%A2-Discovery?node-id=19966-31800), resolved-state reference at [node 19966-31473](https://www.figma.com/design/ORrkWZKXD7aFr12gNQC3lU/Alzheimer-s-Care-Partner-x-ILABS-%E2%80%A2-Discovery?node-id=19966-31473)

## Overview

A 3-screen flow that lets a caregiver pick a topic, land in a chat pre-filled with that topic, send it, and watch the AI "think" before its reply streams in. Built and pixel-checked against Figma in HTML/CSS/JS as an interaction and motion spec — **it is not production code** (see "Prototype-only, do not port" at the bottom before estimating).

**Screen order:** Question select → Chat (prefilled) → Chat (conversation)

---

## Design Tokens Used

| Token | Value | Usage |
|---|---|---|
| `green-100` | `#F6F9F8` | White card / message area background |
| `green-200` | `#EDF3F2` | Input footer background |
| `green-400` | `#DFEBE8` | Screen 1 background, user message bubble, FAQ chip bg |
| `green-500` | `#C3D9D6` | Teal header zone (status bar + nav) on chat screens |
| `green-600` | `#347F71` | CTA border/text, selected-item border, shimmer highlight |
| `green-700` | `#1C4A43` | Primary button fill, active send button |
| `neutral-500` (darker text) | `#11302A` | Headings, primary text, status bar icons |
| `neutral-400` | `#3E4141` | Body text |
| `neutral-300` | `#656766` | Secondary text, disclaimer |
| `neutral-200` | `#8B8E8D` | Placeholder text, inactive send icon |
| `pink-600` | `#BF4075` | Avatar illustration accent |
| `heading-m` | LT Soul SemiBold, 25px/32px, -0.5px | Screen titles ("What's been hardest lately?", "Ask your Care Partner") |
| `body-m-regular` | Poppins Regular, 16px/24px, -0.32px | Body copy, chat text, chips |
| `cta-semibold` | Poppins SemiBold, 16px/24px, -0.32px | Buttons, nav title |
| `caption` | Poppins Regular, 13px/16px, -0.52px | Disclaimer text |

---

## Screen 1 — Question Select

| Element | Spec |
|---|---|
| List | 8 single-select items, `12px` radius, `#F6F9F8` bg, `8px` gap |
| Selected state | `2px solid #347F71` border, checkmark icon (24px) trailing |
| CTA | Full-width pill, disabled = `#B6C9C4` fill + `not-allowed` cursor; enabled = `#1C4A43` fill, only after a selection is made |
| Close (X) | Top-left, links out of the flow (currently → `index.html`) |

**Interaction:** tapping an item sets selection + enables Continue. Continue populates the Screen 2 input with the selected item's label verbatim and navigates forward.

---

## Screen 2 — Chat (prefilled)

| Element | Spec |
|---|---|
| Header background | `#C3D9D6` for the **first 92px only** (status bar 44px + nav 48px), then `#F6F9F8` for the rest of the screen — see "Background split" below |
| Nav title ("AI Assistant") | `opacity: 0` on **this screen only**. Figma hides it here because the avatar overlaps the nav row; it becomes visible again on Screen 3 |
| Avatar | 108×108px, horizontally centered, `top: 51px` from screen top → straddles the header/card seam: 41px sits above the seam (teal), 67px below (white) |
| Heading + subtext | `padding-top: 79px` inside the white card, positioned directly under the avatar |
| Sample question chips | 3 items, `280px` fixed width, horizontal scroll (`overflow-x: auto`), `8px` gap, text clamped to **2 lines** with ellipsis overflow |
| Input bar | Pre-filled with the Screen-1 selection; textarea auto-grows with content |
| Send button | 32px circle; **inactive** = transparent bg + `#8B8E8D` icon, cursor `not-allowed`; **active** (text present) = `#1C4A43` bg + white icon, cursor `pointer` |

### Background split (important, non-obvious)
The header (status bar + nav) and the content card are **not** two separately-colored siblings — the whole screen background is one hard-stop gradient: `#C3D9D6` for 0–92px, `#F6F9F8` from 92px down. This matters because the content card, the sample-question row, and the input footer all have rounded top corners; if the header and content were separate flat-colored boxes, those corners would reveal the wrong color behind them (we hit this bug during build — see Figma node `19966:31605`, where message area *and* footer live inside one continuous white card, not two).

**Interaction:**
- Tapping a sample-question chip populates the input with that question (same behavior as Screen 1 → Screen 2 prefill) and focuses it.
- Enter (no Shift) = Send, same as tapping the send button.
- Send navigates to Screen 3 and immediately starts the AI response sequence below.

---

## Screen 3 — Chat (conversation)

Nav title is visible here (opacity 1). User message renders as a right-aligned bubble (`#DFEBE8` bg, `12px` radius with a `4px` corner on the top-right pointing toward the sender). AI replies render as **plain text, no bubble, no avatar** — this matches Figma's resolved-state frame exactly (node `19966:31473`), which was a correction made mid-build after initially (incorrectly) giving AI replies a bordered bubble + avatar.

### AI response sequence (the actual "animation" being handed off)

```
user sends message
  → user bubble appears (fade + 8px rise, 320ms, cubic-bezier(.22,1,.36,1))
  → "thinking" row appears (fade, 200ms ease-out)
      - single-line shimmer-gradient text, no avatar, no dots
      - cycles through 3 phrases, ~1667ms each (5000ms / 3)
      - tapping the row skips straight to the reply
  → after 5000ms (or on tap-to-skip), thinking row is removed
  → reply streams in word-by-word
      - one word appended every 83ms
      - each word: opacity 0→1, translateY 2px→0, blur(2px)→blur(0), 350ms ease-out
      - tapping the reply block skips straight to full text
  → once streaming completes, copy + bookmark icons fade in (opacity, 300ms ease)
```

| Property | Value |
|---|---|
| Thinking duration | `5000ms` total |
| Thinking phrases | "Reading what you shared…" / "Reflecting on this…" / "Finding a thoughtful way to help…" |
| Shimmer animation | `1.65s` linear, infinite, gradient sweep `120% → -120%` background-position |
| Shimmer gradient | `linear-gradient(90deg, #B6C9C4 0%, #11302A 38%, #347F71 55%, #B6C9C4 78%)`, `background-size: 250% 100%`, clipped to text |
| Word reveal rate | `83ms` per word |
| Word entrance | `350ms ease-out`: opacity 0→1, translateY 2px→0, blur 2px→0 |
| Skip affordance | Tapping the thinking row *or* the streaming reply block jumps straight to the end state |

### AI reply content structure
Plain-text paragraph (`16px` Poppins Regular `#3E4141`) + a row of two icon buttons below (`copy-alt`, `bookmark`), each `36×36px` tap target with an `8px` radius hover state (`#EDF3F2` bg). Copy writes to clipboard and shows a `900ms` "saved" color pulse (`#347F71`); bookmark toggles a visual saved state only (no persistence).

**Reply content used for the one topic Figma specifies** ("They don't always recognize me"):
> "It's okay to feel this way. Caring for someone you love takes so much strength. Try placing one hand on your heart and taking three slow breaths. What matters to you most in this moment?"

All other topics/replies are prototype-only placeholder copy (see below) — not signed off content.

---

## Motion Summary Table

| Element | Trigger | Animation | Duration | Easing |
|---|---|---|---|---|
| Screen transition | Continue / Send / Back | translateX slide | 400ms | `cubic-bezier(.32,.72,0,1)` |
| Message bubble / AI block | New message appended | opacity + translateY(8px→0) | 320ms | `cubic-bezier(.22,1,.36,1)` |
| Thinking row | Appears | opacity | 200ms | ease-out |
| Thinking text | Continuous while waiting | gradient background-position sweep | 1650ms, loops | linear |
| Thinking phrase swap | Every 1/3 of thinking duration | text swap, no transition | 1667ms interval | — |
| Streamed word | Each word appended | opacity + translateY(2px→0) + blur(2px→0) | 350ms | ease-out |
| Reply actions (copy/bookmark) | Stream completes | opacity 0→1 | 300ms | ease |
| Copy button "saved" pulse | Copy tapped | color change | 900ms hold | — |
| Progress bar / ring (home screen, unrelated) | n/a | n/a | n/a | n/a |

---

## Making the thinking duration dynamic (real latency, not a fixed timer)

The prototype's `THINKING_DURATION = 5000` is a **stand-in for "however long the real API takes."** Don't hardcode a duration in production — drive the thinking state off the actual request lifecycle instead. Below is the state machine and the config values to expose.

### The problem with a naive port
If a dev just wires the shimmer to "show until `fetch()` resolves," two things break:
- A fast response (e.g. 400ms) makes the shimmer flash and vanish — reads as a glitch, not a considered pause.
- A slow response (e.g. 12s) has no phrase-cycling logic tied to it, because the prototype's cycling is `THINKING_DURATION / 3` — a fixed math split of a fixed total, which doesn't exist anymore once duration is variable.

### State machine

```
IDLE
  │  user sends message
  ▼
THINKING  ──────────────────────────────────────────────┐
  │  • start request to AI service                      │
  │  • start a phrase-cycle interval on its own clock    │
  │    (NOT derived from total duration — see below)     │
  │  • start a request timer                             │
  │                                                       │
  ├─ response arrives before MIN_THINKING_MS elapsed ─────┤
  │     → wait out the remainder of MIN_THINKING_MS,      │
  │       THEN transition (avoids the flash-and-vanish)   │
  │                                                       │
  ├─ response arrives after MIN_THINKING_MS ──────────────┤
  │     → transition immediately                          │
  │                                                       │
  ├─ no response after MAX_WAIT_MS ───────────────────────┤
  │     → transition to ERROR / retry state (not built    │
  │       in the prototype — needs its own spec)          │
  ▼
STREAMING
  │  tokens/words appended as they actually arrive
  │  (or word-reveal fallback — see below)
  ▼
DONE  (copy/bookmark actions fade in)
```

### Config values to expose (not hardcode inline)

| Constant | Prototype value | Production guidance |
|---|---|---|
| `MIN_THINKING_MS` | *(none — fixed 5000ms)* | ~900–1200ms floor. If the real response comes back faster, still hold the shimmer until this floor is reached, so it never flash-vanishes. |
| `PHRASE_CYCLE_MS` | `5000 / 3 ≈ 1667` (derived from total) | Set as its own constant (~1600–2000ms), independent of when the response actually arrives. Cycle phrases on this clock for as long as the thinking state is showing — 1, 3, or 7 cycles, doesn't matter, it's not tied to a known total anymore. |
| `MAX_WAIT_MS` | *(not implemented)* | Needs a real timeout (e.g. 20–30s) → error/retry state. The prototype has no failure path at all. |
| `WORD_REVEAL_MS` | `83` (simulated) | Only relevant if the AI service is **not** a token stream (see below). If it *is* a stream, delete this entirely and append real chunks as they arrive — see next section. |

### Real token streaming replaces the word-reveal simulation
If the AI backend supports streaming (SSE/websocket, chunked response), the `revealNextWord()` interval in the prototype should be **replaced**, not adapted: append each real chunk to the message as it arrives, reusing the same `.ai-word`-style fade-in animation per chunk for visual consistency, but driven by network events instead of a `setInterval`. If the backend only returns a complete response (no streaming), the prototype's fixed-rate word reveal is a reasonable fallback — but should still use `WORD_REVEAL_MS` as a named constant, not a magic number, so design can tune pacing without touching request logic.

### Tap-to-skip stays relevant either way
Keep the "tap the thinking row / streaming block to jump to the end" affordance in both cases — it's cheap to build and lets a user bail out of the animation once they've decided they don't need to watch it, independent of whether the underlying data arrived instantly or is still streaming in.

---

## Edge Cases (defined in prototype / need confirmation)

- **No selection on Screen 1:** Continue stays disabled — no way to proceed without picking a topic. *(Confirm this is desired; Figma doesn't show an "I'd rather not say" option.)*
- **Empty input on Screen 2/3:** Send button disabled, `cursor: not-allowed`.
- **Long input text:** Textarea auto-grows (`autosize()` on input), no max-height defined — **needs a cap** in production (e.g. scroll after N lines) or it'll push the send row off-screen on long pastes.
- **Chip text longer than 2 lines:** Truncates with ellipsis; full untruncated text is still what gets sent (click handler uses `textContent`, not the visually-clamped string).
- **Rapid repeat sends:** Each `sendToAI()` call is independent and stacks correctly (tested with a follow-up message while a previous reply had already resolved) — but **not tested** for sending while a *previous* thinking/streaming cycle is still in flight. Recommend engineering add a "disable send while a response is pending" guard; the prototype doesn't have one.

---

## Accessibility Notes (not implemented — flagging for engineering)

- No `aria-live` region on the thinking/streaming text — a screen reader user gets no feedback that anything is happening or has completed.
- No focus management between screens (focus isn't moved to the input, a heading, or an error on screen transition).
- Tap-to-skip on the thinking row / streaming block has no keyboard equivalent and no visible focus state.
- Color-only affordance on the send button (opacity/color change) for enabled vs. disabled — should also flip `aria-disabled`.
- Chip truncation to 2 lines with ellipsis hides content from screen readers only if `aria-label` isn't set to the full string — currently isn't.

---

## Prototype-only — do NOT port as-is

This is a **static HTML/CSS/JS spec**, not the app's tech stack. Treat the following as "build this properly," not "copy this code":

1. **No real backend.** The 5-second thinking delay is a hardcoded `setTimeout`; replies are a hardcoded lookup table (`REPLIES` keyed by topic string) plus a rotating `DEFAULT_REPLIES` pool for anything else. **Do not port the fixed 5000ms duration** — see "Making the thinking duration dynamic" above for the state machine, config values, and how real token streaming should replace the simulated word-reveal.
2. **No handling for real-world latency.** What happens at 15s? On timeout? On a network error? Not designed here — needs its own spec.
3. **No accessibility implementation** (see above).
4. **Bookmark doesn't persist** — visual toggle only.
5. **Not framework-native.** If the shipping app is iOS/Android/React Native/etc., this file is a reference for timing, easing, and layout ratios to reimplement — not a component to port line-by-line. The CSS tokens/timings above are the portable part.
6. **Single viewport tested** (iPhone-width mock at 375px). No tablet, dynamic type, or landscape behavior defined.
7. **Screen 1's close (X) and Screen 2/3's back arrows** are wired to this prototype's own screen-stack, not real app navigation/routing.
