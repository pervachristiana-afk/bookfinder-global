# BookFinder Global

A single-page app (React + Tailwind + Leaflet, no build step needed) that:

- Searches Open Library live, with Google Books as a backup source
- Browses by category via Open Library's subject API
- Shows real book covers, descriptions, and details
- Finds nearby bookshops using your real GPS location + the free OpenStreetMap Overpass API, shown on a live Leaflet map
- Lets anyone add their own bookshop (saved in the visitor's browser via localStorage) — useful for shops not mapped on OpenStreetMap, e.g. roadside or informal sellers anywhere in the world
- Dark/light mode toggle

Everything lives in one file: `index.html`. There is nothing to install and nothing to build.

## Deploy on Vercel (free, ~10 minutes, no coding required)

**Option A — easiest, no account setup needed on your computer:**

1. Go to https://vercel.com and sign up (free — you can use your email or GitHub).
2. Once logged in, click **"Add New..." → "Project"**.
3. Choose **"Deploy without Git"** / look for a drag-and-drop upload area (Vercel calls this a "template" or direct upload flow — if you don't see it immediately, use Option B below, which is the more standard path).
4. Drag the whole `bookfinder-global` folder (containing `index.html`) into the upload area.
5. Click **Deploy**. Vercel will give you a live URL like `bookfinder-global.vercel.app` within about a minute.

**Option B — the standard path (works every time):**

1. Create a free GitHub account at https://github.com if you don't have one.
2. Create a new repository (e.g. `bookfinder-global`), and upload `index.html` and this `README.md` to it (GitHub's website lets you drag files in directly — no command line needed).
3. Go to https://vercel.com, sign up using your GitHub account.
4. Click **"Add New..." → "Project"**, then select the `bookfinder-global` repository you just created.
5. Leave all settings as default (Vercel auto-detects a static site) and click **Deploy**.
6. After about a minute, you'll get a live link you can share with anyone in the world.

## Notes

- The app needs the visitor's **location permission** to find nearby shops — their browser will ask for this the first time they tap the location icon.
- The Overpass API (used for real bookshop data) is a free public service and can occasionally be slow or rate-limited under heavy traffic — this is normal and outside this app's control.
- Google Books is only used as a backup if Open Library returns no results for a search.

## Optional: make community-added bookshops visible to everyone (Supabase setup)

By default, "Add Your Bookshop" only saves on the visitor's own device. To make every added shop visible to **all** visitors, connect a free shared database:

1. Go to https://supabase.com and sign up (free, no credit card).
2. Click **"New Project"**. Give it any name, set a database password (save it somewhere), pick any region, and click **Create**. Wait about a minute for it to finish setting up.
3. In your new project, click **"SQL Editor"** in the left sidebar, then **"New query"**, and paste in this exact code, then click **Run**:

   ```sql
   create table sellers (
     id uuid primary key default gen_random_uuid(),
     name text not null,
     area text not null,
     address text,
     phone text not null,
     lat double precision not null,
     lng double precision not null,
     created_at timestamp with time zone default now()
   );

   alter table sellers enable row level security;

   create policy "Anyone can view sellers"
     on sellers for select
     using (true);

   create policy "Anyone can add a seller"
     on sellers for insert
     with check (true);
   ```

4. In the left sidebar, click **"Project Settings" → "API"**. You'll see two values you need:
   - **Project URL** (looks like `https://xxxxx.supabase.co`)
   - **anon public** key (a long string under "Project API keys")
5. Open `index.html` in a text editor, find this section near the top of the `<script type="text/babel">` block:

   ```js
   const SUPABASE_URL = "";
   const SUPABASE_ANON_KEY = "";
   ```

   Paste your Project URL and anon public key between the quotes.
6. Save the file, and re-deploy (re-upload to Vercel, or push to GitHub if using Option B — Vercel redeploys automatically on every push).

That's it — every bookshop anyone adds from anywhere will now show up for everyone else too. If you skip this setup, the app still works exactly as before, just saving added shops per-device.
