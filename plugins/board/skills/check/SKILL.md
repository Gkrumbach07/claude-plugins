---
name: board-check
description: Perform a full coordination check-in — read the blackboard, assess directives and blockers, then write back current status
---

# Board Check-In Skill

You are performing a **full blackboard coordination check-in**. This combines reading the board, assessing the situation, and writing back your status in one flow.

## Protocol

- **Repo:** `ambient-code/platform`
- **Label:** `blackboard`
- **Agent name:** Use env var `$AGENT_NAME`, default to `local`

## Step 1: Identify the Blackboard Issue

```bash
BRANCH=$(git rev-parse --abbrev-ref HEAD 2>/dev/null)
ISSUE=$(gh issue list --repo ambient-code/platform --label blackboard --state open --search "$BRANCH" --json number -q '.[0].number')
```

If `$ISSUE` is empty, tell the user: "No blackboard issue found for branch `$BRANCH`. Use /board-create to create one." Then **stop**.

## Step 2: Read the Full Board

### Read the issue body (contracts)

```bash
gh issue view "$ISSUE" --repo ambient-code/platform --json body -q '.body'
```

The issue body contains:
- **Contracts section** — shared interfaces, schemas, API boundaries
- **Agents table** — summary of all agents and their current status

### Read all comments (agent statuses)

```bash
gh issue view "$ISSUE" --repo ambient-code/platform --comments
```

Each agent maintains exactly one comment containing a JSON status object. Comments are identified by containing `"agent":"<agent-name>"` in the body.

## Step 3: Assess the Situation

Analyze everything you read and answer these questions:

1. **Directives for this agent:** Are there any directives or questions addressed to `$AGENT_NAME` (or `@all`) in the contracts or other agents' comments?
2. **Blockers from other agents:** Are any other agents blocked or reporting issues that would affect this agent's work?
3. **Contract changes:** Have any contracts changed that this agent should know about? Compare against what this agent was last working with.
4. **`[?BOSS]` questions:** Are there any `[?BOSS]` questions from other agents that this agent can help answer or that need escalation?
5. **Coordination opportunities:** Are any agents working on related areas where coordination would help?

## Step 4: Report Findings to the User

Present a clear summary:

```
=== Board Check-In for agent: <agent-name> ===
Branch: <branch>
Issue: #<number>

DIRECTIVES FOR YOU:
- (list any directives/questions aimed at this agent, or "None")

BLOCKERS FROM OTHERS:
- (list relevant blockers from other agents, or "None")

CONTRACT CHANGES:
- (list any contract changes, or "No changes detected")

OPEN QUESTIONS ([?BOSS]):
- (list any unanswered boss questions, or "None")

COORDINATION NOTES:
- (any observations about what other agents are doing, or "All clear")
```

## Step 5: Write Back Status

### Assess current work

Look at recent git changes to understand what this agent has been doing:

```bash
# Recent commits on this branch
git log --oneline -10

# Current working state
git status --short

# Files changed vs main
git diff --name-only main...HEAD 2>/dev/null | head -30
```

### Build the status JSON

Construct a single-line JSON object with the exact same schema used by `/board-write`:

```json
{"agent":"<AGENT_NAME>","status":"active|blocked|done","summary":"one-line summary","items":["what was done"],"next":["next steps"],"blockers":["if any"],"questions":["[?BOSS] if any"],"updated":"<ISO 8601 timestamp>"}
```

Generate the timestamp with: `date -u +%Y-%m-%dT%H:%M:%SZ`

### Find or create this agent's comment

Search existing comments for one containing this agent's name:

```bash
COMMENT_ID=$(gh api "repos/ambient-code/platform/issues/${ISSUE}/comments" --jq ".[] | select(.body | try fromjson | .agent == \"${AGENT_NAME}\") | .id")
```

**If `COMMENT_ID` is non-empty**, update it:
```bash
gh api "repos/ambient-code/platform/issues/comments/${COMMENT_ID}" -X PATCH -f body='<JSON>'
```

**If `COMMENT_ID` is empty**, create it:
```bash
gh api "repos/ambient-code/platform/issues/${ISSUE}/comments" -X POST -f body='<JSON>'
```

### Update the agents table in the issue body

Read the current issue body, find the `## Agents` markdown table, and update or add a row for this agent. Then write back:

```bash
BODY=$(gh issue view "$ISSUE" --repo ambient-code/platform --json body -q '.body')
# Parse table, update/add row for this agent
gh issue edit "$ISSUE" --repo ambient-code/platform --body "$UPDATED_BODY"
```

Table format:
```
| Agent | Status | Summary | Updated |
|-------|--------|---------|---------|
| <name> | <status> | <summary> | <timestamp> |
```

## Step 6: Summarize

End with a brief summary of what was read and written:

```
=== Check-In Complete ===
READ: Issue #<number>, <N> agent comments
WROTE: Updated status for <agent-name> (status: <status>)
NEXT: <suggested next action based on assessment>
```

## When to Invoke This Skill

Use this skill when the user says things like:
- "Check the board"
- "Board check-in"
- "What's happening on the blackboard?"
- "Sync with the team"
- "Check in"
