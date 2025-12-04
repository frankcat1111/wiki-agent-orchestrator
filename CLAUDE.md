# CLAUDE.md Example

Add the following section to your project's CLAUDE.md file.

---

## Agent Orchestration

**All tasks must pass through the orchestrator before execution.**

### Workflow

1. Read `.claude/agents/orchestrator.md` for orchestration logic
2. Scan all skill sheets in `.claude/agents/manifests/`
3. Follow the orchestrator's decision matrix to determine optimal agent configuration
4. Execute task with selected/generated agent
5. Report results including any agent evolution

### Agent Management Rules

- **New agents**: Always create a skill sheet in `.claude/agents/manifests/`
- **Merged agents**: Record parent lineage in `parent_agents` field
- **Naming convention**: 
  - Specialized: `{domain}-expert` (e.g., `db-expert`)
  - Merged: `merged-{domain1}-{domain2}` (e.g., `merged-api-db`)

### Quick Reference

```
Coverage 90%+  → Use existing agent
Coverage 60-90% → Merge agents
Coverage <60%  → Create new agent
```
