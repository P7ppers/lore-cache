# Lore Cache

A DnD campaign manager for an Eberron campaign, built with SvelteKit frontend, a backend API, and Supabase/Postgres data storage. This is a living document with information about the the project ideas, features, progress, tech-stack and architecture.

## Project Overview

- **Goal:** Build a professional campaign dashboard for player characters and the campaign map.
- **Main panels:**
  - `Character Panel` for authenticated player sheets, stats, items, spells, notes, and session journaling.
  - `Map Panel` for campaign maps, region markers, location detail pages, NPCs, landmarks, and story scenes.
- **Future expansion:** DM/Player mode, combat panel, dice CLI, level-up system, deployable portfolio app.

## Tech Stack

- Frontend: `Svelte` / `SvelteKit`, `Tailwind CSS`
- Auth: `Supabase Auth`
- Backend: `Go` or `Node.js`
- Database: `Postgres` via `Supabase`

## Architecture

1. **Frontend**
   - Character and map UI components
   - Supabase auth integration
2. **Backend API**
   - Character endpoints: `GET`, `POST`, `PUT`
   - Map endpoints: locations, markers, campaign data
   - Notes/session endpoints
   - JWT auth middleware using Supabase tokens
3. **Database**
   - Supabase users via Auth
   - Characters, stats, spells, items, maps, notes, etc.

## Current Focus/Steps MVP

1. Database schema (design it out, create Supabase project)
2. Backend API (Go or Node, simple endpoints for Character + Map) (if needed might skip for now)
3. SvelteKit components (Character Panel, Map Panel UI)
4. Authentication (hook Supabase Auth into Sveltekit)
5. Deploy (Vercel + Railway, test end-to-end)
6. Polish (responsive design, error handling)

### Recommended Supabase setup

1. Create a Supabase project and database.
2. Define tables for users, characters, stats, spells, items, map locations, markers, and notes.
3. Use Supabase Auth for user sign-in.
4. Keep credentials out of source control using a `.env` file.

## Next Steps

- Design the Supabase schema
- Build the backend API
- Implement `CharacterPanel` and `MapPanel` in SvelteKit
- Integrate Supabase auth and deploy
