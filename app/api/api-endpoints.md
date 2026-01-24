# AirCade - API Endpoints

This document defines every REST endpoint and the WebSocket protocol for the AirCade platform. The backend is built with Rust / Axum 0.8 and communicates over HTTPS (REST) and WSS (WebSocket).

---

## Table of Contents

01. [Conventions](#1-conventions)
02. [Authentication](#2-authentication)
03. [Users](#3-users)
04. [Games](#4-games)
05. [Sessions](#5-sessions)
06. [Community](#6-community)
07. [Discovery](#7-discovery)
08. [Templates](#8-templates)
09. [Administration](#9-administration)
10. [WebSocket Protocol](#10-websocket-protocol)

---

## 1. Conventions

### 1.1 Base URL

All endpoints are prefixed with:

```
/api/v1
```

### 1.2 Authentication

Authenticated endpoints require a Bearer token in the `Authorization` header:

```
Authorization: Bearer <token>
```

Tokens are issued on sign-in and refreshed via the refresh endpoint. Each endpoint specifies its required role:

| Role          | Description                                               |
|:--------------|:----------------------------------------------------------|
| **None**      | No authentication required (public access).               |
| **Guest**     | No account required, but a session context may be needed. |
| **User**      | Requires a signed-in user with `accountStatus: "active"`. |
| **Moderator** | Requires `role: "moderator"` or `"admin"`.                |
| **Admin**     | Requires `role: "admin"`.                                 |

Suspended and deactivated accounts are rejected at the middleware level with `403 Forbidden`.

### 1.3 Request & Response Format

- Request bodies and responses use **JSON** (`Content-Type: application/json`) unless stated otherwise (e.g., file uploads use `multipart/form-data`).
- All identifiers are **UUIDs**.
- All timestamps are **UTC, ISO 8601** (e.g., `2026-02-08T15:30:00Z`).

### 1.4 Pagination

List endpoints support offset-based pagination via query parameters:

| Parameter | Type      | Default | Description                            |
|:----------|:----------|:--------|:---------------------------------------|
| `offset`  | `integer` | `0`     | Number of records to skip.             |
| `limit`   | `integer` | `20`    | Number of records to return (max 100). |

Paginated responses wrap results in a standard envelope:

```json
{
  "data": [],
  "total": 142,
  "offset": 0,
  "limit": 20
}
```

### 1.5 Error Response Format

All errors return a consistent JSON body:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Human-readable description of the problem.",
    "details": {}
  }
}
```

The `details` object is optional and provides field-level information when relevant (e.g., validation errors).

### 1.6 Common HTTP Status Codes

| Code  | Meaning                                                                  |
|:------|:-------------------------------------------------------------------------|
| `200` | Success. Response body contains the requested data.                      |
| `201` | Created. A new resource was created and is returned in the body.         |
| `204` | No Content. The action succeeded with no response body.                  |
| `400` | Bad Request. The request is malformed or missing required fields.        |
| `401` | Unauthorized. No valid token was provided.                               |
| `403` | Forbidden. The authenticated user lacks permission for this action.      |
| `404` | Not Found. The referenced resource does not exist.                       |
| `409` | Conflict. A unique constraint would be violated (e.g., duplicate email). |
| `413` | Payload Too Large. The uploaded file exceeds the size limit.             |
| `422` | Unprocessable Entity. A business rule prevents the action.               |
| `429` | Too Many Requests. Rate limit exceeded.                                  |
| `500` | Internal Server Error.                                                   |

### 1.7 Common Error Codes

| Error Code            | Used When                                                  |
|:----------------------|:-----------------------------------------------------------|
| `VALIDATION_ERROR`    | A field fails format or constraint validation.             |
| `UNAUTHORIZED`        | Missing or invalid authentication token.                   |
| `FORBIDDEN`           | User lacks the required role or ownership.                 |
| `NOT_FOUND`           | The referenced resource does not exist or is soft-deleted. |
| `CONFLICT`            | A unique constraint violation (email, username, etc.).     |
| `EMAIL_NOT_VERIFIED`  | Action requires a verified email.                          |
| `ACCOUNT_SUSPENDED`   | The user's account is suspended.                           |
| `ACCOUNT_DEACTIVATED` | The user's account is deactivated.                         |
| `RATE_LIMIT_EXCEEDED` | Too many requests in the time window.                      |
| `PAYLOAD_TOO_LARGE`   | File or body exceeds size limit.                           |

---

## 2. Authentication

### 2.1 Email Sign-Up

Creates a new user account with email and password. Sends a verification email.

```
POST /api/v1/auth/signup/email
```

**Auth:** None

**Request Body:**

| Field      | Type     | Required | Description                        |
|:-----------|:---------|:---------|:-----------------------------------|
| `email`    | `string` | Yes      | Email address. Must be unique.     |
| `username` | `string` | Yes      | Username. Must be unique.          |
| `password` | `string` | Yes      | Password meeting complexity rules. |

**Response:** `201 Created`

```json
{
  "user": {
    "id": "uuid",
    "email": "mike@example.com",
    "username": "PixelMike",
    "displayName": null,
    "avatarUrl": null,
    "bio": null,
    "emailVerified": false,
    "role": "user",
    "subscriptionPlan": "free",
    "createdAt": "2026-02-08T15:30:00Z"
  },
  "token": "jwt-access-token",
  "refreshToken": "jwt-refresh-token"
}
```

**Errors:**

| Code  | Condition                                |
|:------|:-----------------------------------------|
| `409` | Email or username already exists.        |
| `400` | Password does not meet complexity rules. |

---

### 2.2 Email Sign-In

Authenticates a user with email and password.

```
POST /api/v1/auth/signin/email
```

**Auth:** None

**Request Body:**

| Field      | Type     | Required | Description       |
|:-----------|:---------|:---------|:------------------|
| `email`    | `string` | Yes      | Email address.    |
| `password` | `string` | Yes      | Account password. |

**Response:** `200 OK`

```json
{
  "user": {
    "id": "uuid",
    "email": "mike@example.com",
    "username": "PixelMike",
    "displayName": "Mike Johnson",
    "avatarUrl": "avatars/mike.png",
    "bio": "Party game addict",
    "emailVerified": true,
    "role": "user",
    "subscriptionPlan": "free",
    "createdAt": "2026-01-12T14:32:00Z"
  },
  "token": "jwt-access-token",
  "refreshToken": "jwt-refresh-token"
}
```

Updates `User.lastLoginAt` and `User.lastLoginIp` on success.

**Errors:**

| Code  | Condition                            |
|:------|:-------------------------------------|
| `401` | Invalid email or password.           |
| `403` | Account is suspended or deactivated. |

---

### 2.3 Google OAuth — Initiate

Redirects the user to Google's OAuth 2.0 consent screen.

```
GET /api/v1/auth/oauth/google
```

**Auth:** None

**Query Parameters:**

| Parameter     | Type     | Required | Description                                |
|:--------------|:---------|:---------|:-------------------------------------------|
| `redirectUri` | `string` | No       | Client redirect URI after OAuth completes. |

**Response:** `302 Found` — Redirects to Google's authorization URL with a CSRF `state` parameter.

---

### 2.4 Google OAuth — Callback

Handles the OAuth callback from Google. Creates a new user on first sign-in or signs in an existing user.

```
GET /api/v1/auth/oauth/google/callback
```

**Auth:** None

**Query Parameters:**

| Parameter | Type     | Required | Description                                    |
|:----------|:---------|:---------|:-----------------------------------------------|
| `code`    | `string` | Yes      | Authorization code from Google.                |
| `state`   | `string` | Yes      | CSRF state parameter from the initiation step. |

**Response:** `200 OK` — Returns the same user/token payload as email sign-in.

On first sign-in: creates a `User` (with `emailVerified: true`) and an `AuthProvider` with `provider: "google"`. Profile data (email, name, avatar) is imported from Google.

**Errors:**

| Code  | Condition                                                        |
|:------|:-----------------------------------------------------------------|
| `400` | Invalid or expired authorization code.                           |
| `409` | The Google email is already registered via a different provider. |

---

### 2.5 GitHub OAuth — Initiate

Redirects the user to GitHub's OAuth authorization screen.

```
GET /api/v1/auth/oauth/github
```

**Auth:** None

**Query Parameters:**

| Parameter     | Type     | Required | Description                                |
|:--------------|:---------|:---------|:-------------------------------------------|
| `redirectUri` | `string` | No       | Client redirect URI after OAuth completes. |

**Response:** `302 Found` — Redirects to GitHub's authorization URL.

---

### 2.6 GitHub OAuth — Callback

Handles the OAuth callback from GitHub. Same behavior as the Google callback.

```
GET /api/v1/auth/oauth/github/callback
```

**Auth:** None

**Query Parameters:**

| Parameter | Type     | Required | Description                     |
|:----------|:---------|:---------|:--------------------------------|
| `code`    | `string` | Yes      | Authorization code from GitHub. |
| `state`   | `string` | Yes      | CSRF state parameter.           |

**Response:** `200 OK` — Same user/token payload as email sign-in.

**Errors:**

| Code  | Condition                                                        |
|:------|:-----------------------------------------------------------------|
| `400` | Invalid or expired authorization code.                           |
| `409` | The GitHub email is already registered via a different provider. |

---

### 2.7 Link Auth Provider

Links an additional OAuth provider to the currently signed-in user's account.

```
POST /api/v1/auth/link/{provider}
```

**Auth:** User

**Path Parameters:**

| Parameter  | Type     | Values             | Description           |
|:-----------|:---------|:-------------------|:----------------------|
| `provider` | `string` | `google`, `github` | The provider to link. |

**Request Body:**

| Field  | Type     | Required | Description                                 |
|:-------|:---------|:---------|:--------------------------------------------|
| `code` | `string` | Yes      | OAuth authorization code from the provider. |

**Response:** `201 Created`

```json
{
  "provider": "google",
  "providerEmail": "mike@gmail.com",
  "linkedAt": "2026-02-08T15:30:00Z"
}
```

**Errors:**

| Code  | Condition                                                                                  |
|:------|:-------------------------------------------------------------------------------------------|
| `409` | Provider already linked to this account, or the provider ID is linked to a different user. |
| `400` | Invalid authorization code.                                                                |

---

### 2.8 Unlink Auth Provider

Removes a linked auth provider from the user's account. At least one provider must remain.

```
DELETE /api/v1/auth/link/{provider}
```

**Auth:** User

**Path Parameters:**

| Parameter  | Type     | Values                      | Description             |
|:-----------|:---------|:----------------------------|:------------------------|
| `provider` | `string` | `email`, `google`, `github` | The provider to unlink. |

**Response:** `204 No Content`

**Errors:**

| Code  | Condition                                       |
|:------|:------------------------------------------------|
| `422` | Cannot unlink the last remaining auth provider. |
| `404` | The provider is not linked to this account.     |

---

### 2.9 Verify Email

Verifies a user's email address using the token from the verification email.

```
POST /api/v1/auth/verify-email
```

**Auth:** None

**Request Body:**

| Field   | Type     | Required | Description               |
|:--------|:---------|:---------|:--------------------------|
| `token` | `string` | Yes      | Email verification token. |

**Response:** `200 OK`

```json
{
  "message": "Email verified successfully."
}
```

Sets `User.emailVerified` to `true` and clears the token from `AuthProvider`.

**Errors:**

| Code  | Condition                    |
|:------|:-----------------------------|
| `400` | Token is invalid or expired. |

---

### 2.10 Resend Verification Email

Resends the email verification link. Only applicable for users with an email provider whose email is not yet verified.

```
POST /api/v1/auth/resend-verification
```

**Auth:** User

**Response:** `200 OK`

```json
{
  "message": "Verification email sent."
}
```

Generates a new `verificationToken` and `tokenExpiresAt` on the email `AuthProvider`.

**Errors:**

| Code  | Condition                      |
|:------|:-------------------------------|
| `422` | Email is already verified.     |
| `404` | No email auth provider linked. |

---

### 2.11 Request Password Reset

Sends a password reset email. Always returns success to prevent email enumeration.

```
POST /api/v1/auth/password-reset/request
```

**Auth:** None

**Request Body:**

| Field   | Type     | Required | Description                                  |
|:--------|:---------|:---------|:---------------------------------------------|
| `email` | `string` | Yes      | The email address to send the reset link to. |

**Response:** `200 OK`

```json
{
  "message": "If an account with that email exists, a reset link has been sent."
}
```

Generates a single-use reset token stored in `AuthProvider.verificationToken`.

---

### 2.12 Confirm Password Reset

Resets the user's password using the token from the reset email.

```
POST /api/v1/auth/password-reset/confirm
```

**Auth:** None

**Request Body:**

| Field         | Type     | Required | Description           |
|:--------------|:---------|:---------|:----------------------|
| `token`       | `string` | Yes      | Password reset token. |
| `newPassword` | `string` | Yes      | The new password.     |

**Response:** `200 OK`

```json
{
  "message": "Password has been reset."
}
```

Updates `AuthProvider.passwordHash` and clears the token. The token is single-use.

**Errors:**

| Code  | Condition                                    |
|:------|:---------------------------------------------|
| `400` | Token is invalid or expired.                 |
| `400` | New password does not meet complexity rules. |

---

### 2.13 Change Password

Changes the password for a signed-in user with an email auth provider.

```
POST /api/v1/auth/password/change
```

**Auth:** User

**Request Body:**

| Field             | Type     | Required | Description           |
|:------------------|:---------|:---------|:----------------------|
| `currentPassword` | `string` | Yes      | The current password. |
| `newPassword`     | `string` | Yes      | The new password.     |

**Response:** `200 OK`

```json
{
  "message": "Password changed."
}
```

**Errors:**

| Code  | Condition                                    |
|:------|:---------------------------------------------|
| `401` | Current password is incorrect.               |
| `400` | New password does not meet complexity rules. |
| `404` | No email auth provider linked.               |

---

### 2.14 Refresh Token

Exchanges a valid refresh token for a new access token and refresh token pair.

```
POST /api/v1/auth/refresh
```

**Auth:** None (uses refresh token)

**Request Body:**

| Field          | Type     | Required | Description            |
|:---------------|:---------|:---------|:-----------------------|
| `refreshToken` | `string` | Yes      | A valid refresh token. |

**Response:** `200 OK`

```json
{
  "token": "new-jwt-access-token",
  "refreshToken": "new-jwt-refresh-token"
}
```

**Errors:**

| Code  | Condition                            |
|:------|:-------------------------------------|
| `401` | Refresh token is invalid or expired. |

---

### 2.15 Sign Out

Invalidates the current refresh token.

```
POST /api/v1/auth/signout
```

**Auth:** User

**Request Body:**

| Field          | Type     | Required | Description                      |
|:---------------|:---------|:---------|:---------------------------------|
| `refreshToken` | `string` | Yes      | The refresh token to invalidate. |

**Response:** `204 No Content`

---

## 3. Users

### 3.1 Get Current User

Returns the full profile of the authenticated user.

```
GET /api/v1/users/me
```

**Auth:** User

**Response:** `200 OK`

```json
{
  "id": "uuid",
  "createdAt": "2026-01-12T14:32:00Z",
  "updatedAt": "2026-02-07T20:15:00Z",
  "email": "mike@example.com",
  "username": "PixelMike",
  "displayName": "Mike Johnson",
  "avatarUrl": "avatars/mike.png",
  "bio": "Party game addict",
  "emailVerified": true,
  "role": "user",
  "subscriptionPlan": "free",
  "subscriptionExpiresAt": null,
  "accountStatus": "active",
  "lastLoginAt": "2026-02-07T20:15:00Z",
  "authProviders": [
    {
      "provider": "email",
      "providerEmail": "mike@example.com",
      "linkedAt": "2026-01-12T14:32:00Z"
    },
    {
      "provider": "google",
      "providerEmail": "mike@gmail.com",
      "linkedAt": "2026-01-20T10:00:00Z"
    }
  ]
}
```

---

### 3.2 Update Current User

Updates the authenticated user's profile fields.

```
PATCH /api/v1/users/me
```

**Auth:** User

**Request Body (all fields optional):**

| Field         | Type     | Description                |
|:--------------|:---------|:---------------------------|
| `displayName` | `string` | Friendly display name.     |
| `bio`         | `string` | Short profile description. |
| `avatarUrl`   | `string` | URL to profile picture.    |

**Response:** `200 OK` — Returns the updated user object (same shape as GET `/users/me`).

**Errors:**

| Code  | Condition                      |
|:------|:-------------------------------|
| `400` | A field exceeds length limits. |

> **Note:** `username` and `email` are changed through dedicated endpoints (`PATCH /users/me/username` and `PATCH /users/me/email`) because they involve uniqueness checks and verification flows.

---

### 3.3 Get Public User Profile

Returns the public profile of a user by username.

```
GET /api/v1/users/{username}
```

**Auth:** None

**Path Parameters:**

| Parameter  | Type     | Description                 |
|:-----------|:---------|:----------------------------|
| `username` | `string` | The user's unique username. |

**Response:** `200 OK`

```json
{
  "id": "uuid",
  "username": "PixelMike",
  "displayName": "Mike Johnson",
  "avatarUrl": "avatars/mike.png",
  "bio": "Party game addict",
  "createdAt": "2026-01-12T14:32:00Z",
  "stats": {
    "gamesPublished": 5,
    "totalPlayCount": 3200
  }
}
```

**Errors:**

| Code  | Condition                                                  |
|:------|:-----------------------------------------------------------|
| `404` | User not found, is deactivated, or has no published games. |

---

### 3.4 Upload Avatar

Uploads a profile picture for the authenticated user.

```
POST /api/v1/users/me/avatar
```

**Auth:** User

**Request:** `multipart/form-data`

| Field  | Type   | Required | Description                         |
|:-------|:-------|:---------|:------------------------------------|
| `file` | `file` | Yes      | Image file (PNG, JPG, SVG, or GIF). |

**Response:** `200 OK`

```json
{
  "avatarUrl": "avatars/a8f3e7b1.png"
}
```

**Errors:**

| Code  | Condition                    |
|:------|:-----------------------------|
| `400` | Unsupported file type.       |
| `413` | File exceeds the size limit. |

---

### 3.5 Delete Avatar

Removes the authenticated user's profile picture and reverts to the default.

```
DELETE /api/v1/users/me/avatar
```

**Auth:** User

**Response:** `204 No Content`

---

### 3.6 Change Username

Updates the authenticated user's username. Subject to uniqueness validation.

```
PATCH /api/v1/users/me/username
```

**Auth:** User

**Request Body:**

| Field         | Type     | Required | Description                                                               |
|:--------------|:---------|:---------|:--------------------------------------------------------------------------|
| `newUsername` | `string` | Yes      | The new username. 3–30 characters, alphanumeric plus hyphens/underscores. |

**Response:** `200 OK`

```json
{
  "username": "NewPixelMike"
}
```

**Errors:**

| Code  | Condition                                  |
|:------|:-------------------------------------------|
| `409` | The new username is already in use.        |
| `400` | Username does not meet format constraints. |

---

### 3.7 Change Email

Updates the user's email address and triggers re-verification.

```
PATCH /api/v1/users/me/email
```

**Auth:** User

**Request Body:**

| Field      | Type     | Required | Description                                                                                        |
|:-----------|:---------|:---------|:---------------------------------------------------------------------------------------------------|
| `newEmail` | `string` | Yes      | The new email.                                                                                     |
| `password` | `string` | Cond.    | Required if the user has an email auth provider. Omit if the user only has OAuth providers linked. |

**Response:** `200 OK`

```json
{
  "message": "Email updated. A verification email has been sent to the new address.",
  "email": "newemail@example.com",
  "emailVerified": false
}
```

Sets `emailVerified` to `false` and sends a new verification email.

**Errors:**

| Code  | Condition                                                      |
|:------|:---------------------------------------------------------------|
| `409` | The new email is already in use.                               |
| `401` | Password is incorrect (when an email auth provider is linked). |

---

### 3.8 Deactivate Account

Soft-deletes the authenticated user's account. Profile and published games are hidden. Data is retained for potential reactivation.

```
DELETE /api/v1/users/me
```

**Auth:** User

**Request Body:**

| Field      | Type     | Required | Description                                                                                        |
|:-----------|:---------|:---------|:---------------------------------------------------------------------------------------------------|
| `password` | `string` | Cond.    | Required if the user has an email auth provider. Omit if the user only has OAuth providers linked. |

**Response:** `204 No Content`

Sets `User.accountStatus` to `"deactivated"` and sets `User.deletedAt`.

**Errors:**

| Code  | Condition                                                      |
|:------|:---------------------------------------------------------------|
| `401` | Password is incorrect (when an email auth provider is linked). |

---

### 3.9 Get Creator Statistics

Returns aggregate statistics for the authenticated user's published games.

```
GET /api/v1/users/me/stats
```

**Auth:** User

**Response:** `200 OK`

```json
{
  "gamesPublished": 5,
  "totalPlayCount": 3200,
  "totalPlayTime": 245000,
  "averageRating": 4.1,
  "totalReviews": 87,
  "games": [
    {
      "gameId": "uuid",
      "title": "Space Race",
      "playCount": 1247,
      "totalPlayTime": 89340,
      "avgRating": 4.3,
      "reviewCount": 42
    }
  ]
}
```

---

## 4. Games

### 4.1 Create Game

Creates a new game project in draft status.

```
POST /api/v1/games
```

**Auth:** User

**Request Body:**

| Field                  | Type      | Required | Default     | Description                         |
|:-----------------------|:----------|:---------|:------------|:------------------------------------|
| `title`                | `string`  | Yes      |             | Game title.                         |
| `description`          | `string`  | No       | `null`      | Game description.                   |
| `technology`           | `string`  | No       | `"p5js"`    | Runtime: `p5js`.                    |
| `minPlayers`           | `integer` | No       | `1`         | Minimum players (≥ 1).              |
| `maxPlayers`           | `integer` | No       | `8`         | Maximum players (≥ `minPlayers`).   |
| `visibility`           | `string`  | No       | `"private"` | `public`, `private`, or `unlisted`. |
| `remixable`            | `boolean` | No       | `false`     | Allow others to fork.               |
| `gameScreenCode`       | `text`    | No       | `null`      | Game Screen source code.            |
| `controllerScreenCode` | `text`    | No       | `null`      | Controller Screen source code.      |

**Response:** `201 Created`

```json
{
  "id": "uuid",
  "createdAt": "2026-02-08T15:30:00Z",
  "updatedAt": "2026-02-08T15:30:00Z",
  "creatorId": "uuid",
  "title": "Space Race",
  "description": null,
  "thumbnailUrl": null,
  "technology": "p5js",
  "minPlayers": 2,
  "maxPlayers": 8,
  "status": "draft",
  "visibility": "private",
  "remixable": false,
  "forkedFromId": null,
  "gameScreenCode": null,
  "controllerScreenCode": null,
  "publishedVersionId": null,
  "playCount": 0,
  "totalPlayTime": 0,
  "avgRating": 0.0,
  "reviewCount": 0
}
```

**Errors:**

| Code  | Condition                                     |
|:------|:----------------------------------------------|
| `400` | Missing title or `maxPlayers` < `minPlayers`. |

---

### 4.2 Get Game

Returns the details of a single game. Respects visibility rules.

```
GET /api/v1/games/{gameId}
```

**Auth:** None (public/unlisted games), User (private games — creator only)

**Path Parameters:**

| Parameter | Type   | Description    |
|:----------|:-------|:---------------|
| `gameId`  | `UUID` | The game's ID. |

**Response:** `200 OK`

Returns the full game object (same shape as the creation response) plus the creator's public profile:

```json
{
  "id": "uuid",
  "createdAt": "...",
  "updatedAt": "...",
  "creatorId": "uuid",
  "creator": {
    "username": "PixelMike",
    "displayName": "Mike Johnson",
    "avatarUrl": "avatars/mike.png"
  },
  "title": "Space Race",
  "description": "Race through asteroids!",
  "thumbnailUrl": "thumbnails/space-race.png",
  "technology": "p5js",
  "minPlayers": 2,
  "maxPlayers": 8,
  "status": "published",
  "visibility": "public",
  "remixable": true,
  "forkedFromId": null,
  "publishedVersionId": "uuid",
  "playCount": 1247,
  "totalPlayTime": 89340,
  "avgRating": 4.3,
  "reviewCount": 42,
  "tags": [
    {
      "id": "uuid",
      "name": "Racing",
      "slug": "racing",
      "category": "genre"
    }
  ]
}
```

> **Note:** `gameScreenCode` and `controllerScreenCode` are only included when the requester is the game's creator.

**Errors:**

| Code  | Condition                                                         |
|:------|:------------------------------------------------------------------|
| `404` | Game not found, is soft-deleted, or not visible to the requester. |

---

### 4.3 Update Game

Updates a game's metadata or code. Only the creator can update.

```
PATCH /api/v1/games/{gameId}
```

**Auth:** User (creator only)

**Request Body (all fields optional):**

| Field                  | Type      | Description                         |
|:-----------------------|:----------|:------------------------------------|
| `title`                | `string`  | Game title.                         |
| `description`          | `string`  | Game description.                   |
| `thumbnailUrl`         | `string`  | Cover image URL.                    |
| `minPlayers`           | `integer` | Minimum players.                    |
| `maxPlayers`           | `integer` | Maximum players.                    |
| `visibility`           | `string`  | `public`, `private`, or `unlisted`. |
| `remixable`            | `boolean` | Allow forking.                      |
| `gameScreenCode`       | `text`    | Game Screen source code.            |
| `controllerScreenCode` | `text`    | Controller Screen source code.      |

**Response:** `200 OK` — Returns the updated game object.

**Errors:**

| Code  | Condition                                  |
|:------|:-------------------------------------------|
| `403` | The authenticated user is not the creator. |
| `400` | `maxPlayers` < `minPlayers`.               |
| `404` | Game not found.                            |

---

### 4.4 Delete Game

Soft-deletes a game. Available to the creator or a moderator.

```
DELETE /api/v1/games/{gameId}
```

**Auth:** User (creator) or Moderator

**Response:** `204 No Content`

Sets `Game.deletedAt`. The game is excluded from all public queries. Related assets are also soft-deleted.

**Errors:**

| Code  | Condition                                        |
|:------|:-------------------------------------------------|
| `403` | The user is not the creator and not a moderator. |
| `404` | Game not found.                                  |

---

### 4.5 Publish Game

Publishes the game by creating an immutable `GameVersion` snapshot of the current code.

```
POST /api/v1/games/{gameId}/publish
```

**Auth:** User (creator only, email must be verified)

**Request Body:**

| Field       | Type     | Required | Description                                  |
|:------------|:---------|:---------|:---------------------------------------------|
| `changelog` | `string` | No       | Description of what changed in this version. |

**Response:** `201 Created`

```json
{
  "version": {
    "id": "uuid",
    "createdAt": "2026-02-08T16:00:00Z",
    "gameId": "uuid",
    "versionNumber": 3,
    "changelog": "Added power-ups",
    "publishedById": "uuid"
  },
  "game": {
    "id": "uuid",
    "status": "published",
    "publishedVersionId": "uuid"
  }
}
```

Steps performed:

1. Validates the game has a title and at least one non-empty canvas code.
2. Increments `versionNumber` (starting from 1).
3. Copies `gameScreenCode` and `controllerScreenCode` into a new `GameVersion`.
4. Updates `Game.publishedVersionId` and sets `Game.status` to `"published"`.

**Errors:**

| Code  | Condition                                               |
|:------|:--------------------------------------------------------|
| `422` | Game has no title or both canvas codes are empty.       |
| `422` | Creator's email is not verified (`EMAIL_NOT_VERIFIED`). |
| `403` | The user is not the creator.                            |
| `404` | Game not found.                                         |

---

### 4.6 Archive Game

Sets the game status to `archived`. Removes it from public discovery. Existing direct links still work but the game cannot be loaded into new sessions.

```
POST /api/v1/games/{gameId}/archive
```

**Auth:** User (creator) or Moderator

**Response:** `200 OK` — Returns the updated game object.

**Errors:**

| Code  | Condition                                    |
|:------|:---------------------------------------------|
| `422` | Game is already archived.                    |
| `403` | User is not the creator and not a moderator. |
| `404` | Game not found.                              |

---

### 4.7 Unarchive Game

Returns an archived game to its previous status (`draft` if never published, `published` if it has a `publishedVersionId`).

```
POST /api/v1/games/{gameId}/unarchive
```

**Auth:** User (creator only)

**Response:** `200 OK` — Returns the updated game object.

**Errors:**

| Code  | Condition                       |
|:------|:--------------------------------|
| `422` | Game is not currently archived. |
| `403` | User is not the creator.        |
| `404` | Game not found.                 |

---

### 4.8 Fork Game

Creates a copy of a remixable game under the authenticated user's account.

```
POST /api/v1/games/{gameId}/fork
```

**Auth:** User

**Response:** `201 Created`

Returns the newly created game object with:

- `creatorId` set to the authenticated user.
- `forkedFromId` set to the source game's ID.
- Code copied from the source game's published version.
- `status: "draft"`, `visibility: "private"`.

**Errors:**

| Code  | Condition                                 |
|:------|:------------------------------------------|
| `422` | The source game is not `remixable`.       |
| `422` | The source game has no published version. |
| `404` | Game not found.                           |

---

### 4.9 List Game Versions

Returns all published versions of a game, ordered by version number descending.

```
GET /api/v1/games/{gameId}/versions
```

**Auth:** None (published games), User (creator for draft games)

**Query Parameters:** Supports [pagination](#14-pagination).

**Response:** `200 OK`

```json
{
  "data": [
    {
      "id": "uuid",
      "createdAt": "2026-02-05T12:00:00Z",
      "versionNumber": 3,
      "changelog": "Added power-ups",
      "publishedById": "uuid"
    }
  ],
  "total": 3,
  "offset": 0,
  "limit": 20
}
```

> **Note:** Game code is not included in the list response. Use the detail endpoint to retrieve code.

---

### 4.10 Get Game Version

Returns the details of a specific published version, including the frozen source code.

```
GET /api/v1/games/{gameId}/versions/{versionNumber}
```

**Auth:** None (published games), User (creator for draft games)

**Path Parameters:**

| Parameter       | Type      | Description         |
|:----------------|:----------|:--------------------|
| `gameId`        | `UUID`    | The game's ID.      |
| `versionNumber` | `integer` | The version number. |

**Response:** `200 OK`

```json
{
  "id": "uuid",
  "createdAt": "2026-02-05T12:00:00Z",
  "gameId": "uuid",
  "versionNumber": 3,
  "gameScreenCode": "...",
  "controllerScreenCode": "...",
  "changelog": "Added power-ups",
  "publishedById": "uuid"
}
```

**Errors:**

| Code  | Condition                  |
|:------|:---------------------------|
| `404` | Game or version not found. |

---

### 4.11 Upload Game Asset

Uploads a media file (image, sound, font) attached to a game.

```
POST /api/v1/games/{gameId}/assets
```

**Auth:** User (creator only)

**Request:** `multipart/form-data`

| Field  | Type   | Required | Description                                                                         |
|:-------|:-------|:---------|:------------------------------------------------------------------------------------|
| `file` | `file` | Yes      | The file to upload. Supported types: PNG, JPG, SVG, GIF, MP3, WAV, OGG, TTF, WOFF2. |

**Response:** `201 Created`

```json
{
  "id": "uuid",
  "createdAt": "2026-02-03T09:15:00Z",
  "gameId": "uuid",
  "fileName": "spaceship.png",
  "fileType": "image/png",
  "fileSize": 24576,
  "storageUrl": "assets/d7a1c3e5/spaceship.png"
}
```

**Errors:**

| Code  | Condition                                      |
|:------|:-----------------------------------------------|
| `400` | Unsupported file type.                         |
| `403` | User is not the game's creator.                |
| `413` | File exceeds the size limit.                   |
| `422` | Game has reached the maximum number of assets. |
| `404` | Game not found.                                |

---

### 4.12 List Game Assets

Returns all assets attached to a game.

```
GET /api/v1/games/{gameId}/assets
```

**Auth:** User (creator only)

**Response:** `200 OK`

```json
{
  "data": [
    {
      "id": "uuid",
      "createdAt": "2026-02-03T09:15:00Z",
      "fileName": "spaceship.png",
      "fileType": "image/png",
      "fileSize": 24576,
      "storageUrl": "assets/d7a1c3e5/spaceship.png"
    }
  ],
  "total": 3,
  "offset": 0,
  "limit": 20
}
```

---

### 4.13 Get Game Asset

Returns details for a single asset.

```
GET /api/v1/games/{gameId}/assets/{assetId}
```

**Auth:** User (creator only)

**Response:** `200 OK` — Returns the asset object.

**Errors:**

| Code  | Condition                |
|:------|:-------------------------|
| `404` | Game or asset not found. |

---

### 4.14 Delete Game Asset

Permanently deletes an asset from a game.

```
DELETE /api/v1/games/{gameId}/assets/{assetId}
```

**Auth:** User (creator only)

**Response:** `204 No Content`

**Errors:**

| Code  | Condition                       |
|:------|:--------------------------------|
| `403` | User is not the game's creator. |
| `404` | Game or asset not found.        |

---

### 4.15 List Tags

Returns all platform-managed tags, optionally filtered by category.

```
GET /api/v1/tags
```

**Auth:** None

**Query Parameters:**

| Parameter  | Type     | Required | Description                                            |
|:-----------|:---------|:---------|:-------------------------------------------------------|
| `category` | `string` | No       | Filter by category: `genre`, `mood`, or `playerStyle`. |

**Response:** `200 OK`

```json
{
  "data": [
    {
      "id": "uuid",
      "name": "Racing",
      "slug": "racing",
      "category": "genre"
    },
    {
      "id": "uuid",
      "name": "Trivia",
      "slug": "trivia",
      "category": "genre"
    }
  ]
}
```

---

### 4.16 Set Game Tags

Replaces all tags on a game with the provided set.

```
PUT /api/v1/games/{gameId}/tags
```

**Auth:** User (creator only)

**Request Body:**

| Field    | Type     | Required | Description                 |
|:---------|:---------|:---------|:----------------------------|
| `tagIds` | `UUID[]` | Yes      | Array of tag IDs to assign. |

**Response:** `200 OK`

```json
{
  "tags": [
    {
      "id": "uuid",
      "name": "Racing",
      "slug": "racing",
      "category": "genre"
    }
  ]
}
```

**Errors:**

| Code  | Condition                         |
|:------|:----------------------------------|
| `400` | One or more tag IDs do not exist. |
| `403` | User is not the game's creator.   |
| `404` | Game not found.                   |

---

### 4.17 Get Game Tags

Returns the tags assigned to a game.

```
GET /api/v1/games/{gameId}/tags
```

**Auth:** None

**Response:** `200 OK`

```json
{
  "tags": [
    {
      "id": "uuid",
      "name": "Racing",
      "slug": "racing",
      "category": "genre"
    },
    {
      "id": "uuid",
      "name": "Competitive",
      "slug": "competitive",
      "category": "mood"
    }
  ]
}
```

---

### 4.18 List My Games

Returns all games created by the authenticated user, across all statuses.

```
GET /api/v1/users/me/games
```

**Auth:** User

**Query Parameters:** Supports [pagination](#14-pagination).

| Parameter | Type     | Required | Description                                         |
|:----------|:---------|:---------|:----------------------------------------------------|
| `status`  | `string` | No       | Filter by status: `draft`, `published`, `archived`. |

**Response:** `200 OK` — Paginated list of game objects (without source code fields).

---

### 4.19 List User's Public Games

Returns the published, public games of a user.

```
GET /api/v1/users/{username}/games
```

**Auth:** None

**Path Parameters:**

| Parameter  | Type     | Description             |
|:-----------|:---------|:------------------------|
| `username` | `string` | The creator's username. |

**Query Parameters:** Supports [pagination](#14-pagination).

**Response:** `200 OK` — Paginated list of game summary objects.

**Errors:**

| Code  | Condition       |
|:------|:----------------|
| `404` | User not found. |

---

## 5. Sessions

### 5.1 Create Session

Creates a new live session in lobby status and generates a unique session code.

```
POST /api/v1/sessions
```

**Auth:** User

**Request Body:**

| Field        | Type      | Required | Default | Description                                                                                                      |
|:-------------|:----------|:---------|:--------|:-----------------------------------------------------------------------------------------------------------------|
| `maxPlayers` | `integer` | No       | `8`     | Maximum players allowed. Capped by subscription tier (free: 8, pro: unlimited).                                  |
| `testGameId` | `UUID`    | No       | `null`  | If provided, creates a test session that loads draft code from this game. The game must belong to the host user. |

**Response:** `201 Created`

```json
{
  "id": "uuid",
  "createdAt": "2026-02-07T19:00:00Z",
  "updatedAt": "2026-02-07T19:00:00Z",
  "endedAt": null,
  "hostId": "uuid",
  "gameId": null,
  "gameVersionId": null,
  "sessionCode": "XKCD42",
  "status": "lobby",
  "maxPlayers": 8
}
```

If `testGameId` is provided, the session is created with `gameId` set and the response includes the draft game code. The session transitions directly to `playing` status. Test sessions do not create `PlayHistory` records and do not update game play counters.

The `sessionCode` consists of 4–6 uppercase alphanumeric characters, excluding visually ambiguous characters (0/O, 1/I/L). Codes are unique among active (non-ended) sessions.

---

### 5.2 Get Session

Returns the details of a session by its session code.

```
GET /api/v1/sessions/{sessionCode}
```

**Auth:** None

**Path Parameters:**

| Parameter     | Type     | Description                                          |
|:--------------|:---------|:-----------------------------------------------------|
| `sessionCode` | `string` | The session's human-readable code. Case-insensitive. |

**Response:** `200 OK`

```json
{
  "id": "uuid",
  "createdAt": "2026-02-07T19:00:00Z",
  "updatedAt": "2026-02-07T19:45:00Z",
  "endedAt": null,
  "hostId": "uuid",
  "sessionCode": "XKCD42",
  "status": "playing",
  "maxPlayers": 8,
  "game": {
    "id": "uuid",
    "title": "Space Race",
    "thumbnailUrl": "thumbnails/space-race.png",
    "minPlayers": 2,
    "maxPlayers": 8
  },
  "playerCount": 4
}
```

**Errors:**

| Code  | Condition                   |
|:------|:----------------------------|
| `404` | Session not found or ended. |

---

### 5.3 Update Session

Updates session settings. Only the host can update.

```
PATCH /api/v1/sessions/{sessionId}
```

**Auth:** User (host only)

**Path Parameters:**

| Parameter   | Type   | Description       |
|:------------|:-------|:------------------|
| `sessionId` | `UUID` | The session's ID. |

**Request Body (all fields optional):**

| Field        | Type      | Description           |
|:-------------|:----------|:----------------------|
| `maxPlayers` | `integer` | New max player count. |

**Response:** `200 OK` — Returns the updated session object.

**Errors:**

| Code  | Condition             |
|:------|:----------------------|
| `403` | User is not the host. |
| `404` | Session not found.    |

---

### 5.4 End Session

Ends a session. All WebSocket connections are closed and the session code is freed.

```
DELETE /api/v1/sessions/{sessionId}
```

**Auth:** User (host only)

**Response:** `204 No Content`

Sets `Session.status` to `"ended"` and `Session.endedAt` to the current timestamp. Updates `PlayHistory.duration` for all active players if a game was running. Triggers aggregate counter updates on the `Game` entity.

**Errors:**

| Code  | Condition                           |
|:------|:------------------------------------|
| `403` | User is not the host.               |
| `404` | Session not found or already ended. |

---

### 5.5 Load Game into Session

Loads a published game into the session and transitions to playing.

```
POST /api/v1/sessions/{sessionId}/game
```

**Auth:** User (host only)

**Request Body:**

| Field    | Type   | Required | Description                 |
|:---------|:-------|:---------|:----------------------------|
| `gameId` | `UUID` | Yes      | The published game to load. |

**Response:** `200 OK`

```json
{
  "session": {
    "id": "uuid",
    "status": "playing",
    "gameId": "uuid",
    "gameVersionId": "uuid"
  },
  "gameVersion": {
    "id": "uuid",
    "versionNumber": 3,
    "gameScreenCode": "...",
    "controllerScreenCode": "..."
  }
}
```

Steps performed:

1. Validates the game is `published` and has a `publishedVersionId`.
2. Sets `Session.gameId` and `Session.gameVersionId`.
3. Transitions `Session.status` to `"playing"`.
4. Creates `PlayHistory` records for each connected player.
5. Delivers game code via **two channels**: the REST response returns `gameScreenCode` to the host who made the request, and a `game_loaded` WebSocket message delivers `controllerScreenCode` to all connected players. The host also receives the `game_loaded` message with `gameScreenCode` via WebSocket for consistency.

**Errors:**

| Code  | Condition                                                                            |
|:------|:-------------------------------------------------------------------------------------|
| `403` | User is not the host.                                                                |
| `404` | Session not found, or game not found.                                                |
| `422` | Game is not published or has no published version.                                   |
| `422` | Session is not in `lobby` status.                                                    |
| `422` | Current player count exceeds the game's `maxPlayers`. Remove players before loading. |

---

### 5.6 Unload Game (Return to Lobby)

Stops the current game and returns the session to lobby. Player connections remain active.

```
DELETE /api/v1/sessions/{sessionId}/game
```

**Auth:** User (host only)

**Response:** `204 No Content`

Clears `Session.gameId` and `Session.gameVersionId`. Sets `Session.status` to `"lobby"`. Updates `PlayHistory.duration` for the ended game. Triggers aggregate counter updates on the `Game` entity.

**Errors:**

| Code  | Condition                    |
|:------|:-----------------------------|
| `403` | User is not the host.        |
| `422` | No game is currently loaded. |

---

### 5.7 Pause Game

Pauses the currently playing game. Players remain connected but no input is processed.

```
POST /api/v1/sessions/{sessionId}/pause
```

**Auth:** User (host only)

**Response:** `200 OK`

```json
{
  "status": "paused"
}
```

**Errors:**

| Code  | Condition                           |
|:------|:------------------------------------|
| `403` | User is not the host.               |
| `422` | Session is not in `playing` status. |

---

### 5.8 Resume Game

Resumes a paused game.

```
POST /api/v1/sessions/{sessionId}/resume
```

**Auth:** User (host only)

**Response:** `200 OK`

```json
{
  "status": "playing"
}
```

**Errors:**

| Code  | Condition                          |
|:------|:-----------------------------------|
| `403` | User is not the host.              |
| `422` | Session is not in `paused` status. |

---

### 5.9 Restart Game

Restarts the currently loaded game. Resets game state by re-executing the setup phase.

```
POST /api/v1/sessions/{sessionId}/restart
```

**Auth:** User (host only)

**Response:** `200 OK`

```json
{
  "status": "playing"
}
```

Sends a restart signal to all connected clients via WebSocket, causing both canvases to re-run their setup phase.

**Errors:**

| Code  | Condition                    |
|:------|:-----------------------------|
| `403` | User is not the host.        |
| `422` | No game is currently loaded. |

---

### 5.10 Join Session

Joins an active session as a player.

```
POST /api/v1/sessions/{sessionCode}/join
```

**Auth:** None (guest players supported), User (optional)

**Path Parameters:**

| Parameter     | Type     | Description                         |
|:--------------|:---------|:------------------------------------|
| `sessionCode` | `string` | The session code. Case-insensitive. |

**Request Body:**

| Field         | Type     | Required | Description                          |
|:--------------|:---------|:---------|:-------------------------------------|
| `displayName` | `string` | Yes      | Name shown in the lobby and in-game. |
| `avatarUrl`   | `string` | No       | Optional avatar URL.                 |

**Response:** `201 Created`

```json
{
  "player": {
    "id": "uuid",
    "createdAt": "2026-02-07T19:02:00Z",
    "sessionId": "uuid",
    "userId": "uuid-or-null",
    "displayName": "Player 3",
    "avatarUrl": null,
    "connectionStatus": "connected"
  },
  "session": {
    "id": "uuid",
    "sessionCode": "XKCD42",
    "status": "lobby"
  }
}
```

If the player is authenticated, `Player.userId` is set automatically.

**Errors:**

| Code  | Condition                         |
|:------|:----------------------------------|
| `404` | Session not found or ended.       |
| `422` | Session has reached `maxPlayers`. |

---

### 5.11 List Players

Returns all players in a session.

```
GET /api/v1/sessions/{sessionId}/players
```

**Auth:** None

**Response:** `200 OK`

```json
{
  "data": [
    {
      "id": "uuid",
      "displayName": "Player 3",
      "avatarUrl": null,
      "connectionStatus": "connected",
      "joinedAt": "2026-02-07T19:02:00Z"
    }
  ]
}
```

---

### 5.12 Remove Player

Removes a player from the session. Only the host can remove players.

```
DELETE /api/v1/sessions/{sessionId}/players/{playerId}
```

**Auth:** User (host only)

**Response:** `204 No Content`

Sets `Player.leftAt` and `Player.connectionStatus` to `"disconnected"`. The player's WebSocket connection is closed.

**Errors:**

| Code  | Condition                    |
|:------|:-----------------------------|
| `403` | User is not the host.        |
| `404` | Session or player not found. |

---

### 5.13 Update Player

Allows a player to update their display name or avatar within a session.

```
PATCH /api/v1/sessions/{sessionId}/players/me
```

**Auth:** None (uses player context from WebSocket or session token)

**Request Body (all fields optional):**

| Field         | Type     | Description           |
|:--------------|:---------|:----------------------|
| `displayName` | `string` | Updated display name. |
| `avatarUrl`   | `string` | Updated avatar URL.   |

**Response:** `200 OK` — Returns the updated player object.

---

### 5.14 List My Sessions

Returns sessions hosted by the authenticated user, ordered by most recent.

```
GET /api/v1/users/me/sessions
```

**Auth:** User

**Query Parameters:** Supports [pagination](#14-pagination).

| Parameter | Type     | Required | Description                                    |
|:----------|:---------|:---------|:-----------------------------------------------|
| `status`  | `string` | No       | Filter: `lobby`, `playing`, `paused`, `ended`. |

**Response:** `200 OK` — Paginated list of session summary objects.

---

## 6. Community

### 6.1 Create or Update Review

Creates a review or updates the existing review if one already exists for this user and game (upsert). One review per user per game.

```
PUT /api/v1/games/{gameId}/reviews
```

**Auth:** User

**Request Body:**

| Field    | Type      | Required | Description           |
|:---------|:----------|:---------|:----------------------|
| `rating` | `integer` | Yes      | Star rating, 1–5.     |
| `title`  | `string`  | No       | Optional headline.    |
| `body`   | `string`  | No       | Optional review text. |

**Response:** `200 OK` (updated) or `201 Created` (new)

```json
{
  "id": "uuid",
  "createdAt": "2026-02-06T14:00:00Z",
  "updatedAt": "2026-02-06T14:00:00Z",
  "userId": "uuid",
  "gameId": "uuid",
  "rating": 4,
  "title": "Great party game!",
  "body": "My friends loved it.",
  "user": {
    "username": "PixelMike",
    "displayName": "Mike Johnson",
    "avatarUrl": "avatars/mike.png"
  }
}
```

Triggers recalculation of `Game.avgRating` and `Game.reviewCount`.

**Errors:**

| Code  | Condition                                      |
|:------|:-----------------------------------------------|
| `400` | Rating is not between 1 and 5.                 |
| `403` | User is the creator of the game (self-review). |
| `404` | Game not found.                                |

---

### 6.2 List Game Reviews

Returns reviews for a game, paginated.

```
GET /api/v1/games/{gameId}/reviews
```

**Auth:** None

**Query Parameters:** Supports [pagination](#14-pagination).

| Parameter | Type     | Required | Description                              |
|:----------|:---------|:---------|:-----------------------------------------|
| `sort`    | `string` | No       | `newest` (default), `highest`, `lowest`. |

**Response:** `200 OK`

```json
{
  "data": [
    {
      "id": "uuid",
      "createdAt": "2026-02-06T14:00:00Z",
      "updatedAt": "2026-02-06T14:00:00Z",
      "rating": 4,
      "title": "Great party game!",
      "body": "My friends loved it.",
      "user": {
        "username": "PixelMike",
        "displayName": "Mike Johnson",
        "avatarUrl": "avatars/mike.png"
      }
    }
  ],
  "total": 42,
  "offset": 0,
  "limit": 20
}
```

---

### 6.3 Get My Review

Returns the authenticated user's review for a specific game, or `404` if none exists.

```
GET /api/v1/games/{gameId}/reviews/me
```

**Auth:** User

**Response:** `200 OK` — Returns the review object.

**Errors:**

| Code  | Condition                             |
|:------|:--------------------------------------|
| `404` | No review by this user for this game. |

---

### 6.4 Delete My Review

Soft-deletes the authenticated user's review.

```
DELETE /api/v1/games/{gameId}/reviews/me
```

**Auth:** User

**Response:** `204 No Content`

Sets `Review.deletedAt`. Triggers recalculation of `Game.avgRating` and `Game.reviewCount`.

**Errors:**

| Code  | Condition                             |
|:------|:--------------------------------------|
| `404` | No review by this user for this game. |

---

### 6.5 Delete Review (Moderation)

Soft-deletes any review. Available to moderators and admins.

```
DELETE /api/v1/games/{gameId}/reviews/{reviewId}
```

**Auth:** Moderator

**Response:** `204 No Content`

**Errors:**

| Code  | Condition         |
|:------|:------------------|
| `404` | Review not found. |

---

### 6.6 Toggle Favorite

Toggles the favorite status of a game. If not favorited, adds a favorite. If already favorited, removes it.

```
POST /api/v1/games/{gameId}/favorite
```

**Auth:** User

**Response:** `200 OK`

```json
{
  "favorited": true
}
```

or

```json
{
  "favorited": false
}
```

**Errors:**

| Code  | Condition       |
|:------|:----------------|
| `404` | Game not found. |

---

### 6.7 Check Favorite Status

Returns whether the authenticated user has favorited a game.

```
GET /api/v1/games/{gameId}/favorite
```

**Auth:** User

**Response:** `200 OK`

```json
{
  "favorited": true
}
```

---

### 6.8 List My Favorites

Returns the authenticated user's favorited games, ordered by most recently favorited.

```
GET /api/v1/users/me/favorites
```

**Auth:** User

**Query Parameters:** Supports [pagination](#14-pagination).

**Response:** `200 OK`

```json
{
  "data": [
    {
      "favoritedAt": "2026-02-05T20:00:00Z",
      "game": {
        "id": "uuid",
        "title": "Space Race",
        "thumbnailUrl": "thumbnails/space-race.png",
        "minPlayers": 2,
        "maxPlayers": 8,
        "avgRating": 4.3,
        "playCount": 1247,
        "creator": {
          "username": "PixelMike",
          "avatarUrl": "avatars/mike.png"
        }
      }
    }
  ],
  "total": 12,
  "offset": 0,
  "limit": 20
}
```

---

### 6.9 Get Play History

Returns the authenticated user's play history, ordered by most recent.

```
GET /api/v1/users/me/history
```

**Auth:** User

**Query Parameters:** Supports [pagination](#14-pagination).

**Response:** `200 OK`

```json
{
  "data": [
    {
      "id": "uuid",
      "playedAt": "2026-02-07T19:05:00Z",
      "duration": 720,
      "game": {
        "id": "uuid",
        "title": "Space Race",
        "thumbnailUrl": "thumbnails/space-race.png"
      }
    }
  ],
  "total": 34,
  "offset": 0,
  "limit": 20
}
```

---

### 6.10 Get Game Rating Summary

Returns the aggregate rating data for a game.

```
GET /api/v1/games/{gameId}/rating
```

**Auth:** None

**Response:** `200 OK`

```json
{
  "avgRating": 4.3,
  "reviewCount": 42,
  "distribution": {
    "1": 2,
    "2": 3,
    "3": 5,
    "4": 14,
    "5": 18
  }
}
```

---

### 6.11 Get Game Statistics

Returns play statistics for a game. Available to the game's creator.

```
GET /api/v1/games/{gameId}/stats
```

**Auth:** User (creator only)

**Response:** `200 OK`

```json
{
  "playCount": 1247,
  "totalPlayTime": 89340,
  "avgSessionDuration": 432,
  "avgRating": 4.3,
  "reviewCount": 42
}
```

**Errors:**

| Code  | Condition                       |
|:------|:--------------------------------|
| `403` | User is not the game's creator. |
| `404` | Game not found.                 |

---

## 7. Discovery

### 7.1 Browse Games

Browses the game library. Returns only `published` and `public` games.

```
GET /api/v1/library/games
```

**Auth:** None

**Query Parameters:**

| Parameter    | Type       | Required | Description                                                       |
|:-------------|:-----------|:---------|:------------------------------------------------------------------|
| `tags`       | `string[]` | No       | Filter by tag slugs (comma-separated). Matches any.               |
| `minPlayers` | `integer`  | No       | Filter games that support at least this many players.             |
| `maxPlayers` | `integer`  | No       | Filter games that support at most this many players.              |
| `technology` | `string`   | No       | Filter by runtime: `p5js`.                                        |
| `sort`       | `string`   | No       | `popular`, `top-rated`, `newest`, `trending`. Default: `popular`. |
| `offset`     | `integer`  | No       | Pagination offset. Default: `0`.                                  |
| `limit`      | `integer`  | No       | Pagination limit. Default: `20`, max: `100`.                      |

**Response:** `200 OK`

```json
{
  "data": [
    {
      "id": "uuid",
      "title": "Space Race",
      "description": "Race through asteroids!",
      "thumbnailUrl": "thumbnails/space-race.png",
      "technology": "p5js",
      "minPlayers": 2,
      "maxPlayers": 8,
      "playCount": 1247,
      "avgRating": 4.3,
      "reviewCount": 42,
      "createdAt": "2026-02-01T10:00:00Z",
      "creator": {
        "username": "PixelMike",
        "avatarUrl": "avatars/mike.png"
      },
      "tags": [
        {
          "name": "Racing",
          "slug": "racing",
          "category": "genre"
        }
      ]
    }
  ],
  "total": 142,
  "offset": 0,
  "limit": 20
}
```

Sorting rules:

- `popular` — ordered by `playCount` descending.
- `top-rated` — ordered by `avgRating` descending (minimum review threshold applied).
- `newest` — ordered by publish date descending.
- `trending` — ordered by recent play count growth.

---

### 7.2 Search Games

Full-text search across game titles, descriptions, creator usernames, and tag names.

```
GET /api/v1/library/search
```

**Auth:** None

**Query Parameters:**

| Parameter | Type      | Required | Description          |
|:----------|:----------|:---------|:---------------------|
| `q`       | `string`  | Yes      | Search query string. |
| `offset`  | `integer` | No       | Pagination offset.   |
| `limit`   | `integer` | No       | Pagination limit.    |

**Response:** `200 OK` — Same shape as browse response, ranked by relevance.

**Errors:**

| Code  | Condition                   |
|:------|:----------------------------|
| `400` | Missing or empty query `q`. |

---

### 7.3 Get Trending Games

Returns a curated list of currently trending games based on recent play count growth.

```
GET /api/v1/library/trending
```

**Auth:** None

**Query Parameters:**

| Parameter | Type      | Required | Description                                |
|:----------|:----------|:---------|:-------------------------------------------|
| `limit`   | `integer` | No       | Number of games. Default: `10`, max: `50`. |

**Response:** `200 OK` — Array of game summary objects.

---

### 7.4 Get Featured Games

Returns games featured by platform staff.

```
GET /api/v1/library/featured
```

**Auth:** None

**Query Parameters:**

| Parameter | Type      | Required | Description                                |
|:----------|:----------|:---------|:-------------------------------------------|
| `limit`   | `integer` | No       | Number of games. Default: `10`, max: `50`. |

**Response:** `200 OK` — Array of game summary objects.

---

### 7.5 List Collections

Returns all active collections, ordered by `sortOrder`.

```
GET /api/v1/collections
```

**Auth:** None

**Response:** `200 OK`

```json
{
  "data": [
    {
      "id": "uuid",
      "title": "Family Night Favorites",
      "description": "Games the whole family will love",
      "coverImageUrl": "collections/family.png",
      "gameCount": 8
    }
  ]
}
```

Only collections with `active: true` are returned.

---

### 7.6 Get Collection

Returns a collection with its games.

```
GET /api/v1/collections/{collectionId}
```

**Auth:** None

**Path Parameters:**

| Parameter      | Type   | Description          |
|:---------------|:-------|:---------------------|
| `collectionId` | `UUID` | The collection's ID. |

**Response:** `200 OK`

```json
{
  "id": "uuid",
  "title": "Family Night Favorites",
  "description": "Games the whole family will love",
  "coverImageUrl": "collections/family.png",
  "games": [
    {
      "id": "uuid",
      "title": "Space Race",
      "thumbnailUrl": "thumbnails/space-race.png",
      "minPlayers": 2,
      "maxPlayers": 8,
      "avgRating": 4.3,
      "playCount": 1247,
      "creator": {
        "username": "PixelMike",
        "avatarUrl": "avatars/mike.png"
      }
    }
  ]
}
```

Games are ordered by `CollectionGame.sortOrder`.

**Errors:**

| Code  | Condition                           |
|:------|:------------------------------------|
| `404` | Collection not found or not active. |

---

## 8. Templates

### 8.1 List Templates

Returns all available templates, ordered by `sortOrder`.

```
GET /api/v1/templates
```

**Auth:** None

**Query Parameters:**

| Parameter    | Type     | Required | Description                                        |
|:-------------|:---------|:---------|:---------------------------------------------------|
| `difficulty` | `string` | No       | Filter: `beginner`, `intermediate`, or `advanced`. |
| `technology` | `string` | No       | Filter: `p5js`.                                    |

**Response:** `200 OK`

```json
{
  "data": [
    {
      "id": "uuid",
      "title": "Quiz Show",
      "description": "A trivia game template",
      "thumbnailUrl": "templates/quiz-show.png",
      "technology": "p5js",
      "difficulty": "beginner",
      "sortOrder": 1
    }
  ]
}
```

---

### 8.2 Get Template

Returns the details of a template, including the starter code.

```
GET /api/v1/templates/{templateId}
```

**Auth:** None

**Path Parameters:**

| Parameter    | Type   | Description        |
|:-------------|:-------|:-------------------|
| `templateId` | `UUID` | The template's ID. |

**Response:** `200 OK`

```json
{
  "id": "uuid",
  "createdAt": "2026-01-01T00:00:00Z",
  "updatedAt": "2026-01-15T10:00:00Z",
  "title": "Quiz Show",
  "description": "A trivia game template",
  "thumbnailUrl": "templates/quiz-show.png",
  "technology": "p5js",
  "gameScreenCode": "...",
  "controllerScreenCode": "...",
  "difficulty": "beginner",
  "sortOrder": 1
}
```

**Errors:**

| Code  | Condition           |
|:------|:--------------------|
| `404` | Template not found. |

---

### 8.3 Create Game from Template

Creates a new game by copying the template's code. The game starts in `draft` status with `private` visibility.

```
POST /api/v1/templates/{templateId}/use
```

**Auth:** User

**Request Body:**

| Field   | Type     | Required | Description             |
|:--------|:---------|:---------|:------------------------|
| `title` | `string` | Yes      | Title for the new game. |

**Response:** `201 Created` — Returns the newly created game object.

The new game's `gameScreenCode` and `controllerScreenCode` are copied from the template. The `technology` is inherited from the template.

**Errors:**

| Code  | Condition           |
|:------|:--------------------|
| `404` | Template not found. |

---

## 9. Administration

All administration endpoints require **Admin** role unless noted otherwise.

### 9.1 List Users

Returns a paginated list of all users.

```
GET /api/v1/admin/users
```

**Auth:** Admin

**Query Parameters:** Supports [pagination](#14-pagination).

| Parameter       | Type     | Required | Description                                   |
|:----------------|:---------|:---------|:----------------------------------------------|
| `role`          | `string` | No       | Filter: `user`, `moderator`, `admin`.         |
| `accountStatus` | `string` | No       | Filter: `active`, `suspended`, `deactivated`. |
| `search`        | `string` | No       | Search by username or email.                  |

**Response:** `200 OK` — Paginated list of user objects (including `accountStatus`, `lastLoginAt`, `lastLoginIp`).

---

### 9.2 Get User (Admin)

Returns the full details of any user, including fields not visible on the public profile.

```
GET /api/v1/admin/users/{userId}
```

**Auth:** Admin

**Response:** `200 OK` — Full user object with auth providers, subscription info, and account status.

**Errors:**

| Code  | Condition       |
|:------|:----------------|
| `404` | User not found. |

---

### 9.3 Update User (Admin)

Updates a user's role, subscription plan, or account status.

```
PATCH /api/v1/admin/users/{userId}
```

**Auth:** Admin

**Request Body (all fields optional):**

| Field                   | Type     | Description                              |
|:------------------------|:---------|:-----------------------------------------|
| `role`                  | `string` | `user`, `moderator`, or `admin`.         |
| `subscriptionPlan`      | `string` | `free` or `pro`.                         |
| `subscriptionExpiresAt` | `string` | ISO 8601 timestamp, or `null`.           |
| `accountStatus`         | `string` | `active`, `suspended`, or `deactivated`. |

**Response:** `200 OK` — Returns the updated user object.

**Errors:**

| Code  | Condition       |
|:------|:----------------|
| `404` | User not found. |

---

### 9.4 Suspend User

Suspends a user account. The user cannot sign in. Existing sessions are not interrupted but no new sessions or content can be created.

```
POST /api/v1/admin/users/{userId}/suspend
```

**Auth:** Admin

**Request Body:**

| Field    | Type     | Required | Description            |
|:---------|:---------|:---------|:-----------------------|
| `reason` | `string` | No       | Reason for suspension. |

**Response:** `200 OK`

```json
{
  "userId": "uuid",
  "accountStatus": "suspended"
}
```

**Errors:**

| Code  | Condition                  |
|:------|:---------------------------|
| `404` | User not found.            |
| `422` | User is already suspended. |

---

### 9.5 Restore User

Restores a suspended or deactivated user account to active status.

```
POST /api/v1/admin/users/{userId}/restore
```

**Auth:** Admin

**Response:** `200 OK`

```json
{
  "userId": "uuid",
  "accountStatus": "active"
}
```

Sets `User.accountStatus` to `"active"` and clears `User.deletedAt` if set.

**Errors:**

| Code  | Condition               |
|:------|:------------------------|
| `404` | User not found.         |
| `422` | User is already active. |

---

### 9.6 Archive Game (Moderation)

Archives a game that violates platform guidelines. Available to moderators and admins.

```
POST /api/v1/admin/games/{gameId}/archive
```

**Auth:** Moderator

**Request Body:**

| Field    | Type     | Required | Description           |
|:---------|:---------|:---------|:----------------------|
| `reason` | `string` | No       | Reason for archiving. |

**Response:** `200 OK` — Returns the updated game object.

**Errors:**

| Code  | Condition       |
|:------|:----------------|
| `404` | Game not found. |

---

### 9.7 Restore Game (Admin)

Restores a soft-deleted game.

```
POST /api/v1/admin/games/{gameId}/restore
```

**Auth:** Admin

**Response:** `200 OK` — Returns the restored game object.

Clears `Game.deletedAt`.

**Errors:**

| Code  | Condition                 |
|:------|:--------------------------|
| `404` | Game not found.           |
| `422` | Game is not soft-deleted. |

---

### 9.8 Create Collection

Creates a new curated game collection.

```
POST /api/v1/admin/collections
```

**Auth:** Moderator

**Request Body:**

| Field           | Type      | Required | Default | Description               |
|:----------------|:----------|:---------|:--------|:--------------------------|
| `title`         | `string`  | Yes      |         | Collection title.         |
| `description`   | `string`  | No       | `null`  | Collection description.   |
| `coverImageUrl` | `string`  | No       | `null`  | Cover image URL.          |
| `sortOrder`     | `integer` | No       | `0`     | Display order.            |
| `active`        | `boolean` | No       | `true`  | Whether visible to users. |

**Response:** `201 Created` — Returns the new collection object.

---

### 9.9 Update Collection

Updates a collection's metadata.

```
PATCH /api/v1/admin/collections/{collectionId}
```

**Auth:** Moderator

**Request Body (all fields optional):**

| Field           | Type      | Description             |
|:----------------|:----------|:------------------------|
| `title`         | `string`  | Collection title.       |
| `description`   | `string`  | Collection description. |
| `coverImageUrl` | `string`  | Cover image URL.        |
| `sortOrder`     | `integer` | Display order.          |
| `active`        | `boolean` | Visibility toggle.      |

**Response:** `200 OK` — Returns the updated collection object.

**Errors:**

| Code  | Condition             |
|:------|:----------------------|
| `404` | Collection not found. |

---

### 9.10 Delete Collection

Deletes a collection. The games in the collection are not affected.

```
DELETE /api/v1/admin/collections/{collectionId}
```

**Auth:** Admin

**Response:** `204 No Content`

**Errors:**

| Code  | Condition             |
|:------|:----------------------|
| `404` | Collection not found. |

---

### 9.11 Set Collection Games

Replaces all games in a collection with the provided ordered list.

```
PUT /api/v1/admin/collections/{collectionId}/games
```

**Auth:** Moderator

**Request Body:**

| Field     | Type     | Required | Description                                              |
|:----------|:---------|:---------|:---------------------------------------------------------|
| `gameIds` | `UUID[]` | Yes      | Ordered array of game IDs. Order determines `sortOrder`. |

**Response:** `200 OK`

```json
{
  "collectionId": "uuid",
  "gameCount": 8,
  "games": [
    {
      "gameId": "uuid",
      "title": "Space Race",
      "sortOrder": 0
    },
    {
      "gameId": "uuid",
      "title": "Quiz Night",
      "sortOrder": 1
    }
  ]
}
```

**Errors:**

| Code  | Condition                                               |
|:------|:--------------------------------------------------------|
| `400` | One or more game IDs do not exist or are not published. |
| `404` | Collection not found.                                   |

---

### 9.12 Create Template

Creates a new platform-provided starter template.

```
POST /api/v1/admin/templates
```

**Auth:** Admin

**Request Body:**

| Field                  | Type      | Required | Default      | Description                             |
|:-----------------------|:----------|:---------|:-------------|:----------------------------------------|
| `title`                | `string`  | Yes      |              | Template title.                         |
| `description`          | `string`  | No       | `null`       | Template description.                   |
| `thumbnailUrl`         | `string`  | No       | `null`       | Preview image URL.                      |
| `technology`           | `string`  | No       | `"p5js"`     | Runtime technology.                     |
| `gameScreenCode`       | `text`    | Yes      |              | Starter Game Screen code.               |
| `controllerScreenCode` | `text`    | Yes      |              | Starter Controller Screen code.         |
| `difficulty`           | `string`  | No       | `"beginner"` | `beginner`, `intermediate`, `advanced`. |
| `sortOrder`            | `integer` | No       | `0`          | Display order.                          |

**Response:** `201 Created` — Returns the new template object.

---

### 9.13 Update Template

Updates an existing template.

```
PATCH /api/v1/admin/templates/{templateId}
```

**Auth:** Admin

**Request Body (all fields optional):** Same fields as create.

**Response:** `200 OK` — Returns the updated template object.

**Errors:**

| Code  | Condition           |
|:------|:--------------------|
| `404` | Template not found. |

---

### 9.14 Delete Template

Deletes a template. Existing games created from this template are not affected.

```
DELETE /api/v1/admin/templates/{templateId}
```

**Auth:** Admin

**Response:** `204 No Content`

**Errors:**

| Code  | Condition           |
|:------|:--------------------|
| `404` | Template not found. |

---

### 9.15 Get Platform Analytics

Returns platform-wide statistics.

```
GET /api/v1/admin/analytics
```

**Auth:** Admin

**Response:** `200 OK`

```json
{
  "users": {
    "total": 5430,
    "active": 4200,
    "suspended": 15,
    "deactivated": 1215,
    "pro": 340
  },
  "games": {
    "total": 890,
    "published": 420,
    "draft": 380,
    "archived": 90
  },
  "sessions": {
    "totalCreated": 12500,
    "activeNow": 34
  },
  "plays": {
    "totalPlayCount": 87000,
    "totalPlayTime": 6240000
  }
}
```

---

## 10. WebSocket Protocol

### 10.1 Connection

```
WS /api/v1/sessions/{sessionId}/ws
```

**Query Parameters:**

| Parameter  | Type     | Required | Description                                                |
|:-----------|:---------|:---------|:-----------------------------------------------------------|
| `role`     | `string` | Yes      | `host` or `player`.                                        |
| `playerId` | `UUID`   | Cond.    | Required when `role` is `player`. The player's ID.         |
| `token`    | `string` | No       | Bearer token for authenticated users. Optional for guests. |

**Connection Flow:**

1. Client opens a WebSocket connection with the query parameters above.
2. Server validates the session exists and is not ended.
3. For `player` role: validates the `playerId` exists in the session and sets `connectionStatus` to `"connected"`.
4. For `host` role: validates the requester is the session host (via `token`).
5. On success, the server sends a `connected` message.
6. On failure, the server closes the connection with an appropriate close code.

**Close Codes:**

| Code   | Meaning                         |
|:-------|:--------------------------------|
| `1000` | Normal closure (session ended). |
| `4001` | Session not found.              |
| `4002` | Player not found in session.    |
| `4003` | Unauthorized.                   |
| `4004` | Session is full.                |

---

### 10.2 Message Format

All messages are JSON with this structure:

```json
{
  "type": "message_type",
  "payload": {},
  "timestamp": "2026-02-08T15:30:00Z",
  "messageId": "uuid"
}
```

| Field       | Type     | Description                                       |
|:------------|:---------|:--------------------------------------------------|
| `type`      | `string` | The message type identifier.                      |
| `payload`   | `object` | Message-specific data.                            |
| `timestamp` | `string` | ISO 8601 UTC timestamp when the message was sent. |
| `messageId` | `string` | Unique identifier for this message (UUID).        |

---

### 10.3 Client → Server Messages

#### `player_input`

Sent by a player's Controller Screen to relay input to the Game Screen.

```json
{
  "type": "player_input",
  "payload": {
    "inputType": "button_press",
    "data": {
      "button": "A",
      "pressed": true
    }
  }
}
```

The server relays this to the host as a `player_input_event` with the `playerId` attached. The `payload.data` is opaque to the server — the game code defines the input schema.

#### `game_state_update`

Sent by the host's Game Screen to broadcast game state to all Controller Screens.

```json
{
  "type": "game_state_update",
  "payload": {
    "state": {}
  }
}
```

The `payload.state` is opaque to the server. The server relays this to all connected players as a `game_state` message.

#### `player_ready`

Sent by a player to signal they are ready to start.

```json
{
  "type": "player_ready",
  "payload": {
    "ready": true
  }
}
```

The server relays this to the host.

#### `ping`

Keep-alive message. The server responds with `pong`.

```json
{
  "type": "ping",
  "payload": {}
}
```

---

### 10.4 Server → Client Messages

#### `connected`

Sent to a client immediately after a successful WebSocket connection.

```json
{
  "type": "connected",
  "payload": {
    "sessionId": "uuid",
    "role": "player",
    "playerId": "uuid"
  }
}
```

#### `player_joined`

Broadcast to all clients in the session when a new player joins.

```json
{
  "type": "player_joined",
  "payload": {
    "player": {
      "id": "uuid",
      "displayName": "Player 3",
      "avatarUrl": null
    }
  }
}
```

#### `player_left`

Broadcast when a player leaves or is removed from the session.

```json
{
  "type": "player_left",
  "payload": {
    "playerId": "uuid",
    "reason": "left"
  }
}
```

`reason` values: `"left"` (voluntary), `"removed"` (by host), `"disconnected"` (connection lost, grace period expired).

#### `player_reconnected`

Broadcast when a previously disconnected player reconnects within the grace period.

```json
{
  "type": "player_reconnected",
  "payload": {
    "playerId": "uuid"
  }
}
```

#### `player_input_event`

Sent to the host when a player sends input. Wraps the player's `player_input` payload with their ID.

```json
{
  "type": "player_input_event",
  "payload": {
    "playerId": "uuid",
    "inputType": "button_press",
    "data": {
      "button": "A",
      "pressed": true
    }
  }
}
```

#### `game_state`

Sent to all players when the host broadcasts game state.

```json
{
  "type": "game_state",
  "payload": {
    "state": {}
  }
}
```

#### `session_status_change`

Broadcast when the session status changes.

```json
{
  "type": "session_status_change",
  "payload": {
    "status": "playing",
    "previousStatus": "lobby"
  }
}
```

#### `game_loaded`

Sent to players when the host loads a game into the session. Includes the controller code.

```json
{
  "type": "game_loaded",
  "payload": {
    "gameId": "uuid",
    "gameVersionId": "uuid",
    "controllerScreenCode": "..."
  }
}
```

The host receives this message with `gameScreenCode` instead of `controllerScreenCode`.

#### `game_restarted`

Sent to all clients when the host restarts the game, signaling both canvases to re-run their setup phase.

```json
{
  "type": "game_restarted",
  "payload": {}
}
```

#### `error`

Sent to a client when an operation fails.

```json
{
  "type": "error",
  "payload": {
    "code": "SESSION_FULL",
    "message": "The session has reached the maximum number of players."
  }
}
```

#### `pong`

Response to a `ping` message.

```json
{
  "type": "pong",
  "payload": {}
}
```
