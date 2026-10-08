# Handoff: Interactive Wedding Invitation (mobile)

## Overview
This is a mobile-only, single-page Persian (RTL) wedding invitation.

1. A loader shows first, then a closed envelope (the first frame of a video). The guest's name sits under the wax seal and a tap icon pulses.
2. Tapping plays the envelope-opening video (muted) and starts background music.
3. When the video ends, the full-length invitation image slides up and can be scrolled. A directions button at the bottom opens Balad.

## About the files
`index.html` + `assets/` is a **working, production-ready static site**: plain HTML, CSS and vanilla JS, with no build step and no backend. Deploy the folder as-is to any static host, or port it into another stack while keeping the behavior below. Keep assets as separate files. Do **not** inline them into one big HTML file: an earlier 14 MB self-unpacking build failed to open on mobile and in-app browsers.

## Fidelity
High-fidelity. All visuals come from the provided video and image. Reproduce the behavior and positioning exactly.

## Layers (all `position:absolute; inset:0` inside a fixed root, bg `#9b9b9b`)
1. **#stage**: the envelope video plus overlays. A click on it starts the experience.
2. **#card**: the scrollable invitation. It starts at `translateY(100%)` with `pointer-events:none`.
3. **#loader**: the top layer (z-index 10). It fades out (`opacity .6s`) once the video has loaded.

## Loader
- 36px spinner: 3px `rgba(255,255,255,.35)` border with a `#fff` top border, rotating `.9s linear`.
- Text below it: "در حال بارگذاری…", Azar 16px, white, 18px gap.
- Hidden on the video's `loadeddata` or `canplay` event, or if `readyState >= 2`. A fallback timeout hides it after 8 s.

## Envelope (#stage)
- **Video:** `<video src="assets/card2.mp4#t=0.001" muted playsinline webkit-playsinline preload="auto">`, `object-fit:cover`. The video is 368×800 and 6.04 s long. `#t=0.001` forces iOS to show the first frame.
- **Guest name (`#guest-name`)**
  - Default text: `مهمان گرامی`.
  - Source: the `to` query parameter, read with `URLSearchParams`, then `.normalize('NFC')`, whitespace collapsed and trimmed. It is set via **`textContent`** only. ZWNJ is preserved.
  - Style: Azar 26px, line-height 1.4, color `#5e5e5e`, centered, `text-wrap:balance`, `overflow-wrap:break-word`, `unicode-bidi:plaintext`.
  - Auto-fit: if the name runs past 2 lines, font-size drops in 2px steps to a minimum of 14px. This re-runs on `document.fonts.ready` and on `resize`.
  - Position: the wrapper's `top` tracks the seal inside the cover-fitted video. With `k = max(W/368, H/800)`, `top = H/2 + k*118` px.
  - The name fades out (`.8s`) on tap.
- **Tap hint**
  - Placed at `bottom: calc(14vh + env(safe-area-inset-bottom))` and centered.
  - A 56px circle in `rgba(26,26,26,.55)` with a white tap-hand icon. It pulses `scale 1→1.15`, 1.8s.
  - A 64px white ring expands `scale .6→1.8` and fades out, 1.8s.
  - Hidden on tap.

## Card (#card)
- Content column with `max-width:520px`, centered.
- Image: `<picture>` with `assets/card.webp` (1560×8318) and `assets/card.jpg` as the fallback. Explicit `width`/`height` attributes prevent layout shift.
- Below the image is a column with a 14px gap and padding `28px 20px calc(36px + env(safe-area-inset-bottom))`:
  - **Text:** "جهت مسیریابی از طریق اپلیکیشن‌های مسیریاب کلیک کنید:", Azar 16px, `#1a1a1a`, centered.
  - **Button:** links to `https://balad.ir/p/70s2GB1fHKiJhq` with `target=_blank`. Pill shape, min-height 52px, padding `0 28px`, bg `#1a1a1a`, white Azar 16px, 20px map-pin icon. Label: "مسیریابی تا باغ تالار شاهدخت".
- The card slides in with `transform 1.1s cubic-bezier(.22,1,.36,1)`.

## Behavior
- State `phase` is `idle → playing → card`.
- Tap (only while `idle`):
  1. Hide the name and tap hint.
  2. `music.play()`. This must happen inside the gesture handler because of mobile autoplay rules. Errors are swallowed.
  3. `video.play()`. If the promise rejects, go straight to the card.
  4. A safety timeout shows the card after 12 s.
- The video's `ended` event shows the card. So does a video `error` that happens after the tap.
- **Music:** `assets/music.mp3`, looping, `preload="metadata"`.
  - Paused on `visibilitychange` (hidden) and on `pagehide`.
  - Resumed when the page becomes visible again, but only if it was playing before.

## Tokens
- Colors: bg `#9b9b9b`, ink/button `#1a1a1a`, guest name `#5e5e5e`, white.
- Font: Azar (`assets/Azar.ttf`, `font-display:swap`, preloaded) for all Persian text, with Tahoma as the fallback.

## Assets
- `card2.mp4`: envelope video, ~2 MB.
- `card.webp` / `card.jpg`: invitation artwork, 0.8 MB / 1.25 MB.
- `music.mp3`: background music ("jing-o-jing piano").
- `Azar.ttf`: font.

## Possible improvements
- Re-encode the video as H.264 at about 1–1.5 Mbps with `+faststart`.
- Re-encode the music at about 96–128 kbps.
- Subset the Azar font to Persian glyphs and convert it to woff2.
- Test on iOS Safari, Android Chrome, and the in-app browsers of Telegram, WhatsApp and Instagram.
