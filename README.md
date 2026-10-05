# Handoff: Interactive Wedding Invitation (mobile)

## Overview
A mobile-only, single-page Persian (RTL) wedding invitation for Negin & Mohammad. The guest sees a closed envelope (first frame of a video) with their name under the wax seal and a pulsing tap icon. Tapping plays the envelope-opening video (muted) and starts background music; when the video ends, the full-length invitation image slides up from the bottom and can be scrolled. A "directions" button links to Google Maps.

## About the Design Files
The files here are **design references built in HTML** that show the intended look and behavior. `index.html` is a fully working, self-contained build (all assets inlined, ~14 MB) and can be deployed as-is to any static host if no rebuild is wanted. If re-implementing (e.g. plain HTML/JS, React, Next.js), recreate the behavior below using the target environment's patterns; `Wedding Invitation.dc.html` is the editable source.

## Fidelity
High-fidelity. Visuals come entirely from the provided video and image; reproduce the behavior and positioning exactly.

## Screen 1 — Envelope (idle)
- Full viewport (`position:fixed; inset:0`), background `#9b9b9b`, `overflow:hidden`, `dir="rtl"`.
- `<video src="assets/card2.mp4#t=0.001" muted playsinline preload="auto">`, `width/height:100%`, `object-fit:cover`. Video is 368×800, 6.04 s. `#t=0.001` makes iOS show the first frame as a poster.
- **Guest name** `#guest-name`
  - Default text: `مهمان گرامی`.
  - On load: `new URLSearchParams(location.search).get('to')` → `.normalize('NFC')`, collapse whitespace, trim; if non-empty set via **`textContent`** (never innerHTML). Example: `?to=علی%20رضایی`. ZWNJ (U+200C) passes through untouched.
  - Font: `Azar` (assets/Azar.ttf, `@font-face`, `font-display:swap`), 26px, line-height 1.4, color `#1a1a1a`, centered, `text-wrap:balance`, `overflow-wrap:break-word`, `unicode-bidi:plaintext`.
  - Auto-fit: if the text exceeds 2 lines, reduce font-size by 2px steps down to a 14px minimum. Re-run after `document.fonts.ready` and on `resize`.
  - Position: wrapper is absolutely positioned full-width with 24px side padding. Its top tracks the seal in the `object-fit:cover` video: `k = max(W/368, H/800)`, `top = H/2 + k*118` px (118 = offset in video px below the vertical center, just under the seal).
  - Fades out (`opacity 0`, `.8s ease`) when the user taps.
- **Tap hint**: centered, `bottom: calc(14vh + env(safe-area-inset-bottom))`, non-interactive.
  - 56px circle, `rgba(26,26,26,.55)`, white hand/tap stroke icon 28px (stroke 1.8).
  - Pulse: `scale 1→1.15→1`, opacity `.95→.7`, 1.8s ease-in-out infinite.
  - Outer ring: 64px, 2px `rgba(255,255,255,.9)` border, `scale .6→1.8`, opacity `.8→0`, 1.8s ease-out infinite.
  - Hidden once tapped.

## Screen 2 — Invitation card
- A full-viewport scroll container overlaying the video (`overflow-y:auto`, `-webkit-overflow-scrolling:touch`, bg `#9b9b9b`).
- Starts at `translateY(100%)` with `pointer-events:none`. On reveal it animates to `translateY(0)` over `1.1s cubic-bezier(.22,1,.36,1)`.
- Content column: `max-width:520px`, centered.
  - `<img src="assets/card.png">` at `width:100%`, `height:auto`.
  - Directions button below it: container padding `28px 20px calc(36px + env(safe-area-inset-bottom))`. Link to `https://maps.app.goo.gl/8skM7YPaKcJ3bTpB8`, `target=_blank`. Style: pill (`border-radius:999px`), min-height 52px, padding `0 28px`, bg `#1a1a1a`, white text, 16px, gap 10px. Map-pin icon 20px. Label: `مسیریابی تا باغ تالار شاهدخت`.

## Interactions & State
- `phase`: `'idle' | 'playing' | 'card'`.
- Tap anywhere on the envelope (only while `idle`):
  1. `audio.play()` for `assets/music.mp3` (loop; errors ignored). This must run inside the tap handler for mobile autoplay rules.
  2. Set phase to `playing`, then `video.muted=true; video.play()`. If the play promise rejects, go straight to `card`.
- Video `ended` or `error` → phase `card` → card slides up.
- Music keeps looping. There is no mute control.

## Design Tokens
- Colors: background `#9b9b9b`; ink/button `#1a1a1a`; white `#ffffff`.
- Font: Azar for all Persian text (Tahoma fallback).
- Radius: 999px (pill/circle).

## Assets
- `assets/card2.mp4` — envelope-opening video (user-provided)
- `assets/card.png` — full invitation artwork (user-provided)
- `assets/music.mp3` — background music (user-provided, "mohammad_noori_aroosi")
- `assets/Azar.ttf` — Persian font (user-provided)

## Files
- `index.html` — self-contained working build (deployable directly)
- `Wedding Invitation.dc.html` — editable design source (references `uploads/…` paths; map them to `assets/…`)

## Notes
- Large payload. Compress the video (H.264, ~1–2 Mbps) and convert the PNG to WebP/JPEG before production.
- Test on iOS Safari and Android Chrome. Check: first-frame poster, audio starting on tap, and safe-area insets.
