# quarto-touying-slides

A Quarto extension for creating Typst-based slide presentations using the [Touying](https://touying-typ.github.io/) framework with the Metropolis theme.

## Installation

```bash
quarto add calote/qmd-ptm-ty-slides
```

Or copy the `_extensions/qmd-ptm-ty-slides/` folder into your project.

## Basic Usage

Create a `.qmd` file with the following YAML front matter:

```yaml
---
title: "My Presentation"
author: "Author Name"
date: today
format: qmd-ptm-ty-slides-typst
---
```

Then render with:

```bash
quarto render my-presentation.qmd
```

## YAML Configuration Options

| Parameter | Type | Description | Default |
|---|---|---|---|
| `header-color` | string | Slide title bar background color (hex, with or without `#`) | `"003f72"` |
| `section-color` | string | Section slide background color for H1; if omitted, uses `header-color` | *(same as `header-color`)* |
| `section-color-2` | string | Section slide background color for H2 (only when `section-level >= 3`) | *(same as `section-color`)* |
| `section-color-3` | string | Section slide background color for H3 (only when `section-level >= 4`) | *(same as `section-color`)* |
| `accent-color` | string | Progress bar and accent element color | `"eb811b"` |
| `font-size` | string | Global document font size | `"20pt"` |
| `section-level` | int | Minimum heading level for content slides; levels below become section slides | `2` |
| `slide-numbering` | bool | Add hierarchical prefixes to slide titles (e.g., "1.2.3 Title") | `false` |
| `toc-slide` | bool | Insert a table of contents slide | `false` |
| `toc-levels` | int | Heading depth shown in TOC (1 = H1 only, 2 = H1+H2, etc.) | `1` |
| `toc-font-size` | string | Font size for TOC entries | `"1em"` |
| `toc-title` | string | Title of the TOC slide | `"Contenido"` |
| `toc-columns` | int | Number of columns in the TOC slide (1 = single column, 2 = two columns, etc.) | `1` |
| `center-equations` | bool | Center display math (`$$...$$`) correcting accumulated list indentation | `true` |
| `code-bg-color` | string | Background color for source code blocks; use `"none"` to disable | `"e8f0fe"` |
| `output-bg-color` | string | Background color for code output blocks; use `"none"` to disable | `"f0f4e8"` |
| `code-text-color` | string | Text color inside source code blocks; if omitted, uses the theme default | *(theme default)* |
| `handout-mode` | bool | Handout mode: collapses all `{{< pause >}}` steps, showing only the final state of each slide. **Must be declared at the root level of the YAML** (see [Handout Mode](#handout-mode)) | `false` |

### Color Values

Colors can be specified with or without a leading `#`. Both forms are equivalent and the `#` is automatically stripped for Typst compatibility:

```yaml
header-color: "#003f72"   # works fine (useful for color preview in editors)
header-color: "003f72"    # also works
```

To disable a background, pass the string `"none"` **in quotes**. Without quotes, YAML parses `none` as null and the default color is used instead:

```yaml
code-bg-color: "none"     # disables code background  ← correct
output-bg-color: "none"   # disables output background ← correct
output-bg-color: none     # ← wrong: YAML null, default color is applied
```

### Full Example

```yaml
---
title: "Statistics Course"
author: "Jane Doe"
date: today
format:
  touying-slides-typst:
    header-color: "#003f72"
    section-color: "#AB262D"
    section-color-2: "#7B241C"  # only needed when section-level >= 3
    accent-color: "#eb811b"
    font-size: "20pt"
    section-level: 2
    slide-numbering: false
    toc-slide: true
    toc-levels: 2
    toc-font-size: "0.85em"
    toc-columns: 2           # optional: two-column TOC
    toc-title: "Contents"
    code-bg-color: "#dce8ff"
    output-bg-color: "#e8f5e9"
    code-text-color: "#1a1a2a"
---
```

## Heading Structure

### Default behavior (`section-level: 2`)

By default, H1 headings become full-screen **section slides** and H2–H5 become regular **content slides** with a title bar:

| Heading | Slide Type |
|---|---|
| `#` (H1) | Section slide — full-screen background with `section-color` |
| `##` (H2) | Regular content slide |
| `###` to `#####` (H3–H5) | Nested content slides |

```markdown
# Module 1

## Introduction

Content of slide 1...

## Key Concepts

Content of slide 2...

# Module 2
```

### Configuring the section boundary (`section-level`)

The `section-level` parameter controls which heading level separates section slides from content slides. All levels **below** `section-level` become section slides; `section-level` and above become content slides.

| `section-level` | Section slides | Content slides |
|:---:|---|---|
| `1` | *(none)* | H1, H2, H3, H4, H5 |
| `2` *(default)* | H1 | H2, H3, H4, H5 |
| `3` | H1, H2 | H3, H4, H5 |
| `4` | H1, H2, H3 | H4, H5 |

> **Note:** The parameter is named `section-level` (not `slide-level`) because `slide-level` is a reserved Pandoc option and would be intercepted before reaching the Lua filter. Similarly, `handout-mode` is used instead of `handout` (a Quarto/Beamer reserved word) and must be placed at the root level of the YAML — see [Handout Mode](#handout-mode).

**Example with `section-level: 3`:**

```yaml
format:
  touying-slides-typst:
    section-level: 3
    section-color: "#AB262D"    # H1 section slides → dark red
    section-color-2: "#7B241C"  # H2 section slides → darker red
```

```markdown
# Part 1              ← section slide (H1 color)

## Chapter 1          ← section slide (H2 color)

### Introduction      ← content slide with title bar

### Methods           ← content slide with title bar

## Chapter 2          ← section slide (H2 color)
```

When a section-level heading has content after it (before the next heading), that content is automatically placed in a regular `#slide` with the same title, immediately following the section slide.

### Per-level section colors (`section-color-N`)

When multiple levels are section slides, assign a different background color to each level:

```yaml
section-color: "#003f72"      # H1 → dark blue
section-color-2: "#AB262D"    # H2 → dark red  (section-level >= 3)
section-color-3: "#1A5276"    # H3 → steel blue (section-level >= 4)
```

If `section-color-N` is not specified for a given level, that level falls back to the theme's default `neutral-dark` color (i.e., `section-color`).

All slides are PDF-linkable and can be included in the TOC when `toc-slide: true`.

## Titleless Slides

Use `---` inside the content of a slide to create a continuation slide **without a title**. The horizontal rule acts as a slide boundary: everything before it becomes the first slide (with the heading title), and everything after it becomes a new `#slide(title: none)`.

```markdown
## My Slide

Content of the first slide.

---

This slide has no title bar.
You can use code, tables, images, etc.

---

A third titleless slide.
```

This is particularly useful for:

- Showing a code block on one slide and its output on the next, without repeating the title.
- Creating full-content slides (e.g., a large figure) that look cleaner without a title bar.

> **Note:** In previous versions of the extension, `---` inside a slide rendered as a visual horizontal rule (`#horizontalrule`). If you need a decorative horizontal rule inside a slide, use a raw Typst block instead:
> ````markdown
> ```{=typst}
> #line(length: 100%)
> ```
> ````

## Handout Mode

Set `handout-mode: true` to generate a handout PDF. In handout mode, Touying collapses all incremental steps (`{{< pause >}}`) into a single slide showing all content at once. Each logical slide produces exactly one page, which is ideal for distributing to students or printing.

> **Important:** `handout-mode` must be declared at the **root level** of the YAML front matter, not under `format:`. Quarto intercepts both `handout` (reserved word for Beamer) and format-level boolean options in a way that prevents the value from reaching the Typst template. Placing it at the root level bypasses this.

The recommended workflow is a separate file (e.g., `my-slides-handout.qmd`) that includes the source presentation and adds `handout-mode: true` at the root:

```yaml
---
title: "My Presentation"
subtitle: "Subtitle"
author: "Author Name"
date: today
institute: "University"
handout-mode: true          # ← root level, NOT under format:

format:
  qmd-ptm-ty-slides-typst:
    aspect-ratio: "16-9"
    header-color: "#003f72"
    # ... same options as the original slides file ...
---

{{< include my-slides.qmd >}}
```

The source file `my-slides.qmd` does not need any `handout-mode` key — the default is `false`, handled by the Lua filter when the key is absent.

> **Note:** Handout mode is handled entirely by Touying (`config-common(handout: true)`). The `{{< pause >}}` shortcodes do not need to be removed from the source file.

## Display Math Centering

By default, display math equations (`$$...$$`) are centered across the full text column width, even when nested inside a list. Without this correction, Typst would apply the list's accumulated indentation to the equation, shifting it off-center.

This behavior is controlled by the `center-equations` option, which is `true` by default. To disable it and use Typst's native equation placement:

```yaml
format:
  qmd-ptm-ty-slides-typst:
    center-equations: false
```

> **Note:** If disabling from within `format:` has no effect (due to Quarto's format-level boolean merging), set it at the root level of the YAML instead:
> ```yaml
> ---
> center-equations: false
> format:
>   qmd-ptm-ty-slides-typst:
>     ...
> ---
> ```

## Shortcodes

### `{{< pause >}}`

Incremental reveal — content after the pause appears on the next overlay step.

```markdown
## My Slide

First item (visible immediately).

{{< pause >}}

Second item (appears on click).

{{< pause >}}

Third item (appears on next click).
```

### `{{< fontsize size >}}`

Changes the font size for the rest of the current slide.

```markdown
## Slide with a Table

{{< fontsize 0.75em >}}

| A | B | C |
|---|---|---|
| 1 | 2 | 3 |
```

## Div/Span Classes

### Alignment

Apply alignment to any block or inline content:

```markdown
:::{.center}
This text is centered.
:::

:::{.right}
Right-aligned text.
:::

:::{.center align="horizon"}
Horizontally and vertically centered.
:::
```

### Alert / Emphasis

Highlight or emphasize text:

```markdown
[This text is highlighted]{.alert}

:::{.alert}
An entire block highlighted in accent color.
:::

[Italic text]{.emph}
```

> **Note:** Bold (`**text**`) is rendered without accent coloring. Use `{.alert}` for accent-colored emphasis.

### Two-Column Layouts

Use nested divs for side-by-side columns:

```markdown
::::{.cols gutter="2em"}
:::::{.col}
**Left column**

- Item A
- Item B
:::::
:::::{.col}
**Right column**

- Item C
- Item D
:::::
::::
```

The `gutter` attribute controls the spacing between columns (any valid Typst length, e.g., `"1em"`, `"20pt"`).

## Code Block Styling

The extension supports independent background colors for source code and execution output.

| Block type | When it appears | Parameter | Default |
|---|---|---|---|
| Source code | Static ` ```r ``` ` blocks and `echo: true` chunks | `code-bg-color` | Light blue `"e8f0fe"` |
| Output / results | Output from executed `{r}` chunks | `output-bg-color` | Light green `"f0f4e8"` |
| Source code text | Text color inside source code | `code-text-color` | *(theme default)* |

**Static code block** (displayed only, not executed):

````markdown
```r
x <- c(1, 2, 3)
mean(x)
```
````

**Executed code block** with separate colors for source and output:

````markdown
```{{r}}
#| echo: true
x <- c(1, 2, 3)
mean(x)
```
````

Both block types are detected automatically. The Lua filter processes Quarto's internal cell structure (`Div.cell`, `Div.cell-code`, `Div.cell-output-*`) to apply the correct color to each part.

To disable a background:

```yaml
code-bg-color: "none"    # source code with no background
output-bg-color: "none"  # output with no background
```

> **Important:** `"none"` must be quoted. Unquoted `none` is parsed as YAML null and the default color is applied.

## Color Mapping

The extension maps YAML parameters to Touying Metropolis theme variables:

| YAML Parameter | Metropolis Variable | Visual Effect |
|---|---|---|
| `header-color` | `secondary` | Slide title bar background |
| `section-color` | `neutral-dark` | Section slide background (H1 by default) |
| `section-color-N` | *(passed via `config-page`)* | Section slide background for level N |
| `accent-color` | `primary` | Progress bar and accent elements |

`section-color-N` overrides the page fill of the corresponding section slide by passing `config-page(fill: color)` to Touying's `focus-slide`, which is the official Touying 0.6.1 mechanism for per-slide page customization.

## Slide Numbering

When `slide-numbering: true`, slide titles receive hierarchical numeric prefixes:

```
# Module 1         → section slide (no number shown in body)
## Introduction    → "1. Introduction"
### Details        → "1.1. Details"
#### More          → "1.1.1. More"
```

## Table of Contents

Enable with `toc-slide: true`. The TOC is inserted after the title slide and before the first section. Each entry is a clickable PDF link that jumps to the corresponding slide.

Control which heading levels appear with `toc-levels` (default `1` shows only H1 sections; `2` also shows H2 slides, etc.).

For long TOCs, set `toc-columns: 2` to split the entries into two side-by-side columns:

```yaml
toc-slide: true
toc-levels: 3
toc-columns: 2
```

## Extension File Structure

```
_extensions/touying-slides/
├── _extension.yml          # Extension metadata and configuration
├── typst-template.typ      # Main Typst template (slides() function)
├── typst-show.typ          # Pandoc template connecting YAML to Typst
├── slides-headings.lua     # Main filter: headings → slides, TOC, code block coloring
├── shortcodes.lua          # {{< pause >}} and {{< fontsize >}} shortcodes
└── typst-classes.lua       # Converts Span/Div classes to Typst functions
```

## Requirements

- [Quarto](https://quarto.org/) ≥ 1.4
- [Typst](https://typst.app/) (bundled with Quarto ≥ 1.4)
- [Touying](https://touying-typ.github.io/) 0.6.1 (included via Typst package manager)
