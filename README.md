# Zippee Location Master

A lightweight, single-file store/location master dashboard for Zippee's dark-store network — built to replace the WhatsApp-group-as-a-database workflow. Add, edit, search, and bulk-copy store details for riders and ops teams, backed by Supabase.

<img width="1666" height="911" alt="image" src="https://github.com/user-attachments/assets/03d69a61-d580-40ec-8741-b9339d28b876" />


## Features

- 🔐 **Simple login gate** to keep the dashboard internal
- 🏬 **Store master table** — brand code, location, city, full address, Google Maps link, POC name & phone
- ✏️ **Full CRUD** — add, edit, and delete stores inline
- 🔎 **Search & filter** by brand code or city
- ☑️ **Multi-select → Copy** — select any number of stores and copy their full details to your clipboard in one click, ready to paste into WhatsApp, email, or anywhere else
- 📥📤 **Excel import/export** — bulk-load your existing store sheet, or export the current list back to `.xlsx`
- ☁️ **Supabase-backed** — every add/edit/delete/import writes straight to a real Postgres table, shared live across everyone using the dashboard
- 🎨 Light theme, single HTML file, no build step, no framework


## Tech stack

- Plain HTML/CSS/JS — no build tools, no npm install, just open the file
- [SheetJS (xlsx)](https://cdnjs.com/libraries/xlsx) via CDN for Excel import/export
- [Supabase](https://supabase.com) (Postgres + REST API) as the backend

## Getting started

### 1. Clone this repo

```bash
git clone https://github.com/<your-org>/<your-repo>.git
cd <your-repo>
```

### 2. Create the Supabase table

In your [Supabase project](https://supabase.com/dashboard) → **SQL Editor** → New query, run:

```sql
create table if not exists stores (
  id text primary key,
  store_name text,
  brand_code text,
  location text,
  city text,
  address text,
  maps_link text,
  poc_name text,
  poc_phone text,
  created_at timestamptz default now()
);

alter table stores enable row level security;

create policy "allow all for anon"
  on stores
  for all
  using (true)
  with check (true);

create index if not exists idx_stores_brand_code on stores (brand_code);
create index if not exists idx_stores_city on stores (city);
```

> The `"allow all for anon"` policy is intentionally permissive for an internal tool that already sits behind the dashboard's own login. Tighten it if you ever expose the anon key more broadly.

### 3. Add your Supabase credentials

Open `store-master-dashboard.html`, find the `SUPABASE CONFIG` block near the top of the `<script>` tag, and fill in your project's URL and anon key (**Project Settings → API** in Supabase):

```js
const SUPABASE_URL = "https://your-project.supabase.co";
const SUPABASE_ANON_KEY = "your-anon-public-key";
```

Leave both blank to run in **local mode** instead — data will be stored in the browser session only, useful for a quick demo.

### 4. Set your login credentials

Find the login check inside the `submit` event listener and update the hardcoded ID/password to your own:

```js
if(u==="Bizops2026" && p==="Zippee@123"){
```

### 5. Open it

No build step — just open `store-master-dashboard.html` in a browser, or host it on any static file host (GitHub Pages, Netlify, S3, etc.).

## Excel import/export format

When importing, the sheet's first row should contain these headers (case-insensitive, order doesn't matter):

| Store name | Brand | Address | Location | POC name | POC number |
|---|---|---|---|---|---|

Exporting produces a `.xlsx` file in the same column order.

## Security notes

- **Don't commit real secrets to a public repo.** The Supabase anon key is safe to ship client-side *only* because of the Row Level Security policy above — but the login ID/password in this file are a basic UI gate, not real authentication. If this repo is public, replace both with placeholders before committing, and set the real values only in your deployed copy.
- Consider adding `store-master-dashboard.html` (with real credentials filled in) to `.gitignore` and committing a `store-master-dashboard.template.html` instead, if you want to keep secrets out of version control entirely.

## License

Internal tool — add a license here if you plan to open-source it.
