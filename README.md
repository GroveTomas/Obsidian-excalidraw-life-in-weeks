# Obsidian Excalidraw — Life in Weeks

A small **Excalidraw Automate (EA) script** for the [Obsidian Excalidraw Plugin](https://github.com/zsviczian/obsidian-excalidraw-plugin) that generates a “**life in weeks**” (memento mori) grid:

- **Rows** = years (your chosen life expectancy)
- **Columns** = weeks (52)
- **Each cell** = one week of life (circle / square / diamond)
- Weeks already lived are shaded automatically

This is meant as a gentle reminder to not waste time.

---

## What it generates

- Prompts you for:
  1. **Birth year**
  2. **Life expectancy (years)** (default: `78`)
  3. **Shape type**: Circle / Square / Diamond
  4. (If square) **Corner style**: Sharp / Rounded
- Draws a grid of weekly shapes
- Colors “past” weeks differently from “future” weeks
- Adds **decade separators** (every 10th year)
- Adds a header with:
  - Title: `Your Life in Weeks`
  - Subtitle aligned to the week grid:
    - Left edge: `born ...`
    - Middle: totals / lived / remaining
    - Right edge: `generated 2026-02-26 16:45`

---

## Requirements

- Obsidian
- Excalidraw plugin by zsviczian (with Script Engine / Excalidraw Automate enabled)

---

## Installation

1. In Obsidian, open **Settings → Excalidraw** and find the **Script Engine folder** setting.
2. Copy the script file into that folder:

   - `Life in Weeks.md`

3. Reload Obsidian (or run the “Reload app” command) if the script doesn’t appear immediately.

---

## Usage

1. Open any `.excalidraw` file (canvas).
2. Open the Command Palette (`Ctrl/Cmd + P`)
3. Run the script (it will appear with the file name, e.g. **Life in Weeks**).
4. Answer the prompts.
5. The drawing will be inserted into the current Excalidraw view.

Tip: after generation, use **Fit to content** / zoom out to see the full grid.

---

## Customizing colors (optional)

You can influence the colors and styles used by the script:

- Select **one or two existing shapes** in your drawing *before* running the script.
- The script reads styling from the selected shapes:
  - **1st selected shape** → style for *past* (lived) weeks
  - **2nd selected shape** → style for *future* (remaining) weeks

Notes:
- Selected elements must match the shape type you chose (e.g., select rectangles if you chose square).
- If nothing is selected, default colors are used.

---

## Performance notes

This drawing can contain **thousands of elements**, so it may feel heavy in Excalidraw/Obsidian.

What the script already does to help performance:
- Uses a **single** `ea.addElementsToView(...)` commit at the end (no incremental commits).
- Minimizes repeated style changes inside loops.

If you want the canvas to be even faster long-term, consider:
- Using fewer years (lower life expectancy value)
- Using a smaller shape size in the script constants
- Removing labels / headers (text elements are relatively expensive)

---

## License

Choose one:
- Add an MIT license (recommended for scripts)
- Or keep it unlicensed (default: all rights reserved)

If you want, tell me which license you prefer and I’ll generate a `LICENSE` file.

---

## Credits / inspiration

Inspired by the Excalidraw Automate scripting patterns from the Obsidian Excalidraw plugin and the popular “life in weeks” visualization concept.