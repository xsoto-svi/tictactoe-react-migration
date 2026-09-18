# Tic-Tac-Toe — Vanilla JavaScript → React Structure

The React structure keeps the same application concepts as the existing Vanilla JS project, but moves responsibilities into the places where React handles them naturally.

The main change is not the UI itself. It is **responsibility ownership**: Vanilla JS classes currently create DOM nodes, attach listeners, mutate state, and manage polling. React separates those concerns into pages, components, hooks, state, and services.

## Folder structure

```text
src/
├── App.jsx
├── main.jsx
├── assets/
├── components/
│   ├── common/
│   ├── game/
│   ├── howToPlay/
│   └── lobby/
├── hooks/
├── pages/
├── services/
├── state/
├── styles/
└── utils/
```

### `pages/`

`pages/` contains the four application screens that already exist in the Vanilla JS project:

```text
js/pages/homePage.js        → pages/HomePage.jsx
js/pages/howToPlayPage.js   → pages/HowToPlayPage.jsx
js/pages/lobbyPage.js       → pages/LobbyPage.jsx
js/pages/gamePage.js        → pages/GamePage.jsx
```

A page is responsible for composing the screen and connecting its components to the required state and behavior. The old `Page` base class is not carried over because React already provides the rendering and lifecycle model.

### `components/`

The existing `BoardComponent` and `CellComponent` naturally become React components:

```text
BoardComponent      → components/game/Board.jsx
CellComponent       → components/game/Cell.jsx
```

The component folders group UI by responsibility:

```text
components/
├── game/          Board, Cell, scoreboard, game status, leave-game UI
├── lobby/         Create-game and join-game UI
├── howToPlay/     Instruction UI
└── common/        Shared modal/toast UI
```

The important change is that components no longer create and update DOM elements themselves. Instead, they receive data through props and report user actions through callbacks. React re-renders the affected UI when that data changes.

### `hooks/`

`hooks/` contains behavior that was previously embedded inside page classes.

For example, the current `GamePage` owns asynchronous polling, lifecycle-sensitive cleanup, and other stateful behavior. In React, those responsibilities can be extracted into hooks such as:

```text
useGameState
useGamePolling
useRoomReadyPolling
useRematchPolling
useBeforeUnload
useToast
```

The polling behavior itself stays conceptually the same: the application still polls the backend and keeps an in-flight guard so multiple requests are not running at once. The difference is that React hooks control when polling starts, when it stops, and what state is updated when a response arrives.

### `state/`

The Vanilla JS project has a shared `GameState` object and `gameConstants.js`.

Those responsibilities become:

```text
js/game/gameState.js
    → state/gameReducer.js
    → state/GameContext.jsx
    → hooks/useGameState.js

js/game/gameConstants.js
    → state/gameConstants.js
```

The game data itself does not fundamentally change. What changes is how updates happen. Instead of directly mutating a shared object, components dispatch state changes and React propagates the new state to the components that depend on it.

### `services/`

The existing API abstraction is retained, but the single `TicTacToeApi` wrapper is divided according to the backend's domain controllers:

```text
services/
├── apiClient.js
├── gameService.js
├── roomService.js
└── playerService.js
```

The mapping is:

```text
GameController   → gameService.js
RoomController   → roomService.js
PlayerController → playerService.js
```

`apiClient.js` remains the low-level HTTP layer. The domain services provide the application-facing API so UI code does not need to know endpoint details.

HTTP behavior also stays at this boundary. For example, a `409 Conflict` from the backend is interpreted by the service/error layer and converted into something the page or component can display, rather than being handled throughout the UI.

### `utils/`

`utils/` contains small, reusable logic that is not specific to rendering or React state, such as API error normalization and room-code handling.

### `styles/` and `assets/`

The existing CSS organization is largely retained:

```text
css/global.css              → styles/global.css
css/pages/*                 → styles/pages/*
css/components/*            → styles/components/*
assets/*                    → src/assets/*
```

The styling model does not need to become fundamentally different just because the UI is React.

## How the data flows

The resulting flow is roughly:

```text
User interaction
      ↓
Component
      ↓
Page / hook
      ↓
Domain service
      ↓
apiClient
      ↓
Spring Boot controller
      ↓
Response
      ↓
Service / hook
      ↓
React state
      ↓
Components re-render
```

For multiplayer synchronization, the polling hooks sit between the page/state and the services:

```text
GamePage
   ↓
useGamePolling
   ↓
gameService
   ↓
backend
   ↓
state update
   ↓
Board / ScoreBoard / GameStatusBar
```

This replaces the current pattern where `GamePage` is responsible for both requesting data and manually updating the DOM.

## What stays the same vs. what changes

### Retained

- The same four major screens.
- The same game concepts and state data.
- The Board → Cell component hierarchy.
- The same backend integration concept.
- The same polling-based synchronization strategy.
- The existing CSS organization and visual assets.
- Domain-specific API responsibilities, now aligned with `GameController`, `RoomController`, and `PlayerController`.

### Changed

- `Page` classes become functional page components.
- `ViewTemplate` and manual DOM construction are replaced by JSX.
- `setValue()`, `updateCell()`, and direct element manipulation are replaced by props and state-driven rendering.
- `GameState` becomes React-managed state rather than a mutable shared object.
- Polling and browser lifecycle behavior move into hooks.
- The single `TicTacToeApi` wrapper is split into domain services.
- Routing/application composition moves from the custom imperative router into the React application structure.

## Summary

The migration is therefore mostly a **reorganization of responsibilities rather than a redesign of the application**.

The Vanilla JS project already has useful boundaries around pages, components, state, services, and styles. React keeps those boundaries, but introduces two important layers—`hooks/` and `state/`—to handle behavior and state in a React-native way. The biggest architectural improvement is that UI components become declarative and focused on presentation, while pages compose them, hooks manage effects such as polling, state manages application data, and services handle communication with the Spring Boot backend.
