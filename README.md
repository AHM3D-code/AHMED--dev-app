# Ahmed.Dev

A premium developer portfolio / digital product platform / Discord bot marketplace, built with React + Vite + TypeScript + Tailwind CSS, with a real Discord OAuth backend on Netlify Functions.

## What's real vs. what's mock

- **Real:** Discord login (`netlify/functions/auth.js`), sessions (httpOnly signed cookies), admin role gating via `ADMIN_DISCORD_IDS`.
- **Mock / local-only (no database yet):** projects, bots, free resources, users list, announcements, and site settings (banner image, site name) — all editable from `/admin`, persisted in the browser's `localStorage` so edits survive a reload, but they're per-browser, not shared across visitors. Profile fields like bio/website/avatar are layered on top of the real Discord session the same way.

## 1) Discord OAuth app

1. Go to [discord.com/developers/applications](https://discord.com/developers/applications) → **New Application** → name it "Ahmed.Dev".
2. **OAuth2 → General**: copy the **Client ID** and **Client Secret**.
3. **OAuth2 → General → Redirects**, add both:
   - `http://localhost:8888/api/auth/discord/callback` (local dev, via `netlify dev`)
   - `https://yourdomain.netlify.app/api/auth/discord/callback` (your real deployed domain)
4. To find your own Discord user ID: enable **Developer Mode** (Discord Settings → Advanced), then right-click your name → **Copy User ID**.

## 2) Environment variables

```bash
cp .env.example .env
```

Fill in:
- `JWT_SECRET` — any long random string (`node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"`)
- `DISCORD_CLIENT_ID` / `DISCORD_CLIENT_SECRET`
- `ADMIN_DISCORD_IDS` — your Discord user ID (comma-separate for more than one). Anyone signing in with an ID in this list gets the **Owner** role and `/admin` access; everyone else gets **User**.

## 3) Run locally

```bash
npm install
npm run dev:full
```

`npm run dev:full` runs `netlify dev`, which starts the Vite dev server **and** the Discord OAuth functions together on `http://localhost:8888` — use this URL, not the plain Vite one, whenever you're testing login.

(`npm run dev` alone still works for pure UI iteration, just without a working login — `/api/auth/*` won't exist.)

First run may ask you to log in to Netlify or link a site — you can skip/decline that prompt; it still runs functions locally either way.

## 4) Deploy to Netlify

- Push to GitHub and connect the repo in Netlify (reads `netlify.toml` automatically), **or** drag the `dist` folder (after `npm run build`) to [app.netlify.com/drop](https://app.netlify.com/drop) for a quick one-off deploy — note the drop method won't include the functions, so use the GitHub connection (or `netlify deploy --prod`) for a fully working deploy.
- In **Site settings → Environment variables**, add the same variables from your `.env`.
- Update the Discord app's redirect URI once you know your real Netlify domain (step 1.3 above).

## Project structure

```
src/                        Frontend (React + Vite + TypeScript + Tailwind)
  pages/                     Route pages, incl. pages/admin/
  components/                Shared UI (Glass* design system, Navbar, cards...)
  lib/auth.tsx                Real session state (talks to netlify/functions/auth.js)
  lib/dataStore.ts            Shared, localStorage-persisted content (projects/bots/...)
  lib/siteSettings.ts         Site name/tagline/banner image, same persistence pattern

netlify/functions/
  auth.js                    Discord OAuth: /discord/start, /discord/callback, /me, /logout
  lib/auth.js                 Session JWT + cookie helpers
```
