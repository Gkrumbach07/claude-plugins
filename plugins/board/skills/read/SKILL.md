---
name: board-read
description: Read and summarize the coordination blackboard for the current feature branch, showing contracts, agent statuses, blockers, and open questions.
---

# Board Read

Read the blackboard GitHub Issue for the current branch and present a formatted summary of all agent activity.

## Steps

1. **Get the current branch name:**
   ```bash
   BRANCH=$(git rev-parse --abbrev-ref HEAD)
   ```

2. **Find the blackboard issue for this branch:**
   ```bash
   ISSUE_NUM=$(gh issue list --repo ambient-code/platform --label blackboard --state open --search "$BRANCH" --json number,title -q '.[0].number')
   ```
   - If `ISSUE_NUM` is empty, inform the user that no blackboard exists for this branch and suggest running `/board-create` to create one. Then stop.

3. **Fetch the issue body:**
   ```bash
   gh api repos/ambient-code/platform/issues/$ISSUE_NUM --jq '{body: .body, comments_url: .comments_url}'
   ```

4. **Fetch all comments on the issue:**
   ```bash
   gh api repos/ambient-code/platform/issues/$ISSUE_NUM/comments --jq '.[] | {id: .id, body: .body}'
   ```

5. **Parse each comment.** Try to parse each comment body as JSON matching the agent comment schema. Skip any comments that are not valid agent JSON (e.g., human discussion, bot messages).

   Agent comment schema:
   ```json
   {
     "agent": "name",
     "status": "active|blocked|done",
     "summary": "one line",
     "items": ["details"],
     "next": ["next steps"],
     "blockers": ["if any"],
     "questions": ["[?BOSS] if any"],
     "updated": "ISO timestamp"
   }
   ```

6. **Present a formatted summary** with these sections:

   ### Contracts
   Display the contents of the `## Contracts` section from the issue body.

   ### Agent Status
   Render a table of all agents parsed from comments:

   | Agent | Status | Summary | Updated |
   |-------|--------|---------|---------|
   | (from parsed comments) | | | |

   ### Blockers
   List any blockers from any agent. If none, say "None."

   ### Open Questions
   List any questions tagged `[?BOSS]` from any agent. If none, say "None."

## Notes

- The repo is `ambient-code/platform`.
- The label is `blackboard`.
- Use the `gh` CLI for all GitHub operations.
- Each agent owns exactly one comment (created on first write, edited in place after).
