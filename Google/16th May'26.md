# GitHub Dev Card Generator: Prompt Guide


## Phase 1 — Project setup

### Prompt 1.1 — Set the working context 
```text
/"You are an expert Python developer building a GitHub Dev Card Generator. The stack is: Google ADK for agent orchestration, MCP (FastMCP) for tools, Gemini 2.5 Flash as the LLM, FastAPI as the backend, and React/HTML as the frontend. Everything deploys to Google Cloud Run. Write clean, modular Python. Prefer uv for dependency management."
```
<br>

### Prompt 1.2 — Building the project structure
```text
Create a complete project scaffold for a GitHub Dev Card Generator with this folder structure:


github-card-generator/
  backend/
    mcp_server.py       (MCP tools)
    agent.py            (ADK agent definition)
    main.py             (FastAPI app with runner)
    requirements.txt
    Dockerfile
  frontend/
    index.html          (single-page UI)
    Dockerfile
  docker-compose.yml    (for local testing)
  .env.example


Create all files with the correct boilerplate — empty functions are fine, just get the imports and structure right. Use `uv` for Python deps.
```
---
<br>

## Phase 2 — MCP Server (the tools)

### Prompt 2.1 — Build the MCP server with all 4 tools

```text
In backend/mcp_server.py, implement a FastMCP server with exactly these 4 tools:

1. scrape_github(username: str) -> dict
   - Calls the GitHub REST API (no auth needed for public profiles)
   - Returns: name, bio, location, public_repos, followers, top 6 repos (name, stars, language, description), most used languages aggregated
2. analyze_profile(github_data: dict) -> dict
   - Calls Gemini 2.5 Flash with the github_data
   - Returns a JSON with: developer_vibe (1 sentence personality), top_skills (list of 3), fun_fact (something clever inferred from their repos), card_theme (one of: "hacker", "builder", "researcher", "designer", "open-source-hero")
3. generate_card_html(username: str, github_data: dict, analysis: dict) -> str
   - Generates a self-contained HTML string for a beautiful dev card
   - Card shows: avatar, name, vibe sentence, top skills as badges, repo count, followers, top 3 repos, card_theme styling (dark for hacker, light for builder, etc.)
4. save_card(username: str, html: str) -> str
   - Saves the HTML to static/cards/{username}.html
   - Returns the relative URL path

Run the server with: uv run python mcp_server.py
```
<br>

### Prompt 2.2 — Test the MCP server in isolation with Gemini CLI

```text
Connect to my local MCP server and test it end to end. Run these steps in sequence:

1. Call scrape_github with username "torvalds"
2. Pass that result into analyze_profile
3. Generate an HTML card from the results using generate_card_html
4. Print the card_theme and developer_vibe from the analysis

Tell me if any tool fails and what the error is.
```
---

<br>

## Phase 3 — ADK Agent

### Prompt 3.1 — Build the ADK agent

```text
In backend/agent.py, create an ADK Agent called "github_card_agent" using Gemini 2.5 Flash.
Connect it to the MCP server at backend/mcp_server.py using McpToolset with stdio transport.

System instruction for the agent:
> "You are a GitHub profile analyst and dev card generator. When a user gives you a GitHub username, you ALWAYS follow this exact sequence: first call scrape_github, then analyze_profile with the result, then generate_card_html with all three inputs, then save_card. Never skip steps. Be enthusiastic about developers' work. If the profile is private or doesn't exist, say so clearly."

Export the agent as: github_card_agent
```
<br>

### Prompt 3.2 — Wire up the FastAPI backend

```text
In backend/main.py, create a FastAPI app that:

1. Imports github_card_agent from agent.py
2. Sets up InMemorySessionService and InMemoryMemoryService
3. Creates a Runner bound to the agent and services
4. Exposes POST /generate endpoint that accepts {"username": "someuser"}
   - Creates or reuses a session by username
   - Runs the agent with message "Generate a dev card for {username}"
   - Streams the agent events and returns the final card URL and HTML
5. Exposes GET /card/{username} to serve saved cards from static/cards/
6. Exposes GET /health for Cloud Run health checks
7. Add CORS middleware allowing all origins (for the frontend to call it).

Run with: uvicorn main:app --host 0.0.0.0 --port 8080

```

---

## Phase 4 — Frontend 

### Prompt 4.1 — Vibe-code the full frontend
```text
Create frontend/index.html as a single self-contained file (no build step, no npm) with this UI:

- Dark background (#0d1117, like GitHub's dark mode)
- Centered layout, max-width 600px
- Big heading: "GitHub Dev Card Generator"
- Subheading: "Enter any public GitHub username"
- Input field + "Generate Card" button (styled, animated)
- Loading state: show a pulsing skeleton while the card is being generated
- Result area: render the returned HTML card inline
- Share button: copies the card URL to clipboard
- Error state: friendly message if username not found

The frontend calls POST http://localhost:8080/generate with {"username": "..."} and displays the response. Make it look genuinely polished — not default HTML. Use Inter font from Google Fonts. Animate the button on hover. The card should appear with a fade-in.
```
---
<br>

## Phase 5 — Dockerize & Deploy

### Prompt 5.1 — Write the Dockerfiles

```text
Write Dockerfiles for both services:

**backend/Dockerfile:**
- Base: `python:3.12-slim`
- Install `uv`
- Copy requirements and install deps
- Copy source
- Expose port 8080
- CMD: `uvicorn main:app --host 0.0.0.0 --port 8080`

**frontend/Dockerfile:**
- Base: `nginx:alpine`
- Copy `index.html` into `/usr/share/nginx/html/`
- Replace the hardcoded `localhost:8080` backend URL with an environment variable `BACKEND_URL` using `envsubst` at container start
- Expose port 80

Also write `docker-compose.yml` that wires both containers together for local testing.
```
<br>

### Prompt 5.2 — Deploy via AntiGravity MCP 

```text
Deploy two Cloud Run services to Google Cloud for my GitHub Dev Card Generator project:

**Service 1 — backend:**
- Name: `github-card-backend`
- Source: `./backend` directory
- Port: 8080
- Environment variables: `GOOGLE_CLOUD_PROJECT=<your-project-id>`, `GEMINI_API_KEY=<your-key>`
- Allow unauthenticated requests: yes
- Memory: 512Mi
- Region: us-central1

**Service 2 — frontend:**
- Name: `github-card-frontend`
- Source: `./frontend` directory
- Port: 80
- Environment variable: `BACKEND_URL=<url of backend service above>`
- Allow unauthenticated requests: yes
- Memory: 256Mi
- Region: us-central1

Deploy backend first, get its URL, then deploy frontend with that URL as `BACKEND_URL`. Return both public URLs when done.
```
<br>

### 5.3 — Verify the deployment

```text
Check the health of both my Cloud Run services:
- `GET https://<backend-url>/health` — should return `{"status": "ok"}`
- Open `https://<frontend-url>` in a browser preview

If the backend health check fails, show me the Cloud Run logs for `github-card-backend`.
If the frontend loads but can't reach the backend, check CORS headers on the backend response.
```
---

## Optional: Add Memory Bank (bonus factor)
<br>

### Prompt — Hook up Vertex AI Memory Bank

```text
Upgrade `backend/main.py` to use persistent memory:
Replace `InMemorySessionService` with `VertexAiSessionService` and `InMemoryMemoryService` with `VertexAiMemoryBankService`, both using:
- `project=os.getenv("GOOGLE_CLOUD_PROJECT")`
- `location="us-central1"`
- `agent_engine_id=os.getenv("AGENT_ENGINE_ID")`

Add a script `backend/deploy_memory.py` that:
1. Creates a Vertex AI Agent Engine with a custom memory topic called "card_preferences" that extracts: preferred card theme, favorite languages, whether user wants a dark or light card
2. Prints the `agent_engine_id` to add to `.env`

This way the agent remembers "last time this user asked for a hacker-theme dark card" across sessions.
```

## Commands to run the backend

Cd to the place where the file is

<br>

```text
.\.venv\Scripts\activate
```

<br>

```text
Install dependency
pip install -r requirements.txt
```

<br>

```text
uvicorn main:app --reload --port 8080
```


