# AirCade - Entities

> **Note:** `FK -> (...)` stands for Foreign Key referencing another entity.

## User

A registered account on the platform. Users can create and publish games in the Creative Studio, host sessions, leave reviews, and manage their profile. An account is required for creation and publishing but not for playing.

| FIELD                   | TYPE        | SPECIFICITY             | DESCRIPTION                                                                    | EXAMPLE                |
|:------------------------|:------------|:------------------------|:-------------------------------------------------------------------------------|:-----------------------|
| `id`                    | `UUID`      | _Unique_, _Primary Key_ | The user account unique identifier                                             | `a8f3e7b1-4c2d-...`    |
| `createdAt`             | `TimeStamp` | _Not Modifiable_        | The date and time when this account was created                                | `2026-01-12T14:32:00Z` |
| `updatedAt`             | `TimeStamp` |                         | The date and time when this account was last modified                          | `2026-02-07T20:15:00Z` |
| `deletedAt`             | `TimeStamp` | _Nullable_              | The date and time when this account was soft deleted                           | `null`                 |
| `email`                 | `String`    | _Unique_, _Not Null_    | The user's email address, used for sign-in and notifications                   | `mike@example.com`     |
| `username`              | `String`    | _Unique_, _Not Null_    | The user's unique public name, visible to other players and creators           | `PixelMike`            |
| `displayName`           | `String`    | _Nullable_              | An optional friendly name shown in lobbies and on the creator profile          | `Mike Johnson`         |
| `avatarUrl`             | `String`    | _Nullable_              | URL to the user's profile picture, shown in lobbies and leaderboards           | `avatars/mike.png`     |
| `bio`                   | `String`    | _Nullable_              | A short description the user writes about themselves, visible on their profile | `Party game addict`    |
| `emailVerified`         | `Boolean`   | _Default: false_        | Whether the user has confirmed their email address (required to publish games) | `true`                 |
| `role`                  | `String`    | _Default: "user"_       | The user's platform role: `user`, `moderator`, or `admin`                      | `user`                 |
| `subscriptionPlan`      | `String`    | _Default: "free"_       | The user's subscription tier: `free` or `pro`                                  | `free`                 |
| `subscriptionExpiresAt` | `TimeStamp` | _Nullable_              | The date the Pro subscription expires (null for free-tier users)               | `2026-09-15T00:00:00Z` |
| `accountStatus`         | `String`    | _Default: "active"_     | The account's current state: `active`, `suspended`, or `deactivated`           | `active`               |
| `lastLoginAt`           | `TimeStamp` | _Nullable_              | The date and time of the user's last sign-in                                   | `2026-02-07T20:15:00Z` |
| `lastLoginIp`           | `String`    | _Nullable_              | IP address of the user's last sign-in                                          | `192.168.1.42`         |

---

## AuthProvider

Supports account linking: a single user can have multiple sign-in methods (email & password, Google, GitHub). Each row represents one linked authentication method.

| FIELD               | TYPE        | SPECIFICITY              | DESCRIPTION                                                                         | EXAMPLE                |
|:--------------------|:------------|:-------------------------|:------------------------------------------------------------------------------------|:-----------------------|
| `id`                | `UUID`      | _Unique_, _Primary Key_  | The auth provider link unique identifier                                            | `c4e2f8a1-7d3b-...`    |
| `createdAt`         | `TimeStamp` | _Not Modifiable_         | The date and time when this auth method was linked                                  | `2026-01-12T14:32:00Z` |
| `userId`            | `UUID`      | _Not Null_, _FK -> User_ | The user this auth method belongs to                                                | `a8f3e7b1-4c2d-...`    |
| `provider`          | `String`    | _Not Null_               | The authentication provider: `email`, `google`, or `github`                         | `google`               |
| `providerId`        | `String`    | _Not Null_, _Unique_     | The unique identifier from the provider (email address for email, OAuth ID for SSO) | `1234567890`           |
| `passwordHash`      | `String`    | _Nullable_               | The hashed password (only set when `provider` is `email`)                           | *(not human-readable)* |
| `providerEmail`     | `String`    | _Nullable_               | The email address associated with the OAuth provider account                        | `mike@gmail.com`       |
| `verificationToken` | `String`    | _Nullable_, _Unique_     | Token sent by email for address verification or password reset                      | *(not human-readable)* |
| `tokenExpiresAt`    | `TimeStamp` | _Nullable_               | Expiration time for the verification or reset token                                 | `2026-02-08T15:00:00Z` |

_Unique constraint on (`userId`, `provider`) - a user can link each provider at most once._

---

## Game

A game project created by a user in the Creative Studio. This is the central content entity of the platform.

| FIELD                  | TYPE        | SPECIFICITY                     | DESCRIPTION                                                             | EXAMPLE                     |
|:-----------------------|:------------|:--------------------------------|:------------------------------------------------------------------------|:----------------------------|
| `id`                   | `UUID`      | _Unique_, _Primary Key_         | The game project unique identifier                                      | `d7a1c3e5-9f4b-...`         |
| `createdAt`            | `TimeStamp` | _Not Modifiable_                | The date and time when this game was created                            | `2026-02-01T10:00:00Z`      |
| `updatedAt`            | `TimeStamp` |                                 | The date and time when this game was last modified                      | `2026-02-07T18:30:00Z`      |
| `deletedAt`            | `TimeStamp` | _Nullable_                      | The date and time when this game was soft deleted                       | `null`                      |
| `creatorId`            | `UUID`      | _Not Null_, _FK -> User_        | The user who created this game                                          | `a8f3e7b1-4c2d-...`         |
| `title`                | `String`    | _Not Null_                      | The name of the game, displayed in the library and search results       | `Space Race`                |
| `description`          | `String`    | _Nullable_                      | A longer description of the game, shown on its detail page              | `Race through asteroids!`   |
| `thumbnailUrl`         | `String`    | _Nullable_                      | URL to the game's cover image, shown in the library grid                | `thumbnails/space-race.png` |
| `technology`           | `String`    | _Default: "p5js"_               | The runtime technology used: `p5js` (future: `3d`, `visual`)            | `p5js`                      |
| `minPlayers`           | `Integer`   | _Default: 1_                    | The minimum number of players required to start the game                | `2`                         |
| `maxPlayers`           | `Integer`   | _Default: 8_                    | The maximum number of players supported                                 | `8`                         |
| `status`               | `String`    | _Default: "draft"_              | The game's lifecycle state: `draft`, `published`, or `archived`         | `published`                 |
| `visibility`           | `String`    | _Default: "private"_            | Who can see the game in the library: `public`, `private`, or `unlisted` | `public`                    |
| `remixable`            | `Boolean`   | _Default: false_                | Whether other creators are allowed to fork and remix this game          | `true`                      |
| `forkedFromId`         | `UUID`      | _Nullable_, _FK -> Game_        | The original game this was forked from (null if original creation)      | `null`                      |
| `gameScreenCode`       | `Text`      | _Nullable_                      | The source code for the Game Screen canvas (big screen)                 | *(game code)*               |
| `controllerScreenCode` | `Text`      | _Nullable_                      | The source code for the Controller Screen canvas (smartphone)           | *(controller code)*         |
| `publishedVersionId`   | `UUID`      | _Nullable_, _FK -> GameVersion_ | The currently live published version (null if never published)          | `b2d4f6a8-1c3e-...`         |
| `playCount`            | `Integer`   | _Default: 0_                    | Cached total number of times this game has been played                  | `1247`                      |
| `totalPlayTime`        | `Integer`   | _Default: 0_                    | Cached total play time across all sessions, in seconds                  | `89340`                     |
| `avgRating`            | `Float`     | _Default: 0.0_                  | Cached average rating from reviews (0.0 if no reviews)                  | `4.3`                       |
| `reviewCount`          | `Integer`   | _Default: 0_                    | Cached total number of reviews                                          | `42`                        |

---

## GameVersion

An immutable snapshot of a game's code at the time of publishing. Separates the working draft from published versions so players always get stable code.

| FIELD                  | TYPE        | SPECIFICITY              | DESCRIPTION                                               | EXAMPLE                |
|:-----------------------|:------------|:-------------------------|:----------------------------------------------------------|:-----------------------|
| `id`                   | `UUID`      | _Unique_, _Primary Key_  | The version unique identifier                             | `b2d4f6a8-1c3e-...`    |
| `createdAt`            | `TimeStamp` | _Not Modifiable_         | The date and time when this version was published         | `2026-02-05T12:00:00Z` |
| `gameId`               | `UUID`      | _Not Null_, _FK -> Game_ | The game this version belongs to                          | `d7a1c3e5-9f4b-...`    |
| `versionNumber`        | `Integer`   | _Not Null_               | Sequential version number (1, 2, 3, ...)                  | `3`                    |
| `gameScreenCode`       | `Text`      | _Not Null_               | The frozen Game Screen source code for this version       | *(game code)*          |
| `controllerScreenCode` | `Text`      | _Not Null_               | The frozen Controller Screen source code for this version | *(controller code)*    |
| `changelog`            | `String`    | _Nullable_               | Optional description of what changed in this version      | `Added power-ups`      |
| `publishedById`        | `UUID`      | _Not Null_, _FK -> User_ | The user who published this version                       | `a8f3e7b1-4c2d-...`    |

_Unique constraint on (`gameId`, `versionNumber`) - no duplicate version numbers per game._

---

## GameAsset

Media files (images, sounds, sprites, fonts) uploaded by a creator and attached to a game project.

| FIELD        | TYPE        | SPECIFICITY              | DESCRIPTION                                       | EXAMPLE                         |
|:-------------|:------------|:-------------------------|:--------------------------------------------------|:--------------------------------|
| `id`         | `UUID`      | _Unique_, _Primary Key_  | The asset unique identifier                       | `e1f2a3b4-5c6d-...`             |
| `createdAt`  | `TimeStamp` | _Not Modifiable_         | The date and time when this asset was uploaded    | `2026-02-03T09:15:00Z`          |
| `gameId`     | `UUID`      | _Not Null_, _FK -> Game_ | The game this asset belongs to                    | `d7a1c3e5-9f4b-...`             |
| `fileName`   | `String`    | _Not Null_               | The original file name as uploaded by the creator | `spaceship.png`                 |
| `fileType`   | `String`    | _Not Null_               | The MIME type of the file                         | `image/png`                     |
| `fileSize`   | `Integer`   | _Not Null_               | The file size in bytes                            | `24576`                         |
| `storageUrl` | `String`    | _Not Null_               | The URL or storage path where the file is stored  | `assets/d7a1c3e5/spaceship.png` |

---

## Tag

Normalized labels used to categorize games by genre, mood, or play style.

| FIELD      | TYPE     | SPECIFICITY             | DESCRIPTION                                                  | EXAMPLE        |
|:-----------|:---------|:------------------------|:-------------------------------------------------------------|:---------------|
| `id`       | `UUID`   | _Unique_, _Primary Key_ | The tag unique identifier                                    | `f1a2b3c4-...` |
| `name`     | `String` | _Unique_, _Not Null_    | The human-readable tag name                                  | `Racing`       |
| `slug`     | `String` | _Unique_, _Not Null_    | URL-safe version of the tag name, used in filters and routes | `racing`       |
| `category` | `String` | _Not Null_              | The type of tag: `genre`, `mood`, or `playerStyle`           | `genre`        |

---

## GameTag

Join table that links games to their tags. A game can have multiple tags, and a tag can apply to multiple games.

| FIELD    | TYPE   | SPECIFICITY              | DESCRIPTION                 | EXAMPLE             |
|:---------|:-------|:-------------------------|:----------------------------|:--------------------|
| `gameId` | `UUID` | _Not Null_, _FK -> Game_ | The game being tagged       | `d7a1c3e5-9f4b-...` |
| `tagId`  | `UUID` | _Not Null_, _FK -> Tag_  | The tag applied to the game | `f1a2b3c4-...`      |

_Primary key is (`gameId`, `tagId`)._

---

## Session

A live play session hosted on a big screen. Created when a host launches AirCade and generates a session code for players to join.

| FIELD           | TYPE        | SPECIFICITY                     | DESCRIPTION                                                           | EXAMPLE                |
|:----------------|:------------|:--------------------------------|:----------------------------------------------------------------------|:-----------------------|
| `id`            | `UUID`      | _Unique_, _Primary Key_         | The session unique identifier                                         | `91b2c3d4-5e6f-...`    |
| `createdAt`     | `TimeStamp` | _Not Modifiable_                | The date and time when the session was created                        | `2026-02-07T19:00:00Z` |
| `updatedAt`     | `TimeStamp` |                                 | The date and time when the session was last updated                   | `2026-02-07T19:45:00Z` |
| `endedAt`       | `TimeStamp` | _Nullable_                      | The date and time when the session ended (null while active)          | `null`                 |
| `hostId`        | `UUID`      | _Nullable_, _FK -> User_        | The user hosting the session (null for anonymous hosts)               | `a8f3e7b1-4c2d-...`    |
| `gameId`        | `UUID`      | _Nullable_, _FK -> Game_        | The game currently loaded in this session (null while in lobby)       | `d7a1c3e5-9f4b-...`    |
| `gameVersionId` | `UUID`      | _Nullable_, _FK -> GameVersion_ | The specific published version being played (null while in lobby)     | `b2d4f6a8-1c3e-...`    |
| `sessionCode`   | `String`    | _Unique_, _Not Null_            | The short-lived code displayed on the big screen for players to join  | `XKCD42`               |
| `status`        | `String`    | _Default: "lobby"_              | The session's current state: `lobby`, `playing`, `paused`, or `ended` | `playing`              |
| `maxPlayers`    | `Integer`   | _Default: 8_                    | The maximum number of players allowed in this session                 | `12`                   |

---

## Player

A participant in a session. Players do not need an AirCade account to join - they scan the QR code or enter the session code and play immediately.

| FIELD              | TYPE        | SPECIFICITY                 | DESCRIPTION                                                                   | EXAMPLE                |
|:-------------------|:------------|:----------------------------|:------------------------------------------------------------------------------|:-----------------------|
| `id`               | `UUID`      | _Unique_, _Primary Key_     | The player instance unique identifier                                         | `71a2b3c4-5d6e-...`    |
| `createdAt`        | `TimeStamp` | _Not Modifiable_            | The date and time when this player joined the session                         | `2026-02-07T19:02:00Z` |
| `sessionId`        | `UUID`      | _Not Null_, _FK -> Session_ | The session this player is participating in                                   | `91b2c3d4-5e6f-...`    |
| `userId`           | `UUID`      | _Nullable_, _FK -> User_    | The user's account (null for anonymous/guest players)                         | `null`                 |
| `displayName`      | `String`    | _Not Null_                  | The name shown in the lobby and in-game for this player                       | `Player 3`             |
| `avatarUrl`        | `String`    | _Nullable_                  | URL to the player's avatar image                                              | `avatars/default3.png` |
| `connectionStatus` | `String`    | _Default: "connected"_      | The player's WebSocket connection state: `connected` or `disconnected`        | `connected`            |
| `leftAt`           | `TimeStamp` | _Nullable_                  | The date and time when the player left or disconnected (null while connected) | `null`                 |

---

## Review

A user's rating and written review of a published game. Users must have an account to leave a review.

| FIELD       | TYPE        | SPECIFICITY              | DESCRIPTION                                         | EXAMPLE                |
|:------------|:------------|:-------------------------|:----------------------------------------------------|:-----------------------|
| `id`        | `UUID`      | _Unique_, _Primary Key_  | The review unique identifier                        | `a1b2c3d4-5e6f-...`    |
| `createdAt` | `TimeStamp` | _Not Modifiable_         | The date and time when this review was submitted    | `2026-02-06T14:00:00Z` |
| `updatedAt` | `TimeStamp` |                          | The date and time when this review was last edited  | `2026-02-06T14:00:00Z` |
| `deletedAt` | `TimeStamp` | _Nullable_               | The date and time when this review was soft deleted | `null`                 |
| `userId`    | `UUID`      | _Not Null_, _FK -> User_ | The user who wrote the review                       | `a8f3e7b1-4c2d-...`    |
| `gameId`    | `UUID`      | _Not Null_, _FK -> Game_ | The game being reviewed                             | `d7a1c3e5-9f4b-...`    |
| `rating`    | `Integer`   | _Not Null_               | The star rating, from 1 to 5                        | `4`                    |
| `title`     | `String`    | _Nullable_               | An optional short headline for the review           | `Great party game!`    |
| `body`      | `String`    | _Nullable_               | The full review text                                | `My friends loved it.` |

_Unique constraint on (`userId`, `gameId`) - a user can review each game at most once._

---

## Favorite

A user bookmarking a game for quick access later.

| FIELD       | TYPE        | SPECIFICITY              | DESCRIPTION                                        | EXAMPLE                |
|:------------|:------------|:-------------------------|:---------------------------------------------------|:-----------------------|
| `id`        | `UUID`      | _Unique_, _Primary Key_  | The favorite unique identifier                     | `b2c3d4e5-6f7a-...`    |
| `createdAt` | `TimeStamp` | _Not Modifiable_         | The date and time when the user favorited the game | `2026-02-05T20:00:00Z` |
| `userId`    | `UUID`      | _Not Null_, _FK -> User_ | The user who favorited the game                    | `a8f3e7b1-4c2d-...`    |
| `gameId`    | `UUID`      | _Not Null_, _FK -> Game_ | The game that was favorited                        | `d7a1c3e5-9f4b-...`    |

_Unique constraint on (`userId`, `gameId`) - a user can favorite each game at most once._

---

## PlayHistory

Records of individual game plays, used for tracking play statistics, recommendations, and the "recently played" feed.

| FIELD       | TYPE        | SPECIFICITY                 | DESCRIPTION                                                      | EXAMPLE                |
|:------------|:------------|:----------------------------|:-----------------------------------------------------------------|:-----------------------|
| `id`        | `UUID`      | _Unique_, _Primary Key_     | The play record unique identifier                                | `c3d4e5f6-7a8b-...`    |
| `playedAt`  | `TimeStamp` | _Not Modifiable_            | The date and time when this play session started                 | `2026-02-07T19:05:00Z` |
| `userId`    | `UUID`      | _Nullable_, _FK -> User_    | The user who played (null for anonymous/guest players)           | `a8f3e7b1-4c2d-...`    |
| `gameId`    | `UUID`      | _Not Null_, _FK -> Game_    | The game that was played                                         | `d7a1c3e5-9f4b-...`    |
| `sessionId` | `UUID`      | _Not Null_, _FK -> Session_ | The session in which the game was played                         | `91b2c3d4-5e6f-...`    |
| `duration`  | `Integer`   | _Nullable_                  | The duration of this play in seconds (null if still in progress) | `720`                  |

---

## Template

A platform-provided starter project that new creators can open, study, modify, and use as a base for their own games.

| FIELD                  | TYPE        | SPECIFICITY             | DESCRIPTION                                                     | EXAMPLE                   |
|:-----------------------|:------------|:------------------------|:----------------------------------------------------------------|:--------------------------|
| `id`                   | `UUID`      | _Unique_, _Primary Key_ | The template unique identifier                                  | `d4e5f6a7-8b9c-...`       |
| `createdAt`            | `TimeStamp` | _Not Modifiable_        | The date and time when this template was created                | `2026-01-01T00:00:00Z`    |
| `updatedAt`            | `TimeStamp` |                         | The date and time when this template was last modified          | `2026-01-15T10:00:00Z`    |
| `title`                | `String`    | _Not Null_              | The name of the template                                        | `Quiz Show`               |
| `description`          | `String`    | _Nullable_              | A description of what the template demonstrates                 | `A trivia game template`  |
| `thumbnailUrl`         | `String`    | _Nullable_              | URL to the template's preview image                             | `templates/quiz-show.png` |
| `technology`           | `String`    | _Default: "p5js"_       | The runtime technology: `p5js` (future: `3d`, `visual`)         | `p5js`                    |
| `gameScreenCode`       | `Text`      | _Not Null_              | The starter Game Screen source code                             | *(game code)*             |
| `controllerScreenCode` | `Text`      | _Not Null_              | The starter Controller Screen source code                       | *(controller code)*       |
| `difficulty`           | `String`    | _Default: "beginner"_   | The complexity level: `beginner`, `intermediate`, or `advanced` | `beginner`                |
| `sortOrder`            | `Integer`   | _Default: 0_            | Display order in the template list (lower values appear first)  | `1`                       |

---

## Collection

A curated or featured group of games, managed by platform staff (e.g., "Ice-Breakers for Teams", "Family Night Favorites").

| FIELD           | TYPE        | SPECIFICITY             | DESCRIPTION                                                    | EXAMPLE                            |
|:----------------|:------------|:------------------------|:---------------------------------------------------------------|:-----------------------------------|
| `id`            | `UUID`      | _Unique_, _Primary Key_ | The collection unique identifier                               | `e5f6a7b8-9c0d-...`                |
| `createdAt`     | `TimeStamp` | _Not Modifiable_        | The date and time when this collection was created             | `2026-01-20T08:00:00Z`             |
| `updatedAt`     | `TimeStamp` |                         | The date and time when this collection was last modified       | `2026-02-01T12:00:00Z`             |
| `title`         | `String`    | _Not Null_              | The collection's display name                                  | `Family Night Favorites`           |
| `description`   | `String`    | _Nullable_              | A description of the collection's theme or purpose             | `Games the whole family will love` |
| `coverImageUrl` | `String`    | _Nullable_              | URL to the collection's cover image                            | `collections/family.png`           |
| `sortOrder`     | `Integer`   | _Default: 0_            | Display order on the featured page (lower values appear first) | `2`                                |
| `active`        | `Boolean`   | _Default: true_         | Whether this collection is currently visible to users          | `true`                             |

---

## CollectionGame

Join table that links games to curated collections. A game can appear in multiple collections, and a collection contains multiple games.

| FIELD          | TYPE      | SPECIFICITY                    | DESCRIPTION                                     | EXAMPLE             |
|:---------------|:----------|:-------------------------------|:------------------------------------------------|:--------------------|
| `collectionId` | `UUID`    | _Not Null_, _FK -> Collection_ | The collection containing the game              | `e5f6a7b8-9c0d-...` |
| `gameId`       | `UUID`    | _Not Null_, _FK -> Game_       | The game included in the collection             | `d7a1c3e5-9f4b-...` |
| `sortOrder`    | `Integer` | _Default: 0_                   | Display order of the game within the collection | `3`                 |

_Primary key is (`collectionId`, `gameId`)._
