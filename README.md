# Zippee Location Master

A lightweight, single-file store/location master dashboard for Zippee's dark-store network — built to replace the WhatsApp-group-as-a-database workflow. Add, edit, search, and bulk-copy store details for riders and ops teams, backed by Supabase.

<img width="1666" height="911" alt="Screenshot 2026-07-13 115231" src="https://github.com/user-attachments/assets/797a5206-a924-498f-9e07-20eeb6917204" />


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

## Three levels of access

| File | Who gets the link | What they can do |
|---|---|---|
| `store-master-dashboard.html` | Central/BizOps team | Full access — view, add, edit, delete, import/export Excel |
| `limited-access.html` | Regional manager, a specific team, etc. | Sees **only** a locked subset of stores (by city and/or brand), can **edit** those stores and export them to Excel — but **can't delete anything**, add new stores, or see stores outside their scope |
| `add-store.html` | Store team | Can **only submit** a brand-new store — no table view at all, no edit, no delete |

### Setting up `limited-access.html`

Open the file and set:

```js
const SUPABASE_URL = "https://your-project.supabase.co";     // same as the other two files
const SUPABASE_ANON_KEY = "your-anon-public-key";             // same as the other two files
const LIMITED_ACCESS_CODE = "whatever-you-want-to-share";     // different from the other two access codes

// Lock this copy to a specific slice of stores:
const SCOPE_FILTER = { city: "Hyderabad", brandCode: "" };    // only Hyderabad stores, any brand
// or: { city: "", brandCode: "FKM" }                          only FKM stores, any city
// or: { city: "Mumbai", brandCode: "HY" }                      only HY stores in Mumbai
```

**If you need to give *different* people *different* scopes** (e.g. one person for Hyderabad, another for Mumbai), make a separate copy of this file per person — each with its own `SCOPE_FILTER` and its own `LIMITED_ACCESS_CODE` — and deploy each as its own page (e.g. `limited-hyderabad.html`, `limited-mumbai.html`). They all read/write the same Supabase table, so edits show up in the main dashboard immediately either way.

**Screenshots:**

| Login (scoped to a city) | Dashboard (edit, no delete) |
|---|---|
| ![Limited login](./screenshots/limited-login.png) | ![Limited dashboard](./screenshots/limited-dashboard.png) |

All three pages point at the same Supabase project and the same `allow all for anon` policy from the SQL above — the separation between them is purely **which link and access code you hand out to whom**, not a database-level rule. There's also a quick link to the submission form right in the main dashboard's header (**"Open store submission form"**) so you can always grab the latest link to share.

### Setting up `add-store.html`

Open the file and set:

```js
const SUPABASE_URL = "https://your-project.supabase.co";   // same as the dashboard
const SUPABASE_ANON_KEY = "your-anon-public-key";           // same as the dashboard
const STORE_ACCESS_CODE = "whatever-you-want-to-share";     // a simple shared code, different from the other access codes
```

`STORE_ACCESS_CODE` is just a lightweight gate so a random person who stumbles on the link can't spam-submit — it's not real authentication, just a shared code you hand out to the store team along with the link.

**Screenshots:**

| Access code | Submission form |
|---|---|
| ![Add store gate](./screenshots/add-store-gate.png) | ![Add store form](./screenshots/add-store-form.png) |

## Excel import/export format

When importing, the sheet's first row should contain these headers (case-insensitive, order doesn't matter):

| Store name | Brand | Address | Location | Maps link | POC name | POC number |
|---|---|---|---|---|---|---|

Exporting produces a `.xlsx` file in the same column order.

## Security notes

- **Don't commit real secrets to a public repo.** The Supabase anon key is safe to ship client-side *only* because of the Row Level Security policy above — but the dashboard login ID/password, `LIMITED_ACCESS_CODE`, and `STORE_ACCESS_CODE` are all basic gates, not real authentication. If this repo is public, replace all of them with placeholders before committing, and set the real values only in your deployed copies.
- The real protection here is **which link + code you hand to whom** — since all three pages share the same open Supabase policy, anyone with the main dashboard's credentials has full access. Keep that link and password inside the central team only; give `limited-access.html` to people who need a scoped view, and `add-store.html` to anyone who should only ever submit new stores.
- Consider adding all three HTML files (with real secrets filled in) to `.gitignore` and committing `*.template.html` versions instead, if you want to keep every secret out of version control entirely.

## License

Internal tool — add a license here if you plan to open-source it.
