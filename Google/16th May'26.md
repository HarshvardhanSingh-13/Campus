## This file contains the prompts used to build the Project


### Phase 1 — Project setup

#### Prompt 1.1 — Set the working context 
```text
/"You are an expert Python developer building a GitHub Dev Card Generator. The stack is: Google ADK for agent orchestration, MCP (FastMCP) for tools, Gemini 2.5 Flash as the LLM, FastAPI as the backend, and React/HTML as the frontend. Everything deploys to Google Cloud Run. Write clean, modular Python. Prefer uv for dependency management."
```


### Prompt 2 — Scaffold the project structure
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


### Phase 2 — MCP Server (the tools)

#### Prompt 2.1 — Build the MCP server with all 4 tools

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


