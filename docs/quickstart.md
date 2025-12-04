# Quick Start Guide

Get the agent orchestrator running in your project in 5 minutes.

## Prerequisites

- Claude Code installed and configured
- A project where you want to use orchestrated agents

## Step 1: Copy Files

Copy the `.agents` directory to your project:

```bash
cp -r .agents /path/to/your/project/
```

Your project should now have:
```
your-project/
├── .agents/
│   ├── orchestrator.md
│   └── manifests/
│       └── _template.yaml
```

## Step 2: Configure MCP

If you don't have a `.mcp.json`, copy the example:

```bash
cp .mcp.json.example /path/to/your/project/.mcp.json
```

If you already have `.mcp.json`, add the orchestrator subagent:

```json
{
  "subagents": {
    "orchestrator": {
      "description": "Agent orchestrator - analyzes tasks and determines optimal agent configuration",
      "prompt_file": ".agents/orchestrator.md"
    }
  }
}
```

## Step 3: Update CLAUDE.md

Add orchestration rules to your `CLAUDE.md`:

```markdown
## Agent Orchestration

**All tasks must pass through the orchestrator before execution.**

1. Read `.agents/orchestrator.md` for orchestration logic
2. Scan all skill sheets in `.agents/manifests/`
3. Follow the decision matrix to determine optimal agent configuration
4. Execute task with selected/generated agent
5. Report results including any agent evolution
```

## Step 4: Start Using

That's it! Now when you give Claude Code a task:

1. Claude reads the orchestration rules from CLAUDE.md
2. The orchestrator analyzes your task
3. It scans existing agents (initially empty)
4. It creates or selects the optimal agent
5. Task executes
6. New agents are saved for future use

## First Run Example

Your first task might look like:

```
You: "Create a REST API endpoint for user authentication"

Claude: 
- Orchestrator activated
- Scanning manifests: 0 agents found
- Coverage: 0% - Creating new agent
- Created: auth-api-expert
- Skill sheet saved to .agents/manifests/auth-api-expert.yaml
- Executing task...
[task completion]
```

Next similar task:

```
You: "Add password reset to the auth API"

Claude:
- Orchestrator activated
- Scanning manifests: auth-api-expert found
- Coverage: 92% - Using existing agent
- Executing with auth-api-expert...
[task completion]
```

## Watching Evolution

As you work, check `.agents/manifests/` to see your agent ecosystem grow:

```bash
ls .agents/manifests/
# _template.yaml
# auth-api-expert.yaml
# db-expert.yaml
# merged-auth-db.yaml
```

Merged agents will show their lineage:

```yaml
# merged-auth-db.yaml
name: merged-auth-db
parent_agents: [auth-api-expert, db-expert]
```

## Tips for Best Results

1. **Be specific in tasks**: Clear requirements help the orchestrator make better decisions

2. **Let it evolve**: Don't manually create agents unless needed—let the orchestrator build them from real tasks

3. **Check the reports**: The orchestrator reports what it did—use this to understand your ecosystem

4. **Trust the process**: It might create agents you wouldn't have—that's often a good thing

## Next Steps

- Read [Concept](concept.md) for deeper understanding
- See [Advanced Usage](advanced.md) for optimization strategies
