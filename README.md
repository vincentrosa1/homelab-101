# Homelab 101 - CSS Architecture Preview

The first production CSS system for Homelab 101, a beginner's guide to building a homelab.
`index.html` is a temporary preview page that shows every pattern. Real pages will replace it later. 

**Preview site:** 

## File Structure

index.html - temporary preview page
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

Breakpoints are mobile first: base styles are narrow `min-width: 40rem` is medium and `64rem` is wide.

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
- [X] W3C HTML and CSS validators pass

## Known gaps

- Nav links point to pages to be built later
- Table values and commands are placeholders until the real 
    Pick Your Hardware and Your First Project pages. 
- The W3C validator warns that `role="list"` is uneccesary on `ul`. It's kept on purpose. 
    VoiceOver drops list semantics when `list-style: none` is set. 

## AI Disclosure
- No AI was used