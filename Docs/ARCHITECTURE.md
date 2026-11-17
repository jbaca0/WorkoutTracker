# WorkoutTracker Architecture Documentation

This document provides a comprehensive overview of the WorkoutTracker application, a full-stack workout tracking solution with a React frontend and .NET backend.

## Table of Contents

- [Overview](#overview)
- [Technology Stack](#technology-stack)
- [Project Structure](#project-structure)
- [Frontend Architecture](#frontend-architecture)
- [Backend Architecture](#backend-architecture)
- [Data Flow](#data-flow)
- [Database Schema](#database-schema)
- [Authentication](#authentication)

---

## Overview

WorkoutTracker is a workout logging application that allows users to:
- Create and manage workouts
- Add exercises from a library to workouts
- Track sets, reps, and weights
- View previous performance for exercises
- Access the app via web browser or iOS (via Capacitor)

There is also an **AdminClient** web application for administrative tasks:
- Managing exercise info entries (add, edit, browse)
- Managing users
- Viewing system logs

---

## Technology Stack

| Layer | Technology | Version |
|-------|------------|---------|
| Frontend Framework (Client) | React | 18.3.1 |
| Frontend Language (Client) | TypeScript | 5.6.2 |
| Frontend Build (Client) | Vite | 6.0.5 |
| Frontend Routing (Client) | React Router | 7.1.1 |
| Frontend UI (Client) | Material-UI (MUI) | 6.3.0 |
| Mobile Wrapper (Client) | Capacitor | 7.2.0 |
| Frontend Framework (AdminClient) | React | 19.2.0 |
| Frontend Language (AdminClient) | TypeScript | 5.9.3 |
| Frontend Build (AdminClient) | Vite | 7.3.1 |
| Frontend UI (AdminClient) | Material-UI (MUI) | 7.3.8 |
| Backend Framework | ASP.NET Core | .NET 9.0 |
| Backend Language | C# | Latest |
| ORM | Entity Framework Core | 9.0.0 |
| Database | SQLite | File-based |
| Authentication | ASP.NET Identity | Session Cookies |

---

## Project Structure

```
WorkoutTracker/
├── WorkoutTracker.Client/          # React frontend (user-facing)
│   ├── src/
│   │   ├── Pages/                  # Route page components
│   │   ├── Components/             # Reusable UI components
│   │   ├── Services/               # API service layer
│   │   ├── Hooks/                  # Custom React hooks
│   │   ├── Domain/                 # Domain models
│   │   ├── DataTransferObjects/    # API DTOs
│   │   ├── Enums/                  # TypeScript enums (MuscleGroup)
│   │   ├── Utilities/              # Helper functions
│   │   ├── App.tsx                 # Root component
│   │   └── main.tsx                # Entry point
│   ├── ios/                        # Capacitor iOS project
│   ├── package.json
│   └── vite.config.ts
│
├── WorkoutTracker.AdminClient/     # React frontend (admin web app)
│   ├── src/
│   │   ├── App.tsx                 # Root component
│   │   └── main.tsx                # Entry point
│   ├── package.json
│   └── vite.config.ts
│
└── WorkoutTracker.Server/          # .NET backend (shared by both clients)
    ├── Models/                     # EF Core entities
    │   └── Repositories/           # Data access layer
    ├── DomainObjects/              # Business logic models
    ├── DataTransferObjects/        # API contracts
    │   ├── Common/
    │   ├── Requests/
    │   └── Responses/
    ├── Enums/                      # C# enums (MuscleGroup)
    ├── Routes/                     # API endpoint definitions
    ├── Handlers/                   # Request handlers
    ├── Services/                   # Business services
    ├── Authorization/              # Auth policies & handlers
    ├── Middleware/                 # Custom middleware
    ├── Extensions/                 # Extension methods
    ├── Migrations/                 # EF Core migrations
    ├── Program.cs                  # App startup
    └── WorkoutTracker.db           # SQLite database
```

---

## Frontend Architecture

### Entry Points

| File | Purpose |
|------|---------|
| `src/main.tsx` | Bootstrap React app with BrowserRouter |
| `src/App.tsx` | Root component, routing, theme setup |
| `index.html` | HTML shell mounting to `#root` |

### Pages

| Page | Path | Description |
|------|------|-------------|
| `LandingPage` | `/` | Welcome/entry page for unauthenticated users |
| `LoginPage` | `/login` | User authentication form |
| `HomePage` | `/home` | Dashboard showing user's workouts |
| `WorkoutPage` | `/workout/:id` | Active workout editor |
| `ExerciseInfoListPage` | `/exercises` | Browse exercise library |
| `ExerciseInfoPage` | `/exercise/:id` | Individual exercise details |
| `WorkoutHistoryPage` | `/exercise-history` | List of completed workouts |
| `WorkoutHistoryDetailPage` | `/workout-history/:id` | Read-only view of completed workout |

### Components

| Component | Location | Purpose |
|-----------|----------|---------|
| `AppLayoutComponent` | `Components/AppLayout/` | Main app shell with navigation |
| `ProtectedRoute` | `Components/ProtectedRoute/` | Route guard for authentication |
| `AddExerciseComponent` | `Components/AddExercise/` | Dialog to add exercises to workout |
| `AddExerciseInfoComponent` | `Components/AddExerciseInfo/` | Dialog to create new exercise types |
| `ExerciseComponent` | `Components/Exercise/` | Exercise display with sets editor |
| `ReadOnlyExerciseComponent` | `Components/ReadOnlyExercise/` | Read-only exercise display for history |

### Services

Services handle API communication with the backend.

| Service | Location | Responsibility |
|---------|----------|----------------|
| `BaseService` | `Services/BaseService.ts` | HTTP wrapper with cookie credentials |
| `UserService` | `Services/UserService.ts` | Login/logout operations |
| `WorkoutService` | `Services/WorkoutService.ts` | Workout CRUD operations |
| `ExerciseInfoService` | `Services/ExerciseInfoService.ts` | Exercise library operations |

### Hooks

| Hook | Location | Purpose |
|------|----------|---------|
| `useAuth` | `Hooks/useAuth.tsx` | Authentication context (login state, user info) |
| `useActiveWorkout` | `Hooks/useActiveWorkout.tsx` | Active workout state management |
| `useLocalStorage` | `Hooks/useLocalStorage.tsx` | Persistent localStorage helper |

### Domain Models

| Model | Location | Description |
|-------|----------|-------------|
| `Workout` | `Domain/Workout.ts` | Workout with exercises, date |
| `Exercise` | `Domain/Exercise.ts` | Exercise with sets and previous performance |
| `Set` | `Domain/Set.ts` | Individual set (reps, weight, completed) |
| `ExerciseInfo` | `Domain/ExerciseInfo.ts` | Exercise metadata (name, muscle groups) |
| `UserExerciseInfo` | `Domain/UserExerciseInfo.ts` | User's exercise with history |
| `WorkoutSummary` | `Domain/WorkoutSummary.ts` | Workout summary for history list |
| `LoginResult` | `Domain/LoginResult.ts` | Login response wrapper |

### Configuration

| File | Purpose |
|------|---------|
| `vite.config.ts` | Vite build configuration |
| `tsconfig.json` | TypeScript configuration |
| `.env` | Environment variables (`VITE_SERVER_URL`) |
| `capacitor.config.ts` | iOS app configuration |

---

## Backend Architecture

### Layered Architecture

```
Routes (API endpoints)
    ↓
Handlers (Business logic)
    ↓
Services (Shared business logic)
    ↓
Repositories (Data access)
    ↓
Models (EF Core entities)
```

### Entry Point

**`Program.cs`** - Configures:
- Entity Framework with SQLite
- ASP.NET Identity authentication
- CORS for frontend origins
- Session middleware
- Authorization policies
- API route mapping

### Models (EF Core Entities)

| Entity | Location | Fields |
|--------|----------|--------|
| `Workout` | `Models/Workout.cs` | Id, UserId, Exercises |
| `Exercise` | `Models/Exercise.cs` | Id, WorkoutId, ExerciseInfoId, Sets |
| `Set` | `Models/Set.cs` | Id, ExerciseId, Reps, Weight |
| `ExerciseInfo` | `Models/ExerciseInfo.cs` | Id, Name, PrimaryMuscleGroup, SecondaryMuscleGroupsJson |

**`ApplicationDbContext`** (`Models/ApplicationDbContext.cs`) - Database context with entity configurations.

### Repositories

| Repository | Location | Operations |
|------------|----------|------------|
| `WorkoutRepository` | `Models/Repositories/` | GetById, GetByUserId, Add, Save |
| `ExerciseRepository` | `Models/Repositories/` | GetById, Add |
| `ExerciseInfoRepository` | `Models/Repositories/` | GetAll, Add |

### Domain Objects

Business logic layer separate from EF entities:

| Domain Object | Location | Purpose |
|---------------|----------|---------|
| `Workout` | `DomainObjects/Workout.cs` | Business workout with conversion methods |
| `Exercise` | `DomainObjects/Exercise.cs` | Business exercise with FromModel/ToModel |
| `Set` | `DomainObjects/Set.cs` | Business set |
| `ExerciseInfo` | `DomainObjects/ExerciseInfo.cs` | Business exercise info |
| `UserExerciseInfo` | `DomainObjects/UserExerciseInfo.cs` | Exercise with latest attempt data |

### Handlers

| Handler | Location | Responsibility |
|---------|----------|----------------|
| `WorkoutHandler` | `Handlers/WorkoutHandler.cs` | Workout operations (add, get, save) |
| `ExerciseInfoHandler` | `Handlers/ExerciseInfoHandler.cs` | Exercise library operations |

### Services

| Service | Location | Purpose |
|---------|----------|---------|
| `UserExerciseInfoService` | `Services/UserExerciseInfoService.cs` | Get user's exercise history with latest performance |

### Authorization

| Component | Location | Purpose |
|-----------|----------|---------|
| `ResourceAuthorizationService` | `Authorization/` | Policy registration |
| `WorkoutOwnerHandler` | `Authorization/Handlers/` | Validates user owns workout |
| `WorkoutOwnerRequirement` | `Authorization/Requirements/` | Authorization requirement |
| `WorkoutOwnerFilter` | `Authorization/EndpointFilters/` | Endpoint-level auth filter |

### Middleware

| Middleware | Location | Purpose |
|------------|----------|---------|
| `SessionUserIdMiddleware` | `Middleware/` | Stores authenticated user ID in session |

---

## Data Flow

### Creating a Workout

```
1. User clicks "New Workout" in HomePage
2. React calls WorkoutService.addWorkout()
3. POST /addWorkout → WorkoutHandler.AddWorkout()
4. WorkoutRepository creates workout with UserId
5. Returns Workout DTO
6. Frontend navigates to WorkoutPage
```

### Adding Exercises to Workout

```
1. User selects exercises in AddExerciseComponent
2. React calls WorkoutService.addExercises()
3. POST /addExercises → WorkoutHandler.AddExercises()
4. WorkoutRepository adds exercises to workout
5. UserExerciseInfoService gets previous performance
6. Returns AddExercisesResponse with exercise history
7. Frontend displays exercises with previous sets
```

### Saving Workout

```
1. User modifies sets (reps, weight, completed)
2. React calls WorkoutService.save()
3. POST /save → WorkoutHandler.Save()
4. WorkoutRepository updates workout exercises/sets
5. Returns success
6. Frontend clears active workout and navigates to workout history detail
```

### Viewing Workout History

```
1. User clicks History tab in navigation
2. React calls WorkoutService.getWorkoutHistory()
3. GET /getWorkoutHistory → WorkoutHandler.GetWorkoutHistory()
4. WorkoutRepository.GetWorkoutsWithSummaryAsync() loads workouts with exercises
5. Returns list of WorkoutSummary (id, date, exercise count, first 3 exercise names)
6. User clicks a workout to view details
7. React navigates to /workout-history/:id
8. WorkoutService.getWorkoutDetail() fetches full workout
9. GET /getWorkoutDetail/:id → WorkoutHandler.GetWorkoutDetail()
10. WorkoutRepository.GetWorkoutWithDetailsAsync() loads workout with exercises and sets
11. Frontend displays read-only workout with ReadOnlyExerciseComponent
```

---

## Database Schema

### Entity Relationships

```
IdentityUser (ASP.NET Identity)
    │
    └── Workout (1:many)
            │
            └── Exercise (1:many)
                    │
                    ├── ExerciseInfo (many:1)
                    │
                    └── Set (1:many)
```

### Tables

**Workouts**
| Column | Type | Description |
|--------|------|-------------|
| Id | int | Primary key |
| UserId | string | Foreign key to AspNetUsers |

**Exercises**
| Column | Type | Description |
|--------|------|-------------|
| Id | int | Primary key |
| WorkoutId | int | Foreign key to Workouts |
| ExerciseInfoId | int | Foreign key to ExerciseInfos |

**Sets**
| Column | Type | Description |
|--------|------|-------------|
| Id | int | Primary key |
| ExerciseId | int | Foreign key to Exercises |
| Reps | int | Number of repetitions |
| Weight | decimal | Weight used |

**ExerciseInfos**
| Column | Type | Description |
|--------|------|-------------|
| Id | int | Primary key |
| Name | string | Exercise name |
| PrimaryMuscleGroup | int | Primary muscle group (MuscleGroup enum, defaults to Other=9) |
| SecondaryMuscleGroupsJson | string | JSON array of secondary muscle group integers (defaults to "[]") |

---

## Authentication

### Flow

1. User submits credentials to `/login` (ASP.NET Identity endpoint)
2. Server validates and creates session cookie
3. Frontend stores login state in `useAuth` context
4. All subsequent requests include cookies (`credentials: 'include'`)
5. `SessionUserIdMiddleware` extracts UserId for handlers
6. Authorization policies verify resource ownership

### CORS Configuration

Allowed origins:
- `http://localhost:5173` (Client Vite dev server)
- `capacitor://localhost` (iOS app)
- AdminClient origin will need to be added when its dev server port is configured

### Protected Routes

Frontend uses `ProtectedRoute` component to guard authenticated pages:
- Redirects to `/login` if not authenticated
- Checks `useAuth().isLoggedIn` state

---

## Running the Application

### Backend

```bash
cd WorkoutTracker.Server
dotnet run
# Runs on http://localhost:5231
```

### Frontend (Client)

```bash
cd WorkoutTracker.Client
npm install
npm run dev
# Runs on http://localhost:5173
```

### Frontend (AdminClient)

```bash
cd WorkoutTracker.AdminClient
npm install
npm run dev
```

### iOS (Capacitor)

```bash
cd WorkoutTracker.Client
npm run build
npx cap sync ios
npx cap open ios
```
