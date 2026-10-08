# ⚡ StudyFlow — Cloud-Synced Focus Tracker

A single-file liquid-glass study app: focus timer, planner, rewards, and gamification —
all cloud-synced through Supabase so progress follows you across devices.

Everything lives in one `index.html` (~300 KB, no build step, no dependencies to install).

## ✨ Features

### 🗂 Planner
Three sub-sections, each on its own page, switched from a ribbon on the left of the section:

- **To-Do List (today)** — add and delete tasks, set a **priority** (High / Medium / Low)
  and a **status** (To Do, In Progress, Later, Blocked, Done), tick the **Done** checkbox to
  complete a task, filter by All / Active / Done, and reset every tick with **Uncheck done**
  (clears ticks without deleting anything). Shows today's progress bar.
- **Weekly Schedule** — seven day cards, Monday → Sunday, with today highlighted. Add, tick,
  rename (click the text) and delete tasks per day, keep a free-text note per day (auto-saves),
  and clear the whole week's ticks at once with **Uncheck all done** — tasks are kept.
- **Study Resources** — create your own subjects, then attach **links** and **files** to each
  one. Bare domains like `khanacademy.org` are normalised to `https://` and open in a new tab;
  files up to 2 MB are stored with your account and synced, larger ones are session-only.

Planner data lives in `state.todos`, `state.weekPlan`, `state.resources` and
`state.resourceSubjects`, so it cloud-syncs like the rest of your progress.

### 🎁 Planner rewards

Rewards are paid the moment a task is ticked (or a status is set to *Done*), and shown in a
toast plus an on-page reward table. Bonuses stack — the task that finishes a day pays its own
reward **and** the day bonus; the task that finishes the week pays everything.

| Action | Coins | Wheel tokens | XP |
| --- | --- | --- | --- |
| Task done in the To-Do list | +100 🪙 | +1 🎡 | +200 ✦ |
| Task done in the Weekly Schedule | +150 🪙 | — | +200 ✦ |
| 100% of a day's tasks | — | +2 🎡 | +500 ✦ |
| 100% of the week's tasks | +1000 🪙 | +5 🎡 | +1000 ✦ |

- Unticking a task never pays and never takes anything back — re-ticking pays again.
- The week bonus needs all seven days populated with every task ticked (one finished day is
  not a finished week).
- Reward tokens go into a separate `state.bonusTokens` balance, so they add to the tokens you
  earn from focus time and are spendable on the wheel straight away.
- Two achievements come with it: **Plan Master** (finish every task of one day) and
  **Clean Sweep** (finish the whole week).

### ⏱ Focus & progress
- Focus timer with live XP, coins and tokens while a session runs
- Immersive focus mode with fullscreen support and a session summary
- Daily study goal, editable from the timer or the dashboard
- 🔥 Streaks with freeze cards and warnings
- 📊 Statistics — 14-day bars, subject breakdown, full session history, 13-week heatmap

### 🎮 Gamification
- 🎡 **Lucky Wheel** — 30 weighted reward slots, from break time to legendary cosmetics
- 🎲 **Gamble** — roll three dice and bet Over, Under, Even, Odd or Any Triple with coins,
  tokens or break minutes, with client/server seed + nonce verification
- 🛍 **Reward Shop** — study companions, XP boosts, profile themes, avatar borders and titles
- 🐾 **Virtual study companion** — moods, feeding, buffs and evolutions
- ⏱ **Break Vault** — track and redeem the break time you've earned
- 🏆 **70 achievements** with rewards (XP, coins, tokens, titles)
- 👤 **Profile** — display name, motto, biography, 40 avatars, themes and borders
- ☁️ **Cloud sync** — email/password auth, cloud state persistence, live sync indicator

## 🧭 Navigation

The floating dock at the top switches between ten sections:

1. **Dashboard** — motivation, companion, weekly overview, heatmap
2. **Planner** — To-Do List · Weekly Schedule · Study Resources (left ribbon)
3. **Focus** — the timer, daily goal and session stats
4. **Gamble** — dice betting
5. **Wheel** — token spins and rewards
6. **Shop** — spend coins on cosmetics and boosts
7. **Breaks** — your earned break vault
8. **Awards** — achievement collection
9. **Stats** — longer-term analytics
10. **Profile** — identity, avatar and account

On narrow screens the dock scrolls horizontally, the planner ribbon collapses into a
horizontal strip, and the week grid and resource cards stack into one column.

## ✅ Tests

The workspace includes harnesses I use to verify `index.html` without a real browser
(they need `jsdom` / `playwright-core` in a temp folder — the app itself needs nothing):

```bash
node planner_test.js                      # 127 UI + data assertions for the Planner
node render_planner.js                    # renders the Planner in real Chrome for visual review
node tools/smoke-tests.cjs                # syntax + boot smoke test of the whole app
node tools/func-tests.cjs                 # drives the app's UI with a fake DOM
node tools/audit-scope.cjs                # scope-aware static audit
node tools/build-preview.cjs              # writes tools/preview.html with demo data
```

`planner_test.js` covers all three Planner sub-pages plus the reward system: adding, ticking,
unticking, filtering, renaming, deleting, day notes surviving re-renders, link/file resources,
corrupt-state recovery, reload persistence, and the exact payout of all four reward tiers —
including a regression check that a single completed day can never pay the whole-week bonus.

## 🌐 Live Demo

**[https://goateddhleminh.github.io/studyflow/](https://YOUR-USERNAME.github.io/studyflow/)**

## 🛠 Tech Stack

- Vanilla HTML / CSS / JavaScript — one self-contained file
- Supabase (`supabase-js` via CDN) for auth + cloud state persistence
- Canvas API for the Lucky Wheel
- Web Audio API for sound alerts
- No build step and no package manager — just open the file

## 🚀 Run Locally

1. Download `index.html`
2. Open it in any modern browser
3. Sign in or create an account — the Supabase project is pre-configured in the file
   (you can point it at your own project in Settings if you prefer)

## 🗄 Supabase Setup

Run this SQL in your Supabase project's SQL Editor:

```sql
create table if not exists studyflow_state (
  user_id uuid references auth.users(id) on delete cascade primary key,
  data jsonb not null default '{}'::jsonb,
  updated_at timestamptz default now()
);

alter table studyflow_state enable row level security;

create policy "Users read own state"
  on studyflow_state for select using (auth.uid() = user_id);
create policy "Users insert own state"
  on studyflow_state for insert with check (auth.uid() = user_id);
create policy "Users update own state"
  on studyflow_state for update using (auth.uid() = user_id);
create policy "Users delete own state"
  on studyflow_state for delete using (auth.uid() = user_id);
```

All progress — sessions, XP, coins, achievements, planner tasks, weekly schedule, resources,
inventory and profile — lives in that single `data` JSON column, which is why no schema change
is needed when a new feature is added.

## 📄 License

MIT — free to use, modify, and share.
