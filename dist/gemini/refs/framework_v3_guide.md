<!--
VERBATIM copy from TRMNL core (do NOT hand-edit).
Source: TRMNL core — framework v3 supplement
Keep in sync via: bin/sync-from-core
-->

# Framework v3 — Agent Guide

> **applies to:** plugins on framework v3.x
> **purpose:** supplements `agent_prompt.md` and the base template guide. all existing rules, workflows, tool usage, and gates still apply. this document is the authoritative source for v3-specific colors, grayscale scale, and label variants — when the base guide defers to "the framework supplement," it means this file.

## color system

v3's headline feature. the framework now supports full color for ePaper devices that have color panels.

### chromatic palette

10 hues, 14 lightness steps each (150+ tokens):

**hues:** `red`, `orange`, `yellow`, `lime`, `green`, `cyan`, `blue`, `violet`, `purple`, `pink`

**class pattern:**
- base: `bg--{hue}`, `text--{hue}` (e.g. `bg--red`, `text--blue`)
- with step: `bg--{hue}-{step}`, `text--{hue}-{step}` (e.g. `bg--red-40`, `text--green-60`)
- steps: `10` (darkest) through `75` (lightest), in increments of 5

**automatic device fallback:** chromatic classes adapt to device capability without conditional markup:
- grayscale devices: colors fall back to perceptually equivalent gray values (LAB L* mapping)
- limited-color panels: unavailable colors map to the closest supported hue
- full-color panels: colors render at full saturation

this means you can write `bg--red` and it works everywhere. no `{% if device.has_color %}` needed.

### semantic colors

intent-based classes that map to specific hues:

```
bg--primary   / text--primary   / label--primary   → blue (highlights, accents)
bg--success   / text--success   / label--success   → green (confirmations, positive)
bg--error     / text--error     / label--error     → red (errors, critical)
bg--warning   / text--warning   / label--warning   → orange (cautions, alerts)
```

prefer semantic classes when the intent is clear. use direct hue classes for decorative or brand-specific color.

### label color variants

new in v3. colored badges for status indicators:

```html
<span class="label label--primary">active</span>
<span class="label label--success">complete</span>
<span class="label label--error">failed</span>
<span class="label label--warning">pending</span>
```

the existing `label--filled` (black) is unchanged.

---

## extended grayscale

### the new scale

v3 doubles the grayscale resolution from 7 to 14 distinct shades:

`black` → `gray-10` → `gray-15` → `gray-20` → `gray-25` → `gray-30` → `gray-35` → `gray-40` → `gray-45` → `gray-50` → `gray-55` → `gray-60` → `gray-65` → `gray-70` → `gray-75` → `white`

each step now has its own unique 1-bit dither pattern (16x16 tiles, 6.25% density increment).

### legacy gray names

`gray-1` through `gray-7` still work but are deprecated. prefer `gray-10` through `gray-75`.

### dither lightness shift (1-bit mode)

same class names produce different lightness values in v3 vs v2. most shades shifted by 6.25-12.5%. this is an internal rendering change — class names didn't change, but what they produce on 1-bit hardware did.

if a user says their plugin "looks different" after a framework update, the dither shift is likely the cause. see the **migrating from v2** section below for the full remap table and upgrade workflow.

---

## what's unchanged

these v2 behaviors are stable in v3:

- all existing class names compile and render
- `layout`, `grid`, `flex`, `columns`, `item`, `table`, `progress` class APIs unchanged
- `bg--black`, `bg--white`, `bg--gray-{N}`, `text--black`, `text--white`, `text--gray-{N}` all still work
- `title_bar` structure unchanged
- `data-fit-value`, `data-clamp`, `data-overflow-max-cols` attributes unchanged
- chart patterns and grayscale images unchanged
- template structure (no `view` wrapper, start with `layout`) unchanged
- all tool workflows (show_merge_variables → write_markup → screenshot_markup) unchanged

---

## migrating from v2

when a user arrives with a plugin that was built on v2 and has just been bumped to v3, their existing markup may render differently because of the 1-bit dither shift. follow this workflow to restore visual parity and optionally enhance with color.

### step 1: remap grayscale classes (the dither shift)

the v3 1-bit dither scale changed from 7-step non-linear to 14-step linear. the same `gray-N` class name produces a different lightness on v3 than it did on v2. to preserve the existing visual appearance, remap every gray class in the markup using this table:

| v2 class | v2 lightness | v3 lightness | remap to |
|----------|-------------|-------------|----------|
| `gray-10` | 6.25% | 6.25% | — (unchanged) |
| `gray-15` | 6.25% | 12.5% | `gray-10` |
| `gray-20` | 12.5% | 18.75% | `gray-15` |
| `gray-25` | 12.5% | 25% | `gray-15` |
| `gray-30` | 25% | 31.25% | `gray-25` |
| `gray-35` | 25% | 37.5% | `gray-25` |
| `gray-40` | 50% | 43.75% | `gray-45` |
| `gray-45` | 50% | 56.25% | `gray-40` |
| `gray-50` | 75% | 62.5% | `gray-60` |
| `gray-55` | 75% | 68.75% | `gray-60` |
| `gray-60` | 87.5% | 75% | `gray-70` |
| `gray-65` | 87.5% | 81.25% | `gray-70` |
| `gray-70` | 93.75% | 87.5% | `gray-75` |
| `gray-75` | 93.75% | 93.75% | — (unchanged) |

scan every markup size for `bg--gray-*`, `text--gray-*`, and `label--gray-*` references. apply the remap exactly.

### step 2: update legacy gray names

replace deprecated short names with the new scale:

- `gray-1` → `gray-10`
- `gray-2` → `gray-20`
- `gray-3` → `gray-30`
- `gray-4` → `gray-40`
- `gray-5` → `gray-50`
- `gray-6` → `gray-60`
- `gray-7` → `gray-70`

### step 3: verify visual parity

use the normal write_markup → screenshot_markup loop. the goal is visual parity with the v2 appearance — if grays look noticeably lighter or darker than before, a remap was missed.

### step 4 (optional): offer color enhancements

only after the migration is verified, ask the user if they want color enhancements — semantic labels (`label--success`, `label--error`, `label--warning`), colored backgrounds, or colored text. do not add color without asking. colors fall back to grayscale on monochrome devices.

### what NOT to change during a migration

- template structure (`layout`, `grid`, `flex`, `columns`) — unchanged in v3
- `title_bar`, `data-fit-value`, `data-clamp`, `data-overflow-max-cols` — unchanged
- chart code and `trmnl.com/images/grayscale/gray-{N}.png` references — unchanged
- Liquid template logic, merge variables, data flow — unchanged

the migration is a CSS class migration. don't restructure templates unless the user asks.

---

## when to use color

**do use color when:**
- the data has semantic meaning that color reinforces (errors = red, success = green)
- the user explicitly asks for color
- status indicators benefit from color coding (labels, badges)
- the plugin targets a color-capable device

**don't use color when:**
- grayscale contrast alone conveys the information clearly
- the user hasn't mentioned color and their device is grayscale-only
- you're using it purely for decoration without semantic meaning

**default behavior:** if unsure whether to use color, build with grayscale first — it works everywhere. mention to the user that color classes are available if they want to enhance for color devices.

---

## updated common mistakes

additions to the common mistakes list from `agent_prompt.md`:

- **using inline color styles instead of framework color classes** — `style="color: red"` won't render correctly on e-ink. use `text--error` or `text--red`.
- **assuming all devices support color** — chromatic classes fall back gracefully, but design primarily for grayscale. color is an enhancement.
- **using deprecated gray names in new markup** — prefer `gray-10` through `gray-75` over `gray-1` through `gray-7` in new templates.
- **not checking dither shift when updating v2 plugins** — if editing existing markup that was built for v2, verify grayscale shades look correct on v3. the 1-bit dither patterns changed.

---

## updated quality checklist additions

in addition to the existing checklist from `agent_prompt.md`:

- [ ] if using chromatic colors: verified they serve a purpose (semantic or user-requested)
- [ ] if using chromatic colors: design still readable in grayscale fallback
- [ ] if editing a v2 plugin: checked for dither lightness shift on gray shades
- [ ] using `gray-10` through `gray-75` naming (not deprecated `gray-1` through `gray-7`) in new markup
- [ ] semantic labels use `label--primary/success/error/warning` (not inline color styles)
