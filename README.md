# Justin's Game Launcher

A static site that hosts a collection of browser games with global leaderboards.

Live: deploy to GitHub Pages and link from your portfolio.

## Structure

```
.
├── index.html              # The launcher (game list, header, footer)
├── styles.css
├── config.js               # YOUR config — edit this
├── games/
│   └── tetris-survivor/
│       └── index.html      # The game; reads window.LAUNCHER_CONFIG
└── README.md
```

## 1. Configure

Open `config.js` and fill in:

- `PORTFOLIO_URL` — the link used in the launcher's header and footer.
- `SUPABASE_URL` and `SUPABASE_ANON_KEY` — leave blank for localStorage-only mode (the game still works, scores just stay on the device).

## 2. Set up the global leaderboard

You'll use Supabase's free tier. The anon key is safe to embed in the frontend — Row Level Security restricts what clients can do.

### One-time setup (~3 minutes)

1. Go to https://supabase.com → **Start your project** (sign in with GitHub).
2. **New project** → name it anything → set a DB password → **Create**.
3. Wait for provisioning (~1 min). Then open **SQL Editor** → **New query** → paste the block below → **Run**.
4. Open **Project Settings → API** → copy:
   - **Project URL** → paste into `SUPABASE_URL` in `config.js`
   - **anon public key** → paste into `SUPABASE_ANON_KEY` in `config.js`
5. `git add config.js && git commit -m "Connect Supabase" && git push`

### Setup + reset SQL (paste into Supabase SQL Editor)

This is idempotent — safe to run again anytime to wipe the leaderboard.

```sql
drop table if exists tetris_scores cascade;

create table tetris_scores (
  id uuid primary key default gen_random_uuid(),
  name text not null check (char_length(name) between 1 and 12),
  category text not null check (category in ('height','time')),
  score integer not null check (score >= 0 and score < 1000000),
  created_at timestamptz not null default now()
);

create index tetris_scores_category_score_idx
  on tetris_scores (category, score desc);

alter table tetris_scores enable row level security;

create policy "public read"
  on tetris_scores for select
  using (true);

create policy "public insert"
  on tetris_scores for insert
  with check (true);
```

### Reset only (keep the table)

```sql
truncate tetris_scores;
```

## 3. Deploy to GitHub Pages

```bash
# from this directory
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/<your-username>/<repo-name>.git
git push -u origin main
```

Then on GitHub: **Settings → Pages → Source: deploy from branch → main / (root)**.

After ~1 minute the site is live at `https://<your-username>.github.io/<repo-name>/`.

## Notes

- The Supabase anon key is intended for client use; RLS limits what clients can do.
- If you want to harden against spam, add Cloudflare Turnstile or rate-limit by IP via a Postgres function. Out of scope here.
- For local dev: just open `index.html` in a browser. Supabase calls work via CORS by default.
