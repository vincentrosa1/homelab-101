# Accessibility testing
 
Module 5 notes for Homelab 101. I tested the three real pages against
WCAG 2.2 AA, fixed what I found, and retested. Screenshots are in
`evidence/` at the root of the repo.
 
- **Tools:** Chrome and Firefox on Windows, NVDA with the Speech Viewer,
    Chrome DevTools (device toolbar, Rendering tab, Accessibility pane),
    and axe-core 4.14.1
## 1. Test scope
 
| Page | Why it's in scope |
|---|---|
| Home (`index.html`) | First page anyone lands on. Has the main landmarks, the nav, the SVG network diagram, and the card links to every section. |
| Pick Your Hardware (`hardware.html`) | Has the data table, which is the hardest thing on the site at narrow widths. Also has the responsive photos, the art-directed Pi crop, and outside source links. |
| Your First Project (`projects.html`) | Longest page. Has the sidebar jump links, `:target` highlights, code blocks, the YouTube embed, and the linked router screenshot. |
 
**Not in scope:**
- `preview.html` is the pattern library. It isn't in the nav, so it only
    got the automated check.
- Stay Safe, Glossary, and Resources aren't built yet.
## 2. Semantic structure
 
- Each page has `lang="en"`, its own `<title>`, and one `h1`.
- No skipped heading levels. On Projects the outline is h1, then h2
    "On this page", h2 "Media server" with three h3 steps, and h2
    "Game server" with three h3 steps.
- Landmarks are a banner, the "Primary" nav, main, named regions for
    each labeled section, and contentinfo. Projects also has an
    "On this page" nav.
- The current page in the nav has `aria-current="page"`, and NVDA reads
    it as "current page".
- The skip link goes to `#main`.
Evidence: `a11y-sr-landmarks-home.png`, `a11y-sr-headings-projects.png`.
Findings: A11Y-04, A11Y-06.
 
## 3. Keyboard and focus
 
I tabbed through every page with the mouse put away.
 
| Page | Tab stops | Result |
|---|---|---|
| Home | 15 | Pass |
| Pick Your Hardware | 18 | Pass |
| Your First Project | 24 | Pass |
 
- **Order:** it follows the reading order on all three pages.
- **Focus ring:** the amber ring (3.19:1 against the page) shows on
    every stop, including the table and code block scroll areas.
- **Skip link:** it shows on the first Tab. After Enter, the next Tab
    lands on the first thing in main.
- **Table:** the scroll area takes focus, and the arrow keys scroll it
    at 320px.
- **Jump links:** they move to the step, and the heading gets the
    `:target` highlight.
- **YouTube player:** no keyboard trap. Focus goes in and comes back
    out both ways.
- **Firefox:** the same pass gave the same results.
Evidence: `a11y-kbd-skip-link.png`, `a11y-kbd-focus-table.png`,
`a11y-kbd-sidebar-focus.png`, `a11y-kbd-target.png`.
 
## 4. Zoom, reflow, text spacing, and contrast
 
**Zoom and reflow.** I checked each page at 200% and 400% browser zoom
in Chrome and Firefox, and at 320px wide in the device toolbar.
- The page never scrolls sideways.
- Nothing is cut off, and the nav wraps.
- The table and the code blocks scroll inside their own boxes. The
    table counts as a data table under the reflow exception, and both
    can be scrolled with the keyboard.
- The Pi photo switches to the tall crop.
- The sidebar stacks above the content and doesn't stick.
Evidence: `a11y-zoom-400-home.png`, `a11y-reflow-320-hardware.png`,
`a11y-reflow-320-projects.png`. Finding: A11Y-01.
 
**Text spacing.** I added the WCAG 1.4.12 values as a temporary
stylesheet in DevTools (line height 1.5, letter spacing 0.12em, word
spacing 0.16em, paragraph spacing 2em). I checked all three pages at
full width and at 320px. Nothing clipped or overlapped, and the diagram
labels still fit in their boxes.
 
Evidence: `a11y-spacing-home.png`, `a11y-spacing-projects-320.png`.
 
**Contrast.** Every color pair in my tokens passes AA.
 
| Pair | Ratio | Needs |
|---|---|---|
| Body text on page background | 15.2:1 | 4.5:1 |
| Body text on white | 16.46:1 | 4.5:1 |
| Muted text on page background | 6.39:1 | 4.5:1 |
| Link on page background | 5.59:1 | 4.5:1 |
| Visited link on white | 7.2:1 | 4.5:1 |
| Warning text on white | 5.19:1 | 4.5:1 |
| White on the teal button | 6.05:1 | 4.5:1 |
| Code text on code background | 10.45:1 | 4.5:1 |
| Text on the `:target` highlight | 14.93:1 | 4.5:1 |
| Focus ring on page background | 3.19:1 | 3:1 |
| Borders on page background | 3.46:1 | 3:1 |
 
I spot-checked the muted text with the DevTools color picker, and it
matched at 6.39:1. axe can't measure SVG text, so it flagged the six
diagram labels for review. I checked them by hand: they use the text
color on white or the highlight fill, and "Your private network." uses
the link color. All of those are in the table above.
 
**Forced colors.** I emulated `forced-colors: active` in the Rendering
tab.
- The diagram fallback keeps every box, line, and label visible.
- The button keeps its border.
- The focus ring, the current-page underline, and the `:target`
    outline fallback all show.
Evidence: `a11y-forced-colors-home.png`, `a11y-forced-colors-target.png`.
Finding: A11Y-05.
 
## 5. Forms and tables
 
**Forms.** There are no forms on the site yet. When I add one, it gets
the label, error, and keyboard checks.
 
**Tables.** The hardware table has:
- a `<caption>`
- `scope="col"` on the column headers and `scope="row"` on the
    hardware names
- a focusable scroll area with `role="region"`, named by the caption
NVDA reads it as "table with 4 rows and 5 columns", and it reads the
column header before each cell.
 
Evidence: `a11y-sr-table-hardware.png`, `a11y-sr-table-after.png`.
Findings: A11Y-01, A11Y-02, A11Y-04.
 
## 6. Media, motion, and alternatives
 
- **Network diagram:** `role="img"` with a `<title>`, plus a visible
    caption that describes it in words.
- **Photos:** every photo has alt text that says what's in it, and the
    credits are next to the photos.
- **Router screenshot:**
    - The alt text describes the rule shown and says the link opens the
        full-size screenshot.
    - The port and device are blacked out.
- **Jellyfin video:**
    - `youtube-nocookie.com` and lazy loading.
    - The iframe has a real `title`.
    - It has human-made captions, not auto-generated ones.
    - It doesn't autoplay.
    - There's a "Watch on YouTube" link in case the embed is blocked.
- **Motion:**
    - The only motion is smooth scrolling on jump links, and it only
        runs with `prefers-reduced-motion: no-preference`.
    - With reduce emulated, the jump is instant.
    - Nothing on any page moves on its own.
## 7. Automated and manual evidence
 
**Automated.** axe-core 4.14.1. The run covered all four pages
at 1280px and 320px, with the WCAG 2.0, 2.1, and 2.2 A/AA rules plus
best practices. The YouTube embed was blocked during the run.
 
| Run | Violations | Needs review |
|---|---|---|
| Before | 0 on all pages | Broken `aria-labelledby` on Home (A11Y-06), 6 SVG labels for contrast, YouTube frame not scanned |
| After | 0 on all pages | 6 SVG labels for contrast (checked by hand), YouTube frame not scanned |
 
The YouTube frame is third-party content, so axe can't scan inside it.
 
**Manual.**
 
| Test | Result | Evidence |
|---|---|---|
| Keyboard and focus | Pass | `a11y-kbd-*.png` |
| Screen reader (NVDA) | 2 findings | `a11y-sr-*.png` |
| Zoom and reflow | 1 finding | `a11y-zoom-400-home.png`, `a11y-reflow-*.png` |
| Text spacing | Pass | `a11y-spacing-*.png` |
| Forced colors, contrast, reduced motion | 1 finding | `a11y-forced-colors-*.png` |
| Firefox repeat (keyboard, zoom, jump links) | Pass | Written result only |
 
## 8. Screen reader sampling and its limits
 
I used NVDA with Chrome on Windows, with the Speech Viewer on so I could
screenshot what it said. I sampled:
- landmarks (D and the Elements List)
- headings (H and the Elements List)
- the current-page link
- the diagram (G)
- the table (T and Ctrl+Alt+Arrows)
- photo alt text
- link names
- the code blocks
- the video title
**Limits:**
- **One screen reader and one browser.** I didn't test JAWS, VoiceOver
    on Mac or iPhone, or TalkBack on Android.
- **I'm not a daily screen reader user.** I used the keys I know, and
    people who rely on NVDA every day move through pages differently.
- **The Elements List left out Home's intro region.** Pressing D still
    reaches "Run your first home server, region", and Chrome's
    accessibility tree has it, so it's an NVDA quirk and not my markup.
- **NVDA reads the table caption three times:** as the region name, the
    table name, and the caption itself. That's normal for a scroll
    region named by its caption, so I left it.
- **The YouTube player's own controls are third-party,** and I didn't
    test inside them.
- **I didn't test braille output** or mobile screen readers.
## 9. Remediation log
 
| ID | Issue | Priority | Status |
|---|---|---|---|
| A11Y-01 | Table row names and caption scroll out of view | High | Fixed |
| A11Y-02 | Scroll hint isn't connected to the table | Low | Fixed |
| A11Y-03 | Code blocks have no name or role | Medium | Fixed |
| A11Y-04 | Table region nested inside another region | Low | Fixed |
| A11Y-05 | Code blocks lose their edge in forced colors | Low | Fixed |
| A11Y-06 | Broken `aria-labelledby` on Home | Medium | Fixed |
| A11Y-07 | Links to pages that aren't built yet | Low | Deferred |
 
### A11Y-01: Table row names and caption scroll out of view
- **Evidence:** at 320px, scrolling the table sideways moves "Old laptop",
    "Mini PC", and "NAS" off screen, and the caption is cut off at
    "checked Nov". `a11y-reflow-320-hardware.png`
- **Impact:** on a phone or at 400% zoom, you can't tell which hardware a
    price or noise level belongs to. WCAG 1.3.1 and 1.4.10.
- **Priority:** High. The table is the main content of the page.
- **Fix:** 
    - The row headers are `position: sticky` with a background and a
        divider.
    - The scroll area is a size container, so the caption can stick and
        wrap to `100cqi`.
- **Retest:** pass in Chrome at 320px. The names stay pinned, and the
    caption is fully visible. `a11y-01-table-after.png`
### A11Y-02: Scroll hint isn't connected to the table
- **Evidence:** "Scroll sideways to see every column." was only a visible
    paragraph. Nothing tied it to the scroll area.
- **Impact:** a screen reader user who tabs to the table doesn't hear
    that it scrolls. WCAG 1.3.1.
- **Priority:** Low. The table still works.
- **Fix:** The hint has an `id`, and the scroll area points to
    it with `aria-describedby`.
- **Retest:** pass. Chrome's accessibility tree gives the region the
    description "Scroll sideways to see every column." at 320px and
    1280px.
### A11Y-03: Code blocks have no name or role
- **Evidence:** tabbing to a code block, NVDA said only "list with 3
    items, sudo apt update && sudo apt upgrade -y".
    `a11y-sr-code-block-before.png`
- **Impact:** it's a Tab stop with no name, so a screen reader user
    doesn't know they're on a command to copy. WCAG 4.1.2. axe passed
    it, so only manual testing caught it.
- **Priority:** Medium. It affects every step with a command.
- **Fix:** Each block on Projects has `role="region"` and an
    `aria-label`, like "Command: Update the system".
- **Retest:** pass. NVDA says "Command: Update the system, region" and
    then the command. `a11y-sr-code-block-after.png`
### A11Y-04: Table region nested inside another region
- **Evidence:** NVDA said "Compare starter hardware, region" and then
    right away "Starter hardware (US prices…), region" before the table.
    `a11y-sr-table-hardware.png`
- **Impact:** extra noise before the data, and two landmarks for one
    table. Best practice.
- **Priority:** Low.
- **Fix:** I took `aria-labelledby` off the outer section, so
    it isn't a region anymore. The h2 still shows up with H.
- **Retest:** pass. The outer region is gone, and the landmark tree shows
    one region for the table. The caption is still read three times,
    which is normal NVDA behavior (see section 8).
    `a11y-sr-table-after.png`
### A11Y-05: Code blocks lose their edge in forced colors
- **Evidence:** with `forced-colors: active`, the dark background is
    removed, and the command floats with no box.
    `a11y-forced-colors-code-before.png`
- **Impact:** in high contrast mode it's hard to tell where a command
    starts and ends. WCAG 1.4.11.
- **Priority:** Low.
- **Fix:** A `1px solid transparent` border. It's invisible
    normally, and forced colors draws it.
- **Retest:** pass. Both blocks have an outline in forced colors.
    `a11y-forced-colors-code-after.png`
### A11Y-06: Broken `aria-labelledby` on Home
- **Evidence:** the "Who this is for" section had
    `aria-labelledby="audence-title"`, but the heading's id is
    `audience-title`. axe flagged it for review
    (`aria-valid-attr-value`).
- **Impact:** the section lost its name, so it wasn't exposed as a
    landmark. WCAG 4.1.2.
- **Priority:** Medium.
- **Fix:** Fixed with the other typo fixes.
- **Retest:** pass. The axe review item is gone, and NVDA lists "Who this
    is for" as a landmark. `a11y-sr-landmarks-home.png`
### A11Y-07: Links to pages that aren't built yet
- **Evidence:** Stay Safe, Glossary, and Resources in the nav, the Home
    cards, the Projects callout, and the footer all go to pages that
    don't exist yet.
- **Impact:** a dead end for everyone, and keyboard and screen reader
    users lose their place.
- **Priority:** Low for now. The pages are planned.
- **Fix:** deferred. It's listed in the README Known gaps, and the pages
    get built in a later module.
- **Retest:** not yet.
## 10. Conformance summary
 
**What conforms.** As far as I tested, Home, Pick Your Hardware, and
Your First Project meet WCAG 2.2 AA:
- keyboard access and visible focus
- headings, landmarks, and the current-page link
- reflow at 320px and 400% zoom
- text spacing and contrast
- forced colors
- reduced motion
- text alternatives for the diagram, photos, screenshot, and video
**What was fixed.** A11Y-01 through A11Y-06, all retested.
 
**What's limited.**
- A11Y-07 is deferred until the other pages exist.
- NVDA reads the table caption three times. It's expected behavior, but
    it's still repetitive.
- The YouTube player's controls are third-party, and I didn't test
    inside them.
- Screen reader testing was NVDA with Chrome only.

**AI Disclosure**
- No AI Used
