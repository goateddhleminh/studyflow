# 📚 StudyFlow

> A liquid-glass study timer with deep gamification, cloud sync, and cross-device progress.

**🌐 Live App:** [goateddhleminh.github.io/studyflow](https://goateddhleminh.github.io/studyflow/)

---

## ✨ What is StudyFlow?

StudyFlow is a single-file web app that turns studying into a game. Focus timer meets RPG progression — earn XP, level up, gamble with dice, spin the Lucky Wheel, adopt a virtual companion, and compete for the top spot on the leaderboard. Everything syncs to your Supabase account in real time, so your progress follows you to any device.

---

## 🎯 Core Features

### ⏱️ Focus Timer
- **Count-up stopwatch** — no forced session lengths, just start and study
- **Live XP + coins** awarded as you study (10 XP / 5 min · 1 coin / 3 min)
- **Auto-recovery** — if you close the tab mid-session, your partial time is saved and logged on next login
- **Break Timer** — redeem earned break time from your Break Vault. Counts down and chimes when finished.
- **Custom subjects** — add your own, delete any custom ones
- **Session history** with per-subject stats and a 13-week heatmap

### 🎲 Gamble — Sic Bo / Over-Under
Classic 3-dice Over/Under with multiple bet types:

| Bet | Wins when | Payout |
|---|---|---|
| **Over** | Sum 11–17 (no triple) | 1:1 |
| **Under** | Sum 4–10 (no triple) | 1:1 |
| **Even** | Even total | 1:1 |
| **Odd** | Odd total | 1:1 |
| **Any Triple** | All 3 dice match | 1:30 |

- **Provably fair RNG** — SHA-256(server_seed : client_seed : nonce) → dice values
- Bet with **Coins**, **Spin Tokens**, or **Break Minutes**
- Manual roll — place your bet, watch the 3-second dice animation, instant payout

### 🎡 Lucky Wheel
- **30-slot animated wheel** with weighted rarity distribution
- **1 Spin Token per 30 minutes** of focus time
- Rewards: break time (+5m to +2h), snacks, soda, and a legendary jackpot
- **Fully customizable** — edit icons and text for any reward type

### 🏆 70 Achievements
Tiered progression from "First Step" (complete your first 25-minute session) to "Focus Titan" (1,000 total hours). Rewards include XP, Coins, Spin Tokens, and **custom profile titles**.

### 🐾 Virtual Study Companion
An animated companion whose mood changes based on your study habits:
- **Energized** — hit your daily goal
- **Happy** — active streak
- **Sleepy** — 2 days without studying
- **Sad** — 3+ days away

Feed it for a 24-hour **+5% XP buff**. Unlock 9 different companions in the shop.

### 🛍️ Reward Shop
Spend Study Coins on:
- **Companions** (9 total — Mystic Orb, Clever Fox, Wise Owl, Study Dragon and more)
- **XP Booster 2×** (24-hour doubling)
- **Streak Freeze Cards** (protect your streak, max 3)
- **UI Themes** (Lavender, Mint, Warm)
- **Pet Frames** (Gold, Silver, Neon)
- **Profile Themes** (Sunset, Ocean, Forest, Lavender, Mono)
- **Avatar Borders** (Glow, Gold, Silver, Neon, Rainbow, Double)

### 🏅 Leaderboard
Weekly XP rankings with three filter tabs:
- **Global** — see how you rank against everyone
- **Friends** — your inner circle
- **Subject Class** — compete with your classmates

Top 3 ranks show **gold/silver/bronze** medals.

### 👤 Profile
- **40 avatars** — symbols and emoji (🦊 🐉 🚀 💎 and more)
- **Editable bio, motto, and display name**
- **Profile themes & avatar borders** equipable from the shop
- **Custom titles** unlocked via achievements
- **Your Journey** panel showing level, XP, streak, hours, sessions, and achievements

### 💾 Break Vault
- **Earn break time** from the Lucky Wheel and Gamble wins
- **Live jar visualization** filling up as you accumulate
- **Redeem** in 5m / 15m / 30m / 1h chunks — cancels refund the unused portion
- **Sources breakdown** — see where your break time came from

---

## 🎨 Design

- **Liquid glass** aesthetic — frosted translucent panels with animated gradient orbs floating behind
- **Pure dark theme** with white accents by default
- **Three unlockable UI themes** — Lavender, Mint, Warm
- **Fully responsive** — works beautifully from phone to ultrawide
- **Custom fonts** — Inter for UI, JetBrains Mono for timers

---

## 🛠️ Tech Stack

- **Vanilla HTML/CSS/JavaScript** — one file, no build step, no dependencies
- **Supabase** — auth, cloud sync, Postgres state storage
- **Canvas API** — Lucky Wheel rendering and spin animation
- **Web Crypto API** — provably fair SHA-256 hashing
- **Web Audio API** — procedural chimes and ticks
- **localStorage** — only for caching the Supabase URL/key (never user data)

---

## 🚀 Getting Started

### Try it live
Visit **[goateddhleminh.github.io/studyflow](https://goateddhleminh.github.io/studyflow/)** — no installation needed.

### Run locally
1. Download `index.html`
2. Open it in any modern browser (Chrome, Firefox, Safari, Edge)
3. Sign up and start studying

### Self-host your own backend
StudyFlow ships with a public Supabase project. To use your own:

1. Create a project at [supabase.com](https://supabase.com)
2. Open the **SQL Editor** and run:

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
