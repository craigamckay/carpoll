# Car Poll — Setup Guide

Everything runs from a single `index.html` file.
You need two free accounts: Supabase (database) and Vercel (hosting).
Total time: ~15 minutes.

---

## Step 1 — Create the Supabase database

1. Go to https://supabase.com and sign up (free)
2. Click **New project**, give it a name (e.g. "car-poll"), pick a region near you, set a password
3. Wait ~2 minutes for it to provision
4. Click **SQL Editor** in the left sidebar
5. Paste and run this SQL:

```sql
-- Create the cars table
create table cars (
  id         bigserial primary key,
  name       text not null,
  added_by   text not null default 'Anonymous',
  votes      integer not null default 0,
  created_at timestamptz default now()
);

-- Allow anyone to read and insert (public poll)
alter table cars enable row level security;

create policy "Anyone can read cars"
  on cars for select using (true);

create policy "Anyone can add cars"
  on cars for insert with check (true);

-- Stored functions for safe vote increment/decrement
create or replace function increment_votes(row_id bigint)
returns void language sql as $$
  update cars set votes = votes + 1 where id = row_id;
$$;

create or replace function decrement_votes(row_id bigint)
returns void language sql as $$
  update cars set votes = greatest(votes - 1, 0) where id = row_id;
$$;

-- Allow anyone to call the vote functions
grant execute on function increment_votes(bigint) to anon;
grant execute on function decrement_votes(bigint) to anon;
```

6. Go to **Project Settings → API** (left sidebar)
7. Copy two values:
   - **Project URL** (looks like `https://xxxx.supabase.co`)
   - **anon / public** key (long string under "Project API keys")

---

## Step 2 — Configure index.html

Open `index.html` in any text editor and find these two lines near the top of the `<script>`:

```js
const SUPABASE_URL = 'YOUR_SUPABASE_URL';
const SUPABASE_KEY = 'YOUR_SUPABASE_ANON_KEY';
```

Replace the placeholder strings with your values from Step 1. Example:

```js
const SUPABASE_URL = 'https://abcdefgh.supabase.co';
const SUPABASE_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...';
```

Save the file.

---

## Step 3 — Deploy to Vercel

1. Go to https://vercel.com and sign up with GitHub (free)
2. Click **Add New → Project**
3. Choose **"Deploy without a Git repository"** (the drag-and-drop option)
   - Or: put `index.html` in a folder, drag the folder into Vercel
4. Vercel will give you a URL like `https://car-poll-xyz.vercel.app`
5. That's your shareable link — send it to your friends!

### Optional: custom URL
In Vercel's project settings you can add a custom domain if you have one,
or rename the project to get a cleaner URL like `https://our-car-poll.vercel.app`.

---

## How voting works

- Anyone can add any number of cars
- Cars start at 0 votes
- Each person votes per device/browser (stored in localStorage)
- Votes are tallied live — the page refreshes every 8 seconds automatically
- The person who adds a car gets no automatic votes

---

## Troubleshooting

| Problem | Fix |
|---|---|
| "Configure Supabase" banner shows | You haven't replaced the placeholder values in index.html |
| Cars not loading | Check the browser console — likely a wrong URL or key |
| Votes not saving | Make sure you ran all the SQL in Step 1, including the `grant execute` lines |
| Page looks broken | Make sure you're opening the deployed Vercel URL, not the raw HTML file |
