# Clock Button Border-Radius + MISS Button No-Flash Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Give the clock START/STOP button the same 6px border-radius as the center-column scoring buttons, and stop the MISS button from flashing blue on click (since it already turns red while armed).

**Architecture:** Two small, independent CSS/JS tweaks on top of the existing click-flash feature (`app.js`, `styles.css`). No new state, no new files.

**Tech Stack:** Vanilla JS (`app.js`), plain CSS (`styles.css`), no build step, no new dependencies.

## Global Constraints

- No other button's border-radius changes — only `#clk-toggle` gains one.
- No other grid button's flash behavior changes — only `#btn-miss` stops flashing. 2PT, 3PT, FT, OREB, DREB, STL, BLK, AST, TOVR, FOUL, and UNDO must continue to flash blue exactly as before.
- MISS must still turn red while armed (`#btn-miss.armed { background:#dc2626; color:#fff; }` in `styles.css` is untouched).

---

### Task 1: Clock button radius + MISS no-flash

**Files:**
- Modify: `styles.css` (add a rule for `#clk-toggle`)
- Modify: `app.js:1017` (`renderControls`'s MISS button markup), `app.js:1071` (MISS click handler in `wireGame()`)

**Interfaces:** N/A — this task touches only markup/CSS for two specific buttons; nothing here is consumed by other tasks (this is the only task in the plan).

- [ ] **Step 1: Add the clock button border-radius rule**

In `styles.css`, find these two adjacent lines:

```css
#clk-toggle.start { background:#16a34a; color:#fff; border:none; }
#clk-toggle.stop { background:#dc2626; color:#fff; border:none; }
```

Add a new rule immediately above them:

```css
#clk-toggle { border-radius:6px; }
#clk-toggle.start { background:#16a34a; color:#fff; border:none; }
#clk-toggle.stop { background:#dc2626; color:#fff; border:none; }
```

Do not modify the `.start`/`.stop` rules themselves — the new `#clk-toggle` rule applies regardless of which state class is present, since neither existing rule sets `border-radius`.

- [ ] **Step 2: Stop the MISS button from queuing a flash**

In `app.js`, find this line inside `wireGame()`:

```js
  $('btn-miss') && ($('btn-miss').onclick = () => { flashKey = 'btn-miss'; missArm = !missArm; render(); });
```

Replace it with:

```js
  $('btn-miss') && ($('btn-miss').onclick = () => { missArm = !missArm; render(); });
```

(This removes only the `flashKey = 'btn-miss';` assignment — MISS still toggles `missArm` and re-renders exactly as before.)

- [ ] **Step 3: Stop rendering a flash class on the MISS button**

In `app.js`, inside `renderControls`, find this line:

```js
      <button id="btn-miss" class="${missArm?'armed':''}${flashClass('btn-miss')}">MISS</button>
```

Replace it with:

```js
      <button id="btn-miss" class="${missArm?'armed':''}">MISS</button>
```

Do not remove or modify the `flashClass` helper function itself, or any other button's use of it — only this one button's markup changes.

- [ ] **Step 4: Commit**

```bash
git add app.js styles.css
git commit -m "Match clock button radius to scoring buttons; stop MISS button from flashing"
```

- [ ] **Step 5: Manual verification**

There is no automated test for this (pure CSS/markup change, no logic to unit-test — matches the precedent set by the click-flash and linting changes). Since this sandbox has no browser available, ask the user to verify in the actual app:

1. Start a game — confirm the START/STOP button's corners are now rounded to match the 2PT/3PT/FT buttons (6px radius), in both the "start" (green) and "stop" (red) states.
2. Click MISS — confirm it still turns red (armed state) and does **not** flash blue.
3. Click MISS again to disarm, then click any other grid button (e.g. 2PT, with a player selected) — confirm that button still flashes blue as before.
