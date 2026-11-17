# WorkoutTracker

A full-stack workout tracking platform: a React Native iOS app I use at the gym, a .NET 9 REST
API behind it, and an internal admin console for the data it depends on. Built and maintained
solo, deployed to Azure from GitHub Actions.

This repository is the front door — it holds the architecture documentation and points at the
code. Each application lives in its own repository.

<!-- Add screenshots here: 2-3 images of the mobile app (workout logging, history calendar,
     plans) and one of the admin console. A short screen recording of a logged workout reads
     better than any paragraph below. -->

## Repositories

| Repository | Stack | What it is |
| --- | --- | --- |
| [WorkoutTracker.Server](https://github.com/jbaca0/WorkoutTracker.Server) | ASP.NET Core 9, EF Core, SQLite, Identity | Minimal-API backend, auth, data model, logging |
| [WorkoutTracker.MobileApp](https://github.com/jbaca0/WorkoutTracker.MobileApp) | React Native 0.81, Expo 54, Expo Router, TypeScript | The iOS app — primary client |
| [WorkoutTracker.AdminClient](https://github.com/jbaca0/WorkoutTracker.AdminClient) | React 19, MUI 7, Vite, TypeScript | Admin console for users, exercise catalog and logs |

An earlier React + Capacitor web client (`WorkoutTracker.Client`) was superseded by the React
Native app and is retired.

## How it fits together

```
 MobileApp (iOS)          AdminClient (web)
 React Native / Expo      React 19 / MUI 7
        |                        |
        |  JWT bearer + refresh  |
        +-----------+------------+
                    |
          WorkoutTracker.Server
          ASP.NET Core 9 minimal API
   Routes -> Handlers -> Repositories -> EF Core
                    |
                 SQLite
                    |
        Azure App Service  <--  GitHub Actions (OIDC)
```

Both clients mirror the server's DTOs and share the same layering — services over domain models
over wire contracts — so the same shapes appear on both sides of the network boundary.

## Documentation

| Document | Contents |
| --- | --- |
| [ARCHITECTURE.md](Docs/ARCHITECTURE.md) | System structure, project layout, data flow |
| [API_REFERENCE.md](Docs/API_REFERENCE.md) | Endpoints, request and response shapes |
| [FRONTEND_REFERENCE.md](Docs/FRONTEND_REFERENCE.md) | Client components, services, hooks and patterns |
| [FEEDBACK_PIPELINE.md](Docs/FEEDBACK_PIPELINE.md) | Design spec for in-app feedback and admin-driven issue triage |

## Engineering notes

**Auth.** ASP.NET Core Identity issues a 15-minute access token and a 7-day refresh token.
Refresh rotates both and revokes the old one; a background service prunes expired tokens. On the
client, a `401` triggers one refresh and a retry, with concurrent requests sharing a single
in-flight refresh so an expiry can't cause a stampede.

**Authorization.** Ownership is enforced per resource, not per route: custom authorization
requirements, handlers and endpoint filters check that the workout or plan being touched belongs
to the caller. Admin surfaces sit behind a separate role policy.

**Data.** EF Core models for workouts, exercises, sets, templates, plans and goals, evolved
across 37 migrations. IDs are GUIDs throughout.

**Testing.** 285 xUnit tests over handlers, repositories, `JwtService` and the authorization
rules, against an in-memory database; 236 Jest / React Native Testing Library tests over the
app's services, domain models, hooks and screens.

**Operations.** Every request is logged to the database through middleware and a custom
`ILoggerProvider`, and the admin console reads those logs back. Pushing to the backend's default
branch builds and deploys to Azure App Service, authenticating with OIDC federated credentials
rather than a stored publish profile; the admin console deploys to Azure Static Web Apps with
per-PR preview environments.

## Running it locally

Start the backend first — both clients need it:

```bash
cd WorkoutTracker.Server && dotnet run          # http://localhost:5231
cd WorkoutTracker.MobileApp && npx expo start --ios
cd WorkoutTracker.AdminClient && npm run dev    # https://localhost:5174
```

The server needs a `Jwt:Secret` of at least 32 bytes or it refuses to start; each repository's
README covers its own configuration.
