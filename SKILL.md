---
name: subagent-dashboard
displayName: Subagent Dashboard | OpenClaw Skill
description: Web dashboard for real-time monitoring and management of OpenClaw subagents. View active agents, transcripts, task progress, and manage stalled sessions.
version: 1.1.0
---

# Subagent Dashboard | OpenClaw Skill

Web dashboard for real-time monitoring and management of OpenClaw subagents.

## Installation

```bash
cd workspace/skills/subagent-dashboard/scripts
./start_dashboard.sh
```

Or manually:
```bash
python3 -m venv venv
source venv/bin/activate
pip install -r ../requirements.txt
python3 dashboard.py
```

## Usage

Start the dashboard and open http://localhost:8080 in your browser.

The dashboard shows:
- **All sessions** from sessions.json: main (orchestrator), subagents, and optionally cron jobs
- Real-time updates (auto-refresh every 3 seconds)
- Agent details: model, age, tokens, task progress; role badges (Main / Subagent / Cron)
- Transcript viewing for each agent
- Stalled agent detection (>30 min inactive)

## Purpose

Provides a web UI to:
- Monitor active subagents in real-time
- View agent transcripts and activity
- Detect and manage stalled agents
- Track task progress and token usage

## Security

- **Localhost-only by default**: Binds to 127.0.0.1 (set `HOST=0.0.0.0` to expose to network)
- **CORS restricted**: Only localhost origins allowed
- **Path traversal protection**: All file/git endpoints validate paths stay within workspace
- **Branch name validation**: Git checkout/push validate branch names against safe character set
- **Debug mode disabled**: Werkzeug debugger is off in production
- **Git path injection blocked**: Stage/unstage endpoints validate each path individually

## API Endpoints

### Subagent Monitoring
- `GET /api/subagents` - List all active subagents
- `GET /api/subagent/<id>/status` - Get detailed status
- `GET /api/subagent/<id>/transcript?lines=N` - Get transcript (default 50 lines)
- `POST /api/subagent/<id>/refresh` - Request refresh
- `POST /api/subagent/<id>/kill` - Cancel a subagent
- `POST /api/subagent/<id>/resume` - Resume stalled agent
- `GET /api/stalled` - Get list of stalled agents

### File Management (workspace-scoped)
- `GET /api/files/tree` - File tree (workspace only)
- `GET /api/files/content?path=...` - Read file content
- `PUT /api/files/content` - Write file content
- `DELETE /api/files/content` - Delete file
- `POST /api/files/rename` - Rename file

### Git Operations (workspace-scoped)
- `GET /api/git/status` - Repository status
- `POST /api/git/stage` - Stage files
- `POST /api/git/unstage` - Unstage files
- `POST /api/git/commit` - Commit staged changes
- `POST /api/git/push` - Push to remote
- `POST /api/git/pull` - Pull from remote
- `GET /api/git/log` - Commit history
- `GET /api/git/branches` - List branches
- `POST /api/git/checkout` - Switch branch

## Dependencies

- Flask (web server)
- flask-cors (CORS support)
- Subagent-tracker skill (for monitoring data)

## Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `PORT` | `8080` | Server port |
| `HOST` | `127.0.0.1` | Bind address (use `0.0.0.0` for network access) |
| `OPENCLAW_HOME` | `~/.openclaw` | OpenClaw home directory |

## Integration

The dashboard uses the subagent-tracker skill to fetch data. It reads:
- `~/.openclaw/agents/main/sessions/sessions.json` - Session list
- `~/.openclaw/agents/main/sessions/*.jsonl` - Transcript files
- `~/.openclaw/agents/main/subagents/runs.json` - Task mapping
