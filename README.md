# slidev-addon-autofit

A [Slidev](https://sli.dev) addon that automatically scales a slide down until its content fits, similar to PowerPoint's "shrink text on overflow".

## Features

- **Automatic Fit**: Slides whose content overflows the layout are scaled down to the largest factor that still fits
- **No Visible Shrinking**: The scale is applied before the slide is painted – slides appear at their final size, also during slide transitions
- **Respects Line Wrapping**: The factor is searched step by step, so changed line breaks at smaller scales are taken into account
- **Late Content**: Re-fits when content changes after the slide is shown (Mermaid diagrams, syntax highlighting, fonts, images)
- **Uses Slidev's Zoom**: Drives the same mechanism as `zoom:` in the frontmatter, so it works with any theme and layout
- **Per-Slide Control**: A manual `zoom:` always wins; `autofit: false` disables it for a single slide

## Requirements

- **Slidev**: 52 or newer
- **Browser**: any browser supporting `ResizeObserver` and the CSS `scale` property (all current browsers)

## Installation

```bash
npm install slidev-addon-autofit
```

## Usage

Add the addon to your Slidev presentation:

### Option 1: In `slides.md` frontmatter

```yaml
---
addons:
  - slidev-addon-autofit
---
```

### Option 2: In `package.json`

```json
{
  "slidev": {
    "addons": ["slidev-addon-autofit"]
  }
}
```

Every slide whose content overflows its layout is now scaled down automatically.

## Options

Set the options in the headmatter (first slide, applies to all slides) or in the frontmatter of a single slide:

```yaml
---
autofit: false          # disable autofit
---
```

```yaml
---
autofit:
  min: 0.5              # smallest allowed scale (default: 0.4)
---
```

| Option | Default | Description |
|---|---|---|
| `autofit: false` | – | Disables autofit (globally or for one slide) |
| `autofit.min` | `0.4` | Smallest scale the addon will apply; content that still doesn't fit overflows |

A manual `zoom:` in a slide's frontmatter always takes precedence over the automatic value.

## How It Works

1. **Measure**: A per-slide layer (`slide-top.vue`) watches the slide layout with a `ResizeObserver` and a `MutationObserver`. The content overflows when the layout's `scrollHeight` (or `scrollWidth`) is larger than its visible size.

2. **Fit**: The addon searches the largest scale at which the content fits and sets Slidev's `--slidev-slide-zoom-scale` – the same variable `zoom:` in the frontmatter sets. Each step forces a synchronous reflow, so line wrapping at that scale is measured.

3. **No Flicker**: Observer callbacks run after layout but before paint, so intermediate steps are never visible. Hidden slides have no size and are fitted the moment they become visible.

4. **Transitions**: Slidev's slide transitions use `transition: all`, which would animate the scale while a slide enters. The addon limits the transition on auto-fitted slide pages to `translate`, `transform`, `opacity`, `rotate`, `filter` and `clip-path`.

## Layouts and Themes

The addon scales the whole slide, exactly like `zoom:` in the frontmatter. Layouts that want to keep titles, frames or footers at a fixed size while only the content shrinks can compensate the zoom variable:

```css
.my-layout h1 {
  font-size: calc(2rem / var(--slidev-slide-zoom-scale, 1));
}
```

## Troubleshooting

### A slide is not scaled
- Check whether the slide has a manual `zoom:` in its frontmatter – it takes precedence
- Check for `autofit: false` in the slide's frontmatter or in the headmatter
- The content may be clipped inside a scroll container (e.g. a code block with `maxHeight`) – such content doesn't overflow the layout

### Content is still too large
- The scale doesn't go below `autofit.min` (default `0.4`); lower it or shorten the slide

### A custom transition no longer scales the slide
- On auto-fitted slides, `scale`, `width` and `height` of the slide page are not transitioned
- Disable autofit for that slide with `autofit: false`

## License

MIT

## Author

Denis Sowa

## Contributing

Contributions are welcome! Please open an issue or pull request on GitHub.
