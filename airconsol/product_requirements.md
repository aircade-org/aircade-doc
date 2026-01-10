# AirConsole - Product Requirements Document

## 1. Core Value Proposition

**The Problem:**
Traditional console gaming creates high barriers to entry for social play. It requires expensive hardware (consoles), costly proprietary controllers (often limited to 2-4 per household), and significant setup time. Casual social groups often lack the equipment to play together spontaneously.

**The Solution:**
AirConsole removes hardware barriers by turning devices people already own-web browsers/TVs and smartphones-into a console and controllers. It offers immediate, accessible local multiplayer gaming for groups of any size without the need for cables, downloads, or expensive peripherals.

**Why It Matters:**
By leveraging cloud technology and universal smartphone ownership, we democratize "couch gaming." We enable spontaneous social play in environments where traditional consoles cannot go: hotel rooms, classrooms, office breaks, and parties where guests simply need a phone to participate.

## 2. Target User Personas

**Persona A: The Social Host (Primary)**
    - **Profile:** Enjoys hosting dinner parties, game nights, or family gatherings.
    - **Pain Point:** Has guests who aren't "gamers" and finds traditional controllers intimidating for them. Doesn't have enough controllers for everyone.
    - **Goal:** Wants a low-friction activity that is inclusive, easy to understand, and fun for a mixed group of ages and skill levels.

**Persona B: The Casual Gamer/Traveler**
    - **Profile:** Plays games occasionally but doesn't own a dedicated console. Travels frequently or has limited living space.
    - **Pain Point:** Wants to play games on a larger screen (TV/Laptop) without carrying hardware.
    - **Goal:** Instant access to entertainment without carrying a physical console.

**Persona C: The Automotive Passenger**
    - **Profile:** Families or individuals charging electric vehicles or waiting in parked cars.
    - **Pain Point:** Boredom during charging stops or waiting periods.
    - **Goal:** High-quality shared entertainment using the car’s existing infotainment screen.

## 3. Key Features & User Journey

### 3.1 The "Big Screen" Experience (The Console)

The primary display serves as the shared view for all players.

- **Platform Agnostic Access:** Users must be able to access the console via any modern web browser, Smart TV app, or compatible car entertainment system.
- **Connection Hub:** The screen clearly displays a unique, short "Session Code" or QR code immediately upon launch.
- **Lobby Interface:** Once players join, their avatars appear on the big screen. The host can browse a visual catalog of games, filtered by genre, player count, or mood.

### 3.2 The "Controller" Experience (The Smartphone)

The user's smartphone acts as the input device.

- **No-Install Access:** Users can join by simply visiting a URL or opening the mobile app.
- **Sync Mechanism:** Users enter the "Session Code" displayed on the big screen to pair instantly. No Bluetooth pairing or Wi-Fi network matching should be strictly required (though recommended for speed).
- **Dynamic Interface:** The phone screen changes automatically based on the game context.
  - *Example:* If playing a quiz, the phone shows A/B/C/D buttons.
  - *Example:* If playing a racing game, the phone displays a "tilt" icon or steering wheel.
- **Private Information:** The phone must display private game data (hidden cards, secret roles) that is not visible on the main screen.

### 3.3 The Gameplay Lifecycle

A typical user session follows this flow:
    1.  **Launch:** The Host opens AirConsole on their TV or Laptop.
    2.  **Join:** Guests pull out phones, scan the QR code, and instantly appear in the lobby.
    3.  **Selection:** The Host (or any designated player) uses their phone to scroll through the game library on the TV and selects a title (e.g., "The Neighborhood").
    4.  **Loading:** The game loads on the big screen. Controller layouts are pushed to all phones simultaneously.
    5.  **Play:** Users play the game using phones. Latency must be low enough to feel responsive.
    6.  **Transition:** When the game ends, the group can instantly return to the main lobby to switch games without disconnecting their phones.
