# AirCade - Technical Stack

## Hosting

**Railway.com** hosts the entire platform: database, API server, and web frontend. Railway provides container-based deployments with built-in CI/CD from Git, automatic TLS, environment management, and managed PostgreSQL - eliminating the need for separate infrastructure tooling.

---

## Database

**PostgreSQL** serves as the sole persistent data store. It handles user accounts, game project metadata, published game assets, session records, community content (ratings, reviews), and subscription state.

PostgreSQL is accessed exclusively through SeaORM on the backend (see below). No raw SQL is written in application code.

---

## Backend - API Server

The backend is written in **Rust** and exposes an HTTP/WebSocket API consumed by the frontend.

### Web Framework - Axum

**Axum** is the async web framework used for routing, middleware, and request handling. It provides:

- Type-safe route extractors and response types
- Tower-based middleware for authentication, logging, and error handling
- Native WebSocket support for real-time game session communication

### ORM - SeaORM

**SeaORM** handles all database interaction. It provides:

- Async-native query building and execution
- Schema migration management
- Compile-time model definitions that stay in sync with the database schema

### Authentication

The API handles user authentication with three sign-in methods:

- **Email & Password** - Classic registration with hashed passwords and email verification
- **Google OAuth 2.0** - One-click sign-in via Google accounts
- **GitHub OAuth 2.0** - One-click sign-in via GitHub accounts

OAuth flows are handled server-side. On first OAuth sign-in, an AirCade account is created automatically from the provider's profile data. Users can link multiple auth methods to a single account.

### Real-Time Communication

Game sessions require low-latency, bidirectional communication between the big screen, player smartphones, and the server. This is handled via **WebSockets** through Axum's built-in WebSocket support. The server relays game state updates between connected clients within a session.

### Dependencies

|                        Crate | Version | Purpose                                                                             |
|-----------------------------:|:-------:|:------------------------------------------------------------------------------------|
|   **Web Framework & Server** |         |                                                                                     |
|                       `axum` |   0.8   | Async web framework for routing, extractors, and WebSocket support                  |
|                      `tower` |   0.5   | Service and middleware abstractions used by Axum                                    |
|                 `tower-http` |   0.6   | HTTP-specific middleware — CORS policies and request tracing                        |
|            **Async Runtime** |         |                                                                                     |
|                      `tokio` |  1.49   | Async runtime powering all non-blocking I/O (full feature set)                      |
|           **Database & ORM** |         |                                                                                     |
|                    `sea-orm` |   1.1   | Async ORM with PostgreSQL support via SQLx, Tokio+Rustls runtime, and derive macros |
|          `sea-orm-migration` |   1.1   | Schema migration system for SeaORM                                                  |
|            **Serialization** |         |                                                                                     |
|                      `serde` |   1.0   | Serialization/deserialization framework with derive macros                          |
|                 `serde_json` |   1.0   | JSON parsing and generation                                                         |
| **Error Handling & Logging** |         |                                                                                     |
|                     `anyhow` |   1.0   | Ergonomic error handling with context propagation                                   |
|                    `tracing` |   0.1   | Structured, async-aware logging framework                                           |
|         `tracing-subscriber` |   0.3   | Log subscriber with environment-based filter configuration                          |
|            **Configuration** |         |                                                                                     |
|                    `dotenvy` |  0.15   | Loads environment variables from `.env` files                                       |
|                **Utilities** |         |                                                                                     |
|                     `chrono` |   0.4   | Date and time types for timestamps and scheduling                                   |
|                `async-trait` |   0.1   | Async trait support, used by SeaORM migrations                                      |
|                       `rand` |   0.8   | Random number generation for game session codes                                     |
|                       `uuid` |   1.0   | UUID generation for entity identifiers                                              |

---

## Frontend - Web Application

The frontend is built with **TypeScript** and **Next.js 16**. It serves three distinct user-facing experiences from a single application:

1. **The Creative Studio** - browser-based code editor where creators build games using the dual-canvas model
2. **The Console (Big Screen)** - lobby, game library, and game rendering displayed on TVs, laptops, or projectors
3. **The Controller (Smartphone)** - dynamic, game-specific touch interface displayed on each player's phone

Next.js handles server-side rendering, routing, and API layer integration. TypeScript provides type safety across the entire frontend codebase.

### UI Framework - shadcn/ui

**shadcn/ui** provides the component library. Unlike traditional npm packages, shadcn/ui copies component source code directly into the project, giving full control over styling and behavior. Components are built on Radix UI primitives and styled with Tailwind CSS.

### State Management - Zustand

**Zustand** manages client-side state. It provides a minimal, hook-based store API without boilerplate. Separate stores handle different domains: authentication state, game session state, editor state, and WebSocket connection state.

### Dependencies

|                   Package | Purpose                                                    |
|--------------------------:|:-----------------------------------------------------------|
|             **Framework** |                                                            |
|                    `next` | React framework with SSR, routing, and API routes          |
|     `react` / `react-dom` | UI rendering library                                       |
|              `typescript` | Static type checking                                       |
|          **UI & Styling** |                                                            |
|               `shadcn/ui` | Component library (copied into project, built on Radix UI) |
|             `tailwindcss` | Utility-first CSS framework                                |
|            `lucide-react` | Icon library used by shadcn/ui components                  |
| `clsx` + `tailwind-merge` | Conditional and conflict-free class name composition       |
|      **State Management** |                                                            |
|                 `zustand` | Lightweight, hook-based state management                   |
|           **Code Editor** |                                                            |
|    `@monaco-editor/react` | Monaco Editor (VS Code engine) for the Creative Studio     |
|          **Game Runtime** |                                                            |
|        `p5` / `@types/p5` | 2D creative coding library for game rendering              |
|      **HTTP & Real-Time** |                                                            |
|                   `axios` | HTTP client for API communication                          |
|             **Utilities** |                                                            |
|                     `zod` | Schema validation for forms and API responses              |
|                `date-fns` | Lightweight date formatting and manipulation               |

---

## Game Runtime

Creators write games in **JavaScript** or **TypeScript** using the **p5.js** creative coding library. p5.js provides a simple drawing API well-suited for 2D game graphics and is widely taught in educational settings.

Every game consists of two synchronized canvases (the dual-canvas model):

- **Game Screen canvas** - rendered on the big screen, shows the shared game world
- **Controller Screen canvas** - rendered on each player's phone, shows private controls and information

Both canvases share the same game state and communicate through the platform's WebSocket layer. The game runtime executes in the browser on both the big screen and each connected smartphone.

### Planned Additions

- **3D rendering engine** - a browser-based 3D option for games with spatial environments
- **Visual game builder** - a graphical editor for creators who prefer a low-code workflow

Both will integrate into the same dual-canvas architecture.

---

## Architecture Overview

```txt
┌────────────────────────────────────────────────────────────┐
│                        Railway.com                         │
│                                                            │
│  ┌──────────────┐  ┌───────────────────┐  ┌──────────────┐ │
│  │  PostgreSQL  │  │     Rust API      │  │   Next.js    │ │
│  │  (Database)  │◄─┤  (Axum + SeaORM)  │  │  (Frontend)  │ │
│  └──────────────┘  └─────────┬─────────┘  └──────┬───────┘ │
│                              │                   │         │
└──────────────────────────────┼───────────────────┼─────────┘
                               │                   │
                     WebSocket + HTTPS        HTML/TS/CSS
                               │                   │
                    ┌──────────┴───────────────────┴─────────┐
                    │                Browsers                │
                    │                                        │
                    │  ┌──────────────┐  ┌─────────────────┐ │
                    │  │  Big Screen  │  │   Smartphones   │ │
                    │  │  (Console)   │  │  (Controllers)  │ │
                    │  └──────────────┘  └─────────────────┘ │
                    └────────────────────────────────────────┘
```

---

## Summary

|        Layer | Technology                                   |
|-------------:|:---------------------------------------------|
|      Hosting | Railway.com                                  |
|     Database | PostgreSQL                                   |
|      Backend | Rust, Axum, SeaORM, Tokio                    |
|         Auth | Email & Password, Google OAuth, GitHub OAuth |
|     Frontend | TypeScript, Next.js 16, shadcn/ui, Zustand   |
|    Real-Time | WebSockets (via Axum)                        |
| Game Runtime | JavaScript/TypeScript, p5.js                 |
|  Code Editor | Monaco Editor                                |
