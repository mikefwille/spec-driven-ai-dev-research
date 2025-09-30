# AI Development Research

A curated collection of AI development methodologies, production implementations, and research materials focused on effective AI-assisted software development.

## Repository Structure

Research is organized by **maturity level** with supporting production tooling:

```
ai-dev/
├── tools/               # 🔧 Production implementation tools
│   ├── agents/          # Custom Claude Code agents
│   └── commands/        # Custom Claude Code commands
├── 00-sources/          # 📚 Raw, unprocessed material
│   ├── videos/          # YouTube transcripts and analysis
│   ├── articles/        # Saved articles and papers
│   └── repos/           # Code examples and implementations
├── 01-methodologies/    # 📘 Processed, actionable frameworks
├── 02-examples/         # 💡 Practical applications and workflows
├── 03-experiments/      # 🧪 Work-in-progress, testing ideas
└── templates/           # 📋 Reusable document templates
```

### Benefits of This Structure

- Files sort naturally by maturity stage
- Clear progression: source → methodology → example → experiment
- Easy to identify what's "ready to use" vs "still processing"
- Production tools separate from research materials

## Core Methodology: Spec-Driven AI Development

**The primary methodology documented in this repository**: A three-phase workflow (Research → Plan → Implement) for production-ready AI-assisted development in large codebases.

### Key Documents

- 📘 **[[spec-driven-ai-guide]]** - Complete implementation guide with production examples
- 📹 **[[context-engineering-boundary]]** - Source video analysis (Boundary)
- 📹 **[[context-engineering-core]]** - Core principles (YC Root Access)

### Key Concepts

- **40% context utilization threshold** - Keep agent context lean and focused
- **Three-phase workflow** - Research → Plan → Implement with human checkpoints
- **Sub-agent patterns** - Delegate context-heavy tasks to specialized agents
- **Specification-first** - The spec is the primary asset, not the code
- **Hierarchy of leverage** - Review research/plans, not just code

### Production Tools

The `tools/` directory contains production-ready Claude Code extensions:

**Agents** (`tools/agents/`):
- `codebase-locator.md` - Finds WHERE code lives (no analysis, pure search)
- `codebase-analyzer.md` - Analyzes HOW specific code works
- `codebase-pattern-finder.md` - Finds similar implementations to model after
- `thoughts-locator.md` - Discovers historical documentation
- `thoughts-analyzer.md` - Extracts insights from specific documents
- `web-search-researcher.md` - Conducts focused web research

**Commands** (`tools/commands/`):

*Core Three-Phase Workflow:*
- `/research_codebase` - Phase 1: Spawns parallel sub-agents for comprehensive research
- `/research_codebase_generic` - Generic version without HumanLayer-specific patterns
- `/create_plan` - Phase 2: Creates detailed, phased implementation plans
- `/create_plan_generic` - Generic version without HumanLayer-specific patterns
- `/implement_plan` - Phase 3: Executes plans with progress tracking
- `/validate_plan` - Reviews implementation plans for completeness

*Automated Workflow (Ralph):*
- `/ralph_research` - Fetches "research needed" tickets and spawns research
- `/ralph_plan` - Fetches "ready for spec" tickets and creates plans
- `/ralph_impl` - Fetches "ready for dev" tickets and implements

*Git & PR Management:*
- `/commit` - Creates well-formatted git commits with proper messages
- `/describe_pr` - Generates comprehensive PR descriptions from diffs
- `/create_worktree` - Manages git worktrees for parallel work

*Integration & Utilities:*
- `/linear` - Linear ticket management and workflow automation
- `/local_review` - Reviews code changes before committing
- `/debug` - Debugging assistance and troubleshooting
- `/founder_mode` - High-level strategic decision making

**Note**: Claude Code automatically discovers these tools when working in this directory.

## Document Metadata Standards

All research documents include structured YAML frontmatter:

```yaml
---
# Provenance
source_type: [youtube|article|synthesized|codebase_analysis]
source_url: [original source URL]
date_captured: YYYY-MM-DD

# Document Status
doc_type: [analysis|implementation_guide|example_workflow|experiment]
maturity: [raw|analyzed|production_validated]
implementation_status: [not_started|in_progress|battle_tested]

# Relationships
parent_doc: "[[parent-document]]"
child_docs:
  - "[[derivative-document]]"
related_concepts:
  - concept-tags

# Discoverability
primary_topic: "main focus area"
secondary_topics:
  - "related topics"
applicable_to:
  - "use cases"
---
```

## Quick Start

### Using the Methodology

1. **Start with the guide**: Read `01-methodologies/spec-driven-ai-guide.md`
2. **Explore the tools**: Check `tools/agents/` and `tools/commands/`
3. **See examples**: Review documents in `02-examples/`

### Processing New Research

1. **Identify source type** (video, article, repo)
2. **Use appropriate template** from `templates/`
3. **Save to correct folder** based on maturity
4. **Link relationships** in YAML frontmatter

### Finding Content

- **By maturity**: Browse folder structure (`00-sources/` → `01-methodologies/` → `02-examples/`)
- **By topic**: Use Obsidian search with metadata tags
- **By relationship**: Follow wiki-links in document frontmatter

## Implementation Status

### Production Ready ✅
- **Spec-Driven AI Development** - Complete methodology Claude subagents and Claude commands
- **Custom Claude Code Tools** - 6 agents + 17 commands ready to use

### In Progress 🔄
- Processing 47+ YouTube videos on AI development
- Creating additional workflow examples
- Developing templates for common scenarios

## Contributing

This is a personal research repository, but the methodologies and tools are designed to be generally applicable. Feel free to adapt for your own use.

## Sources & Attribution

Primary sources are credited in each document's YAML frontmatter. Key contributors:
- **Boundary** - Three-phase workflow methodology
- **HumanLayer** - Production implementation patterns in the following repo: https://github.com/humanlayer/humanlayer
- **YC Root Access** - Core context engineering principles

---

*Last Updated: 2025-09-30*
*Core Methodologies: 1*
*Production Tools: 6 agents, 17 commands*
*Documents: 6+ organized by maturity*
