Weather Services Using MCP
==========================

An example project that exposes a Weather Alerts tool via an MCP (Model Context Protocol) server and provides an interactive client powered by `mcp-use` and Groq’s LLM through `langchain-groq`.

This repo demonstrates how to:
- Build an MCP server using `FastMCP` and publish tools.
- Launch the server via a JSON config and interact with it using an agent.
- Call the National Weather Service (NWS) API and present results in a user-friendly format.


Features
--------
- MCP server with a weather tool:
	- `get_alerts(state: str)` returns active NWS alerts for a US state (e.g., `CA`, `NY`).
- Agent-based interactive chat client with optional conversation memory.
- Clean, user-facing output for alerts (event, area, severity, description, instructions).
- Workspace‑relative MCP configuration for portability across machines.


Architecture
------------
- `server/weather.py` (MCP server)
	- Uses `FastMCP("weather")` from `mcp.server.fastmcp`.
	- Declares tools with `@mcp.tool()`.
	- Uses `httpx.AsyncClient` to call the NWS API (`https://api.weather.gov`) with proper headers.
- `server/weather.json` (MCP config)
	- Instructs MCP runtime to start the server using `uv run --with mcp[cli] mcp run server/weather.py`.
	- Uses a workspace‑relative path to avoid environment‑specific breakage.
- `server/client.py` (interactive agent client)
	- Loads environment variables via `python-dotenv` (`.env`).
	- Creates an `MCPClient` and `MCPAgent` from `mcp_use`.
	- Uses `ChatGroq` from `langchain-groq` with `GROQ_API_KEY`.
- `pyproject.toml`
	- Project metadata and dependencies managed by `uv`.
	- Note: The project name must not be `mcp` to avoid self‑dependency conflicts.


Project Structure
-----------------
```
.
├── main.py
├── pyproject.toml
├── README.md
└── server/
		├── client.py         # Interactive REPL using MCPAgent + ChatGroq
		├── weather.py        # MCP server exposing the get_alerts tool
		└── weather.json      # MCP config to launch the weather server
```


Prerequisites
-------------
- Python (managed by `uv`).
- `uv` installed on your system.
- Groq API key set in environment or `.env` file: `GROQ_API_KEY`.


Install & Setup
---------------
1. Install dependencies (captured in `pyproject.toml`).
	 - If needed, add/refresh deps:
		 - `mcp[cli]` for running the MCP server.
		 - `mcp-use` for the client and agent orchestration.
		 - `langchain-groq` for Groq model access.
2. Create a `.env` file at the project root with your Groq key:
	 ```env
	 GROQ_API_KEY=your-key-here
	 ```


How to Run
----------
- Start the interactive client (recommended):
	```powershell
	uv run server/client.py
	```
	Then type a prompt, e.g.:
	- `Provide me the weather alerts for CA`
	- `Provide me the weather alerts for NY`

- Optionally, run the MCP server directly for debugging:
	```powershell
	uv run --with mcp[cli] mcp run server/weather.py
	```


Tool: get_alerts(state)
-----------------------
Input
- `state` (str): Two‑letter state code, e.g. `CA`, `NY`.

Behavior
- Calls `https://api.weather.gov/alerts/active/area/{state}`.
- Returns a formatted string for each alert, including:
	- Event, Area, Severity, Description, Instructions.
- If no alerts or an error occurs, returns a helpful message.


Troubleshooting
---------------
- Connection closed when using the client
	- Ensure `server/weather.json` uses `server/weather.py` (relative path), not an absolute path to another machine.
	- Confirm `mcp[cli]` is installed and available to `uv`.
	- Verify your `.env` contains a valid `GROQ_API_KEY` (required by `ChatGroq`).

- Self‑dependency error when adding `mcp`
	- If `pyproject.toml` `[project].name` is `mcp`, `uv add mcp[cli]` will fail.
	- Rename the project (e.g., `weather-services-using-mcp`) and retry.

- No alerts for a state
	- The endpoint may legitimately return no active alerts. Try another state or check NWS status.


Extending the Server
--------------------
- Add more tools by decorating async functions with `@mcp.tool()` in `server/weather.py`.
- Keep outputs user‑friendly (plain text) unless the consumer expects structured data.
- Reuse `httpx.AsyncClient` for external calls; include appropriate headers.


License
-------
This project is provided as-is for educational/demo purposes.
