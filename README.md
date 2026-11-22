# Homelab 101

A beginner's guide to building a homelab. The site has three real pages so far:
Home, Pick Your Hardware, and Your First Project. `preview.html` is the pattern
library that shows every component in one place.

**Live site:** - https://vincentrosa1.github.io/homelab-101/ 

## File Structure

index.html - Home
hardware.html - Pick Your Hardware
projects.html - Your First Project
preview.html - pattern library (not in the nav)
css/
main.css - layer order and imports (the only linked file)
reset.css - browser default fixes
tokens.css - custom properties
base.css - element defaults
layout.css - l- layout primitives
components.css - components
states.css - hover, focus, current page, target
utilities.css - u- helpers
overrides.css - temporary and environment fixes
print.css - print only
docs/
refactoring-notes.md
media-and-type.md - asset inventory, alt text, and Module 4 testing
accessibility-testing.md - Testing, remediation log, and conformance
evidence/ - screenshots from testing
images/ - WebP photos and screenshot

## Layer and order strategy
`main.css` declares the order once:

## CSS
@layer reset, tokens, base, layout, 
components, states, utilities, overrides, print;


A later layer always beats an earlier one, no matter how specific the selector is. 
So almost every selector is a single class (0,1,0), and nothing needs IDs, long descendant chains, 
or `!important` to win. 

- **reset / base** set defaults that everything else builds on.
- **layout** places things; **compentnts** styles them.
- **states** comes after components, so `.button:hover` only
    swaps a variable and never fights the component rule.
- **utilities** come after components, so `.u-visually-hidden`
    wins without `!important`.
- **overrides** hold the rare exceptions, each with a comment saying why
- **print** is last and only loads for print (`@import ... layer(print) print`).



- Every rule lives in a layer. Unlayered CSS beats all layers, which is why the first draft
    `styles.css` had to go.
- The only `!important` is the reduced-motion rule in reset. Important declarations reverse 
    layer order, so putting it in the first layer makes it win everywhere. 
- Plain link states (`a:visited`, `a:hover`) live in **base**, not states, so components
    links like `.nav__link` and `.button` can set their own colors.
- Table cells are styled as `.data-table th` (0,1,1) instead of a class on every cell. 
    This is a deliberate exception. 


## Token decisions

- **Two tiers.** Primitives describe a value (`--teal-700`). 
    Roles describe its job (`--color-link`). Components only use roles,
    so a color changes in one place.
- **Contrast.** Text tokens meet 4.5:1 on the page and surface
    colors. Borders and the focus ring meet 3:1. The focus color is amber
    so it stands apart from the teal links.
- **Type.** A system font stack, a monospace stack for code, a fluid `clamp()`
    scale, and `--measure: 70ch` for line length.
- **Spacing.** One rem scale plus role tokens: `--gutter`, `--section-space`, 
    `--tap-target` (44px).
- **Shape and focus.** `--borer` and `--focus-ring` are shorthand tokens, 
    so components never repat `1px solid ...`.
- **Component-scoped tokens** `.button` (`--button-bg`, `--button-text`), 
    `.card` (`--card-padding`, `--card-radius`), and `.callout` (`--callout-color`)
    keep private variables. States and variants swap the variable, not the property.
- **Print swaps tokens** Instead of restyling components, which only works because
    no component use a raw hex value.

## Naming approach

| Prefix / pattern | Meaning | Example |
|---|---|---|
| `l-` | layout primitive | `.l-grid`, `.l-sidebar` |
| no prefix, BEM | component | `.card`, `.card__title`, `.callout--warning` |
| `u-` | single-purpose utility | `.u-flow`, `.u-measure` |
| pseudo-class / ARIA | state | `:focus-visible`, `[aria-current="page"]` |
| `preview-` | preview page only, delete later | `.preview-swatches` |

State comes from the HTML (`aria-current`, `:hover`, `:target`),
not classes like `.active`, so what screen readers announce and what users see can't drift apart.

Breakpoints are mobile first and content driven. 

## Responsive layout system

Built inside the existing layers, with no new files

- **Foundation** `.l-container` and `.l-section` use 
`max-inline-size`, `margin-inline: auto`, 
`padding-inline`, and `padding-block`, `--gutter`
and `--section-space` are fluid `clamp()` tokens.
- **Intrinsic grid 1, `.l-grid`:**
`repeat(auto-fit, minmax(min(100%, var(--grid-min)), 1fr))`.
Section cards go 1, 2, then 3 columns with no 
media queries. `min(100%, ...)` prevents overflow
at 320px.
- **Intrinsic grid 2, `.l-sidebar`:**
`fit-content(var(--sidebar)) minmax(0, 1fr)`. The
Table of Contents is only as wide as its links, up to 14rem.
- **Subgrid, `.card--subgrid`:** cards share their 
row's lines, so titles, text, and links align 
across a row even when one card's text wraps longer.
- **Container query, `.callout`:** a named
`callout` container. At 36rem or wider, the label
moves into a 10rem column beside the text. The 
same callout is used in the main column and in
the article, so it adapts to wherever it's placed.
- **User preferences:** `prefers-contrast: more`
swaps tokens for darker text, borders, and links
and 2px borders. Jump links scroll smoothly only 
with `prefers-reduced-motion: no-preference`. 
- **Fallbacks:** subgrid sits in
`@supports (grid-template-rows: subgrid)`, so 
without it the cards stay flex columns. Browsers
without container queries keep the stacked
callout. Both defaults are fully usable. 

### Content-driven breakpoints

| Breakpoint | Change | Content reason |
|---|---|---| 
| 40rem | Nav grid to flex row | Six links fit at natural width |
| 40rem | Table hint hidden | The 36rem table fits without scrolling | 
| 56rem | Name beside nav | Name plus one-row nav (~50rem) fit |
| 56rem | TOC (Table of Contents) sidebar | Article keeps about 36rem beside the TOC | 
| 36rem (container) | Callout label column | 10rem label plus about 24rem of text |

The old 40rem / 64rem card grid queries were removed. 
`auto-fit` handles the columns. 

### Layout testing

- [X] Narrow (320px): 1 card column, nav 3 x 2, 
    callouts stacked, no page sideways scroll
- [X] Medium (768px): 2 card columns, nav one row
    under the name, TOC above the article, 
    callout labels beside the text
- [X] Wide (1280px): 3 card columns with aligned
    titles, text, and links; nav beside the name;
    sticky TOC
- [X] Container query (~900px): top callouts show 
    the label column while the narrow article
    callouts stays stacked
- [X] 200% zoom at 1280px: reflows like 640px
    (1 column, stacked callouts), nothing clipped,
    no sideways scroll
- [X] Keyboard: skip link first, focus ring on every
    link, button, table, and code block; TOC links
    jump to each step
- [X] Preference: DevTools Rendering, emulate
    `prefers-contrast: more`: darker, thicker borders
    and darker muted text
- [X] Preferece: emulate `prefers-reduced-motion: reduce`:
    jump links jump instantly
- [X] Fallback: delete the `@supports` and 
    `@container` blocks in DevTools: cards and 
    callouts stay readable

## Browser checks

- [X] Chrome 
- [X] Firefox
- [X] 320px: no page sideways scroll, only the table and code block scroll
- [X] 375 / 768 / 1280px: cards go 1, 2, then 3 columns, and the
      nav wraps to 3 x 2, then one row, then beside the site name
- [X] Keyboard: skip link appears first, focus ring on every link,
      table, and code block; hover matches focus
- [X] Jump links: `:target` highlight shows
- [X] Print preview: nav hidden, black on white, code wraps,
      table complete, external URLs printed
- [X] Forced colors (DevTools Rendering): focus and target still
      visible
- [X] CSS disabled: content reads in a logical order
- [X] DevTools Styles shows the layer order
- [X] W3C HTML validator passes; CSS passes except the container query false errors
    (see known gaps)


## Media and typography (Module 4)

Built on the Module 3 layout, with no new CSS files. Each new piece
lives in the layer it belongs to: `.l-split` in layout, `.figure`,
the diagram, and `.video__frame` in components, and one print rule.

- **Network diagram.** Inline SVG in the intro, so it uses my color
    tokens. It has `role="img"` and a `<title>`,
    and the visible caption describes it in words.
- **Hardware photos.** `srcset` and `sizes` with 480, 800, and 1200 px
    WebP files. The Pi photo uses `<picture>` for art direction: a wide
    4:3 crop when the grid has two columns, and a tall 4:5 crop on phones
    so the Pi and drive dock don't get lost in the laptop stickers.
- **Router screenshot.** One lossless WebP at its real 988px width, 16 KB.
    No `srcset` on purpose: a 640px version came out 79 KB, so the bigger
    file is actually the smaller one. Port and device are blacked out.
- **Jellyfin video.** `youtube-nocookie.com`, `loading="lazy"`, a real
    `title`, captions noted in the caption, and a link to watch on YouTube
    if the embed is blocked.
- **Layout stability.** Every `img` has `width` and `height`, the video
    frame reserves 16:9 with `aspect-ratio`, and the SVG has a `viewBox`.
- **Fonts.** Still system fonts only. No web fonts means no font 
    requests and nothing shifts when text loads.

The full asset inventory, alt text, and testing are in 
`docs/media-and-type.md`.

## Accessibility (Module 5)

I tested Home, Pick Your Hardware, and Your First Project against
WCAG 2.2 AA with axe-core, keyboard only, NVDA, zoom and reflow,
text spacing, forced colors, reduced motion, and a Firefox repeat.

- **Found 6 issues and fixed 6.** The big one was the hardware table:
    at 320px the row names scrolled away. They're pinned now.
- **Code blocks** now have a name and role for screen readers, and
    a border that shows in high contrast mode.
- **The table hint** is tied to the table with `aria-describedby`,
    and the extra region around the table is gone.
- **axe:** 0 violations before and after. Two of the fixes were only
    found by hand.

The test scope, remediation log, and conformance summary are in
`docs/accessibility-testing.md`. Screenshots are in `evidence/`.


## Known gaps

- Stay Safe, Glossary, and Resources aren't built yet, so their nav links don't go anywhere.
- The W3C validator warns that `role="list"` is uneccesary on `ul`. It's kept on purpose. 
    VoiceOver drops list semantics when `list-style: none` is set.
- The W3C CSS validator flags `container` and `@container` , `container-type`, and `cqoi`
    in the components.css (lines 140 156, 200, and 218). It doesn't support container 
    queries yet. Browsers do.
- NVDA reads the hardware table caption three times (region name, table name, caption).
    That's normal for this table pattern.

## AI Disclosure
- No AI was used