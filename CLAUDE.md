# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

This is the **documentation repository** for AirCade — a browser-based multiplayer game creation and play platform. It contains no source code. All files are Markdown specifications, requirements, and architecture documents.

The AirCade project spans three repositories:
- **aircade-doc** (this repo) — Specifications and product documentation
- **aircade-api** — Rust backend (Axum + SeaORM + PostgreSQL)
- **aircade-web** — TypeScript/Next.js 16 frontend

## Repository Structure

- `README.md` — Entry point with links to all repos and documents
- `overview.md` — How AirCade works conceptually
- `product_requirements.md` — Full PRD: personas, features, user journeys
- `technical-stack.md` — Technology choices and architecture diagram
- `descriptions.md` — Marketing copy at various lengths
- `entities.md` — Entity definitions (in progress)
- `specifications.md` — Technical specification (in progress)
- `milestones.md` — Roadmap (template)
- `inspirations/` — Competitor/inspiration analysis (AirConsol)

## Key Architecture Concepts

**Dual-canvas model** is the core technical differentiator: every game renders two synchronized canvases — a Game Screen (big screen/TV) and a Controller Screen (each player's smartphone). Both share game state via WebSockets.

**Three frontend experiences** served from one Next.js app:
1. Creative Studio — browser code editor for building games
2. Console (Big Screen) — lobby, library, and game rendering
3. Controller (Smartphone) — dynamic per-game touch interface

**Game runtime** uses p5.js for 2D rendering. Creators write games in JavaScript/TypeScript.

## Writing Guidelines

- This is a documentation-only repo. Changes are Markdown files — no build, lint, or test commands.
- Keep documents internally consistent: if you change a concept name or architecture detail, check all other docs for references.
- `product_requirements.md` is the authoritative source for feature definitions and user personas.
- `technical-stack.md` is the authoritative source for technology choices.
