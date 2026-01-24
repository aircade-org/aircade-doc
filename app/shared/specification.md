# AirCade - Specification

This document defines the functional and technical specification for the AirCade platform. It describes **what** the system does and the rules governing its behavior. Implementation details such as API endpoint definitions, database queries, and UI layouts are covered in their respective documents.

---

## 1. Authentication & Authorization

### 1.1 Sign-Up Methods

The platform supports three sign-up methods. A user account is created the first time a person authenticates through any of them.

| Method             | Flow                                                                                                                                                                                                                                               |
|:-------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Email/Password** | User submits email and password. The system creates a `User` record and an `AuthProvider` record with `provider: "email"`. A verification email is sent. Password is hashed before storage.                                                        |
| **Google OAuth**   | User authenticates with Google via OAuth 2.0. On first sign-in, the system creates a `User` and an `AuthProvider` with `provider: "google"` and the Google account ID as `providerId`. Profile data (email, name, avatar) is imported from Google. |
| **GitHub OAuth**   | Same flow as Google but using GitHub as the OAuth provider with `provider: "github"`.                                                                                                                                                              |

### 1.2 Sign-In

A returning user authenticates through any provider linked to their account:

- **Email/Password:** User submits credentials. The system verifies the password hash against the stored `AuthProvider` record.
- **Google/GitHub OAuth:** User completes the OAuth flow. The system matches the `providerId` to an existing `AuthProvider` record and returns the associated `User`.

On successful sign-in, the system updates `User.lastLoginAt` and `User.lastLoginIp`.

### 1.3 Account Linking

A signed-in user can link additional auth providers to their account. The system creates a new `AuthProvider` record pointing to the same `User`. Each provider can only be linked once per user (enforced by the unique constraint on `userId` + `provider`). A provider ID that is already linked to a different user cannot be re-linked.

### 1.3.1 OAuth Email Conflict Resolution

If a user attempts to sign in via Google or GitHub OAuth and the OAuth provider's email matches an existing AirCade account registered through a different method, the system returns a `409 Conflict` error. The response message instructs the user to:

1. Sign in using their existing account (email/password or the originally linked OAuth provider).
2. Navigate to account settings and manually link the new OAuth provider.

The system does **not** auto-link providers or auto-merge accounts. This prevents unauthorized account takeover if someone gains access to an OAuth account with a matching email address.

### 1.4 Email Verification

- When a user signs up with email/password, `User.emailVerified` is set to `false`.
- A verification token is stored in `AuthProvider.verificationToken` with an expiration in `AuthProvider.tokenExpiresAt`.
- The user receives an email containing a verification link.
- When the link is followed and the token is valid and not expired, `User.emailVerified` is set to `true` and the token is cleared.
- Users who sign up via Google or GitHub have `emailVerified` set to `true` immediately (the provider has already verified the email).

### 1.5 Password Reset

- A user with an email/password provider can request a password reset.
- The system generates a reset token, stores it in `AuthProvider.verificationToken` with an expiration, and sends an email.
- The user follows the link, submits a new password, and the system updates `AuthProvider.passwordHash` and clears the token.
- The token is single-use and expires after the time defined in `AuthProvider.tokenExpiresAt`.

### 1.6 Token Lifecycle

- **Access tokens** are short-lived JWTs (15 minutes) containing the user's ID and role. They are stateless — the server validates them using the JWT secret without a database lookup.
- **Refresh tokens** are longer-lived JWTs (7 days). They are stored in a server-side allowlist (in the database) and are invalidated on sign-out. Each refresh returns a new access token and a new refresh token (rotation), and the previous refresh token is invalidated.
- **Verification tokens** (email verification, password reset) are single-use, random strings stored in `AuthProvider.verificationToken` with an expiration of 24 hours in `AuthProvider.tokenExpiresAt`.

### 1.7 Authorization Rules

| Role          | Capabilities                                                                                                                                             |
|:--------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Guest**     | Join sessions as a player (no account required). Browse public game library. View game details.                                                          |
| **User**      | Everything a guest can do, plus: create games, publish games (requires verified email), host sessions, leave reviews, favorite games, view play history. |
| **Moderator** | Everything a user can do, plus: manage reported content, remove inappropriate reviews, archive games that violate guidelines.                            |
| **Admin**     | Full platform access: manage users (suspend, deactivate), manage collections and templates, assign moderator roles, view platform analytics.             |

### 1.8 Account Status

| Status          | Effect                                                                                                                                                     |
|:----------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------|
| **active**      | Full access based on role.                                                                                                                                 |
| **suspended**   | Cannot sign in. Existing sessions are not interrupted but no new sessions or content can be created. Published games remain visible but cannot be updated. |
| **deactivated** | Cannot sign in. Profile and published games are hidden from other users. Data is retained for potential reactivation.                                      |

---

## 2. User Profile

### 2.1 Profile Fields

Users manage the following profile information:

- **username** - Unique, public. Used in URLs and @mentions. Cannot be changed to a username currently held by another user. Changed via a dedicated endpoint.
- **displayName** - Optional friendly name shown in lobbies and on the creator profile page.
- **email** - Unique. Used for sign-in and notifications. Changing the email requires re-verification. For users with an email auth provider, the current password is required for confirmation. Users who only have OAuth providers linked can change their email without a password.
- **avatarUrl** - Profile picture. Users can upload an image or use a default generated avatar.
- **bio** - Short free-text description visible on the user's public profile.

> **OAuth-only accounts:** Users who signed up exclusively via Google or GitHub and have no email/password auth provider can perform sensitive actions (email change, account deactivation) without a password. The system verifies their identity through the active authenticated session. If higher security is needed, they should first link an email/password provider to their account.

### 2.2 Public Profile

Every user with at least one published game has a public profile page showing:

- Username, display name, avatar, and bio.
- Published games (sorted by most recent).
- Aggregate statistics: total games published, total play count across all games.

> **Note:** Users who have not published any games do not have a public profile. The public profile endpoint returns `404` for users with no published games, deactivated accounts, or non-existent usernames. This is intentional — public profiles are a creator-facing feature, and having published content is a prerequisite for visibility.

---

## 3. Game Management

### 3.1 Game Lifecycle

A game project progresses through the following states:

```
draft ──► published ──► archived
  ▲            │
  └────────────┘
```

| State         | Description                                                                                                                                                                       |
|:--------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **draft**     | The game is being developed in the Creative Studio. Only the creator can see and test it.                                                                                         |
| **published** | The game is live in the library. Players can discover and play it. The creator can continue editing the draft and publish updates.                                                |
| **archived**  | The game is removed from public discovery. Existing direct links still work but the game cannot be loaded into new sessions. The creator can return it to `draft` or `published`. |

**Allowed state transitions:**

| From        | To          | Trigger                                                                               |
|:------------|:------------|:--------------------------------------------------------------------------------------|
| `draft`     | `published` | Creator publishes the game (creates a `GameVersion`).                                 |
| `published` | `archived`  | Creator or moderator archives the game.                                               |
| `published` | `draft`     | Not allowed. A published game cannot revert to draft. Archive first if needed.        |
| `archived`  | `published` | Creator unarchives a game that has a `publishedVersionId` (was previously published). |
| `archived`  | `draft`     | Creator unarchives a game that has no `publishedVersionId` (was never published).     |

### 3.2 Visibility

Visibility is independent of lifecycle state and controls who can discover the game:

| Visibility   | Behavior                                                                                         |
|:-------------|:-------------------------------------------------------------------------------------------------|
| **public**   | Appears in the game library, search results, and collections. Anyone can play it.                |
| **private**  | Only visible to the creator. Can still be loaded in sessions the creator hosts (for testing).    |
| **unlisted** | Not discoverable in the library or search, but anyone with a direct link can access and play it. |

A game must be `published` **and** `public` to appear in the community library.

### 3.3 Publishing

Publishing creates an immutable `GameVersion` snapshot:

1. The system validates that the game has a title, at least one non-empty canvas code (game screen or controller screen), and that the creator's email is verified.
2. The `versionNumber` is incremented (starting from 1).
3. The current `gameScreenCode` and `controllerScreenCode` are copied into a new `GameVersion` record.
4. `Game.publishedVersionId` is updated to point to the new version.
5. `Game.status` is set to `published` (if not already).

The draft code in the `Game` record remains editable. The published version is frozen. When a session loads a game, it always uses the code from the `GameVersion` referenced by `publishedVersionId`.

### 3.4 Versioning

- Each publish creates a new `GameVersion` with a sequential `versionNumber`.
- Previous versions are retained for reference.
- The `publishedVersionId` on the `Game` entity always points to the most recent published version.
- Sessions that are already running continue to use the version they loaded at start.

### 3.5 Remixing and Forking

If `Game.remixable` is `true`, any authenticated user can fork the game:

1. A new `Game` record is created with the forking user as `creatorId`.
2. `forkedFromId` is set to the original game's ID.
3. The game screen code and controller screen code are copied from the source game's published version.
4. The new game starts in `draft` status with `private` visibility.

The original creator retains full credit. The forked game's detail page shows attribution to the original.

### 3.6 Game Assets

Creators can upload media files (images, sounds, sprites, fonts) to use in their games:

- Each asset is linked to a single game via `GameAsset.gameId`.
- Supported file types are validated on upload (images: PNG, JPG, SVG, GIF; audio: MP3, WAV, OGG; fonts: TTF, WOFF2).
- Assets are stored in the database and served via the API at their `storageUrl` path. Asset files for published games are publicly accessible (no authentication required) so that game code running in any player's browser can load them at runtime. The asset management endpoints (list, upload, delete) remain creator-only.
- Deleting a game soft-deletes its assets. Assets are not independently versioned.

### 3.7 Tags

Games are categorized using tags from three categories:

| Category        | Purpose                         | Examples                                   |
|:----------------|:--------------------------------|:-------------------------------------------|
| **genre**       | The game's genre classification | Racing, Trivia, Drawing, Strategy          |
| **mood**        | The atmosphere or energy level  | Competitive, Cooperative, Relaxed, Chaotic |
| **playerStyle** | How players interact            | Free-for-all, Teams, Turn-based, Real-time |

Tags are platform-managed (users cannot create arbitrary tags). Creators select tags when setting up their game metadata. A game can have multiple tags.

---

## 4. Game Runtime & Dual-Canvas Architecture

### 4.1 Dual-Canvas Model

Every AirCade game consists of two synchronized canvases:

| Canvas                | Runs On                                         | Purpose                                                                                                          |
|:----------------------|:------------------------------------------------|:-----------------------------------------------------------------------------------------------------------------|
| **Game Screen**       | Big screen (TV, laptop, projector, car display) | Displays the shared game world visible to all players: game board, arena, scores, animations.                    |
| **Controller Screen** | Each player's smartphone                        | Displays private controls (buttons, joysticks, sliders) and private information (cards, roles, personal scores). |

Both canvases execute in the browser. They share game state through the platform's real-time communication layer.

### 4.2 Game Code Structure

A game's code defines behavior for both canvases. Each canvas has two phases:

| Phase      | Purpose                                                                                  |
|:-----------|:-----------------------------------------------------------------------------------------|
| **Setup**  | Runs once when the game loads. Initializes the canvas, loads assets, sets initial state. |
| **Update** | Runs continuously (each frame). Handles rendering, input processing, and game logic.     |

The creator writes the setup and update logic for both the Game Screen and the Controller Screen within a single project.

### 4.3 Supported Technologies

| Technology | Status    | Description                                                                                                  |
|:-----------|:----------|:-------------------------------------------------------------------------------------------------------------|
| **p5js**   | Available | 2D creative coding library. JavaScript/TypeScript. Handles drawing, animation, and input. Launch technology. |
| **3d**     | Planned   | Browser-based 3D rendering engine for spatial game environments.                                             |
| **visual** | Planned   | Visual game builder with graphical editor and scripting. Lower barrier to entry.                             |

All technologies integrate into the same dual-canvas model.

> **Validation:** At launch, the API only accepts `"p5js"` as the `technology` value. Requests with `"3d"` or `"visual"` are rejected with a `400` validation error. The schema stores these as a string to allow future expansion without migrations.

### 4.4 Game State Synchronization

Game state is synchronized between the Game Screen and Controller Screens through the server's WebSocket layer:

- **Player input** flows from Controller Screen → Server → Game Screen.
- **Game state updates** flow from Game Screen → Server → Controller Screens.
- The server acts as a relay within a session. It does not interpret or validate game logic - the game code running in the browsers handles all game rules.
- The server ensures that messages are routed only within the session they belong to.

### 4.5 Game Runtime API

The platform injects a runtime API into the game's sandboxed execution context. This API is the only way game code interacts with session data and other players. The API is available as a global `AirCade` object in both canvases.

#### Game Screen API (available in Game Screen code)

| Function / Property                    | Description                                                                                                  |
|:---------------------------------------|:-------------------------------------------------------------------------------------------------------------|
| `AirCade.getPlayers()`                 | Returns an array of currently connected players: `[{ id, displayName, avatarUrl }]`.                         |
| `AirCade.onPlayerJoin(callback)`       | Registers a callback invoked when a player joins. Callback receives the player object.                       |
| `AirCade.onPlayerLeave(callback)`      | Registers a callback invoked when a player leaves. Callback receives the player ID.                          |
| `AirCade.onPlayerInput(callback)`      | Registers a callback invoked when any player sends input. Callback receives `{ playerId, inputType, data }`. |
| `AirCade.broadcastState(state)`        | Sends a game state object to all connected Controller Screens. `state` is any JSON-serializable object.      |
| `AirCade.sendToPlayer(playerId, data)` | Sends data to a specific player's Controller Screen.                                                         |
| `AirCade.getSessionInfo()`             | Returns session metadata: `{ sessionId, sessionCode, status, maxPlayers }`.                                  |

#### Controller Screen API (available in Controller Screen code)

| Function / Property                  | Description                                                                                                                     |
|:-------------------------------------|:--------------------------------------------------------------------------------------------------------------------------------|
| `AirCade.getPlayer()`                | Returns the current player's info: `{ id, displayName, avatarUrl }`.                                                            |
| `AirCade.sendInput(inputType, data)` | Sends input to the Game Screen. `inputType` is a string label (e.g., `"button_press"`), `data` is any JSON-serializable object. |
| `AirCade.onStateUpdate(callback)`    | Registers a callback invoked when the Game Screen broadcasts state. Callback receives the state object.                         |
| `AirCade.onPrivateMessage(callback)` | Registers a callback invoked when the Game Screen sends data to this specific player.                                           |
| `AirCade.setReady(ready)`            | Signals that this player is ready (or not). The Game Screen receives this via `onPlayerInput` with `inputType: "player_ready"`. |
| `AirCade.getSessionInfo()`           | Returns session metadata: `{ sessionId, sessionCode, status, maxPlayers }`.                                                     |

#### Common Rules

- All data passed through the runtime API must be JSON-serializable.
- The runtime API is the sole communication channel between canvases. Direct WebSocket access is not exposed to game code.
- Callbacks are invoked asynchronously. Game code should not rely on synchronous responses.
- The `AirCade` object is injected by the platform before the game's setup phase runs.

### 4.6 Runtime Isolation

- Each game runs in a sandboxed execution context within the browser (sandboxed iframe).
- Games cannot access browser APIs outside the platform-provided runtime (DOM manipulation outside the canvas, localStorage, navigation, etc.).
- Games cannot communicate with external servers. All communication goes through the platform's WebSocket layer via the `AirCade` runtime API.
- The platform provides the `AirCade` global object (described in 4.5) as the controlled API for games to interact with session data.

---

## 5. Sessions & Real-Time Communication

### 5.1 Session Lifecycle

```
             ┌──────────────────────┐
             ▼                      │
lobby ──► playing ──► paused ──► playing ──► ended
  ▲          │                                 ▲
  │          │                                 │
  └──────────┘ (unload game / game ends)       │
  │                                            │
  └────────────────────────────────────────────┘
        (host ends session from any state)
```

| State       | Description                                                                                                                      |
|:------------|:---------------------------------------------------------------------------------------------------------------------------------|
| **lobby**   | Session is active. Players can join. No game is loaded. The big screen shows the connection code and player list.                |
| **playing** | A game is loaded and running. Player input is being processed. New players may still join if the game and player count allow it. |
| **paused**  | The game is temporarily halted by the host. Players remain connected. No input is processed.                                     |
| **ended**   | The session is over. All connections are closed. The session record is retained for history.                                     |

### 5.2 Session Creation

1. A host opens AirCade on a big screen device.
2. The system creates a `Session` record with status `lobby`.
3. A unique `sessionCode` is generated - a short, human-readable alphanumeric string (e.g., `XKCD42`). The code is unique among all currently active (non-ended) sessions.
4. The big screen displays the session code and a QR code that encodes the join URL with the session code.
5. The host must be a signed-in user. `Session.hostId` is set to their user ID. All session management endpoints (game loading, pausing, player removal, etc.) require authentication as the host user. Anonymous session creation is not supported — a user account is required to host sessions.

### 5.3 Joining a Session

1. A player opens AirCade on their smartphone.
2. The player enters the session code or scans the QR code.
3. The system validates that the session exists, is not ended, and has not reached `maxPlayers`.
4. A `Player` record is created with `connectionStatus: "connected"`.
5. The player chooses a display name (and optionally an avatar). If the player is signed in, `Player.userId` is set; otherwise it remains null.
6. A WebSocket connection is established between the player's device and the server.
7. The big screen lobby updates in real time to show the new player.

### 5.4 Session Code Rules

- Codes consist of uppercase letters and digits, excluding visually ambiguous characters (0/O, 1/I/L).
- Codes are 4–6 characters long, providing sufficient uniqueness for concurrent sessions.
- A code is valid only for the lifetime of its session. Once a session ends, its code can be reused by a future session after a 5-minute cooldown period to prevent confusion with recently-ended sessions.
- Codes are case-insensitive on input.

### 5.4.1 Session Identifier Design

The API uses two different identifiers for sessions depending on context:

- **`sessionCode`** (human-readable, e.g., `XKCD42`) — Used in player-facing endpoints where the user knows the code: `GET /sessions/{sessionCode}` (lookup) and `POST /sessions/{sessionCode}/join` (join).
- **`sessionId`** (UUID) — Used in host-facing management endpoints where the host received the UUID at session creation: `PATCH /sessions/{sessionId}`, `DELETE /sessions/{sessionId}`, game load/unload, pause/resume, player management, and the WebSocket connection.

This separation ensures players only need the short code, while host operations use the stable UUID.

### 5.5 Player Management

- **Max players:** Enforced at join time based on `Session.maxPlayers` and the current game's `maxPlayers` (the lower value applies). When loading a game into a session, if the number of currently connected players exceeds the game's `maxPlayers`, the load request is rejected with a `422` error. The host must remove excess players before loading that game.
- **Disconnection:** If a player's WebSocket connection drops, `Player.connectionStatus` is set to `disconnected`. The player slot is preserved for a reconnection grace period. If the player reconnects within that period, their state is restored.
- **Leaving:** A player can voluntarily leave a session. `Player.leftAt` is set and `connectionStatus` becomes `disconnected`. Their slot is freed.
- **Host privileges:** The host can remove players from the session.

### 5.6 Game Loading

1. While in `lobby` or after a game ends, the host selects a game from the library.
2. The system validates that the game is `published` and has a `publishedVersionId`.
3. `Session.gameId` and `Session.gameVersionId` are set.
4. The game version's `gameScreenCode` is sent to the big screen. The `controllerScreenCode` is sent to each connected player's device.
5. Both canvases execute their setup phase.
6. `Session.status` transitions to `playing`.

### 5.7 Game Transitions

- When a game ends (either by game logic or host action), the session returns to the `lobby` state.
- `Session.gameId` and `Session.gameVersionId` are cleared.
- All player connections remain active - players do not need to rejoin.
- The host can select a new game from the lobby.

### 5.8 WebSocket Communication

The WebSocket layer handles bidirectional, real-time messaging between clients and the server within a session:

- **Connection:** Each client (big screen and each player phone) maintains one WebSocket connection to the server.
- **Message routing:** Messages are scoped to a session. The server routes messages between clients within the same session. It does not broadcast across sessions.
- **Message types:** The protocol supports at minimum:
    - Player input events (controller → server → game screen)
    - Game state updates (game screen → server → controllers)
    - Session control events (join, leave, game load, pause, resume, end)
    - Player status updates (connect, disconnect)
- **Latency:** The communication path should maintain latency low enough that player input feels responsive on the game screen.

### 5.9 Play History Tracking

- When a game starts in a session, a `PlayHistory` record is created for each connected player.
- `PlayHistory.duration` is updated when the game ends.
- If a player has a `userId`, the record is linked to their account for statistics and "recently played" functionality.
- Aggregate counters on the `Game` entity (`playCount`, `totalPlayTime`) are updated when a game session ends.

---

## 6. Creative Studio

### 6.1 Overview

The Creative Studio is a browser-based development environment where authenticated users build games. It provides a code editor, live preview, and game configuration in a single interface.

### 6.2 Code Editor

- Powered by Monaco Editor (the VS Code engine).
- Supports JavaScript and TypeScript with syntax highlighting, auto-completion, and inline error detection.
- Two editor panes: one for Game Screen code, one for Controller Screen code.
- Changes are auto-saved to the `Game` record's `gameScreenCode` and `controllerScreenCode` fields.

### 6.3 Live Preview

- The studio includes a preview panel that renders the Game Screen canvas.
- A simulated controller preview allows the creator to test controller interactions without a second device.
- The preview executes the game code in the same sandboxed runtime used in production sessions.

### 6.4 Testing

- Creators can start a private test session directly from the studio.
- The test session behaves like a real session: it generates a session code, and other devices can join as players.
- Test sessions use the current draft code (not a published version), allowing rapid iteration.
- **Implementation:** The session creation API accepts an optional `testGameId` parameter. When provided, the session bypasses the normal "load published game" flow and instead loads the game's current draft `gameScreenCode` and `controllerScreenCode` directly from the `Game` record. The `testGameId` must belong to the authenticated user. Test sessions are functionally identical to regular sessions but skip the publishing requirement.
- Test sessions do not create `PlayHistory` records and do not increment `Game.playCount` or `Game.totalPlayTime`.

### 6.5 Templates

- Templates are platform-provided starter projects stored as `Template` records.
- Each template includes pre-written code for both canvases, a description, and a difficulty level (`beginner`, `intermediate`, `advanced`).
- When a creator selects a template, the system creates a new `Game` with the template's code copied into `gameScreenCode` and `controllerScreenCode`.
- Templates are maintained by platform administrators and are ordered by `sortOrder`.

---

## 7. Game Library & Content Discovery

### 7.1 Library Browse

The game library presents all games that are `published` and `public`. Games are displayed with their thumbnail, title, creator username, player count range, average rating, and play count.

### 7.2 Sorting and Filtering

| Filter / Sort       | Description                                                   |
|:--------------------|:--------------------------------------------------------------|
| **Tags**            | Filter by one or more tags (genre, mood, player style).       |
| **Player count**    | Filter to games that support a given number of players.       |
| **Technology**      | Filter by runtime technology (p5js, 3d, visual).              |
| **Sort: Popular**   | Ordered by `playCount` descending.                            |
| **Sort: Top Rated** | Ordered by `avgRating` descending (minimum review threshold). |
| **Sort: Newest**    | Ordered by publish date descending.                           |
| **Sort: Trending**  | Ordered by recent play count growth.                          |

### 7.3 Search

- Text search matches against `Game.title`, `Game.description`, `User.username` (creator), and `Tag.name`.
- Results are ranked by relevance.

### 7.4 Game Detail Page

A game's detail page shows:

- Title, description, thumbnail.
- Creator profile (username, avatar, link to public profile).
- Player count range (`minPlayers`–`maxPlayers`).
- Tags.
- Average rating and review count.
- Play count and total play time.
- Version history (version numbers and changelogs).
- Reviews (paginated, sorted by most recent).
- "Forked from" attribution if applicable.
- Play button (starts or joins a session with this game).

### 7.5 Collections

- Collections are curated groups of games managed by platform staff. Moderators and admins can create, update, and manage collection contents. Only admins can delete collections.
- Each collection has a title, description, and cover image.
- Collections are displayed on the library's featured section, ordered by `Collection.sortOrder`.
- Only `active` collections are visible.
- A game can appear in multiple collections.

---

## 8. Community Features

### 8.1 Reviews

- Only authenticated users can submit reviews.
- A user cannot review their own game (enforced at the API level — the creator's `userId` cannot match the review's `userId` for the same `gameId`).
- A user can review a game at most once (enforced by the unique constraint on `userId` + `gameId`).
- A review consists of a star rating (integer, 1–5), an optional title, and an optional body text.
- Reviews can be edited by the author and soft-deleted by the author or a moderator.
- When a review is created, updated, or deleted, `Game.avgRating` and `Game.reviewCount` are recalculated.

### 8.2 Favorites

- Authenticated users can favorite a game, adding it to their personal favorites list.
- A user can favorite a game at most once.
- Favoriting is a toggle: favoriting an already-favorited game removes the favorite.
- The user's favorites list is accessible from their profile or dashboard.

### 8.3 Play History

- Authenticated users can view their play history: a list of games they have played, sorted by most recent.
- Each entry shows the game, the date played, and the duration.
- Play history is used to power a "recently played" section on the user's dashboard.

### 8.4 Creator Statistics

Creators can view statistics for their published games:

- Total play count and total play time across all games.
- Per-game play count, average session duration, rating distribution, and review count.
- Trend data (plays over time).

---

## 9. Subscription & Business Model

### 9.1 Subscription Tiers

| Feature                       | Free                                                | Pro          |
|:------------------------------|:----------------------------------------------------|:-------------|
| Play games                    | Yes                                                 | Yes          |
| Create and publish games      | Yes                                                 | Yes          |
| Game library access           | Full library (some games may be marked as Pro-only) | Full library |
| Intermissions during sessions | Yes (every 10 minutes)                              | No           |
| Max players per session       | 8 players                                           | Unlimited    |
| Max games per creator         | 5 published games                                   | Unlimited    |
| Max assets per game           | 20 assets                                           | 50 assets    |
| Game code size per canvas     | 100 KB                                              | 500 KB       |

### 9.2 Subscription Management

- Subscription state is stored on the `User` entity: `subscriptionPlan` (`free` or `pro`) and `subscriptionExpiresAt`.
- When a Pro subscription expires, the user reverts to the `free` plan.
- Subscription status is checked at session creation and game load to enforce tier-based limits.
- **For the initial launch, there is no self-service payment or checkout flow.** Subscriptions are managed exclusively by admins via the `PATCH /api/v1/admin/users/{userId}` endpoint, which can set `subscriptionPlan` and `subscriptionExpiresAt`. A payment provider integration may be added in a future phase.

### 9.3 Intermissions

- Free-tier sessions include periodic intermissions: a brief pause is injected every 10 minutes of continuous gameplay within a single session.
- Intermissions last approximately 15 seconds and display a platform-branded overlay on the Console screen.
- The intermission timer resets when a new game is loaded into the session.
- Pro-tier sessions skip intermissions entirely.
- The intermission schedule is controlled server-side and is not influenced by game code.

---

## 10. Content Moderation

### 10.1 Moderation Capabilities

Moderators and admins can:

- Review reported games and take action (archive or remove).
- Soft-delete reviews that violate community guidelines.
- Suspend user accounts that repeatedly violate guidelines.

### 10.2 Soft Deletion

All moderation actions use soft deletes (`deletedAt` timestamps). Soft-deleted records:

- Are excluded from public queries (library, search, profile pages).
- Remain in the database for audit purposes.
- Can be restored by an admin if the action is reversed.

---

## 11. Platform Constraints & Limits

### 11.1 Session Constraints

| Constraint                    | Value / Rule                                                                       |
|:------------------------------|:-----------------------------------------------------------------------------------|
| Session code length           | 4–6 uppercase alphanumeric characters (excluding 0/O, 1/I/L)                       |
| Max concurrent players (free) | 8 players per session                                                              |
| Max concurrent players (pro)  | Unlimited (up to system capacity)                                                  |
| Reconnection grace period     | 30 seconds after disconnect before a player slot is freed                          |
| Session timeout               | Idle sessions (no activity) are automatically ended after 30 minutes of inactivity |

### 11.2 Game Constraints

| Constraint            | Value / Rule                                                                 |
|:----------------------|:-----------------------------------------------------------------------------|
| `minPlayers`          | Integer, minimum 1                                                           |
| `maxPlayers`          | Integer, must be ≥ `minPlayers`                                              |
| Game code size        | 500 KB maximum per canvas (`gameScreenCode` and `controllerScreenCode` each) |
| Asset upload size     | 5 MB maximum per asset file                                                  |
| Assets per game       | 50 assets maximum per game                                                   |
| Supported asset types | Images: PNG, JPG, SVG, GIF. Audio: MP3, WAV, OGG. Fonts: TTF, WOFF2.         |

### 11.3 User Constraints

| Constraint       | Value / Rule                                                                                          |
|:-----------------|:------------------------------------------------------------------------------------------------------|
| Username         | Unique, 3–30 characters, alphanumeric plus hyphens and underscores, must start with a letter          |
| Display name     | 1–50 characters                                                                                       |
| Email            | Must be a valid email format, unique across the platform                                              |
| Password         | Minimum 8 characters, must contain at least one uppercase letter, one lowercase letter, and one digit |
| Avatar           | PNG, JPG, SVG, or GIF, maximum 2 MB                                                                   |
| Bio              | Maximum 500 characters                                                                                |
| Review title     | Maximum 100 characters                                                                                |
| Review body      | Maximum 2000 characters                                                                               |
| Game title       | 1–100 characters                                                                                      |
| Game description | Maximum 2000 characters                                                                               |

---

## 12. Data Conventions

### 12.1 Identifiers

All entities use UUIDs as primary keys. UUIDs are generated server-side at record creation.

### 12.2 Timestamps

- Most entities include `createdAt` (immutable, set at creation) and `updatedAt` (set at creation, updated on modification).
- `createdAt` is never modified after initial creation.
- All timestamps are stored and transmitted in UTC (ISO 8601 format).

**Exceptions to the `createdAt`/`updatedAt` convention:**

| Entity           | Reason                                                                                                                                                                       |
|:-----------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `GameVersion`    | Immutable snapshot — has `createdAt` only. No `updatedAt` because versions are never modified.                                                                               |
| `GameAsset`      | Has `createdAt` only. Assets are replaced by deleting and re-uploading, not edited in place.                                                                                 |
| `Tag`            | Platform-managed seed data — no timestamps. Tags are created at seed time and rarely change.                                                                                 |
| `GameTag`        | Join table — no timestamps. The relationship is set or removed atomically.                                                                                                   |
| `CollectionGame` | Join table — no timestamps. The relationship is set or removed atomically via the set-collection API.                                                                        |
| `Favorite`       | Has `createdAt` only. Favorites are toggled (created/deleted), never edited.                                                                                                 |
| `PlayHistory`    | Has `playedAt` (serves as `createdAt`). `duration` is updated when the game ends but no `updatedAt` is tracked — the update is a one-time finalization, not an ongoing edit. |

### 12.3 Soft Deletes

Entities that support soft deletion include a nullable `deletedAt` timestamp. When set:

- The record is excluded from standard queries.
- The record remains in the database and can be restored.
- Cascading soft deletes are not automatic - related records must be handled explicitly.

Entities with soft deletes: `User`, `Game`, `GameAsset`, `Review`.

### 12.4 Cached Aggregates

The `Game` entity maintains cached aggregate fields (`playCount`, `totalPlayTime`, `avgRating`, `reviewCount`). These are denormalized for query performance and are recalculated when the underlying data changes (new play session ends, review is created/updated/deleted).
