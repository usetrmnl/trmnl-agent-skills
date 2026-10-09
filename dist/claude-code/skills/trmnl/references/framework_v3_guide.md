<!--
VERBATIM copy from TRMNL core (do NOT hand-edit).
Source: TRMNL core — framework v3 supplement
Keep in sync via: bin/sync-from-core
-->

# Framework v3 — Agent Guide

> **applies to:** plugins on framework v3.x (v3.0.0+, currently v3.4.0)
> **purpose:** supplements `agent_prompt.md` and the base template guide. all existing rules, workflows, tool usage, and gates still apply. this document is the authoritative source for v3-specific colors, grayscale scale, and label variants — when the base guide defers to "the framework supplement," it means this file.

---

## framework version context

the TRMNL design system framework is now at v3.4.0. key releases:

| version | what changed |
|-------|------------|
| 2.3.7 | last v2 release (fixed font asset URLs) |
| 3.0.x | color support, CSS variable architecture, extended grayscale. fixes: `data-fit-value` text shrinking, `gap` with flex and grid |
| 3.1.x | TRMNL pixel fonts become the default, 3x3 mashups, scale and text scale modifiers, `inverse` class |
| 3.2.x | open source, themes, `TRMNLPaint`, `TRMNLCharts` (adaptive charts), `image--adaptive` icons. deprecates `border--h-1` to `border--h-7` and `table-overflow-counter` |
| 3.3.x | maps (`TRMNLMaps`), position utilities (`absolute`, `top--*`, `z--*`), `outline--muted`. fix: `.mashup-cell` without a parent |
| 3.4.x | custom typefaces (`typeface` class), card border art keeps its corners |

a plugin uses the latest release unless its owner pinned an older version. a pinned plugin has only the features up to its pinned version. if a 3.2+ feature (`TRMNLCharts`, `TRMNLPaint`, `image--adaptive`) renders nothing or the chart is empty, tell the user that the plugin may be pinned to an older framework version.

---

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
- chart patterns and grayscale images unchanged (existing charts keep working; build new charts with `TRMNLCharts`, see **v3.2.x: charts** below)
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

**map each class only once, and always from the original markup.** `gray-40` becomes `gray-45` and `gray-45` becomes `gray-40`. if you remap `gray-40` to `gray-45` first and then remap all `gray-45` to `gray-40`, both end up as `gray-40`. the same error turns `gray-20` into `gray-10` in two steps.

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

## v3.0.x fixes

- **`data-fit-value`:** sub-pixel glyph differences no longer shrink text to `minFontSize` by mistake. when `data-value-fit-max-height` is not set, fitting checks the width only.
- **`gap`:** `gap--small`, `gap--medium` and the other gap classes now work correctly with flex and grid layouts.

**for the agent:** no markup changes needed. continue to use `data-fit-value="true"` as before.

---

## v3.1.x: TRMNL pixel fonts are the default

from 3.1, low-density screens use the TRMNL12, TRMNL16 and TRMNL21 pixel fonts by default. before 3.1 they used the Classic fonts. high-density screens still use Inter Variable.

**for the agent:** no markup changes needed. do not add a font class. if a user says the text looks different after an update to 3.1 or later, the new default font is the cause.

---

## v3.1.x: inverse

add the `inverse` class to an element. the element and all of its children change to the opposite color scheme: dark background with light text on a light screen, and light background with dark text on a dark-mode screen. the other elements on the screen do not change.

use it to make one row or one card stand out from the others, for example the active item, the current event or an occupied room:

```html
<div class="item inverse">
  <div class="meta"></div>
  <div class="content">
    <span class="title">Board room</span>
    <span class="description">Occupied · 11:00 - 12:30</span>
  </div>
</div>
<div class="item">
  <div class="meta"></div>
  <div class="content">
    <span class="title">Huddle room</span>
    <span class="description">Available</span>
  </div>
</div>
```

**for the agent:** `inverse` works on every bit depth, color panel and theme. use it on a container (`item`, a card, a table row). for a single label, use `label--inverted`.

---

## v3.2.x: themes

3.2 adds themes. a theme changes the colors of every framework class on the screen: backgrounds, text, borders and chart colors. dark mode works the same way.

a theme can only change colors that come from the framework. a color in an inline style, or a hex color in JavaScript, does not change. on a dark or colored theme it can become unreadable.

**for the agent:**
- take every color from a framework class (`bg--*`, `text--*`, `label--*`, `border--*`).
- in JavaScript, take every color from `TRMNLCharts` or `TRMNLPaint`. never write a hex color such as `#000000` or a color name such as `"black"`.
- you do not need to know which theme the user has. the framework applies it.

---

## v3.2.x: charts (`TRMNLCharts`)

`TRMNLCharts` gives Highcharts the colors, fonts, grid lines and axes of the current screen. the chart then matches the device, the bit depth, dark mode and the theme. `TRMNLCharts` ships in the framework runtime, so it needs no script tag of its own.

**this section overrides template guide §13 for colors and fonts.** keep the §13 script tags, chart types and data patterns. do not copy the hex colors, the `colors: ["#000000", ...]` lists, the font settings or the `trmnl.com/images/grayscale/gray-N.png` pattern fills from §13. `TRMNLCharts` gives all of them.

### Highcharts

```html
<script src="https://trmnl.com/js/highcharts/12.3.0/highcharts.js"></script>
<script src="https://trmnl.com/js/highcharts/12.3.0/pattern-fill.js"></script>

<div class="layout layout--col">
  <div class="flex gap--small">
    <span class="w--4 h--4 rounded--full" data-chart-series="0"></span><span class="label">Sent</span>
    <span class="w--4 h--4 rounded--full" data-chart-series="1"></span><span class="label">Read</span>
  </div>
  <div id="chart" class="w--full"></div>
</div>

<script>
  // TRMNLCharts arrives with the framework runtime, Highcharts with its own script tag: wait for both.
  function whenReady(cb) {
    var tries = 0;
    (function attempt() {
      if (window.TRMNLCharts && window.Highcharts) return cb();
      if (++tries > 200) return;
      setTimeout(attempt, 50);
    })();
  }

  whenReady(function () {
    var el = "chart";
    TRMNLCharts.watch(el, function () {
      var chart = Highcharts.chart(el, TRMNLCharts.merge(TRMNLCharts.options({ el: el }), {
        chart: { type: "column", height: TRMNLPaint.px(180, { el: el }) },
        xAxis: { categories: {{ days | json }} },
        series: [
          { name: "Sent", data: {{ sent | json }}, color: TRMNLCharts.series(0, 2, { el: el }) },
          { name: "Read", data: {{ read | json }}, color: TRMNLCharts.series(1, 2, { el: el }) }
        ]
      }));
      TRMNLCharts.applySwatches({ el: el });
      return chart;
    });
  });
</script>
```

### Chartkick

pass the series colors in `colors` and the merged options in `library`:

```javascript
whenReady(function () {   // same whenReady as above, but wait for window.Chartkick
  var el = "chart";
  TRMNLCharts.watch(el, function () {
    return new Chartkick.LineChart(el, {{ visits | json }}, {
      adapter: "highcharts",
      points: false,
      colors: [TRMNLCharts.series(0, 1, { el: el })],
      library: TRMNLCharts.merge(TRMNLCharts.options({ el: el }), {
        chart: { height: TRMNLPaint.px(260, { el: el }) },
        plotOptions: { series: { lineWidth: TRMNLPaint.px(4, { el: el }) } }
      })
    });
  });
});
```

### rules

- **always build the chart inside `TRMNLCharts.watch(el, function () { ... })`, and return the chart from the function.** the chart then rebuilds when the device, scale, dark mode or theme changes.
- **always start from `TRMNLCharts.options({ el: el })`.** it already sets the axes, grid lines, labels, data labels and fonts, and it turns off animation, tooltips, credits and the background. do not set these again.
- **`options()` also turns off the Highcharts legend and the point markers.** to show a legend, draw it in HTML (see the legend rule below) or set `legend: { enabled: true }`. to show points on a line, set `plotOptions: { series: { marker: { enabled: true } } }`.
- **add your own settings with `TRMNLCharts.merge(TRMNLCharts.options({ el: el }), { ... })`.** give `xAxis` and `yAxis` as objects, not arrays. `merge()` replaces an array, so an array removes the framework axis styles.
- **color series `i` of `n` series with `TRMNLCharts.series(i, n, { el: el })`.** count from 0. for one palette color, use `TRMNLCharts.paint("gray-30", { el: el })`.
- **pass every height, width, offset and line width through `TRMNLPaint.px(n, { el: el })`.** Highcharts numbers do not follow the device scale on their own.
- **load `pattern-fill.js` with Highcharts.** on 1-bit and 2-bit screens the series colors are patterns, and Highcharts needs this module to draw them.
- **for a legend in HTML,** add `data-chart-series="0"`, `data-chart-series="1"` and so on to the legend markers, and call `TRMNLCharts.applySwatches({ el: el })` inside `watch()`. each marker gets the color of its series.

wrong — fixed colors do not follow dark mode or the theme:

```javascript
Highcharts.chart("chart", { colors: ["#000000", "#888888"], xAxis: { lineColor: "black" } });
```

right:

```javascript
TRMNLCharts.watch("chart", function () {
  return Highcharts.chart("chart", TRMNLCharts.merge(TRMNLCharts.options({ el: "chart" }), {
    series: [{ data: {{ values | json }}, color: TRMNLCharts.series(0, 1, { el: "chart" }) }]
  }));
});
```

---

## v3.2.x: adaptive icons

add `image--adaptive` to a single-color icon. the framework paints the icon in the icon color of the screen, so the icon follows dark mode and the theme. the framework uses only the shape of the icon and ignores its own colors. it works with an SVG or a PNG that has a transparent background.

```html
<img class="image image--adaptive" src="https://trmnl.com/images/plugins/trmnl--render.svg" alt="TRMNL">
```

**for the agent:** use `image--adaptive` for every single-color icon and logo, including the `title_bar` icon. do not use it on a photo or a multi-color logo, because it paints the whole shape in one color.

---

## v3.2.x: deprecated classes and attributes

these still render, but do not use them in new markup:

| deprecated | use instead | note |
|------------|-------------|------|
| `border--h-1` to `border--h-7` | `border--h-10` to `border--h-75` | 4.0 removes the old classes. the same applies to `border--v-1` to `border--v-7`. |
| `table-overflow-counter` | `data-table-overflow-counter` | only the `data-` form is valid HTML. `data-table-overflow-counter="false"` hides the "and X more" row. |

**for the agent:** use the new names in new markup. do not rename them in existing markup unless the user asks: the old border levels do not map one-to-one to the new shades, so a rename can change how the plugin looks.

---

## v3.3.x: maps

`TRMNLMaps` draws MapLibre GL JS maps. every map layer paints from a framework slot, so a map adapts to bit depth, color panels, dark mode and themes with no device checks.

**for the agent:** the full map reference is in the Design System Template Guide §20 (maps). read it before you build any map plugin.

### a card over a map

the 3.3 position utilities put one element over another: `absolute`, `relative`, `top--{size}`, `right--{size}`, `bottom--{size}`, `left--{size}`, `inset--{size}`, and `z--0` to `z--3`. put the card inside the `.map` container, so the offsets start at the map's corner:

```html
<div id="map" class="map stretch w--full rounded--base">
  <div class="absolute top--2 left--2 z--2 p--4 w--max-60 bg--canvas outline outline--muted">
    <!-- card content -->
  </div>
</div>
```

- give the card a solid fill: `bg--canvas`, `bg--white` or `bg--black`. a gray fill is a dither tile, and small text on it is unreadable on 1-bit.
- draw the edge with `outline`, or `outline--muted` over a busy map. never use a box-shadow.
- keep the bottom-right corner clear. the OpenStreetMap credit sits there.
- the framework does not fit or clamp an element out of flow. give the card a width and keep its text short.

---

## v3.4.x: custom typefaces

the `typeface` class and the `--framework-typeface` variable set one font for every text role inside a container.

**for the agent:** do not use it. a custom font needs an `@font-face` rule, and the no-custom-styles rule bans it. the device fonts are the default.

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

- **using inline color styles instead of framework color classes** — `style="color: red"` won't render correctly on ePaper. use `text--error` or `text--red`.
- **assuming all devices support color** — chromatic classes fall back gracefully, but design primarily for grayscale. color is an enhancement.
- **using deprecated gray names in new markup** — prefer `gray-10` through `gray-75` over `gray-1` through `gray-7` in new templates.
- **not checking dither shift when updating v2 plugins** — if editing existing markup that was built for v2, verify grayscale shades look correct on v3. the 1-bit dither patterns changed.
- **remapping a gray class twice during a v2 migration** — `gray-40` and `gray-45` swap places. map each class once, from the original markup.
- **hex colors in a chart** — `colors: ["#000000"]` or `lineColor: "black"` does not follow dark mode or the theme. use `TRMNLCharts.series()` and `TRMNLCharts.options()`.
- **a chart built outside `TRMNLCharts.watch()`** — it does not rebuild when the device, scale, dark mode or theme changes.
- **`xAxis` or `yAxis` as an array in `TRMNLCharts.merge()`** — the array replaces the framework axis styles. give an object.
- **a fixed pixel size in a chart** — `height: 180` does not follow the device scale. use `TRMNLPaint.px(180, { el: el })`.
- **a single-color icon without `image--adaptive`** — the icon keeps its own color and can disappear on a dark theme.
- **deprecated names in new markup** — use `border--h-10` to `border--h-75`, not `border--h-1` to `border--h-7`, and `data-table-overflow-counter`, not `table-overflow-counter`.

---

## updated quality checklist additions

in addition to the existing checklist from `agent_prompt.md`:

- [ ] if using chromatic colors: verified they serve a purpose (semantic or user-requested)
- [ ] if using chromatic colors: design still readable in grayscale fallback
- [ ] if editing a v2 plugin: checked for dither lightness shift on gray shades
- [ ] using `gray-10` through `gray-75` naming (not deprecated `gray-1` through `gray-7`) in new markup
- [ ] semantic labels use `label--primary/success/error/warning` (not inline color styles)
- [ ] if editing a v2 plugin: each gray class was remapped once, from the original markup
- [ ] if the markup has a chart: built inside `TRMNLCharts.watch()`, options from `TRMNLCharts.options()`, series colors from `TRMNLCharts.series()`, sizes through `TRMNLPaint.px()`, no hex colors
- [ ] single-color icons use `image--adaptive`
- [ ] new markup uses `border--h-10` to `border--h-75` and `data-table-overflow-counter`
