# Advanced Usage

Strategies for optimizing your agent ecosystem.

## Evolution Strategies

### Encouraging Strong Mergers

When you notice certain agent combinations working well together, you can hint this in skill sheets:

```yaml
# api-expert.yaml
synergy_hints:
  - db-expert: Excellent for data-driven endpoints
  - cache-expert: Essential for high-traffic APIs
  - auth-expert: Required for protected routes
```

These hints guide the orchestrator toward proven combinations.

### Permanent Mergers

If a merged agent is selected repeatedly, consider making it permanent:

1. Review the merged agent's skill sheet
2. Refine its capabilities based on observed usage
3. Update synergy_hints for future combinations
4. (Optional) Deprecate parent agents if merger fully supersedes them

### Controlled Specialization

Sometimes you want to prevent over-generalization. Add clear constraints:

```yaml
# db-expert.yaml
constraints:
  - No application logic
  - No API design
  - Query optimization only, not business rules
```

Clear boundaries prevent "god agents" that try to do everything poorly.

## Ecosystem Maintenance

### Identifying Dead Agents

Agents that haven't been selected in many tasks may be:
- Obsolete (superseded by mergers)
- Too specialized (narrow use case)
- Poorly defined (low coverage scores)

Review periodically and consider:
- Merging into a broader agent
- Refining capabilities for better matching
- Deprecating if truly unused

### Preventing Fragmentation

Too many tiny agents creates overhead. Watch for:
- Agents with only 1-2 capabilities
- High overlap between agents
- Frequent "create new" decisions for similar domains

Solution: Encourage merging by adding synergy hints.

### Capability Gap Analysis

Track "create new" events. Patterns reveal:
- Domains you work in frequently
- Missing coverage in your ecosystem
- Opportunities for proactive agent creation

## Advanced Skill Sheet Patterns

### Versioned Evolution

Track agent evolution through versions:

```yaml
name: api-expert
version: "2.0"  # Upgraded from 1.0
changelog:
  - "2.0: Added GraphQL support"
  - "1.0: Initial REST focus"
```

### Conditional Capabilities

Some capabilities only apply in context:

```yaml
capabilities:
  - REST API design
  - GraphQL (when project uses Apollo)
  - gRPC (when project uses Protocol Buffers)
```

### Inheritance Chains

For merged agents, document the full lineage:

```yaml
name: merged-fullstack-v2
parent_agents: [merged-frontend-backend, devops-expert]
lineage:
  - Generation 1: frontend-expert, backend-expert
  - Generation 2: merged-frontend-backend
  - Generation 3: merged-fullstack-v2 (current)
```

## Orchestrator Customization

### Adjusting Thresholds

The default decision matrix:
- 90%+ → Use existing
- 60-90% → Merge
- <60% → Create new

You can adjust these in `orchestrator.md` based on your preferences:

**Conservative (prefer existing):**
```
85%+ → Use existing
50-85% → Merge
<50% → Create new
```

**Aggressive (prefer specialization):**
```
95%+ → Use existing
70-95% → Merge
<70% → Create new
```

### Adding Decision Factors

Beyond coverage, consider:

```markdown
### Additional Factors

- **Task criticality**: High-stakes tasks prefer proven agents
- **Time constraints**: Urgent tasks use existing when possible
- **Learning mode**: Experimental tasks encourage new agents
```

## Multi-Project Ecosystems

### Shared Core Agents

Create a repository of proven agents:

```
shared-agents/
├── db-expert.yaml
├── api-expert.yaml
├── auth-expert.yaml
└── testing-expert.yaml
```

Import these as starting points for new projects.

### Project-Specific Adaptations

Start with shared agents, let them evolve per-project:

```yaml
# Project A's db-expert might evolve differently than Project B's
name: db-expert
version: "1.0-projectA"
parent_agents: [shared/db-expert]
```

### Cross-Pollination

Strong mergers from one project might benefit others:

1. Export successful merged agent
2. Add to shared repository
3. Import into other projects
4. Let it continue evolving

## Debugging the Orchestrator

### Verbose Mode

Add to orchestrator.md for detailed reasoning:

```markdown
### Debug Output

When making decisions, always explain:
1. Task capability requirements extracted
2. Each agent's coverage score
3. Why the final decision was made
4. For merges: what was combined and why
```

### Coverage Breakdown

Request detailed scoring:

```markdown
### Coverage Report Format

Agent: [name]
- Capability match: X/Y (Z%)
- Constraint violations: [list]
- Synergy bonus: +N% from [hint]
- Final score: W%
```

## Future Possibilities

### Self-Improving Orchestrator

The orchestrator could learn from outcomes:
- Track task success rates per agent
- Adjust coverage calculations based on results
- Recommend agent improvements

### Agent Retirement

Automated deprecation:
- Flag agents unused for N tasks
- Suggest merger or removal
- Archive rather than delete (reversible)

### Ecosystem Analytics

Dashboard showing:
- Agent usage frequency
- Evolution tree visualization
- Coverage gaps
- Merge success rates
