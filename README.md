# AI Travel Planner

A small full-stack engineering assessment demo for a multi-user AI travel planner. Users can register, log in, generate trip itineraries, estimate budgets, edit activities, regenerate individual days, and view hotel suggestions.

## Tech Stack

- Frontend: Next.js, TypeScript, Tailwind CSS
- Backend: Node.js, Express, JavaScript
- Database: MongoDB with Mongoose
- Auth: JWT access token plus bcrypt password hashing
- AI: OpenAI chat completions when `OPENAI_API_KEY` is set, with a deterministic fallback generator for demo environments

The frontend and backend are intentionally separate folders so each can be deployed, scaled, and configured independently.

## Features

- User registration and login
- Protected dashboard
- Strict user-level data isolation on every trip API query
- Trip input form for destination, days, budget type, interests, and pace
- Day-by-day itinerary generation
- Budget estimation for flights, accommodation, food, activities, and transport
- Hotel suggestions by destination and budget tier
- Editable itinerary with add activity, remove activity, and regenerate day actions
- Custom feature: Smart Buffer local tips

## Custom Feature: Smart Buffer

The Smart Buffer feature adds practical local tips alongside the itinerary, such as clustering activities by neighborhood, keeping an indoor backup, and reserving popular stops ahead. It solves a common AI-itinerary problem: generated plans often look good on paper but fail when transit time, weather, or booking constraints appear. The feature nudges the user toward a more realistic plan without adding complexity to the main form.

## Local Setup

### 1. Backend

Start MongoDB locally. If Docker is available:

```bash
docker compose up -d
```

```bash
cd backend
cp .env.example .env
npm install
npm run dev
```

Required backend environment variables:

```bash
PORT=5000
MONGODB_URI=mongodb://127.0.0.1:27017/ai-travel-planner
JWT_SECRET=replace-with-a-long-random-secret
CLIENT_ORIGIN=http://localhost:3000
OPENAI_API_KEY=
OPENAI_MODEL=gpt-4o-mini
```

`OPENAI_API_KEY` is optional for local demos. If it is empty, the app uses the fallback generator so the full workflow still works.

### 2. Frontend

```bash
cd frontend
cp .env.example .env.local
npm install
npm run dev
```

Required frontend environment variable:

```bash
NEXT_PUBLIC_API_URL=http://localhost:5000
```

Open `http://localhost:3000`.

## Architecture

```text
frontend/
  src/app            Next.js routes for auth and dashboard
  src/components     Reusable UI components
  src/lib/api.ts     API client and token storage
  src/types          Shared frontend types

backend/
  src/models         Mongoose User and Trip schemas
  src/controllers    Request handlers
  src/routes         Express routes
  src/middleware     Auth and error handling
  src/services       Token and itinerary generation services
```

The backend exposes `/api/auth/*` for authentication and `/api/trips/*` for protected trip operations. Trip ownership is enforced by querying with both `_id` and `user`, for example `Trip.findOne({ _id: req.params.id, user: req.user._id })`.

## Authentication And Authorization

- Passwords are hashed with bcrypt before storage.
- Login and registration return a signed JWT.
- The frontend stores the token in `localStorage` for this assessment demo.
- Protected backend routes require `Authorization: Bearer <token>`.
- Users can only list, read, edit, regenerate, or delete trips that match their own user id.

For a production app, I would move tokens into secure httpOnly cookies, add refresh-token rotation, and add a full audit trail for generated/edited itinerary changes.

## AI Agent Design

The itinerary service has one responsibility: convert structured trip inputs into structured travel output. It asks the LLM to return JSON containing itinerary days, budget, hotels, local tips, and the generation source. The controller persists that structured object directly after validation of the user input.

The fallback generator keeps the project demoable without paid API access. This is useful for reviewers because auth, persistence, editing, and authorization can be evaluated immediately.

## Deployment

Recommended deployment:

- Frontend: Vercel
- Backend: Render, Railway, or Fly.io
- Database: MongoDB Atlas

Deployment environment variables:

- Set backend `MONGODB_URI` to the MongoDB Atlas connection string.
- Set backend `JWT_SECRET` to a strong secret value.
- Set backend `CLIENT_ORIGIN` to the deployed frontend URL.
- Set frontend `NEXT_PUBLIC_API_URL` to the deployed backend URL.
- Store `OPENAI_API_KEY` only in the backend environment, never in the frontend.

## Key Decisions And Trade-Offs

- Separated frontend and backend to match the assessment and keep deployment boundaries clear.
- Used JWT for a compact demo auth flow, while documenting why httpOnly cookies would be better in production.
- Stored generated itineraries in MongoDB so user edits persist and can be isolated by owner.
- Used a fallback generator to avoid making the demo brittle when an LLM key is unavailable.
- Kept the UI dense and dashboard-focused rather than building a landing page, because the assessment evaluates the product workflow.

## Known Limitations

- No automated test suite is included yet.
- Token storage uses `localStorage`, which is acceptable for a demo but not ideal for production.
- Hotel suggestions are AI-generated estimates, not live availability or pricing.
- Budget estimates are approximate and should be presented as planning guidance, not booking quotes.
- Public deployment and walkthrough video still need to be created from the finished repository.

## Suggested Walkthrough Video Script

1. Register a new user and show the empty dashboard.
2. Generate a trip with destination, days, budget, interests, and pace.
3. Show budget, hotel suggestions, and Smart Buffer tips.
4. Add an activity, remove an activity, and regenerate one day.
5. Log out, register a second user, and show that the first user's trips are not visible.
6. Briefly explain the frontend/backend separation and user-owned trip queries.
