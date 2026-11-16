# Media and typography

Module 4 notes for Homelab 101. Everything here is on the preview
page (`index.html`) until the real pages are built. 

## 1. Asset inventory

| Asset | What it shows | Source | License / provenance | Original | Delivered |
|---|---|---|---|---|---|
| `images/thinkpad-t420-*.webp` | Old laptop tier | "Lenovo ThinkPad T420 Ubuntu Linux" by Sven (Flickr: flickrsven), 2011, via [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Lenovo_ThinkPad_T420_Ubuntu_Linux.jpeg) | [CC BY 2.0](https://creativecommons.org/licenses/by/2.0/). Credited in the caption. Resized and converted, noted in the caption | JPEG, 2560 x 1920, 2.31 MB | WebP q75 at 480, 800, 1200 w (4:3) |
| `images/pi-nas-wide-*.webp` | My Raspberry Pi 4 on a 4 TB drive dock | My iPhone photo, edited in Lightroom Mobile | My own work | JPEG, 1919 x 2000, 3 MB | Cropped to 1919 x 1439 (4:3), WebP q75 at 800, 1200 w |
| `images/pi-nas-tall-*.webp` | Same photo, tight crop | Same | My own work | Same | Cropped to 1200 x 1500 (4:5), WebP q75 at 480, 800 w |
| `images/router-port-forward-988.webp` | AT&T gateway NAT/Gaming page | My screenshot | My screenshot of AT&T's interface, used to show where the setting lives. Logo and footer cropped, port and device blacked out | PNG, 988 x 491, 47 KB | Lossless WebP, 988 w only (see section 2) |
| Network diagram | Internet, router, and devices | Drawn by me as inline SVG | My own work | n/a | Inline in `index.html`, `viewBox="0 0 400 420"` |
| Jellyfin video | First media library setup | "How to Set Up Jellyfin in 5 Minutes - Beginners Guide" on [YouTube](https://www.youtube.com/watch?v=ZDg6OG-KyoQ) | Embedded with YouTube's own player. Not downloaded or re-hosted | n/a | `youtube-nocookie.com` iframe |
| Body and code fonts | All text | Fonts already on the visitor's device | Licensed with the operating system. Nothing is served | n/a | System font stacks, zero requests |

### Exported file sizes

| File | Dimensions | Size |
|---|---|---|
| `thinkpad-t420-480.webp` | 480 x 360 | 36 KB |
| `thinkpad-t420-800.webp` | 800 x 600 | 93 KB |
| `thinkpad-t420-1200.webp` | 1200 x 900 | 189 KB |
| `pi-nas-wide-800.webp` | 800 x 600 | 66 KB |
| `pi-nas-wide-1200.webp` | 1200 x 900 | 143 KB |
| `pi-nas-tall-480.webp` | 480 x 600 | 39 KB |
| `pi-nas-tall-800.webp` | 800 x 1000 | 103 KB |
| `router-port-forward-988.webp` | 988 x 491 | 16 KB |

Every photo is under the 220 KB target, and the heaviest
possible page (both 1200 w photos plus the screenshot) is about
350 KB of images, well under the 1 MB budget.

**Privacy:** the Pi photo came off my phone with GPS location in it.
Squoosh drops metadata on export, and I checked each WebP in
File Explorer (Properties > Details) to make sure no GPS shows.

## 2. Responsive images

| Image | Files | `sizes` | Why |
|---|---|---|---|
| ThinkPad | 480, 800, 1200 w | `(min-width: 78rem) 35rem, (min-width: 36rem) calc(50vw - 2.5rem), calc(100vw - 2rem)` | Matches the photo grid: one column on phones, two columns from about 36rem, capped at 35rem by the 75rem container |
| Pi (wide) | 800, 1200 w | `(min-width: 78rem) 35rem, calc(50vw - 2.5rem)` | Only used when the grid has two columns |
| Pi (tall) | 480, 800 w | `calc(100vw - 2rem)` | Phones only, full width |
| Router screenshot | 988 w only | none | No `srcset` on purpose. I exported a 640 w version too, but it came out 79 KB vs. 16 KB for the 988. Shrinking a screenshot blends its sharp edges into lots of new colors, and lossless WebP can't pack those well. One small file beats a "smaller" one that's 5x bigger. The figure is capped at 988px so it's never upscaled |
| Network diagram | none | none | Inline SVG scales with no extra files |

All photos are WebP. The screenshot is lossless WebP because it's
mostly text and flat color, and lossy compression blurs small text.
Photos below the first screen use `loading="lazy"`.

## 3. Art direction

- **Pi photo: `<picture>`.** On wide screens the 4:3 crop shows the
  sticker-covered laptop behind the Pi, which gives some context.
  At phone width those stickers turn into noise next to a small Pi,
  so phones get a tall 4:5 crop of just the Pi and the drive dock.
  The `<source>` switches at 36rem, the same point the grid goes to
  two columns.
- **ThinkPad: same composition everywhere.** The laptop fills the
  frame and sits in the middle, so it reads the same at 288 px or
  556 px.
- **Router screenshot: same composition everywhere.** Cropping would
  hide where the setting lives. Instead, the image links to the full-size
  file, and the paragraph above it says where to click.

## 4. Alt text and alternatives

| Asset | Purpose | Alt text or alternative |
|---|---|---|
| Network diagram | Complex | `<title>`: "How a basic home network connects." The visible caption gives the full description in words. |
| ThinkPad | Informative | "An older Lenovo ThinkPad T420 laptop, open and running Ubuntu Linux." |
| Pi | Informative | "A Raspberry Pi 4 in a black case sitting on a hard drive dock that holds a 4 TB Western Digital drive." Both crops use the same alt. |
| Router screenshot | Functional (inside a link) | "NAT/Gaming page in the AT&T gateway settings, showing one hosted application rule for the game server. Opens the full-size screenshot." |
| Jellyfin video | Embed | `title`: "Video: How to Set Up Jellyfin in 5 Minutes, Beginners Guide" |

Image credits go in the caption, not the alt text.

## 5. SVG, media, and embed accessibility

- **Diagram.** `role="img"` with `aria-labelledby` pointing at its
  `<title>`. I dropped the `<desc>` I planned in the inventory, because
  the caption already shows the description and screen readers would
  hear it twice. Solid vs. dashed lines carry the Ethernet vs. Wi-Fi
  difference, so it doesn't rely on color. A `forced-colors` rule
  switches it to system colors in high contrast mode.
- **Labels.** Diagram text is 20 units in a 400-unit `viewBox`, so it
  stays about 14px at a 288px width and grows to 22px at the 28rem cap.
- **Video.** Nothing autoplays. YouTube's player has visible controls
  and keyboard support. The uploader added English captions, and the
  caption tells people to turn them on with CC. The steps in the article
  are the text alternative for the setup part.
- **Fallback.** "Watch on YouTube" is a plain link, so the video is
  still reachable if the embed is blocked. In print the player is
  hidden and the link prints with its URL.
- **Privacy and performance.** `youtube-nocookie.com` holds off
  YouTube cookies until someone presses play, and `loading="lazy"` waits
  to load the player until it's near the screen.

## 6. Typography system

- **Font stacks.** Body: `system-ui, -apple-system, "Segoe UI", Roboto,
  sans-serif`. Code: `ui-monospace, "SF Mono", Menlo, Consolas, monospace`.
- **Type scale.** Fluid `clamp()` steps from `--step--1`
  (0.875 to 0.9375rem) up to `--step-3` (1.875 to 2.75rem). Every
  min and max is in rem, so 200% zoom still doubles the text.
- **Line length.** `--measure: 70ch` on paragraphs, captions, and the
  article column.
- **Line height.** 1.6 for body text, 1.2 for headings.
- **Spacing.** One rem scale (`--space-3xs` to `--space-xl`) plus
  fluid `--gutter` and `--section-space`.
- **Fallback behavior.** If a named font isn't on the device, the
  stack falls through to the generic `sans-serif` or `monospace`, so
  text always renders in something readable.

## 7. Font loading

No web fonts. Every font is already on the visitor's device, so there's
no `@font-face`, no font requests, no licensing to track, and no layout
shift from a font swapping in. If I ever add a web font, it needs
`font-display: swap`, a matched fallback stack, and a license note here.

## 8. Layout stability

- Every `<img>` has `width` and `height`, so the browser reserves the
  right box before the file arrives.
- The Pi `<source>` has its own `width` and `height` (1200 x 900), so
  the reserved box switches shape with the crop.
- `.figure img` gets a surface background, so a slow image shows as a
  calm box instead of a gap.
- `.video__frame` reserves 16:9 with `aspect-ratio` before the iframe
  loads.
- The diagram has `width`, `height`, and a `viewBox`.

## 9. Testing

Tested on the live site in Chrome DevTools with cache disabled.

- [X] 320px: one photo column, Pi shows the tall crop, no page
    sideways scroll
- [X] 768px: two photo columns, Pi shows the wide crop
- [X] 1280px: intro and diagram side by side, two photo columns
- [X] Device pixel ratio 1 vs. 2: `currentSrc` picks a bigger file at 2
- [X] 200% zoom at 1280px: everything reflows, nothing clipped
- [X] Slow 4G: image and video spaces hold their size while loading,
    nothing below jumps
- [X] Lighthouse: CLS under 0.1
- [X] File sizes match the table above, and every page stays under 1 MB
- [X] Alt text: WAVE or axe shows no missing-alt or empty-link errors
- [X] Captions: CC button shows English captions on the video
- [X] Forced colors (DevTools Rendering): diagram stays visible
- [X] Print preview: video hidden, "Watch on YouTube" URL printed
- [X] No GPS data in any exported image
- [X] W3C HTML validator passes. CSS validator passes except two false
    errors for container queries (`container` and `@container`), which
    the validator doesn't support yet

## Changed since the Module 4 inventory

- The Jellyfin screenshot was replaced with the embedded video.
- The diagram cap went from 40rem to 28rem. The vertical layout reads
  fine at that size, and bigger just made the labels huge.
- The Pi got two crops for art direction instead of one.
- The screenshot ships as one 988 w file. The 640 w export was
  bigger, so it isn't in the repo.