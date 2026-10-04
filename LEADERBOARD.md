# Leaderboard

The ranking panel in the game shows whatever this page gives it. Scores are
kept in one of two places:

- **This browser only** (the default): `localStorage`. Nothing to set up, but
  each player only ever sees their own scores on their own device. It starts
  with three made-up players (`SEED` in `index.html`); bumping `SEED_VERSION`
  wipes every browser's saved scores back to that list.
- **Online, shared by everyone**: a free [Supabase](https://supabase.com)
  table. Set it up once, as below, then fill in `LEADERBOARD` at the top of the
  script in `index.html`.

## Setting up Supabase

1. Create a free project at supabase.com.
2. In **SQL Editor**, run:

   ```sql
   create table public.scores (
     id bigint generated always as identity primary key,
     name text not null check (char_length(name) between 1 and 9),
     score integer not null check (score between 1 and 1000000),
     created_at timestamptz not null default now()
   );

   alter table public.scores enable row level security;

   -- Anyone can read the ranking and add a score; nobody can edit or delete.
   create policy "read scores" on public.scores
     for select to anon using (true);
   create policy "add a score" on public.scores
     for insert to anon with check (true);

   create index scores_rank on public.scores (score desc, id);
   ```

3. Optionally, start the ranking with the same three made-up players the
   local version uses:

   ```sql
   insert into public.scores (name, score) values
     ('CICRANO', 6000),
     ('FULANO', 4000),
     ('ASTOLFO', 2000);
   ```

   To clear the ranking later: `truncate public.scores restart identity;`

4. In **Project Settings → API**, copy the **Project URL** and the **anon
   (publishable) key** into `index.html`:

   ```js
   const LEADERBOARD = {
     supabaseUrl: 'https://abcdefgh.supabase.co',
     supabaseKey: 'eyJhbGciOi...',
   };
   ```

The anon key is meant to be public; the policies above are what limit it to
reading and adding scores. It can't stop someone from sending a made-up score
straight to the table. The checks cap the damage, and you can delete a bad
row in Supabase's **Table Editor**.

## How the page and the game talk

Through the game's view model (`scene.rml`, `game.luau`):

| Property | Direction | Meaning |
|---|---|---|
| `gameEnded` (trigger) | game → page | a game just ended |
| `finalScore` | game → page | its score |
| `rankingData` | page → game | the list, best first: one `NAME<tab>SCORE` line per score (the page sends up to 100) |
| `rankHighlight` | page → game | position (1, 2, …) in that list to highlight, 0 for none |
| `rankingWheel` | page → game | mouse-wheel scrolling, in artboard pixels; the game reads it and sets it back to 0 |
| `rankingStatus` | page → game | a note shown while there are no rows ("loading..."); empty for none |
| `openRanking` (trigger) | page → game | opens the ranking panel |
