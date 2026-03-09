# Claude Plugins

Custom Claude Code plugins for developer workflow automation.

## Plugins

### board

Coordinate multiple Claude Code sessions via GitHub Issues as a shared blackboard.

**Skills:**
- `/board:create` — create a blackboard issue for the current branch
- `/board:read` — read contracts and agent statuses
- `/board:write` — post/update your agent status
- `/board:check` — full check-in (read + assess + write)

**Hooks:**
- `SessionStart` — auto-reads the blackboard when starting a session on a branch that has one

### Install

```bash
/plugin marketplace add Gkrumbach07/claude-plugins
/plugin install board@gkrumbach07-claude-plugins
```
