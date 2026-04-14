# MTG Limited Land Calculator - Technical Documentation

## Overview
The **MTG Limited Land Calculator** is a single-page web application designed to help Magic: The Gathering players optimize their manabase for Limited formats (Draft/Sealed). It calculates the distribution of basic lands based on mana pips and non-basic land contributions, with a hard-coded safety logic for "splashed" colors.

## 1. Mathematical Model

The engine uses a weighted distribution algorithm inspired by the Hamilton/Largest Remainder Method, adjusted for non-basic land "mana value."

### Basic Proportion
The initial requirement for a land type is determined by the ratio of that color's pips to the total pips in the deck:
$$Fraction = \frac{ColorPips}{TotalPips} \times TotalLandSlots$$

### Non-Basic Adjustment
Non-basic lands (Duals and Tris) reduce the requirement for basic lands. Their contribution is subtracted from the basic land fraction:
- **Dual Lands:** Subtract $0.5$ from the requirement of both colors.
- **Tri Lands:** Subtract $0.33$ from the requirement of all three colors.

### Splash Protection
To prevent a color from being rounded to zero (rendering a card unplayable), the following logic is applied before the remainder distribution:
- If a color has **> 0 pips** but the calculation results in **0 basic lands** and **0 non-basic sources**, the algorithm forcefully assigns **1 basic land** to that color.
- Its remainder is set to $0$ to ensure it doesn't receive a second land during the rounding phase unless the math explicitly requires it.

### Remainder Distribution
After determining the "floor" (whole number) for each land type:
1. **Shortfall:** If the sum of floors is less than the available slots, lands are added to colors with the **highest fractional remainders**.
2. **Over-allocation:** If the sum exceeds slots (often due to Splash Protection), lands are removed from colors with the **most basic lands** currently assigned.

---

## 2. Data Schema (LocalStorage)

The application persists user state in the browser's `LocalStorage` under the key `mtg_deck_state`.

### State Object Structure
```json
{
  "total": 17,
  "pips": {
    "w": 15,
    "b": 15,
    "r": 2
  },
  "nb": {
    "nb-wb": 1,
    "nb-rwb": 1
  }
}
```

### Persistence Rules
- **Manual Save:** State is saved only when the "Calculate" button is clicked.
- **Auto-Load:** On page load, the app hydrates the UI and expands the Dual/Tri land menus if they contain saved data.

---

## 3. UI Component Architecture

### Dynamic Grid Generation
The input grids for Non-Basic lands are generated via JavaScript. This allows for:
- **Dynamic Styling:** Backgrounds are rendered as linear gradients using the specific mana color variables.
- **Theme Support:** Colors are mapped to CSS variables (`--w`, `--u`, etc.), enabling the Light/Dark mode toggle to change the entire UI's look without re-rendering the DOM.

### Accessibility and UX
- **Theme Persistence:** The user's Light/Dark mode preference is saved in `LocalStorage` separately from the deck data (Key: `theme`, Values: `<"dark"|"light">`).
- **Error Handling:** The UI validates that total land slots are set and that non-basic lands do not exceed the total land count before running calculations.

---

## 4. Developer Directives
- **Single File:** All HTML, CSS, and JS must reside in one file for portability.
- **Sanitization:** All inputs must be parsed via `parseInt(val, 10) || 0` to ensure mathematical stability.
- **No Dependencies:** Use only Vanilla JS; no external libraries (like jQuery) are permitted. This ensures the app is fully portable and runs out-of-the-box with no setup, package managers, or build dependencies required.

---

## License

Copyright (c) 2026 Daniel Hoover

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

-