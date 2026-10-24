# Design System — Cream & Forest

The theme file is `theme/Cream_Forest_Theme_v2.json`. This document records the values
and, importantly, **what the theme cannot do for you**.

---

## Palette

### Surfaces

| Token | Hex | L\* | Use |
|---|---|---|---|
| Canvas | `#F7F4EC` | 96.2 | Page background |
| Panel | `#E3DCCA` | 87.9 | Every visual background |
| Band | `#DBD3BE` | — | Table banded rows |
| Table header | `#D3C9B2` | 82.4 | Matrix and table headers |
| Border | `#CDC4AE` | — | Panel borders, dividers |
| Masthead / footer | `#1F3B2C` | — | The two dark bands |

Canvas and panel are **8.4 L\* apart** — roughly double the separation in a typical
cream dashboard, so panels read as distinct objects on the page rather than tinted
areas.

### Ink

| Token | Hex | Use |
|---|---|---|
| Primary | `#1F2A22` | Figures, headings, body |
| Secondary | `#54594F` | Supporting text |
| Tertiary | `#7E7B6E` | Labels, captions |
| On dark | `#F7F4EC` | Text on the forest bands |
| On dark, muted | `#B9C6B4` | Inactive nav, footer text |

### Data ink — monochrome forest ramp

| Step | Hex | Contrast on panel |
|---|---|---|
| 1 | `#1F3B2C` | 8.93:1 |
| 2 | `#2C543B` | 6.30:1 |
| 3 | `#396A4C` | 4.60:1 |
| 4 | `#4A7A5A` | 3.64:1 |
| 5 | `#52805B` | 3.34:1 |

All five clear 3:1 against the panel. A monochrome ramp is used because the categories
in this dataset are ordered by magnitude, not by kind — a categorical rainbow would
imply a distinction that does not exist.

### Semantic

| Meaning | Hex | Glyph |
|---|---|---|
| Good | `#2E5A3F` | ▲ |
| Bad | `#A4553A` | ▼ |
| Neutral | `#726C5D` | – |

**The glyphs are mandatory, not decorative.** No three-colour semantic set survives all
colour-vision deficiency types on hue alone; tested with Viénot deuteranopia and
protanopia transforms. Colour alone must never carry the meaning.

### Accent

| Token | Hex | Permitted uses |
|---|---|---|
| Gold | `#C19242` | **Two only:** the active-page underline, and the clear-filters icon |
| Gold, muted | `#8E6420` | Reserved |

One accent, used twice. Adding a third use dilutes it to decoration.

> **Note:** your existing report also uses `#DDA705` as gold on three Help-page buttons.
> Pick one gold and apply it everywhere, or the page reads as two design systems.

---

## Typography

| Role | Font | Size | Colour |
|---|---|---|---|
| Page title | Segoe UI Semibold | 32 | `#1F3B2C` |
| Hero figure | Segoe UI Semibold | 40 | `#1F3B2C` |
| Figure | Segoe UI Semibold | 26 | `#1F2A22` |
| Volume figure | Segoe UI Semibold | 28 | `#1F2A22` |
| Section label | Segoe UI | 9, caps | `#7E7B6E` |
| Body | Segoe UI | 11 | `#54594F` |
| Sub-line | Segoe UI | 10 | `#54594F` |
| Nav | Segoe UI / Semibold | 12 | `#F7F4EC` / `#B9C6B4` |
| Caption | Segoe UI | 9 | `#7E7B6E` |

**Four levels of hierarchy, no more.** Use `Segoe UI Semibold` from the font list rather
than pressing **B** — bolding Segoe UI produces a heavier, less even weight.

Theme text classes: `title` and `header` use Segoe UI Semibold; `callout` uses Segoe UI
Light; `label` and `largeLabel` use Segoe UI.

---

## Layout

| Rule | Value |
|---|---|
| Canvas | 1280 × 720, 16:9 |
| Outer margin | 28 |
| Inner panel padding | 28 |
| Panel corner radius | 6 |
| Panel border | 1px `#CDC4AE` |
| Left edges | 28 (page) and 56 (inside panels) — **only these two** |
| Right edge | 1252 |
| Masthead | y 0 – 84 |
| Working field | y 84 – 652 |
| Footer | y 652 – 720 |

The three-band structure is not a style choice. Your social icons exist as **white SVGs
only**; a dark footer is what lets them be used unchanged. The masthead follows for
symmetry and because the white wordmark and nav work there too.

---

## What the theme does NOT control

This is the section to read before blaming the theme file.

| Object type | Theme reaches it? | You must set by hand |
|---|---|---|
| **Shapes** | **No** | Fill, border, corner radius, shadow |
| **Buttons** | **No** | Fill, border, text colour, size, icon |
| **Text boxes** | **No** | Font, size, colour, alignment, background |
| **Images** | **No** | Nothing can recolour an imported image |
| Cards | Partly | Theme sets defaults; any explicit setting wins |
| Slicers | Partly | Explicit `items.background` overrides the theme |
| Charts | Partly | A **conditional or gradient fill rule overrides `dataColors` completely** |

### The three things that caught this project out

1. **14 bar charts carry a gradient fill rule on `dataPoint`.** `dataColors` in a theme
   has no authority over a fill rule. The bars stayed blue and coral through four theme
   revisions. Clear each one: Format visual → Bars → Colors → the **fx** button → remove
   the rule.

2. **12 slicers carry an explicit `items.background` of `#07345F`.** Their *font* colour
   is theme-bound, so after applying a green theme they rendered dark green on navy.
   Clear the background: Values → Background → colour picker → **Revert to default**.

3. **A theme must set colours, never switch features on.** An early version of this
   theme set `"show": true` on `title` and `background` under the global `"*"` selector.
   Every button and text box in the report then displayed its internal object name.
   The shipped theme sets `show: true` nowhere.

---

## Theme file facts **[VERIFIED]**

| | |
|---|---|
| File | `Cream_Forest_Theme_v2.json` |
| Size | 26,713 bytes |
| Valid JSON | Yes |
| Named visual types styled | 25 |
| Global `"*"` selector keys | `outspacePane`, `filterCard` only |
| `"show": true` under global `"*"` | **None** |
| `pivotTable` block | **Absent** — deliberately; the 12 pivotTables in this report are sparklines and images |
| Imposed font sizes | **None** — an imposed `fontSize` clipped card text in an earlier version |
| Slicer properties set | 11 |
| Malformed hex values | None |

---

## Design decisions worth keeping

**The landing page is the income statement.** Gross less discounts is net; net less cost
is profit. Laying the page out as that chain makes the structure carry meaning instead
of decoration, and it gives every number a reason to sit where it sits.

**Baseline alignment across mixed sizes.** The five chain figures share a baseline and a
box height; Profit is 40pt against 26pt for the others. Different sizes on a shared
baseline read as deliberate; different sizes on different baselines read as broken.

**No icon beside every figure.** An icon per tile is the generic move. The chain carries
the meaning.

**The limitations panel is on the landing page.** This is the clearest available signal
that the person who built the report knows what it can and cannot carry.
