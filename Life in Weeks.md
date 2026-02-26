/*

## Life in Weeks — Memento Mori

A life visualised as weekly shapes (life expectancy × 52 weeks).

Weeks already lived are shaded automatically. Fill in the rest as you age — a quiet reminder not to waste time.

## How to use

1. Run the script from the Command Palette.
2. Enter your **birth year** when prompted.
3. Enter your **life expectancy** (default 78).
4. Pick a **shape** from the suggester list.
5. If you picked Square, pick **corner style**.

## Customise colours

Optionally select **one or two shapes** before running:
- 1st shape → style for *past* (lived) weeks
- 2nd shape → style for *future* (remaining) weeks

```javascript
*/

// ─────────────────────────────────────────
//  Layout constants
// ─────────────────────────────────────────
const WEEKS_PER_YEAR = 52;

const SHAPE_SIZE  = 14;
const COL_GAP     = 5;
const ROW_GAP     = 10;

const CELL_SIZE  = SHAPE_SIZE + COL_GAP;
const ROW_HEIGHT = SHAPE_SIZE + ROW_GAP;

const LABEL_WIDTH  = 72;
const HEADER_H     = 30;
const START_X      = 0;
const START_Y      = 0;

// ─────────────────────────────────────────
//  Style defaults
// ─────────────────────────────────────────
let COLOR_PAST_FILL     = "#5c5c5c";
let COLOR_PAST_STROKE   = "#3a3a3a";
let COLOR_FUTURE_FILL   = "#ffffff";
let COLOR_FUTURE_STROKE = "#aaaaaa";
let COLOR_TEXT          = "#000000";
let COLOR_DECADE_LINE   = "#cc4444";

let STROKE_WIDTH_PAST   = 1;
let STROKE_WIDTH_FUTURE = 1;
let FILLSTYLE_PAST      = "solid";
let FILLSTYLE_FUTURE    = "solid";

const ROUGHNESS   = 0;
const FONT_FAMILY = 3;

const FONT_SIZE_LABEL    = 9;
const FONT_SIZE_HEADER   = 8;
const FONT_SIZE_TITLE    = 18;
const FONT_SIZE_SUBTITLE = 9;

// ─────────────────────────────────────────
//  Ask for birth year
// ─────────────────────────────────────────
const currentYear = new Date().getFullYear();
let birthYearStr = await utils.inputPrompt(
  "Enter your birth year",
  String(currentYear - 30),
  String(currentYear - 30)
);
const birthYear = parseInt(birthYearStr, 10);
if (isNaN(birthYear) || birthYear < 1900 || birthYear > currentYear) {
  new Notice("Please enter a valid birth year (1900–" + currentYear + ").");
  return;
}

// ─────────────────────────────────────────
//  Ask for life expectancy
// ─────────────────────────────────────────
let lifeExpStr = await utils.inputPrompt(
  "Life expectancy in years? (Central Europe avg: ~78)",
  "78",
  "78"
);
const YEARS = parseInt(lifeExpStr, 10);
if (isNaN(YEARS) || YEARS < 1 || YEARS > 130) {
  new Notice("Please enter a valid life expectancy (1–130).");
  return;
}

// ─────────────────────────────────────────
//  Total weeks derived from life expectancy
// ─────────────────────────────────────────
const TOTAL_WEEKS = YEARS * WEEKS_PER_YEAR;

// ─────────────────────────────────────────
//  Shape picker
// ─────────────────────────────────────────
const shapeLabels = ["⬤  Circle", "■  Square", "◆  Diamond"]; 
const shapeValues = ["ellipse",   "rectangle", "diamond"  ];

const chosenShape = await utils.suggester(
  shapeLabels,
  shapeValues,
  "Choose week shape"
);
if (!chosenShape) {
  new Notice("No shape selected — cancelled.");
  return;
}

// ─────────────────────────────────────────
//  Corner style picker — only for rectangles
// ─────────────────────────────────────────
let useRoundedCorners = false;
if (chosenShape === "rectangle") {
  const cornerChoice = await utils.suggester(
    ["■  Sharp corners", "▣  Rounded corners"],
    ["sharp",            "round"             ],
    "Choose corner style"
  );
  if (!cornerChoice) {
    new Notice("No corner style selected — cancelled.");
    return;
  }
  useRoundedCorners = cornerChoice === "round";
}

const CORNER_RADIUS = useRoundedCorners ? Math.round(SHAPE_SIZE * 0.35) : 0;

// ─────────────────────────────────────────
//  Apply styles from pre-selected elements
// ─────────────────────────────────────────
const sel = ea.getViewSelectedElements().filter(e => e.type === chosenShape);
if (sel.length >= 1) {
  COLOR_PAST_FILL     = sel[0].backgroundColor;
  COLOR_PAST_STROKE   = sel[0].strokeColor;
  STROKE_WIDTH_PAST   = sel[0].strokeWidth;
  FILLSTYLE_PAST      = sel[0].fillStyle;
}
if (sel.length >= 2) {
  COLOR_FUTURE_FILL   = sel[1].backgroundColor;
  COLOR_FUTURE_STROKE = sel[1].strokeColor;
  STROKE_WIDTH_FUTURE = sel[1].strokeWidth;
  FILLSTYLE_FUTURE    = sel[1].fillStyle;
}

// ─────────────────────────────────────────
//  Helper: draw one week shape
// ─────────────────────────────────────────
function addWeekShape(x, y) {
  if (chosenShape === "ellipse") {
    ea.addEllipse(x, y, SHAPE_SIZE, SHAPE_SIZE);
  } else if (chosenShape === "diamond") {
    ea.addDiamond(x, y, SHAPE_SIZE, SHAPE_SIZE);
  } else {
    const id = ea.addRect(x, y, SHAPE_SIZE, SHAPE_SIZE);
    if (useRoundedCorners && CORNER_RADIUS > 0) {
      ea.elementsDict[id].roundness = { type: 3, value: CORNER_RADIUS };
    }
  }
}

// ─────────────────────────────────────────
//  Calculate weeks lived
// ─────────────────────────────────────────
const today      = new Date();
const birthDate  = new Date(birthYear, 0, 1);
const msPerWeek  = 7 * 24 * 60 * 60 * 1000;
const weeksLived = Math.min(
  Math.floor((today - birthDate) / msPerWeek),
  TOTAL_WEEKS
);
const weeksLeft = Math.max(0, TOTAL_WEEKS - weeksLived);

// ─────────────────────────────────────────
//  Timestamp
// ─────────────────────────────────────────
const pad = n => String(n).padStart(2, "0");
const timestamp = today.getFullYear() + "-"
  + pad(today.getMonth() + 1) + "-"
  + pad(today.getDate()) + " "
  + pad(today.getHours()) + ":"
  + pad(today.getMinutes());

// ─────────────────────────────────────────
//  Derived grid geometry
// ─────────────────────────────────────────
const gridX          = START_X + LABEL_WIDTH;
const totalGridWidth = WEEKS_PER_YEAR * CELL_SIZE;

const gridRight = gridX + totalGridWidth;

const titleLineH  = FONT_SIZE_TITLE + 8;
const subtitleY   = START_Y + titleLineH;
const gridY       = subtitleY + FONT_SIZE_SUBTITLE + 8 + HEADER_H;

// ─────────────────────────────────────────
//  Approximate char width for right-alignment
//  Cascadia (font 3) at given size: ~0.55× ratio
// ─────────────────────────────────────────
const CHAR_RATIO = 0.55;
function textWidth(str, fontSize) {
  return str.length * fontSize * CHAR_RATIO;
}

// ─────────────────────────────────────────
//  Line 1 — title  (centred over the grid)
// ─────────────────────────────────────────
ea.style.fontSize    = FONT_SIZE_TITLE;
ea.style.strokeColor = COLOR_TEXT;
ea.style.fontFamily  = FONT_FAMILY;

const titleStr = "Your Life in Weeks";
const titleX   = gridX + (totalGridWidth - textWidth(titleStr, FONT_SIZE_TITLE)) / 2;
ea.addText(titleX, START_Y, titleStr);

// ─────────────────────────────────────────
//  Line 2 — three anchored subtitle segments
//
//  LEFT   → "born YYYY · life exp. N yrs"     starts at gridX   (= week 1 left edge)
//  CENTRE → "TOTAL wks · LIVED lived · LEFT remaining"  centred over grid
//  RIGHT  → "generated YYYY-MM-DD HH:MM"  right-edge at gridRight (= week 52 right edge)
// ─────────────────────────────────────────
ea.style.fontSize    = FONT_SIZE_SUBTITLE;
ea.style.strokeColor = "#555555";
ea.style.fontFamily  = FONT_FAMILY;

// LEFT segment
const leftStr = "born " + birthYear + "  ·  life exp. " + YEARS + " yrs";
ea.addText(gridX, subtitleY, leftStr);

// CENTRE segment
const centreStr = TOTAL_WEEKS + " weeks total  ·  "
  + Math.max(0, weeksLived) + " lived  ·  "
  + weeksLeft + " remaining";
const centreX = gridX + (totalGridWidth - textWidth(centreStr, FONT_SIZE_SUBTITLE)) / 2;
ea.addText(centreX, subtitleY, centreStr);

// RIGHT segment — right-aligned to week 52 edge
const rightStr  = "generated " + timestamp;
const rightStrX = gridRight - textWidth(rightStr, FONT_SIZE_SUBTITLE);
ea.addText(rightStrX, subtitleY, rightStr);

// ─────────────────────────────────────────
//  Week-number header (every 5 weeks)
// ─────────────────────────────────────────
ea.style.fontSize    = FONT_SIZE_HEADER;
ea.style.strokeColor = "#888888";

for (let w = 4; w < WEEKS_PER_YEAR; w += 5) {
  const hx = gridX + w * CELL_SIZE + SHAPE_SIZE / 2 - FONT_SIZE_HEADER * 0.5;
  ea.addText(hx, gridY - HEADER_H + 4, String(w + 1));
}

// ─────────────────────────────────────────
//  Main grid
// ─────────────────────────────────────────
let lastIsPast = null;

for (let year = 0; year < YEARS; year++) {
  const rowY         = gridY + year * ROW_HEIGHT;
  const absWeekStart = year * WEEKS_PER_YEAR;
  const labelYear    = birthYear + year;
  const isDecade     = (year + 1) % 10 === 0;

  // ── Decade separator line (below the decade row) ──
  if (isDecade) {
    ea.style.strokeColor = COLOR_DECADE_LINE;
    ea.style.strokeWidth = 1;
    ea.style.roughness   = 0;
    ea.style.fillStyle   = "solid";
    ea.addLine([
      [gridX,      rowY + SHAPE_SIZE + ROW_GAP / 2],
      [gridRight,  rowY + SHAPE_SIZE + ROW_GAP / 2]
    ]);
    lastIsPast = null;
  }

  // ── Year label ──
  ea.style.fontSize    = FONT_SIZE_LABEL;
  ea.style.strokeColor = isDecade ? COLOR_DECADE_LINE : COLOR_TEXT;
  ea.style.fontFamily  = FONT_FAMILY;
  ea.addText(
    START_X,
    rowY + (SHAPE_SIZE - FONT_SIZE_LABEL) / 2,
    String(year + 1).padStart(2, " ") + " · " + String(labelYear)
  );
  lastIsPast = null;

  // ── 52 week shapes ──
  for (let week = 0; week < WEEKS_PER_YEAR; week++) {
    const absWeek = absWeekStart + week;
    if (absWeek >= TOTAL_WEEKS) break;

    const isPast = absWeek < weeksLived;

    if (isPast !== lastIsPast) {
      ea.style.backgroundColor = isPast ? COLOR_PAST_FILL   : COLOR_FUTURE_FILL;
      ea.style.strokeColor     = isPast ? COLOR_PAST_STROKE : COLOR_FUTURE_STROKE;
      ea.style.strokeWidth     = isPast ? STROKE_WIDTH_PAST : STROKE_WIDTH_FUTURE;
      ea.style.fillStyle       = isPast ? FILLSTYLE_PAST    : FILLSTYLE_FUTURE;
      ea.style.roughness       = ROUGHNESS;
      lastIsPast = isPast;
    }

    addWeekShape(gridX + week * CELL_SIZE, rowY);
  }
}

// ─────────────────────────────────────────
//  Single commit to the view
// ─────────────────────────────────────────
await ea.addElementsToView(false, false, true);
new Notice(
  "Life in Weeks ready!  "
  + Math.max(0, weeksLived) + " lived · " + weeksLeft + " remaining."
);