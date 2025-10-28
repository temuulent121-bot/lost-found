# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Commands

This is a static frontend (HTML/CSS/JS) app with no build system, package manager, or test/lint configuration in the repo. You must serve the site over HTTP to load ESM modules (Supabase) correctly; opening files via file:// will not work.

Project root contains a nested web folder at `lost-and-found-system-main/` — serve that directory.

- Serve locally (Python)
  - From the repo root:
    - `cd lost-and-found-system-main`
    - `python -m http.server 5173`
  - Then open: `http://localhost:5173/login.html` (or `home.html`, `admin-dashboard.html`, etc.)

- Serve locally (Node, no install needed)
  - From the repo root:
    - `cd lost-and-found-system-main`
    - `npx serve -l 5173`
  - Then open: `http://localhost:5173/login.html`

Notes
- There is no configured linter or formatter.
- There is no test suite. Single-test commands do not apply.
- If Supabase Auth blocks redirects locally, ensure the Supabase project allows your local origin (e.g., `http://localhost:5173`) in Authentication settings (Site URL/Redirect URLs).

## High-level architecture

- Overall
  - Pure client-side app that uses Supabase (Auth, Database, Storage) via CDN ESM imports in each page script.
  - Pages are independent HTML documents under `lost-and-found-system-main/` with a 1:1 JS file per page; common UI patterns (navigation, theme toggle, overlays, animations) are duplicated across page scripts.
  - `index.html` immediately redirects to `home.html`.

- Authentication and roles
  - `auth.js` handles signup/login with `supabase.auth`. User metadata includes a `role` field; `admin`/`superadmin` users are redirected to `admin-dashboard.html`, others to `home.html`.
  - Most pages guard on session via `supabase.auth.getUser()` and redirect to `login.html` if missing. Logout uses `supabase.auth.signOut()`.

- Data model (Supabase)
  - Tables referenced across scripts:
    - `lost_items` and `found_items`: core item records; both rendered in list pages and on the home page; item edit flows available when `user_id` matches the authenticated user.
    - `reclaimed_items`: used by the admin item tracker; joined to `found_items` for display.
    - `contact_submissions`: admin messages with read/unread management.
  - Storage buckets are used for images when reporting items (e.g., separate buckets for lost/found images); public URLs are generated after upload.
  - Status fields such as `status` (e.g., `Active`, `Resolved`, etc.) drive filters and dashboards.

- Pages and responsibilities (selected)
  - `home.html` + `main.js`: landing page; loads recent Lost/Found items, provides item detail modal; theme and responsive nav.
  - `login.html`/`signup.html` + `auth.js`: signup/login flows, password visibility toggles, success messaging, theme toggle.
  - `lost-items.html` + `lost-items.js` and `found-items.html` + `found-items.js`: searchable/sortable item lists; click-through modals; owner-only edit modals for inline updates.
  - `report-lost.html` + `report-lost.js` and `report-found.html` + `report-found.js`: item reporting with client-side validation and image upload to Supabase Storage; inserts new records to the corresponding tables.
  - `admin-dashboard.html` + `admin.js`: role-protected; shows KPIs, recent items, CRUD tables for Lost/Found, transaction tracker from `reclaimed_items`, and message center for `contact_submissions` (read/unread, delete, auto-mark-read).
  - Additional pages: `claimed-items.html` (history), `profile.html`, `contact.html` have corresponding scripts and styles for their specific UIs.

- UI/UX
  - Uses Poppins (Google Fonts) and Font Awesome icons.
  - Theme preference stored in `localStorage` (`light` vs default dark); applied early inline in each page head for FOUC prevention.
  - Animations rely on classes (`animate-on-load`, `animate-on-scroll`) and `IntersectionObserver`.

## Repository notes for Warp

- No WARP/Cursor/Claude/Copilot rule files exist in this repo.
- `README.md` is minimal; this WARP.md is the authoritative quick-start for local development and architecture.
