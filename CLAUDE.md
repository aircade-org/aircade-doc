# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the **documentation repository** for AirCade — a browser-based multiplayer game creation and playing platform. It contains all product specs, technical architecture, database schemas, and API definitions. There is no code to build, lint, or test here; the repo is pure Markdown documentation.

AirCade has three repositories:
- **aircade-doc** (this repo) — Specifications and documentation
- **aircade-api** — Rust backend (Axum, SeaORM, PostgreSQL)
- **aircade-web** — Next.js 16 frontend (TypeScript, shadcn/ui, Tailwind CSS)

## Repository Structure

- `general/` — Product vision, PRD, descriptions, and milestone roadmap
- `app/shared/` — Cross-cutting specs: technical stack, functional specification, entity/schema definitions
- `app/api/` — REST API endpoint definitions and WebSocket protocol
- `app/db/` and `app/web/` — Reserved for future database and frontend-specific docs
- `inspirations/` — Reference material (AirConsol case study)

## Key Documents

- **`app/api/api-endpoints.md`** — Largest file (~80K). Complete REST API with all endpoints, parameters, responses, and WebSocket protocol for real-time game sessions.
- **`app/shared/specification.md`** — Functional specification covering auth, profiles, game management, sessions, community features, and discovery.
- **`app/shared/entities.md`** — Full database schema with all entity definitions and field types.
- **`general/milestones.md`** — Development roadmap organized by phases (Foundation, Core Platform, Advanced, Community) and categories (Shared, Database, Backend, Frontend, Test).
- **`app/shared/technical-stack.md`** — All technology choices with specific dependency versions.

## AirCade Core Concepts

The platform has three experiences that are central to the architecture:
1. **Creative Studio** — Browser-based code editor (Monaco Editor) for building games with p5.js
2. **Console** (Big Screen) — Game lobby and rendering surface displayed on TV/laptop/projector
3. **Controller** (Smartphone) — Dynamic touch interface that acts as a custom game controller

Game sessions use WebSockets for real-time communication between Console and Controller clients.

## Writing Guidelines

- All documentation is Markdown (.md). Keep formatting consistent with existing files.
- The milestones file uses a specific structure: phases → milestones → categories (Shared/Database/Backend/Frontend/Test) → tasks with checkbox status.
- Entity definitions in `entities.md` follow a consistent pattern: entity name, description, field table (name, type, constraints, description).
- API endpoints in `api-endpoints.md` follow a consistent pattern: endpoint path, method, description, auth requirements, request/response schemas.
