# Weekly GitHub Dev Summary — n8n + Claude

An importable n8n workflow for bounty #5. Every Friday at 17:00 it pulls the previous seven days of GitHub activity, asks Claude for an EN/FR narrative, and posts the result to a Slack incoming webhook. A Manual Test trigger is included for the required first-run validation.

## Setup — 4 steps

1. **Import** `weekly-dev-summary.json` into n8n.
2. **Configure** the `Config` node: set `githubRepo` (`owner/repo`), `slackWebhookUrl`, and `language` (`EN` or `FR`). The sponsor-pinned model remains `claude-sonnet-5`.
3. **Provide secrets** to the n8n process as `GITHUB_TOKEN` (GitHub API access) and `ANTHROPIC_API_KEY` (Claude Messages API), then restart n8n if your deployment requires it.
4. **Run `Manual Test` once**, confirm the Slack message, save the successful execution screenshot as `screenshot-success.png`, then publish/activate the workflow. The scheduled trigger runs Friday at 17:00 in the workflow timezone (`America/New_York` by default).

## What it does

- Runs weekly at Friday 17:00, with a separate manual trigger for first-run validation.
- Uses GitHub's GraphQL API to fetch up to 100 default-branch commits, recently closed issues, and recently merged pull requests, then filters the issue/PR collections to the same seven-day window.
- Calls Anthropic's Messages API with `claude-sonnet-5`, exactly as specified by the bounty.
- Generates the narrative in English or French based on `Config.language`.
- Posts the narrative to the configured Slack incoming webhook.
- Keeps API keys out of the exported workflow by reading `GITHUB_TOKEN` and `ANTHROPIC_API_KEY` from the n8n environment.

## Configuration

| Field | Example | Purpose |
|---|---|---|
| `githubRepo` | `n8n-io/n8n` | GitHub repository to summarize |
| `slackWebhookUrl` | `https://hooks.slack.com/services/...` | Destination Slack channel webhook |
| `language` | `EN` or `FR` | Narrative language |
| `claudeModel` | `claude-sonnet-5` | Bounty-required Claude model |

## Model-availability note

The bounty explicitly pins `claude-sonnet-5`. Anthropic's current model lifecycle documentation lists that model as retired after June 15, 2026. The committed workflow intentionally preserves the sponsor-required model value for acceptance. If a current Anthropic account rejects the retired model during execution, change only `Config.claudeModel` to the sponsor-approved current replacement and record that fact alongside the execution screenshot rather than silently changing the submitted requirement.

## Data shape and failure behavior

The GitHub request fails clearly if the repository is unavailable or GraphQL returns errors. The prompt includes exact activity counts and explicit `None` sections for empty categories so Claude is instructed not to invent events. Slack delivery is the final node; a green execution therefore proves GitHub fetch, Claude generation, extraction, and webhook delivery all completed.
