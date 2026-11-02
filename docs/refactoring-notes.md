# Refactoring notes

I started with a quick single-file draft (`styles.css`, commit
"Add first-draft preview page and styles") and refactored it
into the layered system one pattern at a time. Each "before"
below is from that draft. Each "after" is the current code.

## Summary

| Goal | Where | Result |
|---|---|---|
| Replaced repeated values | whole draft | hard-coded hex and px values replaced with tokens |
| Lowered specificity | cards | (0,2,1) to (0,1,0) |
| Flattened selectors | nav | 5-part chain tied to HTML structure to one class |
| Clarified naming | nav, CTA, cards, callouts | `.active`, `.cta`, `.home-cards .card`, `.callout-note` renamed |
| Reduced duplication | callouts, spacing, hover/focus | two copy-pasted rules to one base plus a 1-line variant |
| Removed unlayered CSS | draft file | deleted, so every rule lives in a layer |

## 1. Repeated values to tokens

The draft repeated the same raw values across rules:

| Value | Times in draft | Replaced by |
|---|---|---|
| `#ffffff` | 5 | `--color-surface`, `--color-on-accent` |
| `#0b6e6e` | 4 | `--color-link`, `--color-accent`, `--color-note` |
| `#7a8594` | 2 | `--color-border` (inside `--border`) |
| `#16202b` | 2 | `--color-text` |
| `#a15c00` | 2 | `--color-warning` |
| `margin-bottom: 16px` | 3 | `u-flow` and `gap` |
| `padding: 16px 24px` | 2 | `--space-s` / `--space-m` |

Now a color changes in one place, `tokens.css`. Print proves it:
`print.css` swaps the tokens and every component updates
without its own print rule.

## 2. Nav: flattened selectors, ARIA state

**Before**
```css
header nav ul li a { color: #16202b; text-decoration: none; padding: 8px; }
header nav ul li a:hover { text-decoration: underline; }
header nav ul li a.active { font-weight: bold; text-decoration: underline; }
```

**After**
```css
/* components layer */
.nav__link {
    color: var(--color-text);
    min-block-size: var(--tap-target);
    padding-inline: var(--space-2xs);
    text-decoration: none;
}

/* states layer */
.nav__link:hover,
.nav__link:focus-visible {
    background: var(--color-bg);
    text-decoration-line: underline;
}

.nav__link[aria-current="page"] {
    font-weight: var(--weight-bold);
    text-decoration-line: underline;
    text-decoration-thickness: 3px;
}
```

- The 5-part chain `header nav ul li a` broke if any wrapper
  changed. `.nav__link` works wherever the link is.
- `.active` became `aria-current="page"`, so screen readers
  announce the current page and the CSS styles that same
  attribute.
- Keyboard users got no feedback before. Hover and focus now
  share one rule.
- The current-page rule wins because the **states layer** comes
  after components, not because of a longer selector.

## 3. Cards: lower specificity, reusable

**Before**
```css
.home-cards .card { background: #ffffff; border: 1px solid #7a8594;
  border-radius: 8px; padding: 24px; margin-bottom: 16px; }   /* 0,2,0 */
.home-cards .card h3 { font-size: 22px; margin-top: 0; }       /* 0,2,1 */
.home-cards .card a { color: #0b6e6e; font-weight: bold; }     /* 0,2,1 */
```

**After**
```css
.card {                                    /* 0,1,0 */
    --card-padding: var(--space-m);
    --card-radius: var(--radius-md);
    display: flex;
    flex-direction: column;
    gap: var(--space-xs);
    padding: var(--card-padding);
    border: var(--border);
    border-radius: var(--card-radius);
    background: var(--color-surface);
}

.card__title { font-size: var(--step-1); }  /* 0,1,0 */
.card__link { margin-block-start: auto; }   /* 0,1,0 */
```

- Specificity dropped from (0,2,1) to (0,1,0).
- The draft card only worked inside `.home-cards`. The new card
  is reused as the "On this page" sidebar box with no new CSS.
- Layout moved out of the component into `l-grid` (1, 2, then 3
  columns), and margin patches became `gap`.
- The scoped `--card-*` tokens allow future variants without
  new rules.

## 4. CTA: clearer name, variable-driven states

**Before**
```css
.cta { background: #0b6e6e; color: #ffffff; padding: 12px 24px;
  border-radius: 4px; font-weight: bold; text-decoration: none; }
.cta:hover { background: #084f4f; }
```

**After**
```css
.button {
    --button-bg: var(--color-accent);
    --button-text: var(--color-on-accent);
    min-block-size: var(--tap-target);
    border: var(--border-width) solid var(--button-bg);
    background: var(--button-bg);
    color: var(--button-text);
}

.button:hover,
.button:focus-visible { --button-bg: var(--color-link-hover); }
.button:active { --button-bg: var(--color-text); }
```

- `.cta` was vague, and `.button` says what it is.
- States swap one scoped variable instead of rewriting
  properties.
- Focus and active states were added, plus a 44px tap target.

## 5. Callouts: duplication removed

**Before**: two rules where 6 of 7 lines were identical.
```css
.callout-note { background: #ffffff; border: 1px solid #0b6e6e;
  border-left: 4px solid #0b6e6e; border-radius: 4px;
  padding: 16px 24px; margin-bottom: 16px; }
.callout-warning { background: #ffffff; border: 1px solid #a15c00;
  border-left: 4px solid #a15c00; border-radius: 4px;
  padding: 16px 24px; margin-bottom: 16px; }
```

**After**: one base rule plus a one-line variant.
```css
.callout {
    --callout-color: var(--color-note);
    padding: var(--space-s) var(--space-m);
    border: var(--border-width) solid var(--callout-color);
    border-inline-start-width: var(--border-width-thick);
    border-radius: var(--radius-sm);
    background: var(--color-surface);
}

.callout--warning { --callout-color: var(--color-warning); }
```

- A third callout type is now one line.
- `border-left` became the logical property
  `border-inline-start-width`.
- The visible `.callout__label` uses the same variable, so
  meaning never relies on color alone.

## 6. Unlayered draft removed

While `styles.css` existed, its unlayered rules beat every
layer. Each refactor commit deleted the draft rules it
replaced. The final commit removed the file and its `<link>`,
so the cascade is now controlled only by the layer order in
`main.css`.