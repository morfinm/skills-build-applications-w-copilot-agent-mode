# OctoFit Tracker architecture

OctoFit Tracker is a three-tier application for a school fitness program. The starter code includes API contracts and intentionally incomplete implementation points so you can use GitHub Copilot agent mode in a test-driven loop.

![Three-tier OctoFit architecture](../.github/images/octofit-architecture.svg)

## Tier boundaries

- **Presentation tier**: React 19, Vite, React Router, Bootstrap, and a shared API client in `octofit-tracker/frontend/src`.
- **Logic tier**: Express and TypeScript routes/services in `octofit-tracker/backend/src` expose `/api/*` behavior on port `8000`.
- **Data tier**: Mongoose models and seed data in `octofit-tracker/backend/src/models` and `src/scripts/seed.ts` persist to MongoDB on port `27017`.

## Ports and environment rules

- MongoDB: `27017`
- Express API: `8000`
- Vite frontend: `5173`
- The browser calls relative `/api` paths. Vite proxies them to `http://localhost:8000`, so the unauthenticated API port stays private in Codespaces. `VITE_API_BASE_URL` can override the base URL.

## API contract

| Capability | Method and path | Expected behavior |
| --- | --- | --- |
| Activity tracking | `GET /api/activities`, `POST /api/activities` | List and create valid activity records. POST accepts `userId`, `teamId`, `type`, and `minutes`; `userId` and `teamId` must be well-formed IDs for an existing user and team; otherwise respond with a deliberate 4xx (never a 500 from a cast error). A created activity stores the submitted `user`, `team`, `type`, and `minutes` plus computed `points` and `occurredAt`. |
| Teams | `GET /api/teams`, `POST /api/teams` | List and create teams with members. Created and listed teams keep the submitted `name` and `mascot`. Reject a team without a name with `400`. |
| Leaderboard | `GET /api/leaderboard` | Return teams ordered by the sum of their activity points, highest first. Each entry has `teamName`, `totalPoints`, and a 1-based `rank`. |
| Workout suggestions | `GET /api/workouts/suggestions/:userEmail` | Recommend workouts from recent activity patterns for the matching seeded user's email address. The first suggestion's `focus` matches that user's goal or recent activity, with a `reason` explaining why. An email with no matching user returns a deliberate 4xx (for example, `404`). |

## Implementation decisions

Record your plan here before editing code.

- Presentation tier decision: Use React components that read from the shared API client, render loading/empty/error states, and surface accessible alerts instead of bypassing the API contract.
- Logic tier decision: Keep Express route handlers thin and push validation, aggregation, and suggestion logic into services so `/api/*` behavior is deterministic and testable.
- Data tier decision: Define Mongoose schemas with required validation and seed representative users, teams, activities, and workouts so the API can query real persisted data.
- Activity tracking approach: Validate `userId` and `teamId` as existing references, store the normalized user/team IDs plus computed `points` and `occurredAt`, and reject invalid input with a 4xx instead of exposing a CastError.
- Team management approach: Require a team `name`, keep `mascot` and `members` data intact, and return created data in the same shape used by list endpoints.
- Leaderboard approach: Aggregate activity points by team, sort highest first, and assign a 1-based rank alongside `teamName` and `totalPoints`.
- Workout suggestions approach: Match suggestions to the seeded user’s email and recent activity/goal, returning a first recommendation that aligns to their fitness pattern and a clear `reason`.

## Test plan

Record the commands you run and what each failure means.

- Data-tier command: `npm run test:data --prefix octofit-tracker/backend` — validates the Mongoose schemas and seed data contract; failures here mean the database layer is missing required fields or relationships.
- API command: `npm run test:api --prefix octofit-tracker/backend` — validates business logic and HTTP behavior; failures here indicate invalid input handling, leaderboard aggregation, or workout recommendation logic.
- Frontend command: `npm test --prefix octofit-tracker/frontend` — validates the UI contract; failures here indicate placeholder content, missing loading states, or inaccessible error messages in the React components.
- Backend build command: `npm run build --prefix octofit-tracker/backend` — ensures the TypeScript backend compiles cleanly after the data and API implementation.
- Frontend build command: `npm run build --prefix octofit-tracker/frontend` — ensures the React app compiles and the Vite + TypeScript presentation tier is valid after wiring the real API client and UI states.
