# Subagent Dashboard Skill

Web dashboard for real-time monitoring and management of OpenClaw subagents.

## Purpose

Provides a web UI to:
- Monitor active subagents in real-time
- View agent transcripts and activity
- Detect and manage stalled agents
- Track task progress and token usage

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
- All active subagents (active within 60 minutes)
- Real-time updates (auto-refresh every 3 seconds)
- Agent details: model, age, tokens, task progress
- Transcript viewing for each agent
- Stalled agent detection (>30 min inactive)

## Dependencies

- Flask (web server)
- flask-cors (CORS support)
- Subagent-tracker skill (for data)

## Configuration

Set `PORT` environment variable to change the server port (default: 8080).

## Integration

The dashboard uses the subagent-tracker skill to fetch data. It reads:
- `~/.openclaw/agents/main/sessions/sessions.json` - Session list
- `~/.openclaw/agents/main/sessions/*.jsonl` - Transcript files
- `~/.openclaw/agents/main/subagents/runs.json` - Task mapping
