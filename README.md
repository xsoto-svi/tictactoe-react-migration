# Tic-Tac-Toe — Vanilla JavaScript → React Structure

The migration keeps the same application responsibilities, but changes **where those responsibilities live**. The Vanilla JS project is organized around classes that create DOM elements, mutate them, and manage page behavior. The React version keeps the same screens, game rules, API boundary, and visual structure, while separating UI, state, side effects, and backend communication into React-friendly layers.

## Folder structure

```text
src/
├── App.jsx
├── main.jsx
├── assets/
├── components/
│   ├── common/
│   │   ├── Modal.jsx
│   │   ├── AlertModal.jsx
│   │   ├── ConfirmationModal.jsx
│   │   ├── LoadingModal.jsx
│   │   └── ResetGameModal.jsx
│   ├── game/
│   │   ├── Board.jsx
│   │   ├── Cell.jsx
│   │   ├── ScoreBoard.jsx
│   │   ├── GameStatusBar.jsx
│   │   ├── GameErrorMessage.jsx
│   │   └── LeaveGameButton.jsx
│   ├── lobby/
│   │   ├── CreateGameCard.jsx
│   │   └── JoinGameCard.jsx
│   └── how-to-play/
│       ├── InstructionList.jsx
│       └── InstructionItem.jsx
├── pages/
│   ├── HomePage.jsx
│   ├── LobbyPage.jsx
│   ├── HowToPlayPage.jsx
│   └── GamePage.jsx
├── hooks/
│   ├── useGameState.js
│   ├── usePolling.js
│   └── useBeforeUnload.js
├── state/
│   ├── GameContext.jsx
│   ├── gameReducer.js
│   └── gameConstants.js
├── services/
│   ├── apiClient.js
│   ├── gameService.js
│   ├── roomService.js
│   └── playerService.js
├── utils/
│   ├── apiError.js
│   └── roomCode.js
└── styles/
    ├── global.css
    ├── pages/
    └── components/
```

### What each folder does

| Folder | Responsibility | Vanilla JS source it replaces or reorganizes |
|---|---|---|
| `pages/` | Composes complete screens and connects them to state and behavior. | `js/pages/*` and the screen selection logic in `router.js` |
| `components/` | Small, reusable pieces of UI. Components receive data as props and emit user actions instead of creating DOM nodes themselves. | `BoardComponent`, `CellComponent`, modal classes, and parts of `GamePage`, `LobbyPage`, and `HowToPlayPage` |
| `hooks/` | React-specific behavior and side effects such as polling, browser lifecycle events, and access to game state. | `GamePage`/`LobbyPage` timers and event handlers, plus parts of `GameState` access |
| `state/` | Owns shared game data and the state transitions that used to be handled by the mutable `GameState` class. | `js/game/gameState.js` and `js/game/gameConstants.js` |
| `services/` | Handles HTTP communication without exposing `fetch()` details to UI code. | `apiClient.js` and `tictactoeApi.js` |
| `utils/` | Small framework-independent helpers. | Reusable logic extracted from the current classes, such as room-code and API-error handling |
| `styles/` | Keeps the existing CSS responsibilities separate from the React components. | Existing `css/` tree |
| `assets/` | Static game assets such as X and O icons. | Existing `assets/` directory |

## What changes from Vanilla JS to React

**Pages stay, but their responsibility becomes smaller.** `HomePage`, `LobbyPage`, `HowToPlayPage`, and `GamePage` remain the four major screens. They no longer construct elements with `document.createElement()` or attach DOM listeners directly. JSX describes the screen, while hooks and components handle the behavior needed by that screen.

**Components become declarative.** The existing `BoardComponent` and `CellComponent` already suggest a good React hierarchy, so they become `Board` → `Cell`. `GamePage` is further decomposed into `ScoreBoard`, `GameStatusBar`, `GameErrorMessage`, `LeaveGameButton`, and the modal components because those are independent pieces of UI. A change in state causes React to render the new UI instead of calling methods such as `updateCell()` or `updateUI()` manually.

**The mutable `GameState` becomes React state.** The game still needs the same information—room code, player symbol, board, status, winner, and scores—but updates go through a reducer/context or a state hook rather than direct property mutation. This gives every interested component a predictable source of truth.

**The custom `Router` becomes application-level React routing.** Navigation remains a page-level concern, but React controls which page is rendered instead of manually creating and removing page DOM trees.

**Polling is retained, but moved out of page classes.** The current recursive `setTimeout` polling and its in-flight guard are still valid for synchronizing two clients. A reusable `usePolling` hook owns the timer lifecycle, cleanup, and in-flight protection. The game page can use it for board updates, while the lobby/rematch flows can configure the same pattern for room readiness checks.

**The API wrapper is reorganized by backend domain.** `TicTacToeApi` currently contains every endpoint in one class. React keeps the common `apiClient`, but separates application calls into `gameService`, `roomService`, and `playerService`, matching the Spring Boot `GameController`, `RoomController`, and `PlayerController`. HTTP responses and errors stay in this service layer so components are not coupled to backend details.

## Data flow

The main flow becomes:

```text
User action
    ↓
React component
    ↓
Page / hook
    ↓
State update or service call
    ↓
Spring Boot controller
    ↓
Service response
    ↓
React state
    ↓
Components re-render
```

For multiplayer synchronization, polling is simply another side effect in that flow:

```text
GamePage
    ↓
usePolling
    ↓
gameService
    ↓
GameController
    ↓
updated board
    ↓
React state
    ↓
Board / ScoreBoard / GameStatusBar
```

## Naming and decomposition

Component files use **PascalCase** (`Board.jsx`, `GamePage.jsx`). Hooks use the React `useX` convention (`usePolling.js`, `useGameState.js`). Services, reducers, utilities, and CSS files use **camelCase or kebab-case according to their role**, keeping naming predictable across the project.

The important decomposition is that each layer answers a different question: **pages compose screens, components render UI, hooks manage React behavior, state owns application data, and services communicate with the backend.** This reduces the amount of unrelated responsibility currently concentrated in `GamePage` and makes each part easier to change without affecting the others.

## Summary of the migration

The React version is not a redesign of the game. The existing screens, game state, board/cell hierarchy, polling approach, backend integration, CSS, and assets are retained. The main change is the **organization of responsibilities**:

```text
Vanilla JS                         React
────────────────────────────────────────────────────
Page classes                  →   functional page components
DOM construction              →   JSX
Component classes             →   functional components
Mutable GameState             →   Context / reducer / hooks
Page-owned timers             →   reusable hooks
Single TicTacToeApi           →   domain services
Custom DOM router             →   React application routing
Manual DOM updates            →   state-driven rendering
```

That structure preserves what is already working in the Vanilla JS application while making the codebase more modular, predictable, and easier to extend.
