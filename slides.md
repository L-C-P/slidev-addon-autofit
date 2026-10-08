---
theme: default
addons:
  - ./
---

# Autofit Addon Demo

Test presentation for the autofit addon

---

## Features

- Slides that overflow are scaled down automatically
- The scale is applied before the slide is painted
- Works with slide transitions
- A manual `zoom` always wins

---
transition: slide-left
---

## Overflowing list

This slide is scaled down automatically.

- Item 1
- Item 2
- Item 3
- Item 4
- Item 5
- Item 6
- Item 7
- Item 8
- Item 9
- Item 10
- Item 11
- Item 12
- Item 13
- Item 14
- Item 15
- Item 16

---
transition: slide-up
---

## Overflowing table

| Feature | Status | Notes |
|---|---|---|
| Row 1 | ✅ | Some text |
| Row 2 | ✅ | Some text |
| Row 3 | ✅ | Some text |
| Row 4 | ✅ | Some text |
| Row 5 | ✅ | Some text |
| Row 6 | ✅ | Some text |
| Row 7 | ✅ | Some text |
| Row 8 | ✅ | Some text |
| Row 9 | ✅ | Some text |
| Row 10 | ✅ | Some text |
| Row 11 | ✅ | Some text |
| Row 12 | ✅ | Some text |

---
autofit: false
---

## Autofit disabled

`autofit: false` – this slide overflows on purpose.

- Item 1
- Item 2
- Item 3
- Item 4
- Item 5
- Item 6
- Item 7
- Item 8
- Item 9
- Item 10
- Item 11
- Item 12
- Item 13
- Item 14
- Item 15
- Item 16

---
zoom: 0.8
---

## Manual zoom

`zoom: 0.8` in the frontmatter takes precedence over autofit.

---

# Thank You

End of demo presentation
