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

## 2. Set up the global leaderboard (optional, for online scores)

You'll use Supabase's free tier. The anon key is safe to embed in the frontend as long as Row Level Security (RLS) is configured.

1. Create a free project at https://supabase.com
2. In the SQL editor, run:

   ```sql
   create table tetris_scores (
     id uuid primary key default gen_random_uuid(),
     name text not null check (char_length(name) <= 12),
     category text not null check (category in ('height','time')),
     score integer not null check (score >= 0),
     created_at timestamptz not null default now()
   );

   create index tetris_scores_category_score_idx
     on tetris_scores (category, score desc);

   alter table tetris_scores enable row level security;

   create policy "anyone can read"
     on tetris_scores for select
     using (true);

   create policy "anyone can insert"
     on tetris_scores for insert
     with check (
       char_length(name) between 1 and 12
       and category in ('height','time')
       and score >= 0
       and score < 1000000
     );
   ```

3. Project Settings → API: copy the **Project URL** and the **anon public key** into `config.js`.

That's it. The game posts directly to the REST API; no backend code on your side.

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
