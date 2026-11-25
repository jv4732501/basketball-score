# Zero-Install Linting Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Give the project linting-equivalent feedback (catch typos, undefined-variable references, dead code, unused locals) in `app.js` and `logic.test.js` using VS Code's built-in TypeScript language service, with zero npm installs.

**Architecture:** A `jsconfig.json` at the repo root turns on `checkJs` for the two root-level `.js` files. A hand-written `types.d.ts` declares the two CommonJS globals (`require`, `module`) those files use, so TypeScript doesn't need `@types/node` (an npm package) to understand them. `README.md` gets a short "Linting" section documenting this is automatic and zero-setup.

**Tech Stack:** VS Code's bundled TypeScript language service (no `package.json`, no `node_modules`, no npm). Plain JSON/TypeScript-declaration/Markdown files only.

## Global Constraints

- No new dependencies, no build step, no `npm install` — matches the project's existing "no build, no dependencies" stance (`README.md`).
- `strict` mode must stay off in `jsconfig.json` — full strict type-checking against ~1200 lines of existing untyped JS would surface hundreds of pre-existing "implicit any" warnings unrelated to real bugs, per the design spec's rationale.
- Only `app.js` and `logic.test.js` are in scope (`include: ["*.js"]`) — nothing under `docs/` is linted.

---

### Task 1: Add jsconfig.json, types.d.ts, README section; fix any findings

**Files:**
- Create: `jsconfig.json`
- Create: `types.d.ts`
- Modify: `README.md` (insert a new "Linting" section between the existing "## Test" and "## Deploy" sections)
- Possibly modify: `app.js` and/or `logic.test.js` if the Problems panel surfaces real issues (see Step 4)

**Interfaces:**
- N/A — this task adds editor configuration and ambient type declarations, not runtime code. Nothing here is consumed by other tasks (this is the only task in the plan).

- [ ] **Step 1: Create `jsconfig.json`**

Create `jsconfig.json` at the repo root (same level as `app.js`) with exactly this content:

```json
{
  "compilerOptions": {
    "checkJs": true,
    "noEmit": true,
    "target": "ES2020",
    "lib": ["ES2020", "DOM"],
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "strict": false
  },
  "include": ["*.js"]
}
```

- [ ] **Step 2: Create `types.d.ts`**

Create `types.d.ts` at the repo root with exactly this content:

```ts
declare const require: (id: string) => any;
declare const module: { exports: any };
```

- [ ] **Step 3: Add the "Linting" section to `README.md`**

`README.md` currently reads (in full):

```markdown
# HoopScore (single-file basketball scorer)

Static app: `index.html` + `styles.css` + `app.js`. No build, no dependencies.

## Run locally
Open `index.html` in a browser, or serve the folder (e.g. `python -m http.server`).

## Test
`node --test`  (requires Node 17+; no `npm install`)

## Deploy
Commit the three files to a GitHub repo and enable GitHub Pages on the branch.

## Manual QA checklist
- [ ] First run: create a team, add opponent, tap tip → game starts, clock runs.
- [ ] Scoring (2/3/FT make+miss), make/miss auto-reset, detailed stats for my team.
- [ ] Opponent limited to points + fouls.
- [ ] Team fouls bonus (7) / double bonus (10) badges.
- [ ] Clock: start/stop, SET, lock-screen catch-up, auto-stop at 0:00, timeout stops clock.
- [ ] Possession toggle; undo; END HALF; +OT; END GAME.
- [ ] "My team is Away": my team renders right, still gets detailed controls + saved correctly.
- [ ] Summary: final score, period table, box scores, log; Print shows only summary; New Game resets.
- [ ] Reload mid-game → resume banner → Resume restores exact state; Discard clears it.
```

Insert a new `## Linting` section directly after the `## Test` section and its content, and before `## Deploy`, so the file reads:

```markdown
# HoopScore (single-file basketball scorer)

Static app: `index.html` + `styles.css` + `app.js`. No build, no dependencies.

## Run locally
Open `index.html` in a browser, or serve the folder (e.g. `python -m http.server`).

## Test
`node --test`  (requires Node 17+; no `npm install`)

## Linting
Type-checking runs automatically in VS Code via `jsconfig.json` — open the project and check the Problems panel, no install needed.
Optional one-off terminal check: `npx tsc --noEmit -p jsconfig.json` (downloads TypeScript temporarily; requires network, not a permanent install).

## Deploy
Commit the three files to a GitHub repo and enable GitHub Pages on the branch.

## Manual QA checklist
- [ ] First run: create a team, add opponent, tap tip → game starts, clock runs.
- [ ] Scoring (2/3/FT make+miss), make/miss auto-reset, detailed stats for my team.
- [ ] Opponent limited to points + fouls.
- [ ] Team fouls bonus (7) / double bonus (10) badges.
- [ ] Clock: start/stop, SET, lock-screen catch-up, auto-stop at 0:00, timeout stops clock.
- [ ] Possession toggle; undo; END HALF; +OT; END GAME.
- [ ] "My team is Away": my team renders right, still gets detailed controls + saved correctly.
- [ ] Summary: final score, period table, box scores, log; Print shows only summary; New Game resets.
- [ ] Reload mid-game → resume banner → Resume restores exact state; Discard clears it.
```

- [ ] **Step 4: Commit the config/docs addition**

```bash
git add jsconfig.json types.d.ts README.md
git commit -m "Add zero-install linting via jsconfig.json checkJs"
```

- [ ] **Step 5: Manual verification — ask the user to check the VS Code Problems panel**

There is no automated test for this (no test framework applies to editor tooling config, and this sandbox has no Node/npm/VS Code available to run a check directly). Ask the user to:

1. Open the `basketballscore` folder in VS Code (if not already open).
2. Open `app.js` and `logic.test.js`.
3. Open the Problems panel (View → Problems, or `Ctrl+Shift+M`).
4. Report back what appears.

**If the Problems panel is clean:** done, no further changes needed.

**If the Problems panel shows real issues** (e.g., a genuine typo, an undefined variable reference, an unused local that's actually dead code): fix each one directly in `app.js`/`logic.test.js`, following the existing code style, then:
- Run `node --test` if Node is available in your environment, to confirm the fix didn't break existing behavior (expected: all tests still pass, matching the pre-fix pass count).
- Ask the user to refresh the Problems panel and confirm it's now clean.
- Commit the fix separately from the Step 4 config commit, e.g.:

```bash
git add app.js
git commit -m "Fix issues surfaced by new checkJs linting"
```

**If the Problems panel shows only noise** (e.g., false positives from `checkJs` inferring an overly narrow type for something legitimately dynamic): do not silence it by weakening `jsconfig.json`'s settings without checking with the user first — report exactly what shows up and ask whether to adjust the config or suppress that one line with a targeted `// @ts-ignore` comment (which documents *why* inline). This is a case where the plan cannot predict the outcome ahead of time, so use judgment and escalate rather than guessing.
