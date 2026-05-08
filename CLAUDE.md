# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Vidly is a full-stack movie management app used as a Docker learning exercise. It has three services orchestrated with Docker Compose:
- **frontend**: React (CRA) on port 3000
- **backend**: Express + Mongoose on port 3001
- **db**: MongoDB 4.0 on port 27017

## Running the Stack

```bash
# Start all services (build images + run containers)
docker-compose up --build

# Run in background
docker-compose up -d
```

The backend `docker-entrypoint.sh` waits for MongoDB, runs migrations, then starts the server via nodemon.

## Development Without Docker

**Backend** (requires a local MongoDB instance):
```bash
cd backend
npm install
npm start          # nodemon, ignores ./tests
npm run db:up      # run pending migrations
```

**Frontend:**
```bash
cd frontend
npm install
npm start          # CRA dev server on port 3000
npm run build
```

## Testing

**Backend** (Jest + Supertest, hits a real MongoDB — no mocks):
```bash
cd backend
npm test                          # watch mode
npx jest tests/app.test.js        # single file
```

Tests connect to the DB configured via `DB_URL` env var (defaults to `mongodb://localhost/vidly`). Each test manages its own data — inserting before and deleting after.

**Frontend** (React Testing Library + msw for API mocking):
```bash
cd frontend
npm test
```

## Architecture

### Backend
- `app.js` — Express app setup (CORS, JSON parsing, route registration). Separated from `index.js` so Supertest can import the app without starting the server.
- `index.js` — Connects to MongoDB then starts the HTTP server.
- `db.js` — Mongoose connection helper; reads `DB_URL` env var, defaults to `mongodb://localhost/vidly`.
- `routes/movies.js` — CRUD endpoints: `GET /api/movies`, `GET /api/movies/:id`, `POST /api/movies`, `DELETE /api/movies/:id`.
- `middleware/validateId.js` — Returns 404 for non-ObjectId `:id` params before hitting the DB.
- `models/movie.js` — Single `Movie` model with a required `title` field.
- `migrations/` — `migrate-mongo` migrations; run via `npm run db:up`.

### Frontend
- `src/services/api.js` — Thin axios wrapper; base URL is `REACT_APP_API_URL` env var or `http://localhost:3001/api`.
- `src/components/App.jsx` — Root component; owns all state (movies list, error). Performs optimistic UI updates for add/delete before the API responds, rolls back on failure.
- `MovieForm.jsx` → `MovieList.jsx` → `MovieListItem.jsx` — Component hierarchy for adding and displaying movies.

### Docker
- Both Dockerfiles use `node:14.16.0-alpine3.13`, run as a non-root `app` user, and copy `package*.json` before source to leverage layer caching.
- `backend/wait-for` — Shell script that polls until `db:27017` is accepting connections before the entrypoint proceeds.
- The `db` service uses a named volume `vidly` for MongoDB data persistence.

## Environment Variables

| Variable | Service | Default | Purpose |
|---|---|---|---|
| `DB_URL` | backend | `mongodb://localhost/vidly` | MongoDB connection string |
| `REACT_APP_API_URL` | frontend | `http://localhost:3001/api` | Backend base URL |
| `PORT` | backend | `3001` | HTTP listen port |
