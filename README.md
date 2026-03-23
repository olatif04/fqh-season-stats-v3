# FQH Season Stats App

Vite + React + Supabase app for saving game recaps, auto-calculating season stats, and exporting season/game graphics as PNG.

## Rules implemented

- Team score is auto-calculated from **player goals only**.
- Goalies can record assists.
- Goalie points are **5 + assists + winning margin if they win**.
- Points combine player points and goalie points in one shared total.
- PPG is total points divided by total appearances.
- A person can appear as a player in one game and a goalie in another.

## Logo

Put your square logo file at:

`public/logo.png`

If it is missing, the export will still work.

## Env vars

Create `.env` from `.env.example`:

```bash
VITE_SUPABASE_URL=your-project-url
VITE_SUPABASE_ANON_KEY=your-publishable-key
```

## Database setup

Run this SQL in the Supabase SQL editor:

```sql
create extension if not exists pgcrypto;

create table if not exists games (
  id uuid primary key default gen_random_uuid(),
  game_date date not null,
  notes text default '',
  created_at timestamptz not null default now()
);

create table if not exists game_entries (
  id uuid primary key default gen_random_uuid(),
  game_id uuid not null references games(id) on delete cascade,
  name text not null,
  team text not null check (team in ('Red', 'Blue')),
  role text not null check (role in ('player', 'goalie')),
  goals integer not null default 0,
  assists integer not null default 0,
  created_at timestamptz not null default now()
);

create index if not exists idx_game_entries_game_id on game_entries(game_id);
create index if not exists idx_game_entries_name on game_entries(name);
```

Then enable RLS and add policies:

```sql
alter table games enable row level security;
alter table game_entries enable row level security;

create policy "public read games"
on games for select
to anon
using (true);

create policy "public insert games"
on games for insert
to anon
with check (true);

create policy "public update games"
on games for update
to anon
using (true);

create policy "public delete games"
on games for delete
to anon
using (true);

create policy "public read game_entries"
on game_entries for select
to anon
using (true);

create policy "public insert game_entries"
on game_entries for insert
to anon
with check (true);

create policy "public update game_entries"
on game_entries for update
to anon
using (true);

create policy "public delete game_entries"
on game_entries for delete
to anon
using (true);
```

## Local development

```bash
npm install
npm run dev
```

## Vercel notes

This package includes the build fixes we worked through:

- `engines.node` pinned to `20.x`
- `tsconfig.json` and `tsconfig.node.json` use `moduleResolution: bundler`
- `vercel.json` uses `npm ci`
- build script uses `node ./node_modules/vite/bin/vite.js build`
- `.gitignore` excludes `node_modules` and `dist`

Vercel env vars to add:

- `VITE_SUPABASE_URL`
- `VITE_SUPABASE_ANON_KEY`

## Deploy

1. Push repo to GitHub.
2. Import into Vercel.
3. Add env vars.
4. Deploy.

## Season selector

The app now includes dropdowns for:

- FQH season number
- season year range

That selection is stored in browser local storage and shown in the top-right of exported PNGs.

## Database reuse

You can keep using the same Supabase database for new repos and new Vercel projects as long as you point them at the same env vars.
This update does not require a schema change, so you do **not** need to rebuild the database for this zip.
