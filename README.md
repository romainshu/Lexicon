# Lexicon — Word Game Engine

A single-file browser word game with Wordle, Quordle, Hexordle, and a dictionary explorer — no install, no server, no dependencies.

---

## Table of Contents

- [Overview](#overview)
- [Quick Start](#quick-start)
- [Game Modes](#game-modes)
  - [Wordle](#wordle)
  - [Quordle](#quordle)
  - [Hexordle](#hexordle)
  - [Explorer](#explorer)
- [Controls & Settings](#controls--settings)
  - [Top Bar](#top-bar)
  - [Keyboard Input](#keyboard-input)
- [Difficulty System](#difficulty-system)
- [Word Validation](#word-validation)
- [Timer](#timer)
- [Reveal Answers](#reveal-answers)
- [Game Protection](#game-protection)
- [Datamuse API Integration](#datamuse-api-integration)
- [Word List & Vocabulary](#word-list--vocabulary)
- [Offline Support](#offline-support)
- [Dictionary Links](#dictionary-links)
- [UI & Animations](#ui--animations)
- [Code Architecture](#code-architecture)
- [Technical Reference](#technical-reference)
- [Known Limitations](#known-limitations)

---

## Overview

**Lexicon** is a self-contained HTML file that runs entirely in the browser. It requires no build tools, no frameworks, no backend, and no API keys. Open it by double-clicking the file in Chrome (or any modern browser) and play immediately.

It combines three word-guessing game modes of increasing difficulty — Wordle (1 board), Quordle (4 boards), and Hexordle (6 boards) — with a live dictionary explorer powered by the [Datamuse API](https://www.datamuse.com/api/). All word lists and validation logic work offline via embedded fallback data.

---

## Quick Start

1. Save `word_game_engine.html` anywhere on your computer (rename it if you like).
2. Double-click it — Chrome opens it as a local file.
3. A 5-letter Wordle game starts automatically.
4. Type a word using your keyboard or the on-screen keyboard, then press **Enter**.

No internet connection is required to play, though an active connection improves word list variety and enables real-time word validation.

---

## Game Modes

Switch between modes instantly using the buttons in the top bar. If you have a game in progress, a confirmation dialog will appear before switching.

### Wordle

The classic single-board word guessing game.

- One hidden word per game.
- Type a guess and press **Enter** to submit.
- Each tile changes color to show how close your guess was:
  - 🟩 **Green** — correct letter, correct position
  - 🟨 **Yellow** — correct letter, wrong position
  - ⬜ **Gray** — letter not in the word
- Win by guessing the word within the allowed number of attempts.
- If you run out of attempts, the answer is revealed in the status bar with a link to look it up.

### Quordle

Four simultaneous Wordle boards. Every guess you submit is applied to all four boards at once.

- Each board has its own unique hidden word — no two boards share the same answer in a single game.
- The color feedback is independent per board.
- A board locks and shows a win (✓) or lose (✗) overlay when it resolves.
- You win only when all four boards are solved within the attempt limit.
- Solved and failed boards are overlaid with the answer; you can click failed-board answers to open them in the dictionary.

### Hexordle

Six simultaneous boards — the hardest mode.

- Same rules as Quordle, but with six independent hidden words.
- All six must be solved with the same shared pool of guesses.
- Requires careful cross-board letter deduction to succeed.

### Explorer

A free-form dictionary and word search tool, separate from the game modes.

- Enter any word or pattern in the search box.
- Choose a search type from the dropdown:

  | Type | Datamuse Parameter | What it finds |
  |---|---|---|
  | Pattern / Spelling | `sp=` | Words matching a pattern, e.g. `c??ld` → could, child |
  | Meaning Related | `ml=` | Words with similar meaning, e.g. `happy` → joyful, glad |
  | Sounds Like | `sl=` | Words that sound like the input, e.g. `knight` → night |
  | Rhymes With | `rel_rhy=` | Perfect rhymes, e.g. `cat` → bat, hat, mat |

- Results appear as clickable chips. Clicking any chip opens the word in Merriam-Webster in a new tab.
- The explorer does not affect game state and has no attempt limit.

---

## Controls & Settings

### Top Bar

All game controls are in the persistent top bar.

| Control | Options | Description |
|---|---|---|
| **Wordle / Quordle / Hexordle / Explorer** | — | Switches the active mode. Prompts for confirmation if a game is in progress. |
| **Length** | 4 · 5 · 6 · 7 · 8 | Sets the number of letters in the hidden word(s). Prompts for confirmation if a game is in progress. |
| **Tries** | 4 – 10 | Sets the maximum number of guesses allowed. Prompts for confirmation if a game is in progress. |
| **Difficulty** | Easy · Normal · Hard | Controls the vocabulary pool (see [Difficulty System](#difficulty-system)). Prompts for confirmation if a game is in progress. |
| **Timer** | Off · 2 min · 3 min · 5 min · 10 min · Custom… | Sets an optional countdown timer for the game (see [Timer](#timer)). Prompts for confirmation if a game is in progress. |
| **New Game** | — | Starts a fresh game in the current mode with the current settings. Prompts for confirmation if a game is in progress. |

### Keyboard Input

Both physical keyboard and on-screen keyboard are fully supported.

| Input | Action |
|---|---|
| Any letter key (A–Z) | Types that letter into the current guess |
| `Enter` | Submits the current guess |
| `Backspace` | Deletes the last typed letter |
| On-screen letter key | Same as physical key |
| On-screen `Enter` | Submits the guess |
| On-screen `⌫` | Deletes the last letter |

The on-screen keyboard updates its key colors as you play, reflecting the best-known state for each letter across all submitted guesses (green takes priority over yellow, yellow over gray).

Physical keyboard input is disabled while any modal dialog is open, and while tile flip animations are playing (to prevent accidentally typing into the wrong row).

---

## Difficulty System

When connected to the internet, Lexicon fetches a pool of words from Datamuse and sorts them by **corpus frequency** — how often the word appears in real English text. The difficulty setting then slices that sorted pool:

| Difficulty | Pool slice | Typical vocabulary |
|---|---|---|
| **Easy** | Top 40% most frequent | Common everyday words: *house, words, plant* |
| **Normal** | Middle 80% (10th–90th percentile) | General vocabulary: *fable, crisp, anvil* |
| **Hard** | Bottom 40% least frequent | Rare, advanced, or archaic words: *culpa, brume, garnet* |

When offline, difficulty maps directly to hand-curated fallback word lists with the same three tiers, covering word lengths 4 through 8.

---

## Word Validation

Every guess is validated before it is accepted. Lexicon uses a two-stage approach:

**Stage 1 — Local cache check (instant)**
If the word appears in any of the embedded fallback word lists (which contain thousands of real English words across all lengths and difficulty tiers), it is accepted immediately without an API call.

Previously validated words are also cached in memory for the session, so repeated guesses of the same word skip the network entirely.

**Stage 2 — Datamuse API check (online only)**
If the word is not in the local lists, Lexicon queries:
```
https://api.datamuse.com/words?sp={word}&md=df&max=20
```
The response is checked for an **exact match** on the word, then:

- **Has `defs` entries** → accepted (the word has a dictionary definition — it is a base/lemma form).
- **No `defs` but `score ≥ 1000`** → accepted (inflected form — plural, past tense, present participle, comparative, superlative, etc.). Datamuse scores real inflected forms highly even when it has no standalone definition for them.
- **Exact match missing, or score < 1000** → rejected with a shake animation and "Not a valid word!" toast.

**What this means in practice:**
- ✅ `walks`, `walked`, `walking` — accepted (inflections of *walk*)
- ✅ `boxes`, `boxed` — accepted (inflections of *box*)
- ✅ `faster`, `fastest` — accepted (comparatives of *fast*)
- ✅ `tried`, `tries` — accepted (inflections of *try*)
- ❌ `asdfg`, `zzzzz`, `xkqjw` — rejected (no match or score too low)

**Offline behavior:** If Datamuse is unreachable, validation falls back to permissive mode (all alphabetic guesses accepted) so the game remains playable without internet.

---

## Timer

An optional countdown timer can be enabled from the **Timer** dropdown in the top bar.

**Available presets:** Off, 2 min, 3 min, 5 min, 10 min, Custom…

Selecting **Custom…** opens a dialog where you can enter any duration from 1 to 99 minutes.

**How it works:**

- The timer is displayed in the status bar as `⏱ MM:SS` and counts down from the moment the game starts.
- At **30 seconds remaining**, the display turns yellow as a warning.
- At **10 seconds remaining**, the display turns red and pulses.
- When the timer reaches zero, the game ends automatically: all boards are revealed, "Time's up!" is shown, and no further guesses are accepted.

**Changing the timer during a game** triggers the same confirmation modal as any other setting change — your current game won't be silently reset.

**Timer and game end:** The timer stops automatically when you win, lose, or manually reveal the answers. It resets cleanly when a new game starts.

---

## Reveal Answers

A **Reveal Answers** button appears in the status bar at the start of every game (in all three game modes). It is hidden once the game ends.

Clicking it opens a confirmation dialog:

> 🔍 **Reveal answers?**
> This will immediately end the game and show you all the answers. Are you sure?

- **Cancel** — closes the dialog, continues the game.
- **Yes, Reveal** — immediately ends the game, stops the timer, and reveals all answer words:
  - In Wordle: shows a dictionary link for the solution above the keyboard.
  - In Quordle / Hexordle: shows a lose overlay on every unsolved board with its answer word, which is clickable to look it up in the dictionary.

This feature is useful for checking a word you were stuck on, studying vocabulary, or ending a session early without having to guess wrong until attempts run out.

---

## Game Protection

Every action that would reset or discard an in-progress game is guarded by a confirmation modal. The modal fires only after at least one guess has been successfully submitted — a fresh, untouched board never prompts.

**Actions that trigger the confirmation:**

| Action | Modal message |
|---|---|
| Clicking a different mode button | "You have a game in progress. Switching will end it and reveal the answer(s). Continue?" |
| Clicking **New Game** | "You have a game in progress. Starting a new game will end it and reveal the answer(s). Continue?" |
| Changing Length, Tries, Difficulty, or Timer | "Changing settings will reset your current game and reveal the answer(s). Continue?" |
| Clicking **Reveal Answers** | "This will immediately end the game and show you all the answers. Are you sure?" |

**Modal options:**

- **Keep Playing** — dismisses the modal, restores any dropdown to its previous value, and returns to the game unchanged.
- **Give Up & Leave** / **Yes, Reveal** — executes the action.

The modal system is generic: a single `openModal(title, text, confirmLabel, callback)` function powers all four confirmation scenarios.

---

## Datamuse API Integration

Lexicon uses the [Datamuse API](https://www.datamuse.com/api/) for three distinct purposes, all via plain `fetch()` calls with no API key required.

| Purpose | Endpoint | Used for |
|---|---|---|
| Word pool generation | `GET /words?sp=?????&max=500&md=f` | Building the pool of potential hidden words at game start |
| Guess validation | `GET /words?sp={word}&md=df&max=20` | Checking whether a submitted guess is a real English word |
| Explorer search | `GET /words?sp=` / `?ml=` / `?sl=` / `?rel_rhy=` | Powering the four search modes in Explorer |

All API calls are wrapped in try/catch with graceful fallbacks. The game never blocks or crashes if Datamuse is slow or unavailable.

---

## Word List & Vocabulary

**Online (Datamuse):** When connected to the internet, Lexicon fetches up to **500 words per query** from Datamuse, frequency-ranked from the full English corpus. This is the primary and far richer source — the pool changes slightly each game as Datamuse returns varied results.

**Offline (embedded fallback):** The file contains hand-curated fallback lists for lengths 4–8 across all three difficulty tiers. These are a limited sample — roughly 100–300 words per length/difficulty combination — and exist purely as a safety net when Datamuse is unreachable.

**Summary:** Online play gives you a vastly wider and more varied vocabulary. The embedded lists are not exhaustive and are not intended to be — Datamuse is the real dictionary engine.

---

## Offline Support

Lexicon is fully playable without an internet connection. Every critical feature has an offline fallback:

| Feature | Online | Offline |
|---|---|---|
| Word pool | Fetched from Datamuse (up to 500 words, frequency-ranked) | Embedded lists: ~100–300 words per length/difficulty combination |
| Word validation | Datamuse API + local cache | Permissive (all alphabetic guesses accepted) |
| Explorer search | Live Datamuse results | Shows "Search failed" error message |
| Dictionary links | Opens Merriam-Webster in new tab | Link still present; browser shows no-connection error |

The embedded fallback lists cover word lengths 4, 5, 6, 7, and 8, each split into easy, normal, and hard tiers, totalling several thousand unique words baked directly into the HTML file.

---

## Dictionary Links

Any time a game ends — win or lose — a **"Look up in dictionary"** link appears above the keyboard for the solution word. Clicking it opens:

```
https://www.merriam-webster.com/dictionary/{word}
```

In Quordle and Hexordle, boards that were **not solved** show the answer word in the board overlay as an underlined link. Clicking it opens the same Merriam-Webster URL for that word.

In Explorer mode, every result chip opens its word in Merriam-Webster when clicked.

---

## UI & Animations

**Tile flip animation**
When a guess is submitted, each tile flips in sequence (280ms stagger). The color state is applied at the midpoint of the flip (when the tile is edge-on and invisible), so the color and letter both appear cleanly on the way back up — avoiding the visual glitch where letters appeared to vanish mid-animation.

**Pop animation**
Each tile briefly scales up (1.12×) when a letter is typed, providing tactile feedback.

**Shake animation**
The current row shakes horizontally if a guess is submitted with too few letters, or if the word fails validation.

**Input locking**
All keyboard and on-screen input is locked while tile flip animations are running. This prevents letters from being accidentally typed into the next row before the previous guess finishes revealing.

**Board overlays (Quordle / Hexordle)**
Boards that have resolved show a semi-transparent overlay with ✓ (green, win) or ✗ (red, lose) and the answer word. The overlay appears after the last tile of that board finishes flipping.

**On-screen keyboard coloring**
Key colors follow a strict priority: green (correct) > yellow (present) > gray (absent). Once a key turns green it never downgrades.

**Timer display**
The `⏱ MM:SS` countdown in the status bar transitions through three states — normal (dim text), warning (yellow, ≤30s), danger (red + pulsing, ≤10s) — to give clear visual urgency without being intrusive during normal play.

**Dark theme**
The entire UI uses a consistent dark color palette defined with CSS custom properties in the `:root` block. Retheme the entire app by editing those variables.

---

## Code Architecture

The entire application is structured in one HTML file with no external dependencies. The JavaScript is organized into clearly labeled sections:

```
FALLBACK WORD LISTS       — embedded word pools for offline play, all lengths & difficulties
STATE                     — all mutable game variables in one place
UTILITY                   — rand(), uniqueWords(), showToast(), setStatus(), updateStatusBar()
WORD VALIDATION           — isRealWord() with local cache + Datamuse two-stage check
WORD FETCHING             — fetchWords(), applyDifficultyFilter(), applyDifficultyFallback()
MODE SWITCHING            — requestSwitchMode(), requestNewGame(), openModal(), closeModal(),
                            openLeaveModal(), modalConfirmAction(), switchMode()
SETTINGS CHANGE           — onSettingChange() with mid-game guard
INIT GAME                 — initGame() — async entry point, wires timer + reveal button
BOARD GENERATION          — generateBoard() (Wordle), generateMultiBoard() (Quordle/Hexordle)
KEYBOARD                  — buildKeyboard(), makeKey(), updateKeyboard()
INPUT HANDLING            — typeLetter(), deleteLetter(), submitGuess() (async)
GUESS PROCESSING          — processWordleGuess(), processMultiBoardGuess(), checkMultiBoardEnd(),
                            showBoardOverlay()
CHECK GUESS               — checkGuess() — two-pass correct/present/absent scoring
REVEAL ROW                — revealRow() — staggered flip with midpoint color injection
SHAKE ROW                 — shakeCurrentRow()
DICT LINK                 — addRevealLink(), openDict()
KEYBOARD EVENTS           — physical keyboard listener
TIMER                     — startTimer(), stopTimer(), renderTimer()
CUSTOM TIMER MODAL        — showCustomTimerModal(), cancelCustomTimer(), applyCustomTimer()
REVEAL ANSWERS            — updateRevealButton(), requestRevealAnswers(), revealAllAnswers()
EXPLORER MODE             — explorerSearch()
BOOT                      — initGame() call on page load
```

**Key design decisions:**

- `inputLocked` is a boolean gate set to `true` on guess submission and released only inside the reveal callback, ensuring animations always complete before new input is accepted.
- `gameStarted` is only set to `true` after the first successful (validated) guess, so the leave-game modal does not fire on a pristine board.
- The modal system is fully generic: `openModal(title, text, confirmLabel, fn)` stores the callback in `modalConfirmFn`; `modalConfirmAction()` captures and clears it before calling, preventing any race where a stale reference could trigger the wrong action.
- `validWordCache` (a `Set`) persists for the session, preventing redundant Datamuse calls for words already validated this session.
- Multi-board `revealRow` callbacks use a shared `pendingCallbacks` counter to detect when all active boards have finished animating before incrementing `currentRow` and evaluating end conditions.
- `timerSeconds` holds the configured duration; `timerLeft` holds the live countdown. `stopTimer()` always clears the interval safely and can be called multiple times without side effects.
- Settings changes snapshot new values into local variables before opening the modal, so if the user cancels, the dropdowns can be read back without needing to track "previous values" separately.

---

## Technical Reference

| Property | Value |
|---|---|
| File type | Single `.html` file |
| File size | ~70 KB (uncompressed) |
| Dependencies | None |
| Frameworks | None |
| API | Datamuse (no key required) |
| Browser support | Chrome, Firefox, Safari, Edge (any modern browser) |
| Minimum word length | 4 letters |
| Maximum word length | 8 letters |
| Minimum attempts | 4 |
| Maximum attempts | 10 |
| Boards in Quordle | 4 |
| Boards in Hexordle | 6 |
| Tile flip duration | 500ms |
| Tile stagger delay | 280ms per tile |
| Validation score threshold | 1000 (Datamuse score units) |
| Timer presets | Off, 2 min, 3 min, 5 min, 10 min, Custom (1–99 min) |
| Timer warning threshold | 30 seconds remaining |
| Timer danger threshold | 10 seconds remaining |

---

## Known Limitations

- **No persistent score tracking.** Win/loss history is not saved between sessions. Closing and reopening the file starts fresh.
- **No share/copy results feature.** The colored emoji grid common in Wordle clones is not implemented.
- **Single language.** Only English words are supported via Datamuse.
- **Offline validation is permissive.** Without internet, any alphabetic string of the correct length is accepted as a guess. This is intentional to keep the game playable without a connection.
- **Datamuse rate limits.** The Datamuse API is free and requires no authentication, but it may occasionally be slow or return empty results under high load. Fallback lists ensure the game always starts.
- **No hard mode enforcement.** Hard mode only affects the vocabulary pool; there is no rule requiring you to reuse revealed letters in subsequent guesses (unlike Wordle's official hard mode option).
- **Local file access.** When opened as a `file://` URL, some browsers may block `fetch()` calls to external APIs due to CORS/mixed-content restrictions. Chrome handles this correctly for `file://` origins. If validation or word fetching fails unexpectedly, try serving the file via a simple local HTTP server: `python3 -m http.server`.
- **Timer does not pause.** There is currently no pause button. Switching to Explorer mode while a timer is running will continue counting down.
