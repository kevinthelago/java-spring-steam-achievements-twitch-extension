# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Build (skips frontend submodule build)
./mvnw clean package -DskipTests

# Run (dev profile by default — targets steam_achievements_test DB)
./mvnw spring-boot:run

# Run tests
./mvnw test

# Run a single test class
./mvnw test -Dtest=SteamAchievementsApplicationTests

# Initialize the frontend submodule
git submodule update --init
```

The frontend React app lives in the `ui/` git submodule. The Maven build (`prepare-package` phase) compiles it via `frontend-maven-plugin` and copies the output to `src/main/resources/static`. For backend-only work you can skip that phase.

## Architecture

This is a Spring Boot backend that proxies and optionally persists Steam Web API data, intended to serve a Twitch panel extension.

### Two parallel API surfaces

| Path prefix | Persistence | Service |
|---|---|---|
| `/api/player/*` | Yes — MySQL via JPA | `PlayerService` |
| `/api/simp/*` | No — live Steam calls only | `SimplifiedService` |

The simplified endpoints (`/api/simp/player`, `/api/simp/achievements`) are the primary path for the Twitch extension UI. The persisted endpoints support richer queries but are partially implemented.

### Steam API client

`SteamFeignClient` (Spring Cloud OpenFeign) calls three Steam Web API endpoints:

- `IPlayerService/GetOwnedGames` — player's game library
- `ISteamUserStats/GetPlayerAchievements` — per-game achievement status
- `ISteamUserStats/GetSchemaForGame` — achievement metadata (names, icons)
- `ISteamUser/GetPlayerSummaries` — player profile details

`SteamService` wraps these Feign calls. `SimplifiedService` delegates to `SteamService` and builds domain models without touching the DB.

### Data layer

- **JPA (primary)**: `GameJpa`, `GameAchievementJpa`, `PlayerRepository`, `PlayerAchievementJpa` extend `JpaRepository`.
- **JDBC (parallel, less used)**: `GameJdbc`, `GameAchievementJdbc`, `PlayerGamesJdbc` implement DAO interfaces using `JdbcTemplate`. SQL constants live in `constants/*Queries.java`.
- Schema DDL is in `src/main/resources/schema.sql`; Hibernate `ddl-auto: update` also manages the schema at startup.

### Config

`Settings.java` reads `env.steam.api-key` from `application.yml` into a static field consumed throughout services. The key and DB credentials are currently hardcoded in `application.yml` — dev profile targets `steam_achievements_test`, prod targets `steam_achievements`.

### Database

MySQL must be running on `localhost:3306` with user `root` / password `root`. The database is created automatically (`createDatabaseIfNotExist=true`).
