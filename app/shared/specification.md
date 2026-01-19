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

### 1.6 Authorization Rules

| Role          | Capabilities                                                                                                                                             |
|:--------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Guest**     | Join sessions as a player (no account required). Browse public game library. View game details.                                                          |
| **User**      | Everything a guest can do, plus: create games, publish games (requires verified email), host sessions, leave reviews, favorite games, view play history. |
| **Moderator** | Everything a user can do, plus: manage reported content, remove inappropriate reviews, archive games that violate guidelines.                            |
| **Admin**     | Full platform access: manage users (suspend, deactivate), manage collections and templates, assign moderator roles, view platform analytics.             |

### 1.7 Account Status

| Status          | Effect                                                                                                                                                     |
|:----------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------|
| **active**      | Full access based on role.                                                                                                                                 |
| **suspended**   | Cannot sign in. Existing sessions are not interrupted but no new sessions or content can be created. Published games remain visible but cannot be updated. |
| **deactivated** | Cannot sign in. Profile and published games are hidden from other users. Data is retained for potential reactivation.                                      |

---

## 2. User Profile

### 2.1 Profile Fields

Users manage the following profile information:

- **username** - Unique, public. Used in URLs and @mentions. Cannot be changed to a username currently held by another user.
- **displayName** - Optional friendly name shown in lobbies and on the creator profile page.
- **email** - Unique. Used for sign-in and notifications. Changing the email requires re-verification.
- **avatarUrl** - Profile picture. Users can upload an image or use a default generated avatar.
- **bio** - Short free-text description visible on the user's public profile.

### 2.2 Public Profile

Every user with at least one published game has a public profile page showing:

- Username, display name, avatar, and bio.
- Published games (sorted by most recent).
- Aggregate statistics: total games published, total play count across all games.

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
- Assets are stored at a platform-managed storage URL and are accessible from game code at runtime.
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

### 4.4 Game State Synchronization

Game state is synchronized between the Game Screen and Controller Screens through the server's WebSocket layer:

- **Player input** flows from Controller Screen → Server → Game Screen.
- **Game state updates** flow from Game Screen → Server → Controller Screens.
- The server acts as a relay within a session. It does not interpret or validate game logic - the game code running in the browsers handles all game rules.
- The server ensures that messages are routed only within the session they belong to.

### 4.5 Runtime Isolation

- Each game runs in a sandboxed execution context within the browser.
- Games cannot access browser APIs outside of the platform-provided runtime (DOM manipulation outside the canvas, localStorage, navigation, etc.).
- Games cannot communicate with external servers. All communication goes through the platform's WebSocket layer.
- The platform provides a controlled API for games to interact with session data (player list, player input, game state broadcast).

---

## 5. Sessions & Real-Time Communication

### 5.1 Session Lifecycle

```
lobby ──► playing ──► paused ──► playing ──► ended
                        │                      ▲
                        └──────────────────────┘
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
5. If the host is signed in, `Session.hostId` is set to their user ID. Anonymous hosting is supported (`hostId` is null).

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
- A code is valid only for the lifetime of its session. Once a session ends, its code can be reused by a future session.
- Codes are case-insensitive on input.

### 5.5 Player Management

- **Max players:** Enforced at join time based on `Session.maxPlayers` and the current game's `maxPlayers` (the lower value applies).
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

- Collections are curated groups of games managed by platform staff (`moderator` or `admin` roles).
- Each collection has a title, description, and cover image.
- Collections are displayed on the library's featured section, ordered by `Collection.sortOrder`.
- Only `active` collections are visible.
- A game can appear in multiple collections.

---

## 8. Community Features

### 8.1 Reviews

- Only authenticated users can submit reviews.
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

| Feature                       | Free    | Pro          |
|:------------------------------|:--------|:-------------|
| Play games                    | Yes     | Yes          |
| Create and publish games      | Yes     | Yes          |
| Game library access           | Limited | Full library |
| Intermissions during sessions | Yes     | No           |
| Max players per session       | Limited | Unlimited    |
| Advanced creation tools       | Basic   | Full access  |

### 9.2 Subscription Management

- Subscription state is stored on the `User` entity: `subscriptionPlan` (`free` or `pro`) and `subscriptionExpiresAt`.
- When a Pro subscription expires, the user reverts to the `free` plan.
- Subscription status is checked at session creation and game load to enforce tier-based limits.

### 9.3 Intermissions

- Free-tier sessions include periodic intermissions (brief pauses between games or at timed intervals).
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

| Constraint                    | Value / Rule                                                        |
|:------------------------------|:--------------------------------------------------------------------|
| Session code length           | 4–6 alphanumeric characters                                         |
| Max concurrent players (free) | Platform-defined limit (enforced at join)                           |
| Max concurrent players (pro)  | Unlimited (up to system capacity)                                   |
| Reconnection grace period     | Time window after disconnect before a player slot is freed          |
| Session timeout               | Idle sessions (no activity) are automatically ended after a timeout |

### 11.2 Game Constraints

| Constraint            | Value / Rule                                                         |
|:----------------------|:---------------------------------------------------------------------|
| `minPlayers`          | Integer, minimum 1                                                   |
| `maxPlayers`          | Integer, must be ≥ `minPlayers`                                      |
| Game code size        | Maximum allowed size for `gameScreenCode` and `controllerScreenCode` |
| Asset upload size     | Maximum file size per asset                                          |
| Assets per game       | Maximum number of assets per game                                    |
| Supported asset types | Images: PNG, JPG, SVG, GIF. Audio: MP3, WAV, OGG. Fonts: TTF, WOFF2. |

### 11.3 User Constraints

| Constraint  | Value / Rule                                                        |
|:------------|:--------------------------------------------------------------------|
| Username    | Unique, alphanumeric with limited special characters, length limits |
| Email       | Must be a valid email format, unique across the platform            |
| Password    | Minimum length and complexity requirements                          |
| Review text | Maximum character length for title and body                         |
| Bio         | Maximum character length                                            |

---

## 12. Data Conventions

### 12.1 Identifiers

All entities use UUIDs as primary keys. UUIDs are generated server-side at record creation.

### 12.2 Timestamps

- All entities include `createdAt` (immutable, set at creation) and `updatedAt` (set at creation, updated on modification).
- `createdAt` is never modified after initial creation.
- All timestamps are stored and transmitted in UTC (ISO 8601 format).

### 12.3 Soft Deletes

Entities that support soft deletion include a nullable `deletedAt` timestamp. When set:

- The record is excluded from standard queries.
- The record remains in the database and can be restored.
- Cascading soft deletes are not automatic - related records must be handled explicitly.

Entities with soft deletes: `User`, `Game`, `Review`.

### 12.4 Cached Aggregates

The `Game` entity maintains cached aggregate fields (`playCount`, `totalPlayTime`, `avgRating`, `reviewCount`). These are denormalized for query performance and are recalculated when the underlying data changes (new play session ends, review is created/updated/deleted).
