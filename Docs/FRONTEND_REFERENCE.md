# WorkoutTracker Frontend Reference

This document provides a detailed reference for the React frontend applications.

---

## Table of Contents

- [Application Setup](#application-setup)
- [Routing](#routing)
- [Context Providers](#context-providers)
- [Custom Hooks](#custom-hooks)
- [Services](#services)
- [Domain Models](#domain-models)
- [Components](#components)
- [Pages](#pages)
- [AdminClient](#adminclient)

---

## Application Setup

### Entry Point

**`src/main.tsx`**
```tsx
ReactDOM.createRoot(document.getElementById('root')!).render(
  <BrowserRouter>
    <App />
  </BrowserRouter>
)
```

### Root Component

**`src/App.tsx`**

The root component configures:
- MUI dark theme via `ThemeProvider`
- `CssBaseline` for consistent styling
- `AuthProvider` for authentication state
- `ActiveWorkoutProvider` for workout state
- React Router routes

---

## Routing

| Path | Component | Auth Required | Description |
|------|-----------|---------------|-------------|
| `/` | `LandingPage` | No | Welcome page |
| `/login` | `LoginPage` | No | Login form |
| `/home` | `HomePage` | Yes | User dashboard |
| `/workout/:workoutId` | `WorkoutPage` | Yes | Workout editor |
| `/exercise-info` | `ExerciseInfoListPage` | Yes | Exercise library |

### Route Protection

Protected routes are wrapped with `ProtectedRoute` component which checks `useAuth().user` and redirects to `/login` if not authenticated.

**Source:** `src/Components/ProtectedRoute/ProtectedRoute.tsx`

---

## Context Providers

### AuthProvider

Manages authentication state and provides login/logout functionality.

**Location:** `src/Hooks/useAuth.tsx`

```tsx
interface AuthContext {
  user: any;                                           // Current user object
  login: (username: string, password: string) => Promise<void>;
  logout: () => void;
  errorMessage: string;                                // Login error message
}
```

**Usage:**
```tsx
import { AuthProvider, useAuth } from './Hooks/useAuth';

// Wrap app
<AuthProvider>
  <App />
</AuthProvider>

// Use in component
const { user, login, logout, errorMessage } = useAuth();
```

**Behavior:**
- Stores user in localStorage via `useLocalStorage`
- On login success, navigates to `/home`
- On logout, clears user and navigates to `/`

---

### ActiveWorkoutProvider

Manages the currently active workout state.

**Location:** `src/Hooks/useActiveWorkout.tsx`

```tsx
interface ActiveWorkoutContext {
  hasActiveWorkout: boolean;    // Whether a workout is in progress
  activeWorkoutId: string;      // Current workout ID
  setActiveWorkout: (workoutId: string) => void;
}
```

**Usage:**
```tsx
import { ActiveWorkooutProvider, useActiveWorkout } from './Hooks/useActiveWorkout';

// Wrap app
<ActiveWorkooutProvider>
  <App />
</ActiveWorkooutProvider>

// Use in component
const { hasActiveWorkout, activeWorkoutId, setActiveWorkout } = useActiveWorkout();
```

**Behavior:**
- Persists active workout ID in localStorage
- Allows resuming workouts across page refreshes

---

## Custom Hooks

### useLocalStorage

Persists state to localStorage with automatic serialization.

**Location:** `src/Hooks/useLocalStorage.tsx`

```tsx
function useLocalStorage<T>(key: string, initialValue: T): [T, (value: T) => void]
```

**Usage:**
```tsx
const [value, setValue] = useLocalStorage('myKey', defaultValue);
```

---

### useAuth

Access authentication context. Must be used within `AuthProvider`.

**Location:** `src/Hooks/useAuth.tsx`

```tsx
const { user, login, logout, errorMessage } = useAuth();
```

**Throws:** Error if used outside `AuthProvider`

---

### useActiveWorkout

Access active workout context. Must be used within `ActiveWorkoutProvider`.

**Location:** `src/Hooks/useActiveWorkout.tsx`

```tsx
const { hasActiveWorkout, activeWorkoutId, setActiveWorkout } = useActiveWorkout();
```

**Throws:** Error if used outside `ActiveWorkoutProvider`

---

## Services

All services extend `BaseService` which handles HTTP communication with cookie credentials.

### BaseService

**Location:** `src/Services/BaseService.ts`

```tsx
class BaseService {
  protected static serverUrl: string;  // From VITE_SERVER_URL env var

  protected static async fetchFromServer<T>(
    endpoint: string,
    body?: any,
    method?: string
  ): Promise<T>;
}
```

**Features:**
- Automatic method detection (GET if no body, POST if body)
- JSON content type
- Cookie credentials included
- Error handling with response text

---

### UserService

Authentication operations.

**Location:** `src/Services/UserService.ts`

```tsx
class UserService extends BaseService {
  static async login(username: string, password: string): Promise<LoginResult>;
  static async logout(): Promise<void>;
}
```

---

### WorkoutService

Workout CRUD operations.

**Location:** `src/Services/WorkoutService.ts`

```tsx
class WorkoutService extends BaseService {
  // Create new workout, returns workout ID
  static async addWorkout(): Promise<string>;

  // Add exercises to workout, returns exercises with previous performance
  static async addExercises(
    workoutId: string,
    exerciseInfoIds: string[]
  ): Promise<Exercise[]>;

  // Save workout state
  static async save(workout: Workout): Promise<void>;
}
```

---

### ExerciseInfoService

Exercise library operations.

**Location:** `src/Services/ExerciseInfoService.ts`

```tsx
class ExerciseInfoService extends BaseService {
  // Get all available exercises
  static async getExerciseInfos(): Promise<ExerciseInfo[]>;

  // Create new exercise type with muscle groups
  static async addExerciseInfo(
    name: string,
    primaryMuscleGroup: MuscleGroup,
    secondaryMuscleGroups: MuscleGroup[]
  ): Promise<void>;
}
```

---

## Domain Models

Domain models contain business logic and conversion between frontend state and API DTOs.

### Workout

**Location:** `src/Domain/Workout.ts`

```tsx
class Workout {
  Id?: string;
  Exercises: Exercise[];

  constructor(data?: Partial<Workout>);
  ToData(): WorkoutData;  // Convert to API DTO
}
```

---

### Exercise

**Location:** `src/Domain/Exercise.ts`

```tsx
class Exercise {
  Id?: string;
  Key: number;                    // Unique key for React lists
  ExerciseInfoId: string;
  ExerciseInfo?: ExerciseInfo;
  PreviousExercise?: Exercise;    // User's last performance
  Sets: Set[];

  constructor(data?: Partial<Exercise>);
  static FromData(data: ExerciseData): Exercise;
  ToData(): ExerciseData;
}
```

---

### Set

**Location:** `src/Domain/Set.ts`

```tsx
class Set {
  Id?: string;
  Key: number;         // Unique key for React lists
  Reps: number;
  Weight: number;
  Completed: boolean;

  constructor(data?: Partial<Set>);
}
```

---

### ExerciseInfo

**Location:** `src/Domain/ExerciseInfo.ts`

```tsx
class ExerciseInfo {
  Id: string;
  Key: number;                              // Unique key for React lists
  Name: string;
  PrimaryMuscleGroup: MuscleGroup;          // Primary muscle group
  SecondaryMuscleGroups: MuscleGroup[];     // Secondary muscle groups

  constructor(data?: Partial<ExerciseInfo>);
  static FromData(data: ExerciseInfoData): ExerciseInfo;
  ToData(): ExerciseInfoData;
}
```

**MuscleGroup Enum** (`src/Enums/MuscleGroup.ts`):
```tsx
enum MuscleGroup {
  Chest, Back, Shoulders, Biceps, Triceps,
  Forearms, Core, Legs, FullBody, Other
}

// Display labels for UI
const MuscleGroupLabels: Record<MuscleGroup, string>;

// All muscle group values
const AllMuscleGroups: MuscleGroup[];
```

---

### UserExerciseInfo

**Location:** `src/Domain/UserExerciseInfo.ts`

```tsx
class UserExerciseInfo {
  ExerciseInfo: ExerciseInfo;
  LatestExercise?: Exercise;  // Most recent performance

  static FromData(data: UserExerciseInfoData): UserExerciseInfo;
}
```

---

### LoginResult

**Location:** `src/Domain/LoginResult.ts`

```tsx
class LoginResult {
  success: boolean;
  errorMessage?: string;
}
```

---

## Components

### AppLayoutComponent

Main application shell with navigation.

**Location:** `src/Components/AppLayout/AppLayoutComponent.tsx`

**Features:**
- App bar with navigation
- Bottom navigation for mobile
- Outlet for nested routes

---

### ProtectedRoute

Route guard that redirects unauthenticated users to login.

**Location:** `src/Components/ProtectedRoute/ProtectedRoute.tsx`

```tsx
const ProtectedRoute: React.FC = () => {
  const auth = useAuth();
  if (!auth || !auth.user) {
    return <Navigate to="/login" />;
  }
  return <Outlet />;
}
```

---

### AddExerciseComponent

Dialog for adding exercises to a workout.

**Location:** `src/Components/AddExercise/AddExerciseComponent.tsx`

**Props:**
```tsx
interface AddExerciseProps {
  open: boolean;
  onClose: () => void;
  onAdd: (exerciseInfoIds: string[]) => void;
}
```

**Features:**
- Displays list of available exercises
- Multi-select support
- Calls `ExerciseInfoService.getExerciseInfos()` on mount

---

### AddExerciseInfoComponent

Dialog for creating new exercise types with muscle group selection.

**Location:** `src/Components/AddExerciseInfo/AddExerciseInfoComponent.tsx`

**Props:**
```tsx
interface AddExerciseInfoProps {
  afterAdd?: () => void;  // Callback after successful add
}
```

**Features:**
- Exercise name text field
- Primary muscle group dropdown (required, defaults to "Other")
- Secondary muscle groups multi-select with chips
- Calls `ExerciseInfoService.addExerciseInfo()` with muscle groups

---

### ExerciseComponent

Individual exercise display with sets editor.

**Location:** `src/Components/Exercise/ExerciseComponent.tsx`

**Props:**
```tsx
interface ExerciseProps {
  exercise: Exercise;
  onUpdate: (exercise: Exercise) => void;
}
```

**Features:**
- Displays exercise name
- Shows previous performance for reference
- Editable sets (reps, weight)
- Add/remove set buttons
- Completion checkboxes

---

## Pages

### LandingPage

Welcome page for unauthenticated users.

**Location:** `src/Pages/LandingPage/LandingPage.tsx`

**Route:** `/`

---

### LoginPage

User authentication form.

**Location:** `src/Pages/LoginPage/LoginPage.tsx`

**Route:** `/login`

**Features:**
- Email/password inputs
- Calls `useAuth().login()`
- Displays error messages

---

### HomePage

User dashboard showing workouts.

**Location:** `src/Pages/HomePage/HomePage.tsx`

**Route:** `/home` (protected)

**Features:**
- Lists user's workouts
- "New Workout" button
- Calls `WorkoutService.addWorkout()` to create workouts
- Uses `useActiveWorkout()` to track active workout

---

### WorkoutPage

Active workout editor.

**Location:** `src/Pages/WorkoutPage/WorkoutPage.tsx`

**Route:** `/workout/:workoutId` (protected)

**Features:**
- Displays exercises with `ExerciseComponent`
- "Add Exercise" button opens `AddExerciseComponent`
- Save functionality via `WorkoutService.save()`
- Loads exercises via `WorkoutService.addExercises()`

---

### ExerciseInfoListPage

Exercise library browser.

**Location:** `src/Pages/ExerciseInfoListPage/ExerciseInfoListPage.tsx`

**Route:** `/exercise-info` (protected)

**Features:**
- Lists all available exercises with primary muscle group chips
- Displays secondary muscle groups in list item subtitles
- "Add Exercise" button opens `AddExerciseInfoComponent`
- Calls `ExerciseInfoService.getExerciseInfos()`

---

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `VITE_SERVER_URL` | Backend API URL | `http://localhost:5231` |

**Location:** `.env`

---

## Styling

- **Framework:** Material-UI (MUI) v6.3.0
- **Theme:** Dark mode (`palette.mode: 'dark'`)
- **Font:** Roboto (via `@fontsource/roboto`)
- **CSS-in-JS:** Emotion (`@emotion/react`, `@emotion/styled`)

---

## Mobile Support

The Client app includes Capacitor configuration for iOS deployment.

**Configuration:** `capacitor.config.ts`

**Build for iOS:**
```bash
npm run build
npx cap sync ios
npx cap open ios
```

---

## AdminClient

The AdminClient (`WorkoutTracker.AdminClient/`) is a separate React web application for administrative tasks. It shares the same backend server as the Client but is focused on admin functionality like managing exercise info, users, and viewing logs. It is web-only (no mobile/Capacitor support).

### Technology Stack

| Layer | Technology | Version |
|-------|------------|---------|
| Framework | React | 19.2.0 |
| Language | TypeScript | 5.9.3 |
| Build | Vite | 7.3.1 |
| UI | Material-UI (MUI) | 7.3.8 |
| CSS-in-JS | Emotion | 11.14.0 |
| Font | Roboto | @fontsource/roboto |

### Project Structure

```
WorkoutTracker.AdminClient/
├── src/
│   ├── App.tsx              # Root component
│   ├── main.tsx             # Entry point
│   ├── App.css
│   ├── index.css
│   └── assets/
├── public/
├── index.html
├── package.json
├── vite.config.ts
├── tsconfig.json
├── tsconfig.app.json
├── tsconfig.node.json
└── eslint.config.js
```

### Current State

The project is freshly scaffolded from a Vite + React + TypeScript template. MUI and Emotion are installed but no application structure has been built yet. The following need to be established:

- **Routing** - react-router-dom is not yet installed
- **Authentication** - Should follow the same session cookie pattern as the Client (useAuth, AuthProvider, ProtectedRoute)
- **Services** - Should use the same BaseService pattern with `credentials: 'include'` for cookie auth
- **Domain models / DTOs** - Should mirror backend DTOs as needed, following the same FromData/ToData pattern
- **CORS** - The AdminClient's dev server origin needs to be added to the backend's CORS configuration

### Patterns to Follow from Client

When building out the AdminClient, follow these established patterns:

- **Services** extend `BaseService` with static methods and cookie credentials
- **Domain models** use `FromData()` / `ToData()` for DTO conversion
- **Auth** uses `AuthProvider` context with `useAuth` hook and `ProtectedRoute` guard
- **Pages** are organized in `Pages/` folder with `.tsx` and `.module.css` files
- **Components** are reusable UI in `Components/` folder
- **Environment** uses `VITE_SERVER_URL` for backend URL configuration

### Intended Features

- Exercise info management (add, edit, browse)
- User management
- System log viewing
