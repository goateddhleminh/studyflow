# ⚡ StudyFlow — Cloud-Synced Focus Tracker

A liquid-glass study timer with gamification, cloud sync, and cross-device progress.

## ✨ Features
- ⏱ Focus timer with live XP per second
- 🔥 Streaks with freeze cards and warnings
- 🎡 Lucky Wheel with 30 weighted reward slots
- 🎲 Gamble minigame (5 risk tiers, coin/token betting)
- 🛍 Reward shop with companions, themes, and cosmetics
- 🐾 Virtual study companion with mood states
- 🏆 18 achievements to unlock
- ⏱ Break Vault — track and redeem earned break time
- 👤 Profile with bio, motto, and avatar
- ☁️ Cloud-synced via Supabase (progress follows you everywhere)

## 🌐 Live Demo
**[https://YOUR-USERNAME.github.io/studyflow/](https://YOUR-USERNAME.github.io/studyflow/)**

## 🛠 Tech Stack
- Vanilla HTML / CSS / JavaScript (single file)
- Supabase for auth + cloud state persistence
- Canvas API for the Lucky Wheel
- No build step, no npm — just open the file

## 🚀 Run Locally
1. Download `index.html`
2. Open it in any modern browser
3. Configure your Supabase URL + anon key on first launch

## 🗄 Supabase Setup

Run this SQL in your Supabase project's SQL Editor:

\`\`\`sql
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
\`\`\`

Then paste your **Project URL** and **anon public key** (from Supabase → Settings → API)
into the app's config dialog.

## 📄 License
MIT — free to use, modify, and share.