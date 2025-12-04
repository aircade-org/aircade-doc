> [< back](./README.md)

# AirCade Specification

## Scope and goals

- A browser‑based "online console" where one large screen (desktop browser, TV browser, etc.) runs the game session and multiple users join using their smartphones as controllers.
- Support local multiplayer party-style games, with a catalog of titles optimized for phone-as-controller gameplay (taps, swipes, tilt, etc.).
- Make the experience hardware-light: no dedicated console or proprietary controllers; only a browser and smartphones are required.
- Deliver a marketing site plus a "Play" experience under the same domain, with clear calls to action for "Start" (host) and "Join" (player).

## Core user flows

- **Host starts a session (big screen)**  
  - User opens the website on a large screen device (desktop/laptop browser, Smart TV browser, or similar).
  - Clicks a prominent "Start"/"Play on this screen" button, which creates a game "room" and displays a short alphanumeric session code and QR code.

- **Player joins as controller (phone)**  
  - On a smartphone, user opens the site or mobile web app and taps "Join Game".
  - Enters the session code from the big screen or scans QR code; upon success, phone switches into controller UI for lobby/game.

- **Browse and launch games**  
  - On the big screen, the host can browse a catalog of multiplayer games with tiles/cards, filters, and search.
  - Host selects a game, views a short detail page or overlay, then starts it; all connected phones automatically switch to the appropriate controller layout.

- **In-game multiplayer session**  
  - Big screen shows main game view, scoreboard, timer, and other shared visuals; phones show per‑player controls and status.
  - Players can join/leave mid-session where supported; game handles reconnect states and late-join behavior.

- **Account, subscription, and retention (optional)**  
  - Users can create an account (email/password, OAuth) to track favorites, play history, and premium access.
  - Implement an optional premium tier (e.g., more games, no ads) with third‑party payment integration; restrict some titles or features to premium users.

## Main pages and routes

- **Home / Landing (`/`)**  
  - Hero section describing concept: play local multiplayer games in the browser using phones as controllers, no console required.
  - Primary CTAs: "Start on this screen" (host) and "Join a game" (for phone visitors).
  - Secondary content: short "How it works" step-by-step, highlight of game categories (party, racing, quiz, etc.), and trust/social proof modules.

- **Play / Console view (`/play`)**  
  - On load, creates or joins a console session; displays a session code and QR code, plus basic connection instructions for players.
  - Lobby UI to show connected players (avatars, names), selected game, and ready status; controls for host (start, kick player, change game).
  - In-game view: canvas or WebGL/engine container for the actual game, player list, scoreboard, pause/exit controls.

- **Controller / Join (`/join` or device-detected variant)**  
  - Minimal onboarding explaining "Enter the code shown on the big screen to join".
  - Code-entry form plus QR code scanner; on success, route to controller UI that is dynamically defined per game.
  - Fallback states for invalid code, session full, or session expired.

- **Game catalog (`/games`)**  
  - Grid or list of game tiles with thumbnail, title, player count range, high-level tags (genre, difficulty, family-friendly).
  - Filters for number of players, category (party, racing, trivia, etc.), and free vs premium.
  - Clicking a tile opens a game detail route (`/games/{slug}`) with description, supported player count, approximate session length, and "Play on this screen" button.

- **About / Info (`/info`)**  
  - Short explanation of the concept: browser acts as console, phones as controllers, no extra hardware, play anytime with anyone.
  - Sections on supported platforms (desktop browsers, some smart TVs, potentially car infotainment in future), plus high-level mission statement.

- **Account & support**  
  - `/login`, `/signup`, `/account` for managing profile, email, password, subscription status, favorite games, and settings (language, content restrictions).
  - `/help` or `/faq` with small knowledge base: connection issues, code not working, controller lag, supported devices, and troubleshooting steps.

## Key functional requirements

- **Session and code system**  
  - System generates short session codes that uniquely map a phone connection to a running console session; codes expire after inactivity or host exit.
  - Join API validates code, session capacity, and game state; returns game and player metadata to the controller app.

- **Device roles and detection**  
  - Detect approximate device type (desktop/TV vs phone) via viewport and user agent; default to console view for large screens, controller/join view for phones.
  - Allow users to override role (e.g., join as controller from small tablet or run console from a laptop).

- **Game integration layer**  
  - Define a game API for: game lifecycle (init, start, pause, end), player management (join, leave, disconnect), input events from controllers, and outputs to big screen UI.
  - Support different controller layouts per game (buttons, joysticks, motion controls where supported) via a configuration descriptor rather than hard-coding each layout in the core UI.

- **Multiplayer networking**  
  - Use WebSockets or similar real-time transport to connect console session, game server (if any), and controller clients, with room-based channels keyed by session code.
  - Implement basic latency compensation strategies appropriate for casual party games (client-side prediction for UI, server-authoritative scoring where necessary).

- **Content management for games**  
  - Admin interface to register games with metadata (name, slug, description, tags, min/max players, free/premium flag, asset references).
  - Ability to publish/unpublish games, set featured games for home page and category pages, and configure localized content.

## Technical and non-functional requirements

- **Platforms and compatibility**  
  - Support modern desktop browsers (Chrome, Firefox, Edge, Safari) for console view and modern mobile browsers (Chrome, Safari) for controllers.
  - Optimize for Wi‑Fi local play; handle poor connectivity gracefully with reconnection logic and clear UI messages on both big screen and controllers.

- **Performance and UX**  
  - Target fast initial load (lazy-load game bundles, defer non-critical scripts) so users can start playing within seconds of visiting the site.
  - Provide responsive layouts and touch‑friendly controls on all controller screens; ensure big-screen UI is readable from several meters away.

- **Security and privacy**  
  - Use HTTPS for all traffic; protect session join APIs against brute‑force of session codes (rate limiting, code length/entropy).
  - Store minimal personal data, comply with relevant privacy regulations, and provide clear privacy and terms pages.

- **Branding and legal**  
  - Create entirely original branding (name, logo, color palette, typography) and avoid visual imitation of AirConsole's specific look and feel.
  - Do not reuse their game art, trademarks, iconography, or marketing text; treat AirConsole purely as an inspiration for feature set and interaction model.

> [< back](./README.md)
