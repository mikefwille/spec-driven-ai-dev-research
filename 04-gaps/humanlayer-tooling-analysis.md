# HumanLayer Tooling Analysis

Analysis of unique mechanisms in the HumanLayer repo that would need equivalents when adapting this tooling for another codebase (e.g., digital-editor).

---

## Unique Mechanisms to Replicate

### 1. SQLite Session Database

**What it is:** A per-branch SQLite database that stores conversation history, sessions, approvals, and events.

**Location:** `~/.humanlayer/daemon-{BRANCH_NAME}.db`

**Why it matters:** Enables persistence across sessions, debugging conversation history, and tracking approvals. Commands like `debug.md` query this database to understand what happened.

**Tables include:**
- `sessions` - Active/past sessions
- `conversation_events` - Full conversation history
- `approvals` - Human approval records

**To replicate:** You'd need a similar persistence layer - could be SQLite, a JSON file, or another store that tracks session state per branch/worktree.

---

### 2. Thoughts System (Knowledge Base)

**What it is:** A directory-based knowledge system that builds up as you use the custom commands. Not a separate tool to build - it's markdown files in a specific structure that agents read/write.

**Directory Structure:**
```
thoughts/
├── shared/              # Team-shared documents
│   ├── research/        # Research docs (YYYY-MM-DD_topic.md)
│   ├── plans/           # Implementation plans (feature-name.md)
│   ├── tickets/         # Ticket docs (eng_XXXX.md)
│   └── prs/             # PR descriptions
├── {username}/          # Personal notes (e.g., allison/)
│   ├── tickets/
│   └── notes/
├── global/              # Cross-repository thoughts
└── searchable/          # READ-ONLY aggregated search index
```

**Path Correction Rule (Critical):**
When searching in `thoughts/searchable/`, you must correct paths by removing `searchable/`:
- `thoughts/searchable/shared/research/api.md` → `thoughts/shared/research/api.md`
- `thoughts/searchable/allison/tickets/eng_123.md` → `thoughts/allison/tickets/eng_123.md`

The `searchable/` directory is a read-only index. Edits go to the actual paths.

**CLI Commands (HumanLayer-specific):**
```bash
humanlayer thoughts init --directory humanlayer  # Initialize
humanlayer thoughts sync                          # Create searchable index
```

**Naming Conventions:**
- Tickets: `eng_XXXX.md`
- Research: `YYYY-MM-DD_topic.md`
- Plans: `feature-name.md`

**Referenced in:**
- `repo/tools/hack/create_worktree.sh:123` - Initializes thoughts on worktree creation
- `repo/tools/hack/dex/flow-tmux.sh` - Syncs thoughts during workflow
- `repo/tools/agents/thoughts-analyzer.md` - Deep analysis of thought docs
- `repo/tools/agents/thoughts-locator.md` - Finding relevant thought docs

**To replicate:**
1. Create the directory structure above
2. The agents/commands will populate it during usage
3. For the `searchable/` index, you could:
	 - Use symlinks
	 - Use a simple script that copies/aggregates files
	 - Skip it entirely and just search the source dirs

---

### 3. Worktree-Based Development

**What it is:** Each feature/ticket gets an isolated git worktree at `~/wt/[repo]/[name]/`.

**Script:** `repo/tools/hack/create_worktree.sh`

**Why it matters:**
- Isolated dependencies per feature
- Separate database per branch (daemon-{BRANCH_NAME}.db)
- Parallel development without conflicts
- Auto-generated human-readable names (e.g., `swift_fix_0930`)

**What the script does:**
1. Creates worktree at `~/wt/humanlayer/[name]`
2. Copies `.claude/` directory for config
3. Runs `make setup`
4. Initializes thoughts system
5. Integrates with Linear via `LINEAR_API_KEY`

**To replicate:** Adapt the script to your repo structure. The core concept (isolated worktrees with auto-setup) is portable, but paths and setup commands need customization.

---

### 4. Daemon Architecture (hld)

**What it is:** A background daemon that coordinates between Claude Code, TUI, desktop app (WUI), and cloud API.

**Architecture:**
```
Claude Code → MCP Protocol → hlyr → JSON-RPC → hld → HumanLayer Cloud API
                                         ↑         ↑
                                    TUI ─┘         └─ WUI (CodeLayer Desktop)
```

**Components:**
- `hld` - Go daemon (approval coordinator)
- `hlyr` - TypeScript CLI with MCP server
- Socket at `~/.humanlayer/daemon.sock`

**Why it matters:** Provides human-in-the-loop approvals before Claude executes certain actions. The database and logs tie into this.

**To replicate:** This is the most complex piece. For simpler use cases, you might not need this - just the session persistence and knowledge base.

---

### 5. Port Allocation System

**What it is:** Dynamic port allocation to prevent conflicts when running multiple worktrees in parallel.

**Script:** `repo/tools/hack/port-utils.sh`

**Functions:**
```bash
is_port_available()        # Check if port is free
find_available_port()      # Daemon ports: 20000-29999
find_available_vite_port() # Dev server ports: 10000-19999
extract_ticket_number()    # Parse ticket IDs for port derivation
```

**Why it matters:** When you have multiple worktrees running dev servers and daemons, you need non-conflicting ports.

**To replicate:** Copy/adapt the port-utils.sh script. The concept is generic.

---

### 6. Linear Integration

**What it is:** Deep integration with Linear issue tracker for ticket-driven development.

**Components:**
- `repo/tools/commands/linear.md` - Main Linear command
- `repo/tools/hack/linear/linear-cli.ts` - TypeScript CLI for Linear API
- `repo/tools/commands/ralph_*.md` - Ralph workflow (research → plan → implement tied to Linear tickets)

**Environment:** Requires `LINEAR_API_KEY`

**Why it matters:** Commands reference ticket IDs, update statuses, pull ticket context for planning.

**To replicate:** If you use Linear, adapt these. If using GitHub Issues, Jira, etc., you'd build equivalent integrations.

---

### 7. Logging System

**What it is:** Structured logs for debugging MCP communication and daemon activity.

**Locations:**
- `~/.humanlayer/logs/mcp-claude-approvals-*.log` - MCP protocol logs
- `~/.humanlayer/logs/wui-${BRANCH_NAME}/codelayer.log` - Desktop app logs
- `~/.humanlayer/logs/daemon-*.log` - Daemon logs

**Log format:** First line shows `[timestamp] starting [service] in [directory]`

**Referenced in:** `repo/tools/commands/debug.md` - Reads these logs for troubleshooting

**Hardcoded paths in debug.md:**
```bash
# These paths are baked into the debug command
~/.humanlayer/logs/daemon-*.log
~/.humanlayer/logs/wui-*.log
~/.humanlayer/daemon.db           # Note: simplified, actual is daemon-{BRANCH}.db
~/.humanlayer/daemon.sock
```

**To replicate:** Set up a logging directory structure. The debug command shows how to query logs effectively.

---

### 8. TODO Annotation System

**What it is:** Priority-based TODO annotations for tracking work.

**Convention:**
- `TODO(0)`: Critical - never merge
- `TODO(1)`: High - architectural flaws, major bugs
- `TODO(2)`: Medium - minor bugs, missing features
- `TODO(3)`: Low - polish, tests, documentation
- `TODO(4)`: Questions/investigations needed
- `PERF`: Performance optimization opportunities

**Referenced in:** `repo/00-sources/repos/humanlayer/CLAUDE.md`

**To replicate:** Adopt the same annotation system or customize priorities for your workflow.

---

## What's Portable (Works Anywhere)

These don't have HumanLayer-specific dependencies:

### Agents (Fully Portable)
| File | Purpose |
|------|---------|
| `repo/tools/agents/codebase-analyzer.md` | Deep code analysis |
| `repo/tools/agents/codebase-locator.md` | Find code by feature |
| `repo/tools/agents/codebase-pattern-finder.md` | Find patterns |
| `repo/tools/agents/web-search-researcher.md` | Web research |

### Commands (Mostly Portable)
| File | Portability |
|------|-------------|
| `repo/tools/commands/create_plan.md` | Portable - generic planning |
| `repo/tools/commands/create_plan_generic.md` | Portable |
| `repo/tools/commands/implement_plan.md` | Portable |
| `repo/tools/commands/commit.md` | Portable - standard git |
| `repo/tools/commands/describe_pr.md` | Portable - standard git |
| `repo/tools/commands/research_codebase.md` | Portable |
| `repo/tools/commands/local_review.md` | Portable |
| `repo/tools/commands/founder_mode.md` | Portable |

### Commands (Need Adaptation)
| File | Dependency |
|------|------------|
| `repo/tools/commands/linear.md` | Linear API |
| `repo/tools/commands/ralph_*.md` | Linear + thoughts system |
| `repo/tools/commands/debug.md` | Database + logs paths |
| `repo/tools/commands/create_worktree.md` | Worktree script |

### Hack Scripts (Adaptable)
| File | Adaptation Needed |
|------|-------------------|
| `repo/tools/hack/create_worktree.sh` | Paths, setup commands |
| `repo/tools/hack/port-utils.sh` | Minimal - generic utility |
| `repo/tools/hack/run_silent.sh` | Minimal - generic utility |
| `repo/tools/hack/setup_repo.sh` | Heavy - repo-specific deps |
| `repo/tools/hack/visualize.ts` | Minimal - generic JSON viz |

---

## Summary: What to Build for digital-editor

| Mechanism | Priority | Complexity | Notes |
|-----------|----------|------------|-------|
| Session persistence (DB or files) | High | Medium | Track conversation state |
| Thoughts directory structure | High | Low | Just create the folders - content builds via usage |
| Searchable index for thoughts | Medium | Low | Symlinks or copy script |
| Worktree setup script | Medium | Low | Adapt paths and setup steps |
| Port allocation | Low | Low | Only if running parallel instances |
| Logging structure | Medium | Low | Directory + debug command |
| TODO annotation system | Low | None | Just adopt the convention |
| Issue tracker integration | Optional | Medium | Only if needed |
| Daemon architecture | Low | High | Only for approval workflows |

**Key insight:** The thoughts system isn't something you build - it's a directory structure that fills up as you use the commands. You just need to:
1. Create the `thoughts/` directory structure
2. Update path references in agents/commands to match your project
3. Optionally create a `searchable/` aggregation mechanism

---

## File Paths Reference

### Primary Tooling
```
repo/tools/
├── agents/                              # Portable AI agents
│   ├── codebase-analyzer.md
│   ├── codebase-locator.md
│   ├── codebase-pattern-finder.md
│   ├── thoughts-analyzer.md             # Needs thoughts system
│   ├── thoughts-locator.md              # Needs thoughts system
│   └── web-search-researcher.md
├── commands/                            # Workflow commands
│   ├── create_plan.md
│   ├── create_plan_generic.md
│   ├── create_plan_nt.md
│   ├── implement_plan.md
│   ├── iterate_plan.md
│   ├── iterate_plan_nt.md
│   ├── validate_plan.md
│   ├── commit.md
│   ├── ci_commit.md
│   ├── describe_pr.md
│   ├── describe_pr_nt.md
│   ├── ci_describe_pr.md
│   ├── linear.md                        # Needs Linear
│   ├── ralph_plan.md                    # Needs Linear + thoughts
│   ├── ralph_impl.md                    # Needs Linear + thoughts
│   ├── ralph_research.md                # Needs Linear + thoughts
│   ├── research_codebase.md
│   ├── research_codebase_generic.md
│   ├── research_codebase_nt.md
│   ├── debug.md                         # Needs DB + logs
│   ├── local_review.md
│   ├── founder_mode.md
│   ├── create_handoff.md
│   ├── resume_handoff.md
│   ├── create_worktree.md               # Needs worktree script
│   ├── oneshot.md
│   └── oneshot_plan.md
├── codelayer/
│   └── installation.md                  # HumanLayer-specific
└── hack/
    ├── create_worktree.sh               # Adapt paths
    ├── cleanup_worktree.sh
    ├── setup_repo.sh                    # Heavy adaptation
    ├── run_silent.sh                    # Portable
    ├── port-utils.sh                    # Portable
    ├── install_platform_deps.sh
    ├── visualize.ts                     # Portable
    ├── spec_metadata.sh
    ├── generate_*.py/.sh                # Tauri-specific
    ├── rotate_icon_colors.py            # Tauri-specific
    ├── *.png                            # Assets
    ├── linear/                          # Needs Linear
    │   ├── linear-cli.ts
    │   ├── README.md
    │   ├── package.json
    │   ├── tsconfig.json
    │   └── ...
    └── dex/
        └── flow-tmux.sh                 # Needs Linear + thoughts
```

### HumanLayer Source Reference
```
repo/00-sources/repos/humanlayer/
├── CLAUDE.md                            # Their dev guidelines
├── .claude/                             # Their Claude config
│   ├── agents/
│   └── commands/
└── hack/                                # Same scripts as tools/hack
```

### Runtime Locations
```
~/.humanlayer/
├── daemon-{BRANCH_NAME}.db              # SQLite database
├── daemon.sock                          # Daemon socket
└── logs/
    ├── mcp-claude-approvals-*.log       # MCP logs
    └── wui-${BRANCH_NAME}/
        └── codelayer.log                # Desktop app logs

~/wt/[repo]/[name]/                      # Worktrees
```
