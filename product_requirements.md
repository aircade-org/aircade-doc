# AirCade - Product Requirements Document

*Create. Play. Share.*

---

## 1. Core Value Proposition

### The Problem

Traditional game development and social play exist in entirely separate worlds. Creating a game typically requires specialized tools, deep programming knowledge, and months of solitary effort before anyone can experience the result. On the other hand, playing local multiplayer games together requires expensive consoles, proprietary controllers, and titles from major studios that leave no room for personal creativity.

This gap means that hobbyist developers, students, educators, and creative professionals have no straightforward way to build an interactive, multiplayer experience and immediately share it with a room full of people using nothing but the devices they already carry in their pockets.

### The Solution

AirCade is a cloud-based platform that unifies game creation and social play in a single seamless experience. It provides a browser-based creative studio where anyone can build games using visual coding tools, then instantly publish and play them on a shared "big screen" with friends, students, or colleagues—all using smartphones as controllers.

No consoles, no downloads, no expensive hardware. A creator writes a game in AirCade's integrated development environment, and within seconds, a room full of people can join and play it by scanning a code on their phones.

### Why It Matters

AirCade sits at the intersection of three powerful trends: the maker movement in game development, the rise of browser-based creative tools, and the universal adoption of smartphones. By combining a creation toolkit with an instant-play platform, AirCade unlocks possibilities that no existing product addresses:

- **Educators** can build custom classroom games and play them with students in real time.
- **Indie developers** can prototype multiplayer ideas and test them with a live audience instantly.
- **Families and friend groups** can design personalized party games tailored to their inside jokes and interests.
- **Creative professionals** can build interactive demos, team-building activities, or event entertainment without hiring a studio.

---

## 2. Target User Personas

### Persona A: The Creative Developer

|                |                                                                                                                                                                                                                                                                            |
|---------------:|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|    **Profile** | A hobbyist programmer, game design student, or indie developer who loves experimenting with interactive ideas. Comfortable with JavaScript or visual scripting but does not have access to expensive game engines or the time to set up complex networking infrastructure. |
| **Pain Point** | Building a multiplayer game from scratch is overwhelmingly complex. Testing it requires convincing friends to install custom software. Sharing it means navigating app stores or hosting servers.                                                                          |
|       **Goal** | A frictionless environment to code, test, and share multiplayer games that anyone can play instantly from their browser.                                                                                                                                                   |

### Persona B: The Educator or Workshop Leader

|                |                                                                                                                                                                            |
|---------------:|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|    **Profile** | A teacher, professor, or corporate trainer who wants to use interactive games to engage their audience. Has basic technical literacy but is not a professional programmer. |
| **Pain Point** | Existing educational games are generic and do not adapt to specific lesson plans. Custom tools like Kahoot are limited to quizzes and lack real interactivity.             |
|       **Goal** | The ability to create tailored interactive experiences (simulations, collaborative challenges, role-based scenarios) that a classroom or workshop can join in seconds.     |

### Persona C: The Social Host

|                |                                                                                                                                                                     |
|---------------:|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|    **Profile** | Enjoys hosting game nights, parties, or family gatherings. May or may not be technical. Values novelty and group fun.                                               |
| **Pain Point** | Commercial party games feel repetitive after a few sessions. Traditional consoles limit the number of players and require everyone to learn unfamiliar controllers. |
|       **Goal** | Access to a library of community-created games—and the option to create personalized ones—that any number of guests can join using only their phones.               |

### Persona D: The Automotive Passenger

|                |                                                                                                                                                              |
|---------------:|:-------------------------------------------------------------------------------------------------------------------------------------------------------------|
|    **Profile** | Families or individuals waiting during electric vehicle charging sessions or long stops. Passengers in parked vehicles with compatible infotainment screens. |
| **Pain Point** | Boredom during charging stops or waiting periods with no shared entertainment option beyond individual phone screens.                                        |
|       **Goal** | Instant access to multiplayer games on the car's infotainment screen, turning downtime into a shared social experience.                                      |

---

## 3. Key Features & User Journey

### 3.1 The Creative Studio

At the heart of AirCade is a browser-based creative studio that allows users to build games directly within the platform. No software installation is required.

#### Integrated Code Editor

The studio provides a professional-grade code editor directly in the browser. The initial creation environment supports JavaScript and TypeScript using the p5.js creative coding library, which is widely taught in universities and creative coding communities. The editor includes syntax highlighting, auto-completion, error detection, and a live preview panel.

#### Dual-Canvas Architecture

Every AirCade game is composed of two synchronized visual canvases that the creator designs simultaneously:

- **The Game Screen:** This is what appears on the big screen (TV, laptop, projector). It displays the shared game world—the board, the arena, the scoreboard—visible to all players.
- **The Controller Screen:** This is what appears on each player's smartphone. The creator designs this interface to show private controls (buttons, joysticks, steering wheels) and private information (hidden cards, secret roles, personal scores).

Both canvases share the same underlying game state, meaning a button press on a player's phone immediately affects the shared game world on the big screen. The creator writes the logic for both screens within a single project, defining a setup phase (initial configuration) and a continuous update phase (real-time behavior) for each canvas.

#### Extensible Technology Framework

While the platform launches with p5.js as its foundational creative toolkit, the architecture is designed to welcome additional technologies over time. Planned future options include:

- **3D Game Engine:** A browser-based 3D rendering option for creators who want to build experiences with depth, lighting, and spatial environments.
- **Visual Game Builder:** A full-featured game engine environment that allows creators to use a graphical editor alongside scripting, lowering the barrier for those who prefer a more visual workflow.

Each technology option will integrate seamlessly into the same dual-canvas model, ensuring that regardless of the tool used, the game-screen-plus-controller-screen paradigm remains consistent.

#### Templates and Starter Projects

To reduce the learning curve, the studio offers a library of starter templates. These are fully functional mini-games that creators can open, study, modify, and publish as their own. Examples include a quiz show template, a racing game template, and a drawing-party template. Each template demonstrates best practices for designing both the game screen and the controller screen.

---

### 3.2 The Big Screen Experience (The Console)

The big screen is the shared display that turns any compatible device into a game console.

#### Platform-Agnostic Access

Users access the AirCade console by opening a website or application on any device with a modern browser: laptops, Smart TVs, tablets, projectors, or compatible car infotainment systems. No plugins, downloads, or special hardware are needed.

#### Connection Hub

Upon launch, the big screen prominently displays a unique, short-lived Session Code and a scannable QR code. This is the single point of entry for all players. The design prioritizes clarity—the code is large, high-contrast, and visible from across a room.

#### Lobby and Game Library

Once players join, their avatars and names appear in a visual lobby. From here, the host (or any designated player) can browse a curated game library. The library includes:

- **Community Creations:** Games built and published by other AirCade creators, organized by genre, player count, popularity, and rating.
- **Your Projects:** The host's own published and in-progress creations, available for instant testing with the group.
- **Featured and Curated:** Staff-picked games, seasonal collections, and thematic bundles (e.g., "Ice-Breakers for Teams" or "Family Night Favorites").

---

### 3.3 The Controller Experience (The Smartphone)

The smartphone is the player's personal, private input device.

#### Instant Join

Players join a session by opening the AirCade website or mobile app on their phone and entering the Session Code, or by scanning the QR code displayed on the big screen. No account creation is required to play. The connection happens in seconds.

#### Dynamic, Context-Sensitive Interface

The phone screen transforms based on the game being played. The controller layout is not fixed; it is designed by the game's creator as part of the dual-canvas architecture. This means:

- A trivia game might show four large answer buttons.
- A racing game might display a tilt-based steering wheel with a boost button.
- A strategy game might show a private map with draggable tokens.
- A drawing game might present a blank canvas with color pickers and brush sizes.

#### Private Information Display

Because each phone screen is visible only to its owner, the controller can display secret game data. In a poker game, the player sees their cards privately. At a party game with hidden roles, the phone reveals the player's secret identity. This mechanic is fundamental to AirCade's design and is a core feature that creators are encouraged to leverage.

---

### 3.4 The Gameplay Lifecycle

A typical AirCade session follows a streamlined flow designed to minimize friction and maximize time spent playing:

1. **Launch:** The host opens AirCade on their big screen (TV, laptop, projector, or car display). A Session Code and QR code appear immediately.
2. **Join:** Guests pull out their phones, scan the QR code or type the code, and appear in the lobby within seconds. No app download is strictly required.
3. **Browse and Select:** The host uses their phone to scroll through the game library displayed on the big screen, filtering by genre, player count, or mood. They select a title.
4. **Load and Configure:** The chosen game loads on the big screen. Simultaneously, each player's phone receives the custom controller layout designed by the game's creator.
5. **Play:** The group plays the game. Input from phones must feel responsive and immediate. The big screen reflects all player actions in real time.
6. **Transition:** When the game ends, the group returns to the lobby instantly. Phones remain connected. A new game can be selected without anyone needing to rejoin.

---

### 3.5 Publishing and Community

A game created in AirCade is only a click away from being available to the world.

#### One-Click Publishing

From the creative studio, a creator can publish their game directly to the AirCade library. Published games become discoverable by all users. The creator sets metadata such as title, description, thumbnail, genre tags, and supported player count range.

#### Community Features

Players can rate and review games they have played. Creators can see play counts, average session length, and feedback. A trending section highlights games gaining popularity, encouraging a vibrant ecosystem of creation and play.

#### Remixing and Forking

If a creator publishes a game with remixing enabled, other users can open a copy of that project in their own studio, learn from it, modify it, and publish their version with credit to the original. This "fork and remix" culture accelerates learning and creativity across the community.
