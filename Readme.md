# DevOps Assessment — Containerization & CI/CD Pipeline

This project containerizes a full-stack application (React frontend + Node.js/Express backend) using Docker and Docker Compose, and automates deployment using a Jenkins pipeline.

## Tech Stack
- Frontend: React (served via Nginx in production)
- Backend: Node.js, Express, Mongoose (MongoDB)
- Containerization: Docker, Docker Compose
- CI/CD: Jenkins

## Setup Instructions

### Prerequisites
- Docker Desktop installed and running
- Git

### Steps to run

1. Clone the repository:
   git clone https://github.com/JAINAMMESSI30/devops-assessment.git
   cd devops-assessment

2. Create a `.env` file in the project root with your MongoDB connection string:
   DB_URI=<your-mongodb-connection-string>

   (Note: this is optional — the app runs and demonstrates full frontend-backend communication even without a valid DB_URI, since the `/users` endpoint used for the Contact form dropdown returns hardcoded data. DB_URI is only needed if you want the `/contact/send` endpoint to actually persist messages to MongoDB.)

3. Build and run everything:
   docker compose up --build

4. Once running, access:
   - Frontend: http://localhost:3000
   - Backend API: http://localhost:4000

No manual steps are required after `docker compose up --build` — both services start automatically and the frontend can immediately communicate with the backend.

## Approach

- Wrote a standard Node.js Dockerfile for the backend, installing dependencies and running the Express server on port 4000.
- Used a multi-stage Dockerfile for the frontend: Stage 1 builds the React app using Node, Stage 2 serves the static build output using a lightweight Nginx image. This keeps the final image small and production-like.
- Used Docker Compose to orchestrate both services on a shared custom bridge network, with environment variables for configuration instead of hardcoded values.
- Wrote a Jenkins declarative pipeline with four stages: Checkout, Build Docker Images, Deploy, and Verify. The Deploy stage runs `docker compose down` before `up -d --build`, ensuring every pipeline run picks up the latest code changes rather than reusing stale containers.

## Debugging Notes (Issues + Fixes)

### 1. Frontend hardcoded backend URL (localhost issue in Docker)
**Issue:** The frontend's Contact.js had `axios.get('http://localhost:4000/users')` hardcoded.
**Root cause:** This works when both services run directly on the host machine, but the URL needed to be configurable for different environments.
**Fix:** Replaced the hardcoded URL with `process.env.REACT_APP_API_URL`, injected at Docker build time via a build argument in docker-compose.yml. Note: since the React app is compiled into static files that run in the user's browser (not inside the Docker network), the API URL must point to localhost:4000 (accessible via the host's port mapping) rather than the Docker service name — container-to-container DNS names only resolve for server-to-server calls, not for a browser-based frontend.

### 2. Docker build failing on node_modules (invalid file request)
**Issue:** docker compose build failed with "invalid file request node_modules/.bin/mime" while transferring the build context.
**Root cause:** The local node_modules folder (containing symlinks) was being copied into the Docker build context unnecessarily, since dependencies are installed fresh inside the container anyway.
**Fix:** Added a .dockerignore file in both backend/ and frontend/ excluding node_modules, .env, and log files.

### 3. React Router 404 on direct URL access (e.g. /contact-us)
**Issue:** Navigating within the app worked, but directly loading or refreshing http://localhost:3000/contact-us returned a 404 from Nginx.
**Root cause:** React Router handles routing client-side in the browser. Nginx, by default, looks for an actual file matching the requested path and returns 404 if none exists.
**Fix:** Added a custom nginx.conf with a try_files fallback rule, so any unmatched route serves index.html and lets React Router take over.

### 4. Jenkins deployment failing due to container name conflict
**Issue:** docker compose up -d --build failed in the Jenkins pipeline with "Conflict. The container name /backend is already in use".
**Root cause:** The docker-compose.yml originally specified fixed container_name values, which caused a naming clash with containers left running from earlier manual testing.
**Fix:** Removed the container_name overrides, allowing Docker Compose to auto-generate project-scoped container names, avoiding conflicts across different runs/environments.

## Assumptions Made
- A real MongoDB Atlas connection string was not required for grading purposes, since the primary API used for demonstrating frontend-backend communication (GET /users) returns hardcoded data regardless of DB connectivity.
- Jenkins runs on the same Windows host as Docker Desktop, so the pipeline uses `bat` steps and assumes docker is available in Jenkins' system PATH.
- The Jenkins pipeline checks out code from the GitHub repository (not the local filesystem) to reflect a realistic CI/CD flow where the pipeline always builds from the latest pushed commit.
