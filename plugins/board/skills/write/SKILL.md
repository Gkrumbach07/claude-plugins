---
name: board-write
description: Post or update this agent's status on the GitHub Issue-based coordination blackboard
---

# Board Write

Post or update your status comment on the blackboard issue for the current branch.

## Protocol

- Repo: `ambient-code/platform`
- Blackboard issues carry the label `blackboard`
- Issue title format: `board: <branch-name>`
- Each agent owns exactly ONE comment on the issue (edit in place, never create duplicates)

## Steps

### 1. Get the current branch

```bash
BRANCH=$(git rev-parse --abbrev-ref HEAD)
```

### 2. Determine agent name

```bash
AGENT_NAME="${AGENT_NAME:-local}"
```

### 3. Find the blackboard issue

```bash
ISSUE_NUMBER=$(gh issue list --repo ambient-code/platform --label blackboard --state open --search "board: $BRANCH" --json number -q '.[0].number')
```

If `ISSUE_NUMBER` is empty, stop and tell the user:
> No blackboard issue found for branch `<branch>`. Run `/board-create` first.

### 4. Assess current work status

Review recent changes, git log, and working tree to understand:
- What was accomplished (items)
- What is next (next steps)
- Any blockers
- Any questions for the human (prefix each with `[?BOSS]`)

Determine the appropriate status:
- `active` — work is in progress
- `blocked` — cannot proceed without resolution
- `done` — task is complete

### 5. Build the JSON payload

Construct a single-line JSON object with this exact schema:

```json
{"agent":"<AGENT_NAME>","status":"active|blocked|done","summary":"one-line summary","items":["what was done"],"next":["next steps"],"blockers":["if any"],"questions":["[?BOSS] if any"],"updated":"<ISO 8601 timestamp>"}
```

Generate the ISO timestamp with: `date -u +%Y-%m-%dT%H:%M:%SZ`

Keep `blockers` and `questions` as empty arrays `[]` when there are none.

### 6. Find this agent's existing comment

```bash
COMMENT_ID=$(gh api "repos/ambient-code/platform/issues/${ISSUE_NUMBER}/comments" --jq ".[] | select(.body | try fromjson | .agent == \"${AGENT_NAME}\") | .id")
```

### 7. Write the comment

**If `COMMENT_ID` is non-empty** — update the existing comment:

```bash
gh api "repos/ambient-code/platform/issues/comments/${COMMENT_ID}" -X PATCH -f body='<JSON>'
```

**If `COMMENT_ID` is empty** — create a new comment:

```bash
gh api "repos/ambient-code/platform/issues/${ISSUE_NUMBER}/comments" -X POST -f body='<JSON>'
```

### 8. Update the Agents table in the issue body

After writing the comment, update the summary table in the issue body:

1. Read the current issue body:
   ```bash
   BODY=$(gh issue view "$ISSUE_NUMBER" --repo ambient-code/platform --json body -q '.body')
   ```

2. Parse the `## Agents` section and its markdown table.

3. If a row for this agent already exists, update its status, summary, and timestamp. If no row exists, append a new row.

   Table format:
   ```
   | Agent | Status | Summary | Updated |
   |-------|--------|---------|---------|
   | <name> | <status> | <summary> | <timestamp> |
   ```

4. Write the updated body back:
   ```bash
   gh issue edit "$ISSUE_NUMBER" --repo ambient-code/platform --body "$UPDATED_BODY"
   ```

### 9. Confirm

Report what was posted: the agent name, status, summary, and whether the comment was created or updated.
