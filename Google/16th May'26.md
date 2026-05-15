## This file contains the prompts used to build the Project


### Phase 1 — Project setup

#### Prompt 1.1 — Set the working context 
"You are an expert Python developer building a GitHub Dev Card Generator. The stack is: Google ADK for agent orchestration, MCP (FastMCP) for tools, Gemini 2.5 Flash as the LLM, FastAPI as the backend, and React/HTML as the frontend. Everything deploys to Google Cloud Run. Write clean, modular Python. Prefer uv for dependency management."

#### Prompt 1.2 — Building the project structure
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

Create all files with the correct boilerplate — empty functions are fine, just get the imports and structure right. Use uv for Python deps.
