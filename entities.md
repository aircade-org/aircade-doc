# AirCade - Entities

## User

| FIELD                    | TYPE        | SPECIFICITY             | DESCRIPTION                                                                            | EXAMPLE                |
|:-------------------------|:------------|:------------------------|:---------------------------------------------------------------------------------------|:-----------------------|
| `id`                     | `UUID`      | _Unique_, _Primary Key_ | The user account unique identifier                                                     | `a8f3e7b1-4c2d-...`    |
| `createdAt`              | `TimeStamp` | _Not Modifiable_        | The date and time when this account was created                                        | `2026-01-12T14:32:00Z` |
| `updatedAt`              | `TimeStamp` |                         | The date and time when this account was last modified                                  | `2026-02-07T20:15:00Z` |
| `deletedAt`              | `TimeStamp` | _Nullable_              | The date and time when this account was soft deleted                                   | `null`                 |
| `email`                  | `String`    | _Unique_, _Not Null_    | The user's email address, used for sign-in and notifications                           | `mike@example.com`     |
| `username`               | `String`    | _Unique_, _Not Null_    | The user's unique public name, visible to other players and creators                   | `PixelMike`            |
| `passwordHash`           | `String`    | _Nullable_              | The hashed password for email & password authentication (null for OAuth-only accounts) | *(not human-readable)* |
| `authProvider`           | `String`    | _Default: "email"_      | The method used to create the account: `email`, `google`, or `github`                  | `email`                |
| `authProviderId`         | `String`    | _Nullable_, _Unique_    | The unique identifier from the OAuth provider (null for email & password accounts)     | `google\|1234567890`   |
| `displayName`            | `String`    | _Nullable_              | An optional friendly name shown in lobbies and on the creator profile                  | `Mike Johnson`         |
| `avatarUrl`              | `String`    | _Nullable_              | URL to the user's profile picture, shown in lobbies and leaderboards                   | `avatars/mike.png`     |
| `bio`                    | `String`    | _Nullable_              | A short description the user writes about themselves, visible on their creator profile | `Party game addict`    |
| `emailVerified`          | `Boolean`   | _Default: false_        | Whether the user has confirmed their email address (required to publish games)         | `true`                 |
| `role`                   | `String`    | _Default: "user"_       | The user's platform role: `user`, `moderator`, or `admin`                              | `user`                 |
| `subscriptionPlan`       | `String`    | _Default: "free"_       | The user's subscription tier: `free` or `pro`                                          | `free`                 |
| `subscriptionExpiresAt`  | `TimeStamp` | _Nullable_              | The date the Pro subscription expires (null for free-tier users)                       | `2026-09-15T00:00:00Z` |
| `accountStatus`          | `String`    | _Default: "active"_     | The account's current state: `active`, `suspended`, or `deactivated`                   | `active`               |
| `emailVerificationToken` | `String`    | _Nullable_, _Unique_    | Token sent by email for address verification                                           | *(not human-readable)* |
| `passwordResetToken`     | `String`    | _Nullable_, _Unique_    | Token sent by email for password reset                                                 | *(not human-readable)* |
| `passwordResetExpiresAt` | `TimeStamp` | _Nullable_              | Expiration time for the password reset token                                           | `2026-02-08T15:00:00Z` |
| `lastLoginAt`            | `TimeStamp` | _Nullable_              | The date and time of the user's last sign-in                                           | `2026-02-07T20:15:00Z` |
| `lastLoginIp`            | `String`    | _Nullable_              | IP address of the user's last sign-in                                                  | `192.168.1.42`         |
