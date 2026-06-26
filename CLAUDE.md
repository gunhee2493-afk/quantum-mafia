# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**양자 마피아: 에이온 침략전** (Quantum Mafia: Aeon Invasion) — a browser-based multiplayer social deduction game with quantum mechanics theme.

The entire game is a single file: [`aeon ver1.21/index.html`](aeon%20ver1.21/index.html). There is no build step, no package manager, and no server-side code. Open the file directly in a browser to run it.

## Architecture

All game logic, UI, and styling live in one HTML file with three sections:

**External CDN dependencies (loaded in `<head>`):**
- Tailwind CSS — utility styling
- Google Fonts (Orbitron, Rajdhani) — UI fonts
- Tone.js — procedural sound effects

**Firebase (ES module imports inside `<script type="module">`):**
- `firebase/app`, `firebase/auth` — anonymous auth
- `firebase/firestore` — real-time multiplayer state via `onSnapshot`

**Game state flow (Firestore-driven):**
All multiplayer state lives in a Firestore `rooms/{roomId}` document. Clients subscribe via `subscribeToRoom()` and react to changes in `onSnapshot`. The host runs all phase-transition logic (`runPhaseLogic()`) as a Firestore transaction; players only submit actions (`submitVote`, `submitNightAction`, card actions).

**Game phases:**
`DAY/NIGHT` × `DISCUSSION → VOTE/ACTION → DAY_RESULT/NIGHT_RESULT` → repeats for `totalRounds` → `END` → `OBSERVATION → MEASUREMENT_READY → MEASUREMENT → RESULT`

**Game modes:**
- `classic` — base game, no ability cards
- `chaos` — each player gets a random ability card from `abilityCardDeck`

**Player roles & quantum state:**
Each player has a `quantumState` array of 6 chips (`'AEON'` or `'CITIZEN'`). `currentRole` is derived from chip composition: all AEON → `'AEON'`, all CITIZEN → `'DEVELOPER'`, mixed → `'CITIZEN'`. Final role is determined probabilistically at end-game by `measurePlayer()`.

**Key global variables:**
- `localPlayer` — `{ id, nickname, number, isHost }`
- `currentRoomId`, `currentRoomData` — current room
- `db`, `auth` — Firebase instances
- `isDevMode` — enables bot controls and relaxed player minimums

**UI pattern:**
Screens are `<div class="screen">` elements; `showScreen(id)` hides all and shows the target. Popups are toggled with `.hidden`. All interactive elements call global functions attached to `window`.
