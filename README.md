# Subagent Dashboard

A real-time web dashboard for monitoring OpenClaw subagents. View active agents, their progress, transcripts, and manage stalled sessions.

## Features

- **Real-time monitoring** - Auto-refreshes every 3 seconds
- **Agent cards** - See model, age, tokens, and task progress
- **Transcript viewing** - View recent activity for each agent
- **Stall detection** - Highlights agents inactive for >30 minutes
- **Refresh controls** - Manual refresh and restart options
- **File management** - Browse, edit, create, delete workspace files
- **Git integration** - Status, stage, commit, push, pull, branch management

## Quick Start

```bash
cd workspace/skills/subagent-dashboard/scripts
./start_dashboard.sh
```

Then open http://localhost:8080 in your browser.

## Manual Start

```bash
cd workspace/skills/subagent-dashboard/scripts
python3 -m venv venv
source venv/bin/activate
pip install -r ../requirements.txt
python3 dashboard.py
```

## Security

The dashboard implements several security measures:

- **Localhost-only binding** - Binds to `127.0.0.1` by default. Only accessible from the local machine. Set `HOST=0.0.0.0` to expose to the network (not recommended without additional authentication).
- **CORS restricted to localhost** - API endpoints only accept requests from `http://localhost:*` and `http://127.0.0.1:*` origins.
- **Path traversal protection** - All file read/write/delete/rename and git operations validate that paths resolve within the workspace directory using `os.path.realpath()`.
- **Git path injection blocked** - Stage/unstage endpoints validate each path individually before passing to git commands. Uses `--` separator to prevent flag injection.
- **Branch name validation** - Git checkout and push validate branch names against a safe character set (`[a-zA-Z0-9_/.-]`).
- **Debug mode disabled** - Flask debug mode (`debug=False`) prevents Werkzeug interactive debugger exposure (which would allow RCE).
- **File tree restricted** - The `/api/files/tree` endpoint restricts the `root` parameter to the workspace directory.

## Usage

The dashboard automatically shows all active subagents (active within the last 60 minutes). Each card displays:

- **Agent number** and model
- **Age** - How long the agent has been running
- **Token usage** - Input and output tokens
- **Task progress** - If available (Task X/Y)
- **Task description** - What the agent is working on

### Actions

- **Transcript** - View recent activity/events for the agent
- **Refresh** - Refresh agent status
- **Kill** - Cancel a running subagent
- **Resume** - Resume stalled agents
- **Restart** - Restart stalled agents (requires gateway access)

### Auto-refresh

The dashboard auto-refreshes every 3 seconds. Click "Pause" to stop auto-refresh, or "Resume" to restart it.

## API Endpoints

### Subagent Monitoring
- `GET /api/subagents` - List all active subagents
- `GET /api/subagent/<session_id>/status` - Get detailed status
- `GET /api/subagent/<session_id>/transcript?lines=N` - Get transcript (default 50 lines)
- `POST /api/subagent/<session_id>/refresh` - Request refresh/restart
- `POST /api/subagent/<session_id>/kill` - Cancel a subagent
- `POST /api/subagent/<session_id>/resume` - Resume stalled agent
- `GET /api/stalled` - Get list of stalled agents

### File Management
- `GET /api/files/tree` - File tree (workspace-scoped)
- `GET /api/files/content?path=...` - Read file
- `PUT /api/files/content` - Write file
- `DELETE /api/files/content` - Delete file
- `POST /api/files/rename` - Rename file

### Git Operations
- `GET /api/git/status` - Repository status
- `POST /api/git/stage` - Stage files (paths validated)
- `POST /api/git/unstage` - Unstage files (paths validated)
- `POST /api/git/commit` - Commit staged changes
- `POST /api/git/push` - Push to remote
- `POST /api/git/pull` - Pull from remote
- `GET /api/git/log` - Commit history
- `GET /api/git/branches` - List branches
- `POST /api/git/checkout` - Switch branch (name validated)

## Requirements

- Python 3.7+
- Flask and flask-cors (installed via requirements.txt)
- Access to OpenClaw session files (`~/.openclaw/agents/main/sessions/`)
- Subagent-tracker skill installed

## Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `PORT` | `8080` | Server port (8080 avoids macOS AirPlay conflict on 5000) |
| `HOST` | `127.0.0.1` | Bind address. Use `0.0.0.0` to expose to network |
| `OPENCLAW_HOME` | `~/.openclaw` | OpenClaw home directory |

## Subagents not showing?

The dashboard reads from `OPENCLAW_HOME/agents/main/sessions/sessions.json` (default `~/.openclaw`). If you don't see spawned subagents:

1. **Same OpenClaw home** -- Start the dashboard with the same `OPENCLAW_HOME` your TUI/gateway use. Example: `OPENCLAW_HOME=/path/to/.openclaw ./scripts/start_dashboard.sh`
2. **Gateway writes sessions** -- Subagents appear only after the gateway has registered them in `sessions.json`. If the TUI runs elsewhere (e.g. different machine or sandbox), that gateway may write to a different path; point `OPENCLAW_HOME` at that path when starting the dashboard.

## Changelog

### v1.1.0 (Security hardening)

- Fixed: `/api/files/tree` now validates `root` parameter against workspace directory (was allowing arbitrary directory traversal)
- Fixed: `/api/git/status` now validates `path` parameter against workspace directory
- Fixed: `/api/git/stage` and `/api/git/unstage` now validate each file path individually (was injecting raw user paths into git commands)
- Fixed: `debug=True` changed to `debug=False` (Werkzeug debugger was exposing potential RCE)
- Fixed: Default bind address changed from `0.0.0.0` to `127.0.0.1` (localhost-only)
- Fixed: `/api/git/push` now validates branch name parameter
- Fixed: SKILL.md frontmatter populated with name, displayName, description, version (was empty)
- Fixed: SKILL.md heading changed from generic "# Skill" to actual skill name
- Fixed: `_meta.json` now includes `main`, `icon`, `color`, and `slug` fields
- Updated: README and SKILL.md document all security measures and API endpoints
