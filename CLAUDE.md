# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is the **documentation repository** for AirCade, a browser-based platform for building and playing multiplayer games using smartphones as controllers. The actual implementation lives in separate repos: `aircade-api` (Rust/Axum backend) and `aircade-web` (Next.js frontend).

This repo contains only Markdown documentation — there are no build, test, or lint commands.

## Documentation Structure

- `general/` — Product-level docs: overview, PRD with personas, descriptions, milestones
- `app/shared/` — Cross-cutting technical docs: tech stack, entity definitions, specification
- `app/api/` — Backend/API documentation (endpoints)
- `app/db/` — Database documentation (placeholder)
- `app/web/` — Frontend documentation (placeholder)
- `inspirations/` — Competitor research (AirConsole analysis)

## Platform Architecture

AirCade uses a **dual-canvas model**: every game renders a shared Game Screen (big screen/TV) and per-player Controller Screens (smartphones), synchronized via WebSockets.

**Tech stack of the described platform:**
- **Backend:** Rust with Axum 0.8, SeaORM, PostgreSQL
- **Frontend:** Next.js 16, React, TypeScript, shadcn/ui, Tailwind, Zustand
- **Game Runtime:** JavaScript/TypeScript with p5.js
- **Real-time:** WebSockets via Axum
- **Hosting:** Railway.com (all services)
- **Auth:** Email/password, Google OAuth 2.0, GitHub OAuth 2.0

## Database Schema

Entity definitions in `app/shared/entities.md` describe 17 entities using UUIDs, soft deletes, and foreign keys. Key entity groups: User/AuthProvider, Game/GameVersion/GameAsset, Session/Player, Review/Favorite/PlayHistory, Template/Collection.

## Writing Conventions

- Documentation follows standard Markdown formatting
- Entity definitions use structured tables (Field, Type, Constraints, Description)
- Tech stack docs include dependency version numbers
