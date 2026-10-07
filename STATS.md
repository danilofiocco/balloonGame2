# Play statistics

`stats/index.html` (published at `/balloonGame2/stats/`) shows how the game is
played. The game page (`index.html`, "Play statistics") sends anonymous events
to the Supabase table `balloon2_events` — only from the published site, not
from localhost (add `?track` to a local URL to send anyway).

Each game is a session (a restart starts a new one):

- `start` when it begins,
- `boss` for each boss defeated: `{ boss, shots }` (bow shots from its
  appearance to its defeat),
- `end` with the totals: `{ reason, shots, earned, spent, time, arrows, cheated, … }`
  where `reason` is `gameover`, `quit` (restarted first) or `left` (closed or
  hid the page; a later `end` for the same session replaces it).

Games where the cheat menu (or its keys Z, A, 1-3) was used are left out of the stats.

## Creating the table

Run once in the Supabase SQL Editor:

```sql
create table if not exists public.balloon2_events (
  id bigint generated always as identity primary key,
  session uuid not null,
  kind text not null check (kind in ('start', 'boss', 'end')),
  data jsonb not null default '{}' check (pg_column_size(data) < 4000),
  version text,
  created_at timestamptz not null default now()
);
alter table public.balloon2_events enable row level security;
create policy "read events" on public.balloon2_events for select to anon using (true);
create policy "add an event" on public.balloon2_events for insert to anon with check (true);
create index if not exists balloon2_events_session on public.balloon2_events (session);
```

To start the stats over: `truncate public.balloon2_events;`
