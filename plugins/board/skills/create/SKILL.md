---
name: board-create
description: Create a coordination blackboard (GitHub Issue) for the current feature branch, used by the Agent Boss Lite system to coordinate multiple Claude Code sessions.
---

# Board Create

Create a blackboard GitHub Issue for the current branch so that multiple agents can coordinate via comments.

## Steps

1. **Get the current branch name:**
   ```bash
   BRANCH=$(git rev-parse --abbrev-ref HEAD)
   ```

2. **Check if a blackboard issue already exists for this branch:**
   ```bash
   ISSUE_NUM=$(gh issue list --repo ambient-code/platform --label blackboard --state open --search "$BRANCH" --json number,title -q '.[0].number')
   ```
   - If `ISSUE_NUM` is non-empty, print the issue number and URL, then stop. Do not create a duplicate.

3. **Create the issue.** Use the title format `board: <branch-name>` and label `blackboard`. If the user provided an optional description argument, include it under the `## Contracts` section. Otherwise leave the section as a placeholder.

   ```bash
   gh issue create --repo ambient-code/platform \
     --title "board: $BRANCH" \
     --label blackboard \
     --body "$(cat <<'BODY'
   ## Contracts

   <!-- Seed contracts / goals here -->
   ${DESCRIPTION_PLACEHOLDER}

   ## Agents

   | Agent | Status | Summary | Updated |
   |-------|--------|---------|---------|
   <!-- Agents register by posting a JSON comment -->
   BODY
   )"
   ```

   Replace `${DESCRIPTION_PLACEHOLDER}` with the user-supplied description if provided, or leave a blank line.

4. **Print the resulting issue number and URL** from the `gh issue create` output.

## Agent Comment Format

When agents later write to this board, each agent owns exactly one comment (created on first write, edited in place after). Comments use this JSON format:

```json
{
  "agent": "<name from $AGENT_NAME env var, default 'local'>",
  "status": "active|blocked|done",
  "summary": "one line",
  "items": ["details"],
  "next": ["next steps"],
  "blockers": ["if any"],
  "questions": ["[?BOSS] if any"],
  "updated": "ISO timestamp"
}
```

## Notes

- The repo is `ambient-code/platform`.
- The label is `blackboard`.
- Use the `gh` CLI for all GitHub operations.
- The agent name comes from the `$AGENT_NAME` environment variable, defaulting to `local`.
