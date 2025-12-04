> [< back](./README.md)

# AirCade Milestones

## M0.1.0: Repository & Infrastructure Setup

**Goal:** Establish the monorepo structure, shared tooling, and initial CI/CD pipeline.

- [ ] **[Shared]** Initialize project root with pnpm workspaces.
- [ ] **[Shared]** Use `create-turbo` to scaffold the monorepo structure.
- [ ] **[Frontend]** Create the Next.js application inside `apps/web`.
- [ ] **[Backend]** Create the NestJS application inside `apps/api`.
- [ ] **[Shared]** Create shared package folders: `packages/database`, `packages/types`, `packages/eslint-config`, `packages/typescript-config`.
- [ ] **[Shared]** Configure `turbo.json` with initial build and dev pipelines.
- [ ] **[Infra]** Set up a basic CI workflow in GitHub Actions to run `pnpm install`, `pnpm lint`, and `pnpm build` on push.

## M0.2.0: Core Backend Services & Database Schema

**Goal:** Implement the foundational backend services, database schema, and user authentication.

- [ ] **[Database]** Provision a new project on Supabase to get a PostgreSQL database.
- [ ] **[Database]** Add the Supabase connection string to a root `.env` file.
- [ ] **[Database]** Initialize Prisma in `packages/database` and connect it to the Supabase instance.
- [ ] **[Database]** Define the `User` model in `schema.prisma` with fields for email, hashed password, and timestamps.
- [ ] **[Database]** Run the first Prisma migration to create the `User` table.
- [ ] **[Backend]** Create an `AuthModule` in the NestJS app.
- [ ] **[Backend]** Implement a `UserService` to handle user creation and lookup.
- [ ] **[Backend]** Implement `bcrypt` for password hashing and comparison in the `AuthService`.
- [ ] **[Backend]** Create `POST /auth/register` and `POST /auth/login` endpoints in an `AuthController`.
- [ ] **[Backend]** Configure `@nestjs/jwt` and a `JwtStrategy` to issue and validate JWTs upon login.
- [ ] **[Backend]** Create a test endpoint protected with `@UseGuards(JwtAuthGuard)` to verify authentication.

## M0.3.0: Session Management & Real-time Connection

**Goal:** Build the system for creating game sessions and connecting controllers to consoles via WebSockets.

- [ ] **[Database]** Define `Session` and `SessionPlayer` models in `schema.prisma` and run a new migration.
- [ ] **[Backend]** Create a `SessionModule` in the NestJS app.
- [ ] **[Backend]** Implement a `SessionService` with a function to generate a unique, 6-character alphanumeric session code.
- [ ] **[Backend]** Create a `POST /sessions` endpoint that creates a session in the database and returns the new code.
- [ ] **[Backend]** Implement a `GatewayModule` using `@nestjs/websockets` and the Socket.IO adapter.
- [ ] **[Backend]** Create a WebSocket event handler for `join_session` that validates the session code.
- [ ] **[Backend]** On successful validation, add the client's socket to a room named after the session code (e.g., `socket.join(code)`).
- [ ] **[Backend]** Emit a `player_joined` event to the room, broadcasting the new player's arrival.
- [ ] **[Frontend]** Create the `/play` page (`ConsoleView`) that calls `POST /sessions` to get a code on page load.
- [ ] **[Frontend]** Display the session code and a QR code (using `qrcode.react`) on the `ConsoleView`.
- [ ] **[Frontend]** Implement the Socket.IO client in `ConsoleView` to listen for `player_joined` events and display connected players.
- [ ] **[Frontend]** Create the `/join` page (`ControllerView`) with a form to enter the session code.
- [ ] **[Frontend]** On form submission, emit the `join_session` event via the Socket.IO client.
- [ ] **[Frontend]** On a success response from the server, navigate the controller to a "Lobby" screen.

## M0.4.0: Game Logic Prototype & Controller Input

**Goal:** Implement a very simple playable prototype to demonstrate synchronized state and real-time input.

- [ ] **[Shared]** Define basic `GameInput` and `GameState` types in `packages/types`.
- [ ] **[Backend]** Create a `GameModule` and a `GameService` to manage the state of active game sessions.
- [ ] **[Backend]** Implement a basic `GameLoop` service (e.g., using `setInterval`) that ticks and updates a simple game state (e.g., a counter).
- [ ] **[Backend]** On each tick, emit a `game_state_update` event to all sockets in the session room.
- [ ] **[Backend]** Create a WebSocket event handler for `game_input` from controllers.
- [ ] **[Frontend]** Design a simple controller UI for the prototype (e.g., a single large "Tap" button).
- [ ] **[Frontend]** When the button is tapped, emit a `game_input` event to the backend.
- [ ] **[Frontend]** On the `ConsoleView`, create a canvas or div that renders the `game_state_update` data from the server.
- [ ] **[Backend]** Implement logic to handle graceful player disconnects (e.g., remove player from state and notify others).

## M0.5.0: Game Catalog & Dynamic Launcher

**Goal:** Allow hosts to browse a catalog of games and launch a selected game, with controllers adapting dynamically.

- [ ] **[Database]** Define a `Game` model in `schema.prisma` (with fields for title, slug, player count, description) and migrate.
- [ ] **[Database]** Seed the database with 2-3 sample games.
- [ ] **[Backend]** Create a `GamesController` with a `GET /games` endpoint to fetch the list of all available games.
- [ ] **[Frontend]** Create a `HomePage` (`/`) that fetches and displays the game catalog from the API.
- [ ] **[Frontend]** On the `ConsoleView` lobby, allow the host to select a game from the catalog.
- [ ] **[Backend]** Update the `join_session` logic to associate the session with the selected game.
- [ ] **[Backend]** Refactor the gateway to load different input handlers based on the active game for the session.
- [ ] **[Shared]** Define controller layout configurations (e.g., JSON schemas) for each game.
- [ ] **[Frontend]** When a controller joins a session, fetch the game's controller layout config and render it dynamically.

## M1.0.0: First Playable Release (MVP)

**Goal:** Polish the experience for a complete end-to-end session of one simple but fully playable party game (e.g., Trivia).

- [ ] **[Backend]** Implement the full game state machine: `Lobby` -> `Instructions` -> `Countdown` -> `In-Game` -> `RoundOver` -> `Scoreboard` -> `End`.
- [ ] **[Frontend]** Create React components for each state of the game on both the `ConsoleView` and `ControllerView`.
- [ ] **[Frontend]** Design and build a visually appealing scoreboard/results screen on the console.
- [ ] **[Frontend]** Add "How to Play" instructions to the controller UI before the game begins.
- [ ] **[Frontend]** Add subtle animations and user feedback (e.g., button presses, screen transitions) to improve UX.
- [ ] **[QA]** Conduct a playtest session with at least 4 players to identify bugs and latency issues.
- [ ] **[QA]** Test on different browsers (Chrome, Firefox, Safari) and devices (iOS, Android).
- [ ] **[Infra]** Configure production environment variables for the API and Web apps.
- [ ] **[Infra]** Set up deployment pipelines to deploy `apps/web` to Vercel.
- [ ] **[Infra]** Set up deployment pipelines to deploy `apps/api` to a Node.js hosting provider (e.g., Railway, Fly.io).
- [ ] **[Documentation]** Write a basic `README.md` explaining how to run the project and play the game.

> [< back](./README.md)
