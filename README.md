# Agent Orchestrator Template

A self-evolving subagent management system for Claude Code. Instead of manually managing agents, let an orchestrator automatically compose, merge, and create agents based on task requirements.

> **Original concept by [@shintaro_sprech](https://x.com/shintaro_sprech)**

## Concept

Traditional workflow:
```
Task → Human selects agent → Execute → Agents accumulate → Human manually reorganizes
```

Orchestrated workflow:
```
Task → Orchestrator analyzes → Optimal agent auto-selected/generated → Execute → Agents evolve naturally
```

## Key Features

- **Autonomous agent selection**: Orchestrator evaluates task against existing agents
- **Dynamic merging**: Combines agents when synergy improves outcomes
- **Automatic specialization**: Creates new focused agents when gaps exist
- **Natural evolution**: Strong agents survive, weak ones fade through disuse
- **Lineage tracking**: Merged agents remember their parents, enabling evolution chains

## Quick Start

### 1. Copy to your project

```bash
# Copy the .agents directory to your project
cp -r .agents /path/to/your/project/

# Copy example files and rename
cp CLAUDE.md.example /path/to/your/project/CLAUDE.md  # or append to existing
cp .mcp.json.example /path/to/your/project/.mcp.json  # or merge with existing
```

### 2. Add orchestration rules to CLAUDE.md

Add the content from `CLAUDE.md.example` to your project's `CLAUDE.md`.

### 3. Start using

The orchestrator will automatically:
1. Analyze each task you give
2. Scan existing agent skill sheets
3. Decide: use existing / merge / create new
4. Execute with optimal configuration
5. Report what agents were used or created

## Directory Structure

```
your-project/
├── .agents/
│   ├── orchestrator.md      # Orchestrator logic
│   └── manifests/           # Agent skill sheets
│       ├── _template.yaml   # Template for new agents
│       ├── agent-a.yaml     # Auto-generated
│       └── merged-a-b.yaml  # Auto-generated merged agent
├── .mcp.json                # Subagent definitions
└── CLAUDE.md                # Project instructions
```

## How It Works

### Coverage-Based Decision

| Task Coverage | Action |
|---------------|--------|
| 90%+ | Use existing agent directly |
| 60-90% | Merge relevant agents |
| Below 60% | Create new specialized agent |

### Skill Sheet Format

Every agent has a skill sheet describing its capabilities:

```yaml
name: db-expert
version: "1.0"
created: 2025-01-15
parent_agents: []

domain: Database operations and optimization

capabilities:
  - SQL query optimization
  - Schema design
  - Index strategy

constraints:
  - No frontend work
  - No API design

synergy_hints:
  - api-expert: Strong for data-driven API endpoints
```

### Evolution Chains

Merged agents can merge again:

```
Generation 1: api-expert + db-expert → merged-api-db
Generation 2: merged-api-db + cache-expert → merged-api-db-cache
Generation 3: merged-api-db-cache + auth-expert → ...
```

Strong combinations survive through repeated use. Unused agents naturally fade.

## Documentation

- [Concept Deep Dive](docs/concept.md) - Philosophy and design principles
- [Quick Start Guide](docs/quickstart.md) - Step-by-step setup
- [Advanced Usage](docs/advanced.md) - Evolution strategies and optimization

## License

MIT License - Use freely, modify freely, share freely.

## Author

Created by [@shintaro_sprech](https://x.com/shintaro_sprech)

## Contributing

Ideas and improvements welcome! This is an evolving concept.
