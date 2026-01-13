# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is the **documentation and product specification hub** for AirCade — a browser-based platform for creating, sharing, and playing multiplayer games using smartphones as controllers. This repo contains no application code; the implementations live in separate repositories:

- **Backend API** (Rust): `github.com/aircade-org/aircade-api`
- **Frontend Web** (TypeScript/Next.js): `github.com/aircade-org/aircade-web`

## Document Map

| File | Purpose |
|------|---------|
| `overview.md` | What AirCade is, how playing and creating works |
| `product_requirements.md` | Full PRD: personas, features, dual-canvas architecture, gameplay lifecycle, publishing |
| `technical-stack.md` | Technology choices, architecture diagram, hosting (Railway), backend (Rust/Axum/SeaORM), frontend (TS/Next.js 16), game runtime (p5.js) |
| `descriptions.md` | Taglines and product descriptions at various lengths |
| `milestones.md` | Milestone roadmap (template structure with `[Shared]`, `[Database]`, `[Backend]`, `[Frontend]` task labels) |
| `specification.md` | Technical specification (placeholder) |
| `airconsol/` | Documentation for AirConsole, the inspiration project |

## Key Concepts

- **Dual-canvas model**: Every game has two synchronized canvases — a Game Screen (big screen) and a Controller Screen (each player's phone). Both share game state via WebSockets.
- **Three frontend experiences**: Creative Studio (code editor), Console (big screen lobby/game), Controller (smartphone touch interface).
- **Session flow**: Host opens big screen → session code/QR displayed → players join on phones → game selected → play → seamless transition to next game.

## Writing Conventions

- Milestone tasks are prefixed with scope labels: `[Shared]`, `[Database]`, `[Backend]`, `[Frontend]`.
- The product tagline is *"Create. Play. Share."*
- When referencing the platform tech stack: Rust/Axum/SeaORM (backend), TypeScript/Next.js 16 (frontend), p5.js (game runtime), PostgreSQL (database), Railway.com (hosting).

## No Build/Test/Lint

This is a pure documentation repository (markdown files only). There are no build commands, test suites, or linters to run.
