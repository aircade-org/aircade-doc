# AirCade - Milestones

> **Note:** Each milestone has tasks sorted in order: shared, database, backend, frontend, test. However, these tags are not mandatory for every milestone.

---

## Phase 1: Foundation

### M0.1.0: Project Bootstrap

**Goal:** Establish the development infrastructure, CI/CD pipeline, and a deployable skeleton for both the API and web application.

- [X] **[Shared]** Define environment variable schema (database URL, JWT secret, JWT expiration durations, OAuth credentials) and document in a shared `.env.example`.
- [X] **[Shared]** Configure Railway.com project with separate services for PostgreSQL, API server, and web frontend.
- [X] **[Database]** Provision PostgreSQL instance on Railway and verify connectivity from the API server.
- [X] **[Backend]** Initialize Rust project with Axum 0.8, Tokio, and project dependency manifest (see technical stack).
- [X] **[Backend]** Configure Tower middleware stack: CORS, request tracing (`tracing` + `tracing-subscriber`), and JSON error responses.
- [X] **[Backend]** Set up SeaORM with `sea-orm-migration` and create an empty initial migration.
- [X] **[Backend]** Implement `GET /api/v1/health` endpoint returning server status, version, and database connectivity check.
- [X] **[Frontend]** Initialize Next.js 16 project with TypeScript, Tailwind CSS, shadcn/ui, and Zustand.
- [X] **[Frontend]** Set up Axios HTTP client with base URL configuration and interceptor for Bearer token injection.
- [X] **[Frontend]** Create application shell with layout, navigation placeholder, and routing structure for Studio, Console, and Controller experiences.
- [X] **[Test]** Set up backend integration test harness with a test database and request helpers.
- [X] **[Test]** Set up frontend component test framework.
- [X] **[Shared]** Configure CI/CD pipeline: automated build, test, and deploy on push to main branch.

---

### M0.2.0: Authentication

**Goal:** Implement the full authentication system so users can sign up, sign in, verify their email, reset passwords, and authenticate via Google and GitHub OAuth.

- [X] **[Database]** Create `User` table migration with all fields: id, email, username, displayName, avatarUrl, bio, emailVerified, role, subscriptionPlan, subscriptionExpiresAt, accountStatus, lastLoginAt, lastLoginIp, createdAt, updatedAt, deletedAt.
- [X] **[Database]** Create `AuthProvider` table migration with all fields: id, userId (FK -> User), provider, providerId, passwordHash, providerEmail, verificationToken, tokenExpiresAt, createdAt. Add unique constraints on (userId, provider) and on providerId.
- [X] **[Backend]** Implement SeaORM entity models for `User` and `AuthProvider`.
- [X] **[Backend]** Implement JWT token generation (access + refresh tokens) and validation middleware that extracts the authenticated user and enforces role-based authorization (Guest, User, Moderator, Admin).
- [X] **[Backend]** Implement `POST /api/v1/auth/signup/email` - create User + AuthProvider (email), hash password, send verification email, return tokens.
- [X] **[Backend]** Implement `POST /api/v1/auth/signin/email` - verify credentials, update lastLoginAt/lastLoginIp, return tokens.
- [X] **[Backend]** Implement `POST /api/v1/auth/verify-email` - validate token, set emailVerified to true.
- [X] **[Backend]** Implement `POST /api/v1/auth/resend-verification` - generate new token, resend email.
- [X] **[Backend]** Implement `POST /api/v1/auth/password-reset/request` and `POST /api/v1/auth/password-reset/confirm` - token-based password reset flow.
- [X] **[Backend]** Implement `POST /api/v1/auth/password/change` - change password for authenticated user.
- [X] **[Backend]** Implement `GET /api/v1/auth/oauth/google` (initiate) and `GET /api/v1/auth/oauth/google/callback` - Google OAuth 2.0 flow with auto-account creation.
- [X] **[Backend]** Implement `GET /api/v1/auth/oauth/github` (initiate) and `GET /api/v1/auth/oauth/github/callback` - GitHub OAuth 2.0 flow with auto-account creation.
- [X] **[Backend]** Implement `POST /api/v1/auth/link/{provider}` and `DELETE /api/v1/auth/link/{provider}` - link/unlink additional auth providers to an existing account.
- [X] **[Backend]** Implement `POST /api/v1/auth/refresh` - refresh access token using refresh token.
- [X] **[Backend]** Implement `POST /api/v1/auth/signout` - invalidate tokens.
- [X] **[Backend]** Enforce account status checks in auth middleware: reject suspended/deactivated accounts with 403.
- [X] **[Frontend]** Build sign-up page with email/password form and validation (Zod schemas).
- [X] **[Frontend]** Build sign-in page with email/password form, Google OAuth button, and GitHub OAuth button.
- [X] **[Frontend]** Implement OAuth callback handler pages for Google and GitHub redirects.
- [X] **[Frontend]** Create Zustand auth store: token storage, user state, auto-refresh logic, and sign-out.
- [X] **[Frontend]** Build email verification landing page and resend verification UI.
- [X] **[Frontend]** Build password reset request page and password reset confirmation page.
- [X] **[Frontend]** Add auth-protected route wrapper that redirects unauthenticated users to sign-in.
- [X] **[Test]** Write backend tests for all auth endpoints: sign-up, sign-in, OAuth flows, email verification, password reset, token refresh, sign-out.
- [X] **[Test]** Write backend tests for auth middleware: role enforcement, account status blocking.

---

### M0.3.0: User Management

**Goal:** Allow users to view and edit their profile, upload avatars, manage their email, and view public creator profiles.

- [X] **[Backend]** Implement `GET /api/v1/users/me` - return the authenticated user's full profile.
- [X] **[Backend]** Implement `PATCH /api/v1/users/me` - update displayName, bio, avatarUrl.
- [X] **[Backend]** Implement `POST /api/v1/users/me/avatar` - upload avatar image (multipart/form-data), store file, update avatarUrl.
- [X] **[Backend]** Implement `DELETE /api/v1/users/me/avatar` - remove avatar, reset to default.
- [X] **[Backend]** Implement `PATCH /api/v1/users/me/username` - change username with uniqueness validation.
- [X] **[Backend]** Implement `PATCH /api/v1/users/me/email` - change email address, reset emailVerified, send new verification email. Requires current password (or account confirmation for OAuth-only users).
- [X] **[Backend]** Implement `DELETE /api/v1/users/me` - soft-deactivate the account (set accountStatus to deactivated, hide profile and games). Requires current password (or account confirmation for OAuth-only users).
- [X] **[Backend]** Implement `GET /api/v1/users/:username` - return public profile with published games, aggregate stats (total games, total play count).
- [ ] **[Frontend]** Build user settings page with profile edit form (displayName, bio) and separate username change form.
- [ ] **[Frontend]** Build avatar upload component with preview and delete functionality.
- [ ] **[Frontend]** Build email change flow with re-verification notice.
- [ ] **[Frontend]** Build account deactivation confirmation dialog.
- [ ] **[Frontend]** Build public user profile page showing creator info, published games list, and aggregate statistics.
- [X] **[Test]** Write backend tests for all user endpoints: profile CRUD, avatar upload/delete, email change, deactivation, public profile.

---

### MS0.3.0s: Pong PoC - Console-to-Controller Synchronization

**Goal:** Demonstrate the core AirCade experience end-to-end: a user opens the web app, selects a hardcoded Pong game from a game list, generates a QR code, connects a phone as a controller, and plays single-player Pong with the paddle controlled from the phone via WebSocket.

**Scope:** This is a vertical slice / Proof of Concept. No multiplayer, no Creative Studio, no game publishing flow. The Pong game (p5.js) is hardcoded in the frontend. Authentication is required only for session hosting (already implemented). The controller join flow supports anonymous guests.

> **Approach:** Each task below is either a **new PoC-specific task** or a **reference to an existing milestone task** (marked with `→ from M0.x.0`). Referenced tasks should be implemented in their simplest form sufficient for the PoC — full production behavior will be completed in their original milestone.

#### Phase A: Database — Session & Player tables

These are prerequisites for the entire session system. Implement them exactly as specified in M0.6.0.

- [X] **[Database]** Create `Game` table migration with all fields including gameScreenCode, controllerScreenCode, publishedVersionId, cached aggregates (playCount, totalPlayTime, avgRating, reviewCount), and forkedFromId self-reference. → *from M0.4.0*
- [X] **[Database]** Create `GameVersion` table migration with unique constraint on (gameId, versionNumber). → *from M0.4.0*
- [X] **[Database]** Create `Session` table migration with all fields: id, hostId, gameId, gameVersionId, sessionCode, status, maxPlayers, createdAt, updatedAt, endedAt. → *from M0.6.0*
- [X] **[Database]** Create `Player` table migration with all fields: id, sessionId, userId, displayName, avatarUrl, connectionStatus, leftAt, createdAt. → *from M0.6.0*

#### Phase B: Backend — Game entity, Session CRUD & WebSocket relay

Implement the backend session lifecycle and WebSocket communication. For the PoC, only the subset of endpoints needed for the Pong flow is required.

- [X] **[Backend]** Implement SeaORM entity models for `Game` and `GameVersion`. → *from M0.4.0*
- [X] **[Backend]** Implement SeaORM entity models for `Session` and `Player`. → *from M0.6.0*
- [X] **[Backend]** Implement session code generation: 4-6 uppercase alphanumeric characters excluding ambiguous characters (0/O, 1/I/L), unique among active sessions, case-insensitive lookup. → *from M0.6.0*
- [X] **[Backend]** Implement `POST /api/v1/sessions` — create a session in lobby status with generated session code. For the PoC, the `testGameId` parameter is not needed; sessions start in lobby. → *from M0.6.0*
- [X] **[Backend]** Implement `GET /api/v1/sessions/{sessionCode}` — return session details including player list and loaded game info. → *from M0.6.0*
- [X] **[Backend]** Implement `POST /api/v1/sessions/{sessionCode}/join` — join a session by session code. Validate session exists, is not ended, and has not reached maxPlayers. Create Player record. Support anonymous join (userId nullable). → *from M0.6.0*
- [X] **[Backend]** Implement `GET /api/v1/sessions/{sessionId}/players` — list all players in a session. → *from M0.6.0*
- [X] **[Backend]** Implement `DELETE /api/v1/sessions/{sessionId}` — set status to ended, set endedAt, close all WebSocket connections. Host only. → *from M0.6.0*
- [X] **[Backend]** Implement `POST /api/v1/sessions/{sessionId}/game` — load a game into the session (for the PoC, this will load the hardcoded Pong game seeded in the database). Validate game exists. Set gameId and gameVersionId. Transition status to `playing`. Deliver `game_loaded` WebSocket message with gameScreenCode to host and controllerScreenCode to players. → *simplified from M0.6.0*
- [X] **[Backend]** Implement WebSocket endpoint `WS /api/v1/sessions/{sessionId}/ws` — establish WebSocket connection for a client (host or player). Accepts `role`, `playerId`, and optional `token` as query parameters. → *from M0.6.0*
- [X] **[Backend]** Implement WebSocket message routing: relay `player_input` from controller to host as `player_input_event`, relay `game_state_update` from host to all players as `game_state`. Handle `connected`, `player_joined`, `player_left`, `game_loaded`, and `session_status_change` messages. → *from M0.6.0*
- [X] **[Backend]** Implement basic player connection lifecycle: track connectionStatus on connect/disconnect. Full reconnection grace period can be deferred. → *simplified from M0.6.0*

#### Phase C: Backend — Seed Pong game data

A one-time seed to have the Pong game available in the database for the PoC.

- [X] **[Backend]** Create a database seed (or migration) that inserts a `Game` record for "Pong" (status: `published`, visibility: `public`, technology: `p5js`, minPlayers: 1, maxPlayers: 1) with placeholder gameScreenCode and controllerScreenCode, and a corresponding `GameVersion` record. The actual p5.js code will be hardcoded in the frontend for the PoC, but the Game/GameVersion records must exist so the session load flow works correctly.

#### Phase D: Frontend — Console experience (big screen)

Build the big-screen experience: game list → session creation → QR code lobby → game rendering.

- [ ] **[Frontend]** Build a simple game list page showing available games. For the PoC this shows the single hardcoded Pong game with title, description, and a "Play" button. This is a minimal precursor to the full library in M0.8.0.
- [ ] **[Frontend]** Implement session creation flow: when the user clicks "Play" on a game, call `POST /api/v1/sessions` to create a session (user must be signed in). The session is created in `lobby` status.
- [ ] **[Frontend]** Build lobby screen: display session code prominently, show QR code (encoding the join URL `{baseUrl}/join/{sessionCode}`), render connected player list with names in real time via WebSocket. → *from M0.6.0*
- [ ] **[Frontend]** Create Zustand session store for managing session state, player list, and WebSocket connection. → *from M0.6.0*
- [ ] **[Frontend]** Implement WebSocket client on the Console: connect as `role=host`, handle `player_joined`, `player_left`, `player_input_event`, `game_loaded`, and `session_status_change` messages. Update session store reactively. → *from M0.6.0*
- [ ] **[Frontend]** Build the "controller connected" confirmation and "Start Game" button: once at least one player is connected, show a confirmation message and enable the "Start Game" button. Clicking it calls `POST /api/v1/sessions/{sessionId}/game` with the Pong game ID.
- [ ] **[Frontend]** Build Game Screen renderer: when the game is loaded (status transitions to `playing`), render the Pong game using p5.js in a sandboxed iframe. The gameScreenCode is hardcoded in the frontend for the PoC. Wire the `AirCade.onPlayerInput()` callback to receive paddle movement from the controller via WebSocket.
- [ ] **[Frontend]** Implement the Game Screen side of the AirCade runtime API (minimal PoC version): `AirCade.onPlayerInput(callback)` to receive controller input and `AirCade.broadcastState(state)` to send game state back to the controller. → *simplified from M0.5.0*

#### Phase E: Frontend — Controller experience (phone)

Build the smartphone controller: join via QR/code → lobby wait → Pong paddle controller.

- [ ] **[Frontend]** Build the Controller join page: input field for session code and join button. The QR code scanned from the Console should navigate directly to this page with the session code pre-filled. → *from M0.7.0*
- [ ] **[Frontend]** Build a minimal player setup: prompt for display name before joining. Call `POST /api/v1/sessions/{sessionCode}/join` with the display name. → *simplified from M0.7.0*
- [ ] **[Frontend]** Build the Controller lobby view: show "Waiting for host to start the game..." status message. → *from M0.7.0*
- [ ] **[Frontend]** Implement WebSocket client on the Controller: connect as `role=player` with the playerId received from the join response. Handle `game_loaded`, `game_state`, and `session_status_change` messages. → *from M0.7.0*
- [ ] **[Frontend]** Build the Controller game view for Pong: render a touch-based paddle controller using p5.js. Capture touch/drag events for vertical paddle movement. Send input via `AirCade.sendInput("paddle_move", { y: ... })` through WebSocket. → *from M0.7.0*
- [ ] **[Frontend]** Implement the Controller Screen side of the AirCade runtime API (minimal PoC version): `AirCade.sendInput(inputType, data)` to send input to the Game Screen and `AirCade.onStateUpdate(callback)` to receive game state for visual feedback. → *simplified from M0.5.0*
- [ ] **[Frontend]** Optimize the controller for mobile: viewport meta tags, prevent zoom/scroll, full-screen-friendly sizing. → *from M0.7.0*

#### Phase F: Hardcoded Pong game code (p5.js)

The actual game logic, temporarily hardcoded in the frontend.

- [ ] **[Frontend]** Write the Pong Game Screen code (p5.js): single-player Pong with a ball bouncing off walls and a player-controlled paddle on one side, an AI paddle on the other. Use `AirCade.onPlayerInput()` to receive paddle position from the controller. Use `AirCade.broadcastState()` to send ball position and score to the controller for optional visual feedback.
- [ ] **[Frontend]** Write the Pong Controller Screen code (p5.js): a simple vertical slider/touch area representing the paddle. Capture touch Y-position and send it via `AirCade.sendInput("paddle_move", { y })`. Optionally display the current score received from `AirCade.onStateUpdate()`.

#### Phase G: Integration test

- [ ] **[Test]** Manual end-to-end test of the full PoC flow: sign in → see Pong in game list → click Play → session created with QR code → scan QR from phone → enter name and join → Console shows "controller connected" → click Start Game → Pong renders on Console → paddle moves from phone → ball responds → game is playable.

---

## Phase 2: Core Platform

### M0.4.0: Game Management

**Goal:** Enable creators to create game projects, write code for both canvases, manage assets and tags, publish versions, and handle the full game lifecycle (draft/published/archived).

- [ ] **[Database]** Create `Game` table migration with all fields including gameScreenCode, controllerScreenCode, publishedVersionId, cached aggregates (playCount, totalPlayTime, avgRating, reviewCount), and forkedFromId self-reference.
- [ ] **[Database]** Create `GameVersion` table migration with unique constraint on (gameId, versionNumber).
- [ ] **[Database]** Create `GameAsset` table migration.
- [ ] **[Database]** Create `Tag` table migration with unique constraints on name and slug.
- [ ] **[Database]** Create `GameTag` join table migration with composite primary key (gameId, tagId).
- [ ] **[Database]** Seed initial `Tag` records for all three categories: genre (Racing, Trivia, Drawing, Strategy, etc.), mood (Competitive, Cooperative, Relaxed, Chaotic, etc.), and playerStyle (Free-for-all, Teams, Turn-based, Real-time, etc.).
- [ ] **[Backend]** Implement SeaORM entity models for `Game`, `GameVersion`, `GameAsset`, `Tag`, and `GameTag`.
- [ ] **[Backend]** Implement `POST /api/v1/games` - create a new game in draft/private status with the authenticated user as creator.
- [ ] **[Backend]** Implement `GET /api/v1/games/:id` - return game details (respecting visibility and ownership rules).
- [ ] **[Backend]** Implement `PATCH /api/v1/games/:id` - update game metadata (title, description, thumbnail, technology, minPlayers, maxPlayers, visibility, remixable) and code (gameScreenCode, controllerScreenCode). Owner only.
- [ ] **[Backend]** Implement `DELETE /api/v1/games/:id` - soft-delete a game and its assets. Owner only.
- [ ] **[Backend]** Implement `POST /api/v1/games/:id/publish` - validate title, non-empty canvas code, and verified email. Create immutable `GameVersion` snapshot, increment versionNumber, set publishedVersionId, transition status to published.
- [ ] **[Backend]** Implement `POST /api/v1/games/:id/archive` and `POST /api/v1/games/:id/unarchive` - lifecycle transitions between published/archived/draft.
- [ ] **[Backend]** Implement `POST /api/v1/games/:id/fork` - create a copy of a remixable game under the authenticated user with forkedFromId attribution.
- [ ] **[Backend]** Implement `GET /api/v1/games/:id/versions` - list all published versions (paginated).
- [ ] **[Backend]** Implement `GET /api/v1/games/:id/versions/:versionId` - get a specific version's code and changelog.
- [ ] **[Backend]** Implement `POST /api/v1/games/:id/assets` - upload game asset (multipart/form-data), validate file type (PNG, JPG, SVG, GIF, MP3, WAV, OGG, TTF, WOFF2) and size limits.
- [ ] **[Backend]** Implement `GET /api/v1/games/:id/assets` - list all assets for a game.
- [ ] **[Backend]** Implement `GET /api/v1/games/:id/assets/:assetId` - get a single asset.
- [ ] **[Backend]** Implement `DELETE /api/v1/games/:id/assets/:assetId` - delete an asset. Owner only.
- [ ] **[Backend]** Implement `GET /api/v1/tags` - list all platform tags, optionally filtered by category.
- [ ] **[Backend]** Implement `PUT /api/v1/games/:id/tags` - set the tags for a game (replace all). Owner only.
- [ ] **[Backend]** Implement `GET /api/v1/games/:id/tags` - get the tags assigned to a game.
- [ ] **[Backend]** Implement `GET /api/v1/users/me/games` - list the authenticated user's own games (all statuses).
- [ ] **[Backend]** Implement `GET /api/v1/users/:username/games` - list a user's published, public games.
- [ ] **[Frontend]** Build "My Games" dashboard listing the creator's games with status badges and quick actions.
- [ ] **[Frontend]** Build game creation dialog/page with title, description, technology selection, and player count configuration.
- [ ] **[Frontend]** Build game settings page for editing metadata, visibility, remixable toggle, and tags.
- [ ] **[Frontend]** Build publish flow with confirmation dialog showing version changelog input.
- [ ] **[Frontend]** Build game asset manager: upload, list, preview, and delete assets.
- [ ] **[Frontend]** Build version history page showing all published versions with changelogs.
- [ ] **[Test]** Write backend tests for game CRUD, publish flow (validation, version creation), archive/unarchive, forking, asset management, and tag operations.

---

### M0.5.0: Creative Studio

**Goal:** Build the browser-based game development environment with dual-pane code editor, live preview, and test session launch capability.

- [ ] **[Frontend]** Build the Creative Studio layout with resizable panels: code editor area, preview area, and game settings sidebar.
- [ ] **[Frontend]** Integrate Monaco Editor (`@monaco-editor/react`) with dual editor panes - one for Game Screen code, one for Controller Screen code.
- [ ] **[Frontend]** Configure Monaco for JavaScript/TypeScript with syntax highlighting, auto-completion, and inline error detection.
- [ ] **[Frontend]** Implement auto-save: debounced saves of gameScreenCode and controllerScreenCode to the backend via `PATCH /api/v1/games/:id`.
- [ ] **[Frontend]** Build the Game Screen preview panel: render the game screen canvas using p5.js in a sandboxed iframe.
- [ ] **[Frontend]** Build the Controller Screen preview panel: simulated phone frame rendering the controller canvas for testing without a second device.
- [ ] **[Frontend]** Implement the game runtime sandbox: isolate game code execution, prevent access to browser APIs outside the canvas, block external network requests, and provide the platform API (player list, input events, state broadcast).
- [ ] **[Frontend]** Build the platform runtime API that games use to interact with session data: `getPlayers()`, `onPlayerInput()`, `broadcastState()`, `onStateUpdate()`.
- [ ] **[Frontend]** Build the "Test Session" button: launch a private session from the studio using draft code (not published version), display session code and QR code for live testing with real devices.
- [ ] **[Frontend]** Create Zustand editor store for managing editor state: active file, unsaved changes, preview mode, and split layout preferences.
- [ ] **[Frontend]** Build asset browser panel within the studio: list game assets, drag-to-insert asset URL into code.
- [ ] **[Test]** Write tests for runtime sandbox isolation: verify games cannot access forbidden APIs, external networks, or DOM outside canvas.
- [ ] **[Test]** Write tests for auto-save and editor state management.

---

### M0.6.0: Sessions & Real-Time

**Goal:** Implement the session system with WebSocket-based real-time communication, enabling the full session lifecycle: lobby, game loading, playing, pausing, and ending.

- [ ] **[Database]** Create `Session` table migration with all fields: id, hostId, gameId, gameVersionId, sessionCode, status, maxPlayers, createdAt, updatedAt, endedAt.
- [ ] **[Database]** Create `Player` table migration with all fields: id, sessionId, userId, displayName, avatarUrl, connectionStatus, leftAt, createdAt.
- [ ] **[Backend]** Implement SeaORM entity models for `Session` and `Player`.
- [ ] **[Backend]** Implement session code generation: 4-6 uppercase alphanumeric characters excluding ambiguous characters (0/O, 1/I/L), unique among active sessions, case-insensitive lookup.
- [ ] **[Backend]** Implement `POST /api/v1/sessions` - create a session in lobby status with generated session code. Requires authenticated user as host (hostId is required). Enforce max player limits based on host subscription tier.
- [ ] **[Backend]** Implement `GET /api/v1/sessions/:id` - return session details including player list and loaded game info.
- [ ] **[Backend]** Implement `PATCH /api/v1/sessions/:id` - update session settings (maxPlayers). Host only.
- [ ] **[Backend]** Implement `DELETE /api/v1/sessions/{sessionId}` - set status to ended, set endedAt, close all WebSocket connections. Host only.
- [ ] **[Backend]** Implement `POST /api/v1/sessions/{sessionId}/game` - load a published game into the session. Validate game is published with a publishedVersionId. Set gameId and gameVersionId. Transition to playing.
- [ ] **[Backend]** Implement `DELETE /api/v1/sessions/{sessionId}/game` - unload game, clear gameId/gameVersionId, return to lobby. Players stay connected.
- [ ] **[Backend]** Implement `POST /api/v1/sessions/:id/pause` and `POST /api/v1/sessions/:id/resume` - toggle between playing and paused states.
- [ ] **[Backend]** Implement `POST /api/v1/sessions/:id/restart` - restart the current game from the beginning.
- [ ] **[Backend]** Implement `POST /api/v1/sessions/{sessionCode}/join` - join a session by session code in path. Validate session exists, is not ended, and has not reached maxPlayers. Create Player record. Support anonymous join (userId nullable).
- [ ] **[Backend]** Implement `GET /api/v1/sessions/:id/players` - list all players in a session.
- [ ] **[Backend]** Implement `DELETE /api/v1/sessions/:id/players/:playerId` - host removes a player from the session.
- [ ] **[Backend]** Implement `PATCH /api/v1/sessions/{sessionId}/players/me` - allow a player to update their own displayName or avatar within a session.
- [ ] **[Backend]** Implement `GET /api/v1/users/me/sessions` - list sessions hosted by the authenticated user.
- [ ] **[Backend]** Implement WebSocket endpoint `WS /api/v1/sessions/{sessionId}/ws` - establish WebSocket connection for a client (host or player). Accepts `role`, `playerId`, and optional `token` as query parameters.
- [ ] **[Backend]** Implement WebSocket message routing: scope all messages to the session, relay player input (controller -> game screen), relay game state updates (game screen -> controllers), handle session control events (join, leave, load, pause, resume, end).
- [ ] **[Backend]** Implement player connection lifecycle: track connectionStatus, handle disconnect with reconnection grace period, restore state on reconnect, free slot on timeout or voluntary leave.
- [ ] **[Backend]** Implement session timeout: automatically end idle sessions after configurable inactivity period.
- [ ] **[Frontend]** Build session creation UI on the Console (big screen) experience.
- [ ] **[Frontend]** Build lobby screen: display session code prominently, show QR code (encoding join URL), render connected player list with avatars and names in real time.
- [ ] **[Frontend]** Create Zustand session store for managing session state, player list, and WebSocket connection.
- [ ] **[Frontend]** Implement WebSocket client: connect, reconnect on drop, handle all message types, update session store.
- [ ] **[Frontend]** Build game loading flow on Console: browse library from lobby, select game, display loading state, render Game Screen canvas when loaded.
- [ ] **[Frontend]** Build host controls overlay: pause, resume, restart, unload game, end session, remove player.
- [ ] **[Test]** Write backend tests for session CRUD, join flow (including max player enforcement), and all session state transitions.
- [ ] **[Test]** Write backend tests for WebSocket message routing, player connection lifecycle, and reconnection grace period.

---

### M0.7.0: Controller System

**Goal:** Build the smartphone controller experience: session join flow, dynamic controller rendering, and bidirectional input/state relay.

- [ ] **[Frontend]** Build the Controller join page: input field for session code, QR code scanner integration, and join button.
- [ ] **[Frontend]** Build the player setup screen: choose display name and optional avatar before entering the lobby.
- [ ] **[Frontend]** Build the Controller lobby view: show session info, connected player list, and "waiting for host" status.
- [ ] **[Frontend]** Build the Controller game view: render the Controller Screen canvas using p5.js in a sandboxed iframe, receiving controllerScreenCode from the loaded GameVersion.
- [ ] **[Frontend]** Implement touch input capture and relay: detect touch events on the controller canvas, serialize as player input messages, send to server via WebSocket.
- [ ] **[Frontend]** Implement device orientation/gyroscope input: capture tilt and motion data for games that use accelerometer-based controls (e.g., steering).
- [ ] **[Frontend]** Implement game state reception: receive state updates from the Game Screen via WebSocket and inject into the Controller Screen canvas runtime.
- [ ] **[Frontend]** Build the Controller paused/ended screens: display appropriate status messages when the host pauses or ends the session.
- [ ] **[Frontend]** Implement controller reconnection UX: detect WebSocket drops, show reconnecting indicator, auto-reconnect within grace period, restore game state on success.
- [ ] **[Frontend]** Optimize the controller experience for mobile: responsive layout, viewport meta tags, prevent zoom/scroll, full-screen mode, and touch-friendly UI sizing.
- [ ] **[Test]** Write tests for controller join flow, input capture and relay, state synchronization, and reconnection behavior.

---

## Phase 3: Community & Discovery

### M0.8.0: Discovery & Browse

**Goal:** Build the game library experience with browsing, searching, filtering, sorting, and game detail pages so players can discover and launch games.

- [ ] **[Backend]** Implement `GET /api/v1/library/games` - paginated game listing filtered to published + public games. Support query parameters: tags (multi-select), technology, minPlayers, maxPlayers, and sort (popular, top-rated, newest, trending).
- [ ] **[Backend]** Implement `GET /api/v1/library/search?q=` - text search across game title, description, creator username, and tag names. Rank by relevance.
- [ ] **[Backend]** Implement `GET /api/v1/library/trending` - games ordered by recent play count growth (e.g., plays in last 7 days).
- [ ] **[Backend]** Implement `GET /api/v1/library/featured` - return games from active featured collections.
- [ ] **[Frontend]** Build game library page with grid/list display of game cards (thumbnail, title, creator, player count, rating, play count).
- [ ] **[Frontend]** Build filter sidebar/bar: tag selection by category (genre, mood, playerStyle), player count range, technology toggle. Apply filters as query parameters.
- [ ] **[Frontend]** Build sort controls: Popular, Top Rated, Newest, Trending.
- [ ] **[Frontend]** Build search bar with live results and search results page.
- [ ] **[Frontend]** Build game detail page: title, description, thumbnail, creator profile link, player count range, tags, average rating, review count, play count, total play time, version history, reviews list, "forked from" attribution, and Play button.
- [ ] **[Frontend]** Build trending section on library home page.
- [ ] **[Frontend]** Build featured collections section on library home page with cover images and links.
- [ ] **[Test]** Write backend tests for browse (filtering, sorting, pagination), search (relevance ranking), trending algorithm, and featured endpoint.

---

### M0.9.0: Reviews & Favorites

**Goal:** Enable the community feedback loop: players can rate and review games, and bookmark their favorites.

- [ ] **[Database]** Create `Review` table migration with unique constraint on (userId, gameId).
- [ ] **[Database]** Create `Favorite` table migration with unique constraint on (userId, gameId).
- [ ] **[Backend]** Implement SeaORM entity models for `Review` and `Favorite`.
- [ ] **[Backend]** Implement `PUT /api/v1/games/:id/reviews` - create or update a review (rating 1-5, optional title and body). Recalculate Game.avgRating and Game.reviewCount.
- [ ] **[Backend]** Implement `GET /api/v1/games/:id/reviews` - paginated list of reviews for a game, sorted by most recent. Exclude soft-deleted reviews.
- [ ] **[Backend]** Implement `GET /api/v1/games/:id/reviews/me` - get the authenticated user's review for a specific game.
- [ ] **[Backend]** Implement `DELETE /api/v1/games/:id/reviews/me` - soft-delete own review. Recalculate cached aggregates.
- [ ] **[Backend]** Implement `POST /api/v1/games/:id/favorite` - toggle favorite (add if not favorited, remove if already favorited).
- [ ] **[Backend]** Implement `GET /api/v1/games/:id/favorite` - check if the authenticated user has favorited a game.
- [ ] **[Backend]** Implement `GET /api/v1/users/me/favorites` - paginated list of the user's favorited games.
- [ ] **[Backend]** Implement `GET /api/v1/games/:id/rating` - rating summary (distribution of 1-5 stars, average, total count).
- [ ] **[Frontend]** Build review submission form on game detail page: star rating selector, optional title and body fields.
- [ ] **[Frontend]** Build reviews list on game detail page with pagination, showing star rating, title, body, author, and date.
- [ ] **[Frontend]** Build edit/delete controls for the user's own review.
- [ ] **[Frontend]** Build favorite toggle button (heart icon) on game cards and game detail page.
- [ ] **[Frontend]** Build "My Favorites" page listing all favorited games.
- [ ] **[Frontend]** Build rating summary display on game detail page: star distribution bar chart and average.
- [ ] **[Test]** Write backend tests for review CRUD (create, update, delete, uniqueness constraint), aggregate recalculation, favorite toggle, and listing.

---

### M0.10.0: Collections & Templates

**Goal:** Implement curated game collections for featured content and starter templates for new creators.

- [ ] **[Database]** Create `Template` table migration with all fields.
- [ ] **[Database]** Create `Collection` table migration with all fields.
- [ ] **[Database]** Create `CollectionGame` join table migration with composite primary key (collectionId, gameId) and sortOrder.
- [ ] **[Backend]** Implement SeaORM entity models for `Template`, `Collection`, and `CollectionGame`.
- [ ] **[Backend]** Implement `GET /api/v1/templates` - list all templates, optionally filtered by technology and difficulty. Ordered by sortOrder.
- [ ] **[Backend]** Implement `GET /api/v1/templates/:id` - get a single template with full code.
- [ ] **[Backend]** Implement `POST /api/v1/templates/:id/use` - create a new Game from a template, copying gameScreenCode and controllerScreenCode into the new game.
- [ ] **[Backend]** Implement `GET /api/v1/collections` - list active collections ordered by sortOrder (public endpoint).
- [ ] **[Backend]** Implement `GET /api/v1/collections/:id` - get collection details with its games (paginated, ordered by CollectionGame.sortOrder).
- [ ] **[Frontend]** Build template browser in the Creative Studio: grid of templates with thumbnails, titles, descriptions, difficulty badges, and technology labels.
- [ ] **[Frontend]** Build template detail view with code preview and "Use This Template" button.
- [ ] **[Frontend]** Build collection display on library page: collection cards with cover images, titles, and descriptions.
- [ ] **[Frontend]** Build collection detail page: header with description and cover image, followed by the collection's game list.
- [ ] **[Test]** Write backend tests for template listing, template-to-game creation, and collection endpoints.

---

### M0.11.0: Play History & Creator Statistics

**Goal:** Track gameplay activity and surface statistics to players and creators.

- [ ] **[Database]** Create `PlayHistory` table migration with all fields.
- [ ] **[Backend]** Implement SeaORM entity model for `PlayHistory`.
- [ ] **[Backend]** Create `PlayHistory` records automatically when a game starts in a session - one record per connected player. Update duration when the game ends.
- [ ] **[Backend]** Update cached aggregates on `Game` entity (playCount, totalPlayTime) when a game session ends.
- [ ] **[Backend]** Implement `GET /api/v1/users/me/history` - paginated list of the user's play history, sorted by most recent. Each entry includes game info, date played, and duration.
- [ ] **[Backend]** Implement `GET /api/v1/games/:id/stats` - game statistics: total play count, total play time, average session duration, rating info. Creator only.
- [ ] **[Backend]** Implement `GET /api/v1/users/me/stats` - creator statistics: total play count and play time across all games, per-game breakdown, rating distribution.
- [ ] **[Frontend]** Build "Recently Played" section on the user's dashboard showing recent play history entries.
- [ ] **[Frontend]** Build full play history page with game thumbnails, dates, and durations.
- [ ] **[Frontend]** Build creator statistics dashboard: total plays, total play time, per-game stats table, rating distributions, and play trend charts.
- [ ] **[Frontend]** Build game statistics section on the game detail page for the game owner.
- [ ] **[Test]** Write backend tests for PlayHistory creation/update lifecycle, aggregate recalculation, and statistics endpoints.

---

## Phase 4: Production Readiness

### M0.12.0: Administration & Moderation

**Goal:** Build the admin panel and moderation tools for managing users, content, collections, templates, and platform analytics.

- [ ] **[Backend]** Implement `GET /api/v1/admin/users` - paginated list of all users with filtering by role, status, and search by username/email. Admin only.
- [ ] **[Backend]** Implement `GET /api/v1/admin/users/:id` - get full user details including auth providers and account status. Admin only.
- [ ] **[Backend]** Implement `PATCH /api/v1/admin/users/:id` - update user role (user/moderator/admin) and other admin-editable fields. Admin only.
- [ ] **[Backend]** Implement `POST /api/v1/admin/users/:id/suspend` and `POST /api/v1/admin/users/:id/restore` - suspend and restore user accounts. Admin only.
- [ ] **[Backend]** Implement `POST /api/v1/admin/games/:id/archive` - moderator/admin archives a game that violates guidelines.
- [ ] **[Backend]** Implement `POST /api/v1/admin/games/:id/restore` - admin restores a previously archived game.
- [ ] **[Backend]** Implement `DELETE /api/v1/games/:gameId/reviews/:reviewId` - moderator/admin soft-deletes a review that violates guidelines.
- [ ] **[Backend]** Implement `POST /api/v1/admin/collections` - create a new collection. Moderator or Admin.
- [ ] **[Backend]** Implement `PATCH /api/v1/admin/collections/:id` - update collection title, description, coverImageUrl, sortOrder, active status. Moderator or Admin.
- [ ] **[Backend]** Implement `DELETE /api/v1/admin/collections/:id` - delete a collection. Admin only.
- [ ] **[Backend]** Implement `PUT /api/v1/admin/collections/:id/games` - set the games in a collection with their sort order. Moderator or Admin.
- [ ] **[Backend]** Implement `POST /api/v1/admin/templates` - create a new template. Admin only.
- [ ] **[Backend]** Implement `PATCH /api/v1/admin/templates/:id` - update template content, metadata, difficulty, and sortOrder. Admin only.
- [ ] **[Backend]** Implement `DELETE /api/v1/admin/templates/:id` - delete a template. Admin only.
- [ ] **[Backend]** Implement `GET /api/v1/admin/analytics` - platform analytics: total users, games, sessions, active users (daily/weekly/monthly), top games by plays, recent signups.
- [ ] **[Frontend]** Build admin dashboard page with platform analytics overview.
- [ ] **[Frontend]** Build admin user management page: user list with search/filter, user detail view, role editing, suspend/restore actions.
- [ ] **[Frontend]** Build admin game moderation page: list reported or flagged games, archive/restore actions.
- [ ] **[Frontend]** Build admin collection management page: create, edit, reorder collections, assign games with drag-and-drop ordering.
- [ ] **[Frontend]** Build admin template management page: create, edit, delete, and reorder templates.
- [ ] **[Frontend]** Build moderator review management: flag and soft-delete inappropriate reviews.
- [ ] **[Frontend]** Add role-based navigation: show admin/moderator sections only to users with appropriate roles.
- [ ] **[Test]** Write backend tests for all admin endpoints: user management, game moderation, collection CRUD, template CRUD, and analytics.

---

### M0.13.0: Subscriptions & Tiers

**Goal:** Implement the freemium business model by enforcing free and pro tier differences across the platform.

- [ ] **[Backend]** Build subscription checking middleware that reads subscriptionPlan and subscriptionExpiresAt from the authenticated user and determines the effective tier (revert to free if expired).
- [ ] **[Backend]** Enforce max player limits at session join based on host subscription tier: free tier gets a platform-defined limited player count, pro tier gets unlimited (up to system capacity).
- [ ] **[Backend]** Enforce max player limits at session creation: set Session.maxPlayers based on host tier.
- [ ] **[Backend]** Implement intermission logic: inject periodic pause events into free-tier sessions at server-controlled intervals. Skip intermissions for pro-tier sessions.
- [ ] **[Backend]** Implement game library access gating: limit the number of games accessible to free-tier users in browse/search results, or mark pro-only games.
- [ ] **[Backend]** Implement creation tool gating: track which advanced studio features require pro tier and enforce at the API level.
- [ ] **[Frontend]** Build subscription status display in user settings: current plan, expiration date, and upgrade prompt.
- [ ] **[Frontend]** Build upgrade prompts at enforcement points: when hitting player limits, during intermissions, and when accessing pro-gated features.
- [ ] **[Frontend]** Build intermission overlay for the Console: display intermission screen during free-tier session pauses.
- [ ] **[Frontend]** Gate advanced creation tools in the Creative Studio based on the user's subscription tier with upgrade prompts.
- [ ] **[Test]** Write backend tests for subscription enforcement: player limits per tier, intermission injection, library gating, and tool access gating.

---

### M1.0.0: Launch Preparation

**Goal:** Harden the platform for production launch with performance optimization, security review, error handling polish, monitoring, and final deployment configuration.

- [ ] **[Shared]** Conduct a full security review: audit authentication flows, input validation, SQL injection prevention (SeaORM parameterized queries), XSS protection, CORS policy, rate limiting, and file upload validation.
- [ ] **[Shared]** Implement rate limiting on all public endpoints (auth, join session, search) to prevent abuse.
- [ ] **[Shared]** Configure production environment variables, secrets management, and Railway production deployment settings.
- [ ] **[Shared]** Set up monitoring and alerting: application health checks, error tracking, and uptime monitoring.
- [ ] **[Backend]** Optimize database queries: add indexes on frequently queried columns (Game.status + visibility, Session.sessionCode + status, GameTag.gameId, PlayHistory.userId, Review.gameId).
- [ ] **[Backend]** Implement request logging and structured error reporting for production debugging.
- [ ] **[Backend]** Review and enforce all platform constraints: game code size limits, asset upload size limits, assets per game limit, username/password/bio character limits, review text length limits.
- [ ] **[Backend]** Implement graceful shutdown: drain active WebSocket connections, finish in-progress requests, close database pool.
- [ ] **[Backend]** Implement session cleanup job: automatically end idle sessions after timeout, recycle expired session codes.
- [ ] **[Frontend]** Optimize bundle size: code splitting, lazy loading for Studio/Console/Controller routes, and tree shaking.
- [ ] **[Frontend]** Implement comprehensive error boundaries and user-facing error messages for all failure scenarios (network errors, session expired, game load failures).
- [ ] **[Frontend]** Optimize WebSocket reconnection strategy with exponential backoff.
- [ ] **[Frontend]** Add loading states and skeleton screens for all data-fetching pages.
- [ ] **[Frontend]** Ensure responsive design across all target devices: desktop (Studio, Console), tablet (Console), and mobile (Controller, general browsing).
- [ ] **[Frontend]** Test cross-browser compatibility: Chrome, Firefox, Safari, Edge, and major mobile browsers.
- [ ] **[Test]** Run full end-to-end test suite covering the critical user journeys: sign up, create game, publish, host session, join session, play game, leave review.
- [ ] **[Test]** Perform load testing on WebSocket connections: validate concurrent session and player capacity.
- [ ] **[Test]** Perform security testing: automated vulnerability scanning and manual penetration testing of auth flows and file uploads.
