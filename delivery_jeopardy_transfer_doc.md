# Delivery Jeopardy — Transfer Doc

## What this is
A two-team, Jeopardy-style classroom game (`jeopardy_game.html`) plus a companion
content-management page (`admin.html`). Students read/perform short prompts
(e.g. "Fake Apologies for $300") with delivery directions attached.

## What changed in this update
- Categories and prompts now live in a **Supabase Postgres database** instead of
  being hardcoded in the HTML. `jeopardy_game.html` fetches them on page load.
- A new **`admin.html`** page lets you add/edit/delete categories and prompts
  through a form — no more hand-editing JS in the HTML file.
- Writing to the database requires signing in (Supabase Auth); reading (what the
  public game page does) does not require login. See "Security model" below.

## Supabase project
- URL: `https://kkjqqyupcuapuwhznxnh.supabase.co`
- The publishable/anon key is embedded in both HTML files' `<script>` sections.
  This is expected and safe for this key type — it's the public key, and actual
  write access is controlled by the Row Level Security (RLS) policies below, not
  by keeping the key secret.

### One-time setup (do this before anything else works)
1. In the Supabase dashboard, open **SQL Editor → New query**, paste in the
   contents of `schema.sql`, and run it. This creates the `categories` and
   `prompts` tables and the RLS policies.
2. Run `seed_data.sql` the same way. This loads in the same 7 categories/70
   prompt slots that were previously hardcoded (MOVIES & TV, INSPIRATIONAL,
   APOLOGIES, CONGRESS VIBES, TOASTS, SCRIMMAGE SPEECHES, IRL POLITICIANS).
3. Create a login for yourself: **Authentication → Users → Add user** (email +
   password). This is the account you'll use to sign in to `admin.html`. Nobody
   else can edit content without this login.

### Security model
- **Anyone can read** categories/prompts (needed for the public game board to
  load — it's embedded on a public Squarespace page with no login).
- **Only a signed-in user can write.** `admin.html` has a login form; until you
  sign in, all the add/edit/delete controls are unreachable.
- The `admin.html` page is not linked anywhere on the public-facing game screen
  except a small "Manage categories & prompts" link in the footer — a visitor
  could click it, but they'd just hit a login screen they can't get past.
- `schema.sql` includes a commented-out stricter option that locks writes to one
  specific email address instead of "any logged-in account," if you want that.

## Current status: **working and live**
- Embedded on the company Squarespace site via an iframe.
- **Important:** Google Drive does NOT work for hosting this — it serves the HTML as raw text instead of executing it. The working live version is hosted on **Netlify** at:
  `https://sweet-pie-f5ad8e.netlify.app/jeopardy_game.html`
- Squarespace embed uses an **Embed block** (not a Code block) with:
  ```html
  <iframe src="https://sweet-pie-f5ad8e.netlify.app/jeopardy_game.html"
          width="100%" height="900px" frameborder="0" style="border:none;">
  </iframe>
  ```
- The user confirmed the background/styling displays correctly once loaded this way.

## How to update the live game
1. Get the latest `jeopardy_game.html` **and `admin.html`** from this chat/Claude.
2. Go to https://app.netlify.com/drop and drag **both files together** onto the
   page (drag a folder containing both, or drag them at the same time) — they
   need to be deployed to the same site so `admin.html`'s "Back to game" link
   and the game's "Manage categories" link resolve correctly.
3. This generates a **new random URL** each time (e.g. `https://xyz-123.netlify.app`).
4. Update the `src` in the Squarespace iframe to the new URL.
5. **Known friction point:** every drag-and-drop to Netlify Drop creates a new URL, so the Squarespace embed must be updated each time. The user is aware and has deferred a more permanent solution (e.g. GitHub Pages with a stable URL, or a proper Netlify account with a fixed site name) to a future session — not urgent for now.

**Content updates no longer require re-uploading the HTML at all** — just open
`admin.html` on the live site, sign in, and edit. The game page always fetches
current data from Supabase on load.

## Things that were tried and don't work
- **Pasting HTML directly into a Squarespace Code Block** — Squarespace's own CSS overrides styles (caused the "white background" bug that took a long time to diagnose) and it may strip/limit JavaScript.
- **Google Drive preview embed** — renders as plain text, not executable HTML.
- **GitHub Pages** — works in principle, but the repo (`https://github.com/joey718/delivery-jeopardy`) had the README showing instead of the game; needs the HTML file renamed to `index.html` OR the URL needs the explicit filename appended (`/jeopardy_game.html`). Left as a "future draft" — Netlify Drop is the current working solution.
- **Live Google Sheets CSV sync** — blocked by CORS restrictions on Google Sheets. Superseded by the Supabase integration in this update, which solves the same underlying problem (editing content without touching code) a different way.

## Data structure / customization
- Categories and prompts are stored in Supabase (`categories` and `prompts`
  tables — see `schema.sql`). Each category has up to 10 prompt slots
  (positions 0–9, mapped to money values $100–$1000).
- **To update content:** open `admin.html`, sign in, pick a category, edit the
  prompt/directions text for a $ slot, click Save. No code editing or
  re-deploying required.
- **To add a brand-new category:** in `admin.html`, type a name in "New
  category name" and click "Add Category" — this creates 10 empty slots you
  can then fill in.
- In-app "Change Categories" button on the game page still lets you pick any 5
  of the available categories to use in a given game session — this list is
  now pulled live from Supabase instead of a hardcoded list.

## Feature list (all implemented and confirmed working)
- Title: "DELIVERY JEOPARDY!"
- Purple gradient background (`#667eea` → `#764ba2`)
- 5x10 game board, equal-width category columns, category text wraps within its box
- Click a cell → full-screen "teleprompter" modal: large prompt text, smaller directions text, minimal header/vertical clutter, preserves line breaks from source text
- Team score tracking; buttons read "Points for Team 1" / "Points for Team 2"; Team 2's name/score right-aligned
- Student name entry (textarea, one name per line) per team
- "Pick Random Student" button per team — selects without removing from the list
- Judges panel + "Select 2 Judges" button — randomly picks one student from each team and **removes** them from their team list, adding them to the judges list
- Extra vertical spacing added: after Reset/Change Categories buttons, and before the judges section
- "Reset Game" and "Change Categories" controls
- Progress (scores, used cells) persists via localStorage with a version check that auto-resets if the data structure changes (prevents stale-cache bugs)
- **New:** categories/prompts loaded from Supabase on page load, with a
  localStorage cache used as a fallback if Supabase is unreachable
- **New:** `admin.html` — login-gated page to add/edit/delete categories and prompts

## Suggested next steps (not yet started)
- Set up a stable hosting URL (GitHub Pages done right, or a named Netlify site/account) so updates don't require re-pasting a new iframe URL into Squarespace each time.
- If you want tighter admin security, switch `schema.sql`'s write policies to
  the commented-out "only this email" version instead of "any authenticated
  user."
- Consider a "duplicate category" button in `admin.html` if you end up
  creating many similar categories.

## Files
- `jeopardy_game.html` — the public game board (fetches from Supabase, read-only)
- `admin.html` — the content management page (requires Supabase login to edit)
- `schema.sql` — run once in Supabase SQL Editor to create tables + RLS policies
- `seed_data.sql` — run once after schema.sql to load the original 7 categories
