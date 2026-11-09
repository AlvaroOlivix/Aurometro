# Aurometro

**The official aura scoreboard.** An Android app for running dance battles, trends and emotes, scoring them with a live jury and following them from the stands with predictions and signs.

> Personal project, built end to end: Android app, backend and public website.
> Currently in closed testing on Google Play. The source code is private; this repository shows what the app does and how it is built.

> [!NOTE]
> These screenshots are from **beta 1.0.0**, still under construction. Design and features may change before release, and ads shown are test ads.

### Progression

<p align="center">
  <img src="screenshots/01-techniques.jpg" width="190" alt="Technique catalogue by rank">
  <img src="screenshots/02-technique-detail.jpg" width="190" alt="Technique detail with video tutorial">
  <img src="screenshots/03-challenges.jpg" width="190" alt="Daily and weekly challenges">
  <img src="screenshots/04-shop.jpg" width="190" alt="Daily shop">
</p>

### Battles

<p align="center">
  <img src="screenshots/05-prediction.jpg" width="220" alt="Audience prediction before the battle">
  <img src="screenshots/06-judge-scoreboard.jpg" width="220" alt="Judge scoreboard during a round">
  <img src="screenshots/07-battle-results.jpg" width="220" alt="Battle results and tournament bracket">
</p>

<p align="center">
  <img src="screenshots/08-stands-landscape.jpg" width="680" alt="Audience view in landscape during a live battle">
</p>

<p align="center">
  <img src="screenshots/09-verdict.jpg" width="400" alt="Battle verdict">
  <img src="screenshots/10-round-scores.jpg" width="400" alt="Average jury scores per round">
</p>

## What it does

**Battles.** Two sides face off one on one, 2 vs 2 or 4 vs 4, best of five rounds. The organizer sets up the teams, judges, visibility and time per turn, and invites people by their bib number or a shareable link.

**Live jury.** Each judge scores every round from their own phone on technique, aura and creativity. The head judge opens and closes rounds and settles ties, either with a rematch or by letting the audience decide.

**The stands.** The audience joins by link or code, makes a prediction before the battle starts and follows it live in landscape: round scoreboard, turn timer, animated alerts and signs to raise for their side.

**Progression.** Players earn aura by competing, judging and predicting correctly, and spend it to unlock techniques (dances, trends, emotes and poses with video tutorials). Unlocking techniques is what moves you up the ranks.

**Sharing.** Every finished battle produces a result card that can be shared. Links open the public website for anyone who doesn't have the app.

## Architecture

```mermaid
flowchart LR
    subgraph App["Android app"]
        UI["Jetpack Compose UI"] --> VM["ViewModel (MVI)"]
        VM --> UC["Use cases"]
        UC --> REPO["Repositories"]
    end

    subgraph Backend["Supabase"]
        AUTH["Auth"]
        DB["Postgres<br/>game logic in SQL + RLS"]
        RT["Realtime"]
        EF["Edge Functions"]
        ST["Storage"]
        CRON["Scheduled jobs"]
    end

    REPO --> AUTH
    REPO --> DB
    REPO --> RT
    REPO --> EF
    EF --> ST
    CRON --> DB

    EF --> PLAY["Google Play<br/>purchases"]
    EF --> FCM["Firebase Cloud<br/>Messaging"]
    EF --> ADMOB["AdMob<br/>reward validation"]

    WEB["Public website<br/>battle links and results"] --> DB
```

More detail in [docs/architecture.md](docs/architecture.md).

### Key decisions

- **Game logic lives on the server.** Scores, rounds, rewards and the in-app economy are resolved by SQL functions in Postgres. The app only displays the state and calls those functions, so a modified client can't award itself points.
- **Clean Architecture by feature.** Each feature (auth, battle, shop, techniques, profile…) is split into domain, data and presentation. The domain layer has no dependency on Supabase or Android.
- **MVI with unidirectional data flow.** Every screen has a contract with its State, Intents and Effects, and a ViewModel that reduces intents into a new state.
- **One primary action per screen.** Based on Hick's law: each screen has a single main button and the rest are secondary links, to keep decisions quick during a live battle.
- **Real time without depending on the phones.** Battles update live through Supabase Realtime, and a scheduled server job advances the rounds even if nobody has the app open.
- **Purchases and rewards validated server-side.** Google Play purchases and rewarded ads are verified by Edge Functions before anything is granted.

## Tech stack

**Android:** Kotlin, Jetpack Compose (Material 3), Coroutines and Flow, Koin with annotations and KSP, Navigation Compose with type-safe routes, DataStore, Coil, Media3 (ExoPlayer) with local video cache.

**Backend:** Supabase (Auth, Postgres, Row Level Security, Realtime, Storage, Edge Functions in TypeScript), Firebase Cloud Messaging.

**Monetization and privacy:** Google Play Billing, Google Mobile Ads, User Messaging Platform with age-based ad settings.

**Quality and tooling:** JUnit and coroutines-test, Ktlint, Detekt, Compose Rules, Android Lint, Gradle convention plugins, Version Catalog, GitHub Actions CI, Sentry and PostHog (with consent).

## Status

MVP 1.0 complete, in closed testing on Google Play.

## Contact

Álvaro Muñoz Vaquero · [LinkedIn](https://www.linkedin.com/in/alvaroo-munoz-vaquero/) · [GitHub](https://github.com/AlvaroOlivix)
