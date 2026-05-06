# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A French-language personal budget tracker that runs entirely in a single `index.html` file. There is no build system, no package manager, no dependencies, and no backend — everything is vanilla HTML/CSS/JS.

## Development

Open `index.html` directly in a browser (no server needed). There are no build, lint, or test commands.

## Architecture

The entire application lives in `index.html`. The structure is: CSS in `<style>`, markup in `<body>`, and all logic in a single `<script>` block starting at line 276.

### Data persistence

All state is stored in `localStorage`. Helper wrappers `ls(key, default)` (read) and `ss(key, value)` (write) are used throughout. Keys:

| Key | Content |
|---|---|
| `b_sal` | Monthly net salary (number) |
| `b_exp` | Current month's expenses (array) |
| `b_goals` | Savings goals (array) |
| `b_hist` | Monthly history, capped at 12 entries (array) |
| `b_cumul` | Cumulative savings across all months (number) |
| `b_months` | Count of closed months (number) |
| `b_cats` | Budget categories with percentages and colors (array) |

### Budget category system

Categories are objects `{ id, name, pct, color }`. The `pct` values should sum to 100. Two category IDs are semantically special — `'savings'` and `'reserve'` — and are identified throughout the code with `c.id === 'savings' || c.id === 'reserve'`. These are excluded from the "Disponible ce mois" (available funds) calculation: only non-savings categories count toward spendable budget. When a month is closed (`resetMonth()`), the savings calculation uses unspent allocation from savings/reserve categories.

### Render flow

State mutations always call the appropriate render functions manually. There is no reactive framework. The key render functions and what they update:

- `renderEnvelopes()` — budget tab cards, disponible ring, triggers `renderCatSelect()` and `renderPie()`
- `renderHistList()` — current month expense list
- `renderGoals()` — goals tab
- `renderCategories()` — categories tab with percentage editor
- `renderChart()` — history tab bar chart and month detail list
- `renderPie()` — canvas donut chart on budget tab

### Month close (`resetMonth()`)

Calculates `savedAmt` as the sum of (allocation − spent) for savings/reserve categories, pushes a record to `history`, increments `monthsTracked` and `cumulSavings`, then clears `expenses`. History is capped at 12 entries (oldest entry is shifted off).

### Charts

- **Donut chart**: drawn on a `<canvas>` (180×180) using the Canvas 2D API; the inner hole is painted over with the background color `#18181c`.
- **Bar chart**: pure HTML `<div>` elements with inline `height` styles; no canvas.
- **Disponible ring**: SVG `<circle>` animated via `stroke-dashoffset`; the circle has radius 22 giving circumference ≈ 138.2, which is hardcoded in the SVG `stroke-dasharray`.

### Goal allocation

`allocGoal(i)` distributes the current month's total savings budget equally across all goals: `share = Math.round(savingsTotal / goals.length)`.
