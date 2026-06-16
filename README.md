# Lexicon — Word & Number Game Engine

A single-file browser word and number game with Wordle, Quordle, Hexordle, Numble, and a dictionary explorer — no install, no server, no dependencies.

---

## Table of Contents

- [Overview](#overview)
- [Quick Start](#quick-start)
- [Game Modes](#game-modes)
  - [Wordle](#wordle)
  - [Quordle](#quordle)
  - [Hexordle](#hexordle)
  - [Numble](#numble)
  - [Explorer](#explorer)
- [Controls & Settings](#controls--settings)
  - [Top Bar](#top-bar)
  - [Keyboard Input](#keyboard-input)
- [Difficulty System](#difficulty-system)
- [Word Validation](#word-validation)
- [Timer](#timer)
- [Reveal Answers](#reveal-answers)
- [Game Protection](#game-protection)
- [Dark Mode](#dark-mode)
- [Sound Effects](#sound-effects)
- [Numble Statistics](#numble-statistics)
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

It combines four game modes — Wordle (1 board), Quordle (4 boards), Hexordle (6 boards), and **Numble** (number guessing) — plus a live dictionary explorer powered by the [Datamuse API](https://www.datamuse.com/api/). All game logic works offline via embedded fallback data.

---

## Quick Start

1. Save `word_game_engine.html` anywhere on your computer.
2. Double-click it — Chrome opens it as a local file.
3. A 5-letter Wordle game starts automatically.
4. Type a word using your keyboard or the on-screen keyboard, then press **Enter**.

No internet connection is required to play, though an active connection improves word list variety and enables real-time word validation.

---

## Game Modes

Switch between modes instantly using the buttons in the top bar. If a game is in progress, a confirmation dialog appears before switching.

### Wordle

The classic single-board word guessing game.

- One hidden word per game.
- Type a guess and press **Enter** to submit.
- Each tile changes color to show how close your guess was:
  - 🟩 **Green** — correct letter, correct position
  - 🟨 **Yellow** — correct letter, wrong position
  - ⬜ **Gray** — letter not in the word
- Win by guessing the word within the allowed number of attempts.
- If you run out of attempts, the answer is revealed with a dictionary link.

### Quordle

Four simultaneous Wordle boards. Every guess applies to all four boards at once.

- Each board has its own unique hidden word — no two boards share the same answer.
- Color feedback is independent per board.
- A board locks with a ✓ (win) or ✗ (lose) overlay when it resolves.
- You win only when all four boards are solved within the attempt limit.

### Hexordle

Six simultaneous boards — the hardest mode.

- Same rules as Quordle, but six independent hidden words.
- All six must be solved with the same shared pool of guesses.

### Numble

A number-guessing game with full Wordle-style feedback.

**Rules:**
- The hidden answer is a random number with a configurable number of digits (4, 5, or 6).
- Digits may or may not repeat depending on the **Repeats** setting.
- You have a configurable number of attempts (4–8) to guess the number.
- After each guess, tiles reveal:
  - 🟩 **Green** — digit is correct and in the correct position
  - 🟨 **Yellow** — digit exists in the answer but is in the wrong position
  - ⬜ **Gray** — digit does not appear in the answer at all

**Example:**
```
Answer:  5 8 3 7
Guess:   5 7 3 8
Result:  ✅ 🟨 ✅ 🟨   (5=green, 7=yellow, 3=green, 8=yellow)
```

**Numble-specific features:**
- Virtual number keyboard (0–9) with Enter and ⌫ keys
- Physical keyboard input (digits, Enter, Backspace)
- Statistics tracking across sessions (games played, win %, streak, best streak, guess distribution)
- Unfinished games are saved to localStorage and restored on next visit
- 📊 Statistics button in the top bar
- Help modal explains rules based on current digit length and repeat setting

### Explorer

A free-form dictionary and word search tool, separate from the game modes.

- Enter any word or pattern in the search box.
- Choose a search type from the dropdown:

  | Type | What it finds |
  |---|---|
  | Pattern / Spelling | Words matching a pattern, e.g. `c??ld` → could, child |
  | Meaning Related | Words with similar meaning, e.g. `happy` → joyful, glad |
  | Sounds Like | Words that sound like the input, e.g. `knight` → night |
  | Rhymes With | Perfect rhymes, e.g. `cat` → bat, hat, mat |

- Results appear as clickable chips. Clicking any chip opens it in Merriam-Webster.

---

## Controls & Settings

### Top Bar

| Control | Options | Description |
|---|---|---|
| **Wordle / Quordle / Hexordle / Numble / Explorer** | — | Switches the active mode. Prompts for confirmation if a game is in progress. |
| **Length** *(word modes)* | 4–8 | Number of letters in the hidden word. Prompts if a game is in progress. |
| **Tries** *(word modes)* | 4–10 | Maximum word guesses allowed. Prompts if a game is in progress. |
| **Difficulty** *(word modes)* | Easy · Normal · Hard | Vocabulary pool tier. Prompts if a game is in progress. |
| **Digits** *(Numble)* | 4 · 5 · 6 | Number of digits in the hidden number. |
| **Tries** *(Numble)* | 4–8 | Maximum number guesses allowed. |
| **Repeats** *(Numble)* | Allowed · No Repeats | Whether digits may repeat in the hidden number. |
| **Timer** | Off · 2 min · 3 min · 5 min · 10 min · Custom… | Optional countdown timer. Shared across all modes. |
| **?** | — | Opens the Help modal with rules for the current mode. |
| **📊** *(Numble only)* | — | Opens the Numble Statistics panel. |
| **🌙 / ☀️** | — | Toggles between dark and light mode. |
| **New Game** | — | Starts a fresh game. Prompts for confirmation if a game is in progress. |

### Keyboard Input

**Word modes:**

| Input | Action |
|---|---|
| A–Z | Types that letter |
| Enter | Submits the current guess |
| Backspace | Deletes the last letter |

**Numble mode:**

| Input | Action |
|---|---|
| 0–9 | Types that digit |
| Enter | Submits the current guess |
| Backspace | Deletes the last digit |

All physical keyboard input is disabled while any modal dialog is open and while tile flip animations are playing.

---

## Difficulty System

When connected to the internet, Lexicon fetches a pool of words from Datamuse and sorts them by corpus frequency. The difficulty setting slices that pool:

| Difficulty | Pool slice | Typical vocabulary |
|---|---|---|
| **Easy** | Top 40% most frequent | Common everyday words |
| **Normal** | Middle 80% (10th–90th percentile) | General vocabulary |
| **Hard** | Bottom 40% least frequent | Rare or advanced words |

When offline, difficulty maps to hand-curated fallback lists covering lengths 4–8.

Difficulty does not apply to Numble — numbers are always randomly generated.

---

## Word Validation

Every word guess is validated before it is accepted. Lexicon uses a two-stage approach:

**Stage 1 — Local cache (instant)**
If the word appears in any embedded fallback list, it is accepted immediately. Words validated this session are cached in memory.

**Stage 2 — Datamuse API**
If not found locally, Lexicon queries `?sp={word}&md=df&max=20` and checks:
- Has `defs` entries → accepted (base dictionary word)
- No `defs` but `score ≥ 1000` → accepted (inflected form: plurals, past tenses, comparatives, etc.)
- No exact match or score too low → rejected with shake animation and "Not a valid word!" toast

**Examples:**
- ✅ `walks`, `walked`, `walking`, `boxes`, `faster`, `tried` — accepted (inflected forms)
- ❌ `asdfg`, `zzzzz`, `xkqjw` — rejected

Numble does not require validation — any complete digit sequence is accepted.

---

## Timer

An optional countdown timer is available from the **Timer** dropdown, shared across all game modes including Numble.

**Presets:** Off, 2 min, 3 min, 5 min, 10 min, Custom (1–99 minutes)

**Behavior:**
- Timer displays as `⏱ MM:SS` in the status bar
- Turns **yellow** at ≤30 seconds remaining
- Turns **red and pulsing** at ≤10 seconds
- When it reaches zero, the game ends: "Time's up!" is shown and answers are revealed

**After time expires, the Reveal Answers button remains visible.** Clicking it shows all answers immediately, so you can still see what you missed even after losing to the clock.

Changing the timer during an active game triggers the same confirmation modal as any other setting change.

---

## Reveal Answers

A **Reveal Answers** button appears in the status bar at the start of every game and stays visible even after the timer expires, so you can always choose to see the answers.

- **During an active game:** clicking opens a confirmation dialog. Confirm to immediately end the game and reveal all answers.
- **After the timer expires:** the button remains active with no confirmation needed — click once to reveal.
- **After the game ends normally (win or lose):** the button is hidden.

Works across all four game modes.

---

## Game Protection

Every action that would reset an in-progress game is guarded by a confirmation modal. The modal only fires after at least one guess has been submitted.

| Action | Modal message |
|---|---|
| Switching to a different mode | "You have a game in progress. This will end it and reveal the answer(s). Continue?" |
| Clicking New Game | "You have a game in progress. Starting a new game will end it. Continue?" |
| Changing Length, Tries, Difficulty, Digits, Repeats, or Timer | "Changing settings will reset your current game. Continue?" |
| Clicking Reveal Answers (active game) | "This will immediately end the game and show all answers. Are you sure?" |

**Modal options:**
- **Cancel** — dismisses, returns to game unchanged
- **Give Up & Leave / Yes, Reveal** — executes the action

---

## Dark Mode

Click the **🌙** button in the top bar to toggle between dark (default) and light mode. The **☀️** icon appears in light mode. The entire UI switches via CSS custom properties — all colors, backgrounds, tile states, and modal surfaces adapt instantly.

---

## Sound Effects

Subtle sound effects are generated entirely with the Web Audio API — no external audio files required.

| Event | Sound |
|---|---|
| Typing a letter or digit | Short high beep |
| Deleting | Short low beep |
| Invalid input / invalid word | Buzzy descending tone |
| Tile reveal | Rising arpeggio (one tone per tile, staggered) |
| Win | Four-note ascending fanfare |
| Lose / time expires | Three-note descending fall |

Sounds play on all game modes. No on/off toggle is provided — browser tab muting works if needed.

---

## Numble Statistics

Statistics are tracked automatically across sessions using localStorage.

| Stat | Description |
|---|---|
| **Played** | Total Numble games played |
| **Win %** | Percentage of games won |
| **Streak** | Current consecutive win streak |
| **Best** | Best win streak ever |
| **Guess Distribution** | Bar chart of how many guesses each win took |

The current game's row is highlighted in the distribution chart when viewing stats after a game.

**Access:** Click the **📊** button in the top bar while in Numble mode.

**Reset:** A Reset Stats button inside the stats panel clears all data after confirmation.

**Unfinished game save:** If you close the tab mid-game, the board state (solution, row history, settings) is saved to localStorage and restored the next time you open the file in Numble mode with the same settings.

---

## Datamuse API Integration

Lexicon uses the [Datamuse API](https://www.datamuse.com/api/) for three purposes, all via plain `fetch()` with no API key required.

| Purpose | Endpoint | Used for |
|---|---|---|
| Word pool generation | `GET /words?sp=?????&max=500&md=f` | Building the pool of potential hidden words |
| Guess validation | `GET /words?sp={word}&md=df&max=20` | Checking whether a guess is a real English word |
| Explorer search | `GET /words?sp=` / `?ml=` / `?sl=` / `?rel_rhy=` | Powering the four Explorer search types |

All calls are wrapped in try/catch. The game never blocks or crashes if Datamuse is unavailable.

---

## Word List & Vocabulary

**Online:** Datamuse returns up to 500 frequency-ranked words per query — the primary and richer source.

**Offline:** Embedded fallback lists cover lengths 4–8 across easy, normal, and hard tiers (~100–300 words per combination). These are a safety net only and are not exhaustive.

As long as you have internet, you get a much wider vocabulary. The local lists exist purely for offline play.

---

## Offline Support

| Feature | Online | Offline |
|---|---|---|
| Word pool | Datamuse (up to 500 words) | Embedded lists |
| Word validation | Datamuse API + local cache | Permissive (all alphabetic guesses accepted) |
| Explorer search | Live Datamuse results | "Search failed" error message |
| Dictionary links | Opens Merriam-Webster | Link present; browser shows no-connection error |
| Numble | Fully offline | Fully offline |
| Stats / save | localStorage (fully offline) | localStorage (fully offline) |

---

## Dictionary Links

When a word game ends, a **"Look up in dictionary"** link appears above the keyboard. In Quordle/Hexordle, failed boards show the answer word as a clickable underlined link. In Explorer, every result chip is clickable. All links open:

```
https://www.merriam-webster.com/dictionary/{word}
```

Dictionary links do not apply to Numble.

---

## UI & Animations

**Tile flip:** Each tile flips on a 280ms stagger. Color is applied at the midpoint of the flip (when the tile is edge-on and invisible) so the color and letter appear cleanly on the return — no visual glitch where letters disappear.

**Pop:** Tiles briefly scale up (1.12×) on letter/digit input.

**Shake:** The current row shakes on invalid input or too-few characters.

**Input locking:** All input is locked while animations play, preventing accidental input into the next row.

**Board overlays:** In Quordle/Hexordle, resolved boards show a semi-transparent win/lose overlay with the answer.

**Timer states:** Normal → yellow (≤30s) → red pulsing (≤10s).

**Dark/light mode:** Full UI retheme via CSS custom properties — instant, no flash.

---

## Code Architecture

```
NUMBLE STATS (localStorage)   — loadNumbleStats, saveNumbleStats, loadNumbleSave, saveNumbleGame
SHARED STATE                  — all mutable variables for both word and Numble modes
SOUND ENGINE                  — Web Audio API tones: sfxType, sfxDelete, sfxInvalid, sfxReveal, sfxWin, sfxLose
DARK MODE                     — toggleDarkMode()
UTILITY                       — rand, uniqueWords, showToast, showValidating, setStatus, updateStatusBar
WORD VALIDATION               — isRealWord() — local cache + Datamuse two-stage check
WORD FETCHING                 — fetchWords, applyDifficultyFilter, applyDifficultyFallback
MODALS                        — openModal, closeModal, modalConfirmAction, openLeaveModal, isInProgress
HELP MODAL                    — showHelp() — content adapts to current mode
MODE SWITCHING                — requestSwitchMode, requestNewGame, switchMode
SETTINGS CHANGE               — onSettingChange (word), onNumbleSettingChange (Numble)
INIT WORD GAME                — initGame() — async, fetches words, resets state, starts timer
BOARD GENERATION              — generateBoard (Wordle), generateMultiBoard (Quordle/Hexordle)
WORD KEYBOARD                 — buildKeyboard, makeKey, updateKeyboard
WORD INPUT                    — typeLetter, deleteLetter, submitGuess (async)
GUESS PROCESSING              — checkGuess (shared), revealRow (shared), shakeRow (shared)
                                processWordleGuess, processMultiBoardGuess, checkMultiBoardEnd
                                showBoardOverlay, addRevealLink, openDict
KEYBOARD EVENTS               — shared keydown listener routing to word or Numble handlers
EXPLORER                      — explorerSearch()
TIMER                         — startTimer, stopTimer, renderTimer, showCustomTimerModal,
                                cancelCustomTimer, applyCustomTimer
REVEAL ANSWERS                — updateRevealButton, requestRevealAnswers, revealAllAnswers
NUMBLE                        — generateNumber, initNumble, buildNumbleBoard, buildNumKeyboard,
                                makeNumKey, nType, nDelete, nSubmit, nRevealAnswer, checkGuess (shared)
NUMBLE STATS UI               — showNumbleStats, hideNumbleStats, resetNumbleStats
BOOT                          — initGame()
```

**Key design decisions:**

- `checkGuess(guess, solution, length)` and `revealRow()` are shared between word and Numble modes — identical two-pass correct/present/absent logic.
- `isInProgress()` checks the correct game state depending on `MODE`, keeping the modal guard universal.
- `updateRevealButton()` keeps the Reveal Answers button visible even after the timer expires, so players can always access answers.
- `timerSeconds` is shared between modes; `startTimer` / `stopTimer` work identically for both.
- Numble saves use a versioned key structure `{solution, len, tries, repeats, row, over, history[]}` — mismatched settings on restore trigger a fresh game.
- `modalConfirmFn` is captured into a local variable before clearing in `modalConfirmAction()` to prevent any race where the callback could be lost.
- Sound effects are generated on-demand with Web Audio — no audio files, no preloading, no CORS issues.

---

## Technical Reference

| Property | Value |
|---|---|
| File type | Single `.html` file |
| File size | ~95 KB (uncompressed) |
| Dependencies | None |
| Frameworks | None |
| External API | Datamuse (no key required) |
| Browser support | Chrome, Firefox, Safari, Edge (any modern browser) |
| Word length range | 4–8 letters |
| Word attempt range | 4–10 |
| Numble digit range | 4–6 digits |
| Numble attempt range | 4–8 |
| Boards in Quordle | 4 |
| Boards in Hexordle | 6 |
| Tile flip duration | 500ms |
| Tile stagger delay | 280ms |
| Validation score threshold | 1000 (Datamuse score) |
| Timer presets | Off, 2 min, 3 min, 5 min, 10 min, Custom (1–99 min) |
| Timer warning threshold | 30 seconds remaining |
| Timer danger threshold | 10 seconds remaining |
| Stats storage | localStorage (`lexicon_numble_stats`, `lexicon_numble_save`) |

---

## Known Limitations

- **No word-mode score tracking.** Win/loss history for Wordle, Quordle, and Hexordle is not saved. Only Numble tracks statistics.
- **No share/copy results.** The colored emoji grid common in Wordle clones is not implemented.
- **Single language.** Only English words via Datamuse.
- **Offline validation is permissive.** Without internet, any alphabetic string of the correct length is accepted.
- **No hard-mode word enforcement.** Hard difficulty only affects vocabulary; reusing revealed letters is not enforced.
- **Timer does not pause.** There is no pause button; switching to Explorer while a timer runs continues counting.
- **Local file access.** When opened as a `file://` URL, most browsers work correctly for `fetch()`. If Datamuse calls fail unexpectedly, try: `python3 -m http.server` and open via `http://localhost:8000`.
- **Sound requires user interaction.** Web Audio API requires at least one user gesture before audio plays. The first keypress or button click in each session unlocks audio automatically.
