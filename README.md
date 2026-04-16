Agentic workflow definitions for testing ambient workflows on the [Ambient Code Platform](https://github.com/ambient-code) (ACP).

## Structure

```
workflows/                          # ACP workflow definitions
  <workflow-name>/                  # One directory per workflow
    .ambient/ambient.json           # Workflow metadata, prompts, result artifacts
    .claude/settings.json           # Tool permissions and plugin config
    CLAUDE.md                       # Agent context and constraints

scheduled-sessions/                 # ScheduledSession API payloads
  <session-name>.json               # Cron schedule + session template

reports/                            # Auto-generated reports (committed by agent sessions)
```

## Adding a Workflow

Each workflow is a directory under `workflows/` with three files:

1. **`.ambient/ambient.json`** -- Metadata: name, description, system prompt, startup prompt, and result artifact patterns
2. **`.claude/settings.json`** -- Tool permissions (which skills, bash commands, and tools the agent can use)
3. **`CLAUDE.md`** -- Agent context document: purpose, constraints, available repos, API details, and instructions

## Adding a Scheduled Session

Create a JSON file in `scheduled-sessions/` following this structure:

```json
{
  "schedule": "0 10 * * 1-5",
  "displayName": "Session Display Name",
  "suspend": false,
  "reuseLastSession": false,
  "sessionTemplate": {
    "initialPrompt": "Run the workflow.",
    "displayName": "Session Name",
    "llmSettings": {
      "model": "claude-sonnet-4-5",
      "temperature": 0,
      "maxTokens": 8192
    },
    "timeout": 2400,
    "inactivityTimeout": 300,
    "stopOnRunFinished": true,
    "repos": [
      {
        "url": "https://github.com/jimdaga/ambient",
        "branch": "main",
        "autoPush": true
      }
    ],
    "activeWorkflow": {
      "gitUrl": "https://github.com/jimdaga/ambient",
      "branch": "main",
      "path": "workflows/<workflow-name>"
    }
  }
}
```

Push to `main` and the GitHub Action will deploy it to ACP automatically.

## Deploying Sessions

Sessions are deployed automatically via GitHub Actions when `scheduled-sessions/*.json` files are pushed to `main`. You can also trigger deployment manually from the Actions tab.

Required repository secrets:
- `ACP_API` -- ACP API base URL
- `ACP_TOKEN` -- ACP authentication token
- `ACP_PROJECT` -- ACP project identifier

## Security

- `.ambient.key` is gitignored and must never be committed
- Jira credentials are injected via environment variables at session creation time, not stored in git
