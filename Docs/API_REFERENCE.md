# WorkoutTracker API Reference

Base URL: `http://localhost:5231`

All endpoints (except authentication) require session cookie authentication.

---

## Table of Contents

- [Authentication](#authentication)
- [Workouts](#workouts)
- [Exercise Info](#exercise-info)
- [Data Types](#data-types)

---

## Authentication

The API uses ASP.NET Core Identity with session cookies. All requests must include credentials.

### Login

```http
POST /login
Content-Type: application/json
```

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "password123"
}
```

**Response:** `200 OK` with session cookie set

**Errors:**
- `401 Unauthorized` - Invalid credentials

### Logout

```http
POST /logout
```

**Response:** `200 OK` - Session cleared

### Register

```http
POST /register
Content-Type: application/json
```

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "password123"
}
```

**Response:** `200 OK` - User created

---

## Workouts

### Get User Workouts

Retrieves all workout IDs for the authenticated user.

```http
GET /getWorkouts
```

**Authorization:** Required (session cookie)

**Response:** `200 OK`
```json
[
  "550e8400-e29b-41d4-a716-446655440000",
  "6ba7b810-9dad-11d1-80b4-00c04fd430c8"
]
```

**Source:** `WorkoutTracker.Server/Handlers/WorkoutHandler.cs:30`

---

### Create Workout

Creates a new empty workout for the authenticated user.

```http
POST /addWorkout
```

**Authorization:** Required (session cookie)

**Response:** `200 OK`
```json
"550e8400-e29b-41d4-a716-446655440000"
```

Returns the new workout's GUID.

**Source:** `WorkoutTracker.Server/Handlers/WorkoutHandler.cs:19`

---

### Get Workout Exercises

Retrieves exercise names for a specific workout.

```http
GET /getExercises/{workoutId}
```

**Authorization:** Required (session cookie) + Workout Owner

**Path Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `workoutId` | GUID | The workout ID |

**Response:** `200 OK`
```json
["Bench Press", "Squat", "Deadlift"]
```

**Source:** `WorkoutTracker.Server/Handlers/WorkoutHandler.cs:40`

---

### Add Single Exercise

Adds a single exercise to a workout.

```http
POST /addExercise
Content-Type: application/json
```

**Authorization:** Required (session cookie) + Workout Owner

**Request Body:**
```json
{
  "workoutId": "550e8400-e29b-41d4-a716-446655440000",
  "exerciseInfoId": "6ba7b810-9dad-11d1-80b4-00c04fd430c8"
}
```

**Response:** `200 OK`
```json
{
  "exerciseId": "7c9e6679-7425-40de-944b-e07fc1f90ae7"
}
```

**Errors:**
- `404 Not Found` - Invalid workoutId or exerciseInfoId

**Source:** `WorkoutTracker.Server/Handlers/WorkoutHandler.cs:51`

---

### Add Multiple Exercises

Adds multiple exercises to a workout and returns user's previous performance for each.

```http
POST /addExercises
Content-Type: application/json
```

**Authorization:** Required (session cookie) + Workout Owner

**Request Body:**
```json
{
  "workoutId": "550e8400-e29b-41d4-a716-446655440000",
  "exerciseInfoIds": [
    "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
    "7c9e6679-7425-40de-944b-e07fc1f90ae7"
  ]
}
```

**Response:** `200 OK`
```json
{
  "userExerciseInfos": [
    {
      "exerciseInfo": {
        "id": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
        "name": "Bench Press"
      },
      "latestExercise": {
        "exerciseInfoId": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
        "sets": [
          { "reps": 10, "weight": 135.0 },
          { "reps": 8, "weight": 145.0 }
        ]
      }
    },
    {
      "exerciseInfo": {
        "id": "7c9e6679-7425-40de-944b-e07fc1f90ae7",
        "name": "Squat"
      },
      "latestExercise": null
    }
  ]
}
```

The `latestExercise` field contains the user's most recent performance for that exercise, or `null` if never performed.

**Errors:**
- `400 Bad Request` - Missing exerciseInfoIds or invalid data

**Source:** `WorkoutTracker.Server/Handlers/WorkoutHandler.cs:71`

---

### Save Workout

Saves the complete workout state including all exercises and sets.

```http
POST /save
Content-Type: application/json
```

**Authorization:** Required (session cookie) + Workout Owner

**Request Body:**
```json
{
  "workoutId": "550e8400-e29b-41d4-a716-446655440000",
  "workout": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "addDateTime": "2024-01-15T10:30:00Z",
    "exercises": [
      {
        "exerciseInfoId": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
        "sets": [
          { "reps": 10, "weight": 135.0 },
          { "reps": 8, "weight": 145.0 },
          { "reps": 6, "weight": 155.0 }
        ]
      }
    ]
  }
}
```

**Response:** `200 OK`

**Errors:**
- `400 Bad Request` - Invalid workout data

**Source:** `WorkoutTracker.Server/Handlers/WorkoutHandler.cs:117`

---

### Get Workout History

Retrieves a summary list of all completed workouts for the authenticated user.

```http
GET /getWorkoutHistory
```

**Authorization:** Required (session cookie)

**Response:** `200 OK`
```json
{
  "workouts": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "addDateTime": "2024-01-15T10:30:00Z",
      "exerciseCount": 3,
      "exerciseNames": ["Bench Press", "Squat", "Deadlift"]
    },
    {
      "id": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
      "addDateTime": "2024-01-14T09:00:00Z",
      "exerciseCount": 2,
      "exerciseNames": ["Pull-ups", "Rows"]
    }
  ]
}
```

Returns workouts ordered by date (newest first). `exerciseNames` contains the first 3 exercise names as a preview.

**Source:** `WorkoutTracker.Server/Handlers/WorkoutHandler.cs`

---

### Get Workout Detail

Retrieves the full details of a completed workout including all exercises and sets.

```http
GET /getWorkoutDetail/{workoutId}
```

**Authorization:** Required (session cookie) + Workout Owner

**Path Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `workoutId` | GUID | The workout ID |

**Response:** `200 OK`
```json
{
  "workout": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "addDateTime": "2024-01-15T10:30:00Z",
    "exercises": [
      {
        "exerciseInfoId": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
        "exerciseInfo": {
          "id": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
          "name": "Bench Press"
        },
        "sets": [
          { "id": "...", "reps": 10, "weight": 135.0 },
          { "id": "...", "reps": 8, "weight": 145.0 }
        ]
      }
    ]
  }
}
```

**Errors:**
- `404 Not Found` - Workout not found

**Source:** `WorkoutTracker.Server/Handlers/WorkoutHandler.cs`

---

## Exercise Info

### Get All Exercise Types

Retrieves all available exercise types in the library.

```http
GET /getExerciseInfos
```

**Authorization:** Not required

**Response:** `200 OK`
```json
{
  "exerciseInfos": [
    {
      "id": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
      "name": "Bench Press",
      "primaryMuscleGroup": "Chest",
      "secondaryMuscleGroups": ["Triceps", "Shoulders"]
    },
    {
      "id": "7c9e6679-7425-40de-944b-e07fc1f90ae7",
      "name": "Squat",
      "primaryMuscleGroup": "Legs",
      "secondaryMuscleGroups": ["Core"]
    },
    {
      "id": "a87ff679-a2f3-71d1-8f4c-2b9b6d8e9f0a",
      "name": "Deadlift",
      "primaryMuscleGroup": "Back",
      "secondaryMuscleGroups": ["Legs", "Core"]
    }
  ]
}
```

**Source:** `WorkoutTracker.Server/Routes/ExerciseInfoRoutes.cs:12`

---

### Add Exercise Type

Creates a new exercise type in the library.

```http
POST /addExerciseInfo
Content-Type: application/json
```

**Authorization:** Not required

**Request Body:**
```json
{
  "name": "Romanian Deadlift",
  "primaryMuscleGroup": "Legs",
  "secondaryMuscleGroups": ["Back", "Core"]
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Exercise name |
| `primaryMuscleGroup` | string | No | Primary muscle group (defaults to "Other") |
| `secondaryMuscleGroups` | string[] | No | Secondary muscle groups (defaults to empty) |

**Response:** `200 OK`

**Source:** `WorkoutTracker.Server/Routes/ExerciseInfoRoutes.cs:11`

---

## Data Types

### Workout

```typescript
interface Workout {
  id: string;           // GUID
  exercises: Exercise[];
  addDateTime: string;  // ISO 8601 datetime
}
```

**Source:** `WorkoutTracker.Server/DataTransferObjects/Common/Workout.cs`

---

### Exercise

```typescript
interface Exercise {
  exerciseInfoId: string;        // GUID referencing ExerciseInfo
  exerciseInfo?: ExerciseInfo;   // Exercise metadata (optional, included in some responses)
  sets: Set[];
}
```

**Source:** `WorkoutTracker.Server/DataTransferObjects/Common/Exercise.cs`

---

### Set

```typescript
interface Set {
  id?: string;     // GUID (optional, present when reading)
  reps: number;    // Integer
  weight: number;  // Decimal
}
```

**Source:** `WorkoutTracker.Server/DataTransferObjects/Common/Set.cs`

---

### ExerciseInfo

```typescript
interface ExerciseInfo {
  id: string;                      // GUID
  name: string;                    // Exercise name
  primaryMuscleGroup: string;      // Primary muscle group (MuscleGroup enum value)
  secondaryMuscleGroups: string[]; // Secondary muscle groups
}
```

**Source:** `WorkoutTracker.Server/DataTransferObjects/Common/ExerciseInfo.cs`

---

### MuscleGroup

Valid muscle group values:

| Value | Description |
|-------|-------------|
| `Chest` | Chest muscles |
| `Back` | Back muscles |
| `Shoulders` | Shoulder muscles |
| `Biceps` | Bicep muscles |
| `Triceps` | Tricep muscles |
| `Forearms` | Forearm muscles |
| `Core` | Core/abdominal muscles |
| `Legs` | Leg muscles |
| `FullBody` | Full body exercises |
| `Other` | Other/uncategorized |

**Source:** `WorkoutTracker.Server/Enums/MuscleGroup.cs`

---

### UserExerciseInfo

Combines exercise metadata with user's historical performance.

```typescript
interface UserExerciseInfo {
  exerciseInfo: ExerciseInfo;
  latestExercise: Exercise | null;  // Most recent performance, null if never done
}
```

**Source:** `WorkoutTracker.Server/DataTransferObjects/Common/UserExerciseInfo.cs`

---

### WorkoutSummary

Summary of a workout for history list display.

```typescript
interface WorkoutSummary {
  id: string;              // GUID
  addDateTime: string;     // ISO 8601 datetime
  exerciseCount: number;   // Total number of exercises
  exerciseNames: string[]; // First 3 exercise names as preview
}
```

**Source:** `WorkoutTracker.Server/DataTransferObjects/Common/WorkoutSummary.cs`

---

## Authorization

### Workout Owner Policy

Endpoints marked with "Workout Owner" authorization verify that the authenticated user owns the workout being accessed. This is enforced via `WorkoutOwnerFilter`.

**Source:** `WorkoutTracker.Server/Authorization/EndpointFilters/WorkoutOwnerFilter.cs`

---

## Error Responses

| Status Code | Description |
|-------------|-------------|
| `200 OK` | Request successful |
| `400 Bad Request` | Invalid request data |
| `401 Unauthorized` | Not authenticated |
| `403 Forbidden` | Not authorized to access resource |
| `404 Not Found` | Resource not found |

---

## CORS

The API accepts requests from:
- `http://localhost:5173` (Vite development server)
- `capacitor://localhost` (iOS app)

Credentials (cookies) are included in CORS responses.
