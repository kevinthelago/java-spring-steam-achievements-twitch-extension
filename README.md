# Steam Achievements Twitch Extension — Backend

Spring Boot backend for a Twitch panel extension that displays a viewer's Steam achievement progress in real time.

## Stack

- **Java 17** / Spring Boot 3.0.4
- **Spring Cloud OpenFeign** — Steam Web API client
- **Spring Data JPA + Hibernate** — persistence layer
- **MySQL 8** — primary database
- **React** (git submodule at `ui/`) — Twitch panel frontend

## Prerequisites

- JDK 17+
- MySQL 8 running on `localhost:3306` (user: `root`, password: `root`)
- Node 19 + Yarn 1.x (only needed to build the frontend)

## Getting Started

```bash
# Clone with submodules
git clone --recurse-submodules <repo-url>

# Run the backend (dev profile, targets steam_achievements_test DB)
./mvnw spring-boot:run
```

The server starts on `http://localhost:8080`. The database and schema are created automatically on first run.

## API

### Simplified (non-persisted)

Live proxies to Steam Web API — no database writes.

| Method | Endpoint | Params | Description |
|---|---|---|---|
| GET | `/api/simp/player` | `steamId` | Player profile + owned games |
| GET | `/api/simp/achievements` | `steamId`, `appId` | Achievement list with player progress |

### Persisted

Fetches from Steam and stores in MySQL for richer querying.

| Method | Endpoint | Params | Description |
|---|---|---|---|
| GET | `/api/player` | `steamId` | Fetch stored player |
| GET | `/api/player/games` | `steamId` | Player with their game list |
| GET | `/api/player/achievements` | `steamId`, `appId` | Player achievements for a game |
| PUT | `/api/player` | `steamId` | Seed player + up to 4 games + achievements from Steam |

## Configuration

`src/main/resources/application.yml` holds all config. The active profile defaults to `dev`.

| Property | Dev value |
|---|---|
| DB URL | `jdbc:mysql://localhost:3306/steam_achievements_test` |
| Steam API key | `env.steam.api-key` |

## Building

```bash
# Backend + frontend (full build)
./mvnw clean package

# Backend only (skip tests + frontend)
./mvnw clean package -DskipTests
```

The frontend build output is copied to `src/main/resources/static` and served by Spring Boot.

## Frontend

The React Twitch extension UI lives at [`ui/`](https://github.com/Kevin-Lago/javascript-react-steam-achievements-twitch-extension-ui) as a git submodule.
