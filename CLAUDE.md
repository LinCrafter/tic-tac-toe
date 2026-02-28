# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the project

No build step. Open `tictactoe.html` directly in a browser:
```bash
start tictactoe.html   # Windows
```

There are no dependencies, no package manager, and no dev server.

## Architecture

Everything lives in a single file — `tictactoe.html` — structured as:

1. **HTML** — static board (9 `.cell` divs with `data-i` indices 0–8), control toggles, score display
2. **CSS** (inline `<style>`) — dark theme, toggle-group button pattern, `.win` highlight class
3. **JavaScript** (inline `<script>`) — all game logic, no frameworks

### Key state variables
- `board` — flat 9-element array (`''` | `'X'` | `'O'`)
- `current` — whose turn it is (`'X'` | `'O'`)
- `gameOver` — boolean; guards cell clicks and AI dispatch
- `mode` — `'human'` (2-player) or `'ai'`
- `difficulty` — `'easy'` | `'medium'` | `'hard'`
- `humanSide` / `aiSide` — which symbol each player uses
- `score` — `{ X, O, tie }` object; persists across restarts but resets when mode/difficulty/side changes

### Key functions
| Function | Purpose |
|---|---|
| `init()` | Reset board state, clear UI, trigger AI if it goes first |
| `applyMove(index, player)` | Write move to board/DOM, check for winner, advance turn or trigger AI |
| `checkWinner(b)` | Check all 8 win lines; returns `{winner, line}` on win, `{winner: null, line: []}` on tie, `null` if game continues |
| `doAiMove()` | Dispatch to `easyMove()` or `bestMove()` based on difficulty |
| `bestMove()` | Iterates empty cells, calls `minimax()` to score each, picks highest |
| `minimax(b, depth, isMax, alpha, beta)` | Recursive alpha-beta pruned minimax; mutates `b` in-place and restores; scores relative to `aiSide`/`humanSide` |

### AI difficulty
- **Easy** — random empty cell
- **Medium** — 50% chance of best move, 50% random
- **Hard** — full minimax (unbeatable)

### UI patterns
- `WINS` is a module-level constant listing all 8 winning index triples
- Toggle groups use event delegation on the parent `<div>`, targeting `[data-mode]`, `[data-diff]`, or `[data-side]` attributes; any toggle change resets the score and calls `init()`
- Board is locked during AI thinking via the `.locked` CSS class; `lockBoard(bool)` toggles it on all cells; AI moves fire after a 350–400 ms `setTimeout`
- Score labels update dynamically ("You"/"AI" vs "X"/"O") via `updateScoreLabels()`
