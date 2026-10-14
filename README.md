# Pitaco API

Pitaco is a football prediction pool app: players join pools, predict match scores and climb a ranking as results come in. This repository holds the HTTP API, written in Rust with axum and MySQL.

| Part | What it is |
| --- | --- |
| `pitaco-api` (this repo) | REST API under `/api/v1`: accounts, pools, predictions, scoring, rankings, notifications and the admin side of tournaments, teams and matches. |
| [`pitaco-web`](https://github.com/Viniciusadm/pitaco-web) | The web app (React + TanStack Router, installable as a PWA) that talks to this API. |

The code started from [rust-backend-template](https://github.com/Viniciusadm/rust-backend-template) and keeps its layout: `domain` holds pure types and rules, `application` holds use cases generic over traits (ports), `infrastructure` implements those ports (MySQL repositories, Argon2, JWT, file storage, FCM) and `api` is the axum layer.

## What it does

- **Accounts**: register, log in with a JWT, profile, avatar upload and notification preferences.
- **Pools**: create a pool with its own scoring rules, join it with an invite code (optionally limited to an email allowlist), list members, leave, and read the ranking and the history of results.
- **Predictions**: one score per match, locked a pool-defined number of minutes before kickoff. Scoring rules give points for the exact score, the right outcome, the right goal difference and, in knockout games, the penalty shoot-out winner (`src/domain/predictions/scoring_policy.rs`).
- **Admin**: tournaments with stages and groups, teams with flags, matches and results. Only users with the admin role can change them, and every change is written to an audit log.
- **Background jobs**: an in-process tokio scheduler moves matches to live, sends reminders before predictions lock and delivers queued notifications as push messages through Firebase Cloud Messaging.

## Requirements

- Rust stable (edition 2021)
- MySQL 8

Main stack: axum 0.7, sqlx 0.8 (MySQL, rustls), jsonwebtoken 9, argon2 0.5, tower-http 0.6, reqwest 0.12.

## Running locally

Copy the environment file and set `DATABASE_URL` and `JWT_SECRET`:

```bash
cp .env.example .env
```

Start the API. Migrations in `migrations/` run on boot unless `RUN_MIGRATIONS=false`:

```bash
cargo run
```

The server listens on `HTTP_HOST:HTTP_PORT` (`127.0.0.1:3001` by default) and answers `GET /health`. Uploaded files are stored under `FILES_LOCAL_DIR`.

Push notifications stay off until `FCM_PROJECT_ID` and `FCM_SERVICE_ACCOUNT_PATH` point to a Firebase project and its service account JSON. Keep that file out of git. Every variable is described in [`.env.example`](.env.example).

## Tests

```bash
cargo test
```

The tests in [`tests`](tests) run the use cases, the scoring policy, the background jobs and the HTTP routes for files, pools and predictions against in-memory fakes of the ports, so they do not need a database.
