# Quick Start: Spec-Driven AI Development Workflow

**One-page reference for developers using the advanced context engineering methodology.**

---

## The 3-Phase Flow

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  RESEARCH   │ --> │    PLAN     │ --> │  IMPLEMENT  │
└─────────────┘     └─────────────┘     └─────────────┘
      ↓                    ↓                    ↓
  🔍 REVIEW          🔍 REVIEW            ✅ TEST
```

---

## Visual Checkpoint Guide

```
START HERE
    ↓
┌─────────────────────────────────┐
│  /research_codebase             │
│  (spawns sub-agents)            │
└─────────────────────────────────┘
    ↓
    ↓ Creates: thoughts/shared/research/[date]-[desc].md
    ↓
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃  🔍 CHECKPOINT 1: REVIEW       ┃
┃  Read the research doc         ┃
┃  Check for misunderstandings   ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
    ↓
    ↓ Approved?
    ↓
┌─────────────────────────────────┐
│  /create_plan                   │
│  (spawns more sub-agents)       │
│  (interactive conversation)     │
└─────────────────────────────────┘
    ↓
    ↓ Creates: thoughts/shared/plans/[date]-[desc].md
    ↓
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃  🔍 CHECKPOINT 2: REVIEW       ┃
┃  Read the plan                 ┃
┃  Check steps & success criteria┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
    ↓
    ↓ Approved?
    ↓
┌─────────────────────────────────┐
│  /implement_plan [plan-path]    │
│  (executes phase by phase)      │
└─────────────────────────────────┘
    ↓
    ↓ Updates plan with ✓ checkboxes
    ↓
┌─────────────────────────────────┐
│  ✅ Test each phase             │
│  • Run automated checks         │
│  • Do manual testing            │
│  • Use /debug if issues arise   │
└─────────────────────────────────┘
    ↓
    ↓ All phases complete?
    ↓
┌─────────────────────────────────┐
│  /commit                        │
│  /describe_pr                   │
│  • Create commit & PR           │
└─────────────────────────────────┘
    ↓
   DONE ✅
```

---

## Phase 1: Research

### What You Do
```bash
/research_codebase
```

Then describe what you're trying to build/fix.

### What Happens
- Agent spawns parallel sub-agents to explore codebase
- Creates research document: `thoughts/shared/research/YYYY-MM-DD-description.md`
- (May ask to run `humanlayer thoughts sync` - approve if you want)

### 🔍 CHECKPOINT: Review the Research
**You must review before moving on!**

Open the research document and check:
- [ ] Core architecture correctly identified
- [ ] File locations make sense (spot-check 3-5)
- [ ] No major misunderstandings about how things work
- [ ] Dependencies and risks identified

**Why this matters:** Bad research → thousands of wrong code lines

### ❌ If Research Isn't Good Enough

**DON'T move forward with incomplete understanding!**

If you spot gaps, misunderstandings, or missing info:

```bash
# Continue in the SAME research session
# Just tell the agent what's wrong or missing
```

**Example conversation:**
```
You: "The research missed the authentication layer.
     How does auth work with the API?"

Agent: [spawns more sub-agents, investigates auth]
       [updates the research document with new findings]

You: [review updated research]
     "Perfect, now I understand. Let's move to planning."
```

**Keep iterating until:**
- You genuinely understand how things work
- No major "?" marks remain in your head
- You could explain the architecture to a teammate

**This is normal!** Research often takes 2-3 rounds of clarification.

### ✅ Next Step
Once research is solid → Move to Phase 2

---

## Phase 2: Planning

### What You Do
```bash
/create_plan
```

Or with the research doc:
```bash
/create_plan thoughts/shared/research/YYYY-MM-DD-description.md
```

### What Happens
- Agent reads research document
- Spawns MORE sub-agents for deeper investigation
- Asks you clarifying questions
- Presents design options
- Creates detailed plan: `thoughts/shared/plans/YYYY-MM-DD-description.md`

**This is interactive** - you'll have a conversation about the approach

### 🔍 CHECKPOINT: Review the Plan
**You must review before moving on!**

Open the plan document and check:
- [ ] Implementation steps are clear and specific
- [ ] Each phase has automated + manual success criteria
- [ ] No more than 5 files changed per step
- [ ] Testing strategy defined
- [ ] **No open questions remain**

**Why this matters:** Bad plan → hundreds of wrong code lines

### ❌ If Plan Isn't Complete

**DON'T start implementation with open questions!**

If you see vague steps, missing details, or unanswered questions:

```bash
# Continue the conversation with the planning agent
# Point out what's unclear or incomplete
```

**Example conversation:**
```
You: "Step 3 says 'update the database schema' but doesn't
     say what fields to add. Can you be more specific?"

Agent: [researches existing schema patterns]
       [updates plan with specific field names and types]

You: "The success criteria don't mention how to test
     the edge case where..."

Agent: [adds specific test cases to success criteria]
```

**The plan is ready when:**
- Every step is concrete enough to implement
- Success criteria are testable (not vague)
- You could hand it to another dev and they'd know what to do
- **All open questions are resolved**

**Take your time here!** A solid plan saves hours of rework.

### ✅ Next Step
Once plan is bulletproof → Move to Phase 3

---

## Phase 3: Implementation

### What You Do
```bash
/implement_plan thoughts/shared/plans/YYYY-MM-DD-description.md
```

### What Happens
- Agent reads the plan
- Implements phase by phase
- Checks off items in the plan as they're completed
- Runs success criteria after each phase
- Updates the plan document with progress

**You can watch progress** - open the plan file and see checkboxes getting marked

### ✅ Testing During Implementation
After each phase completes:
- [ ] Run automated checks (should be in plan's success criteria)
- [ ] Do manual testing (also in plan's success criteria)

### 🐛 Hit a Bug?
```bash
/debug thoughts/shared/plans/YYYY-MM-DD-description.md
```

Describe what's wrong - debug agent investigates logs/database/git without cluttering main context

---

## After Implementation

### Validate (Optional but Recommended)
```bash
/validate_plan thoughts/shared/plans/YYYY-MM-DD-description.md
```

Runs through all success criteria and creates validation report

### Commit Your Changes
```bash
/commit
```

Creates a well-formatted commit with proper message

### Create PR
```bash
/describe_pr
```

Generates comprehensive PR description from your changes

---

## Common Scenarios

### "I have a bug to fix"
```bash
/research_codebase
# Describe the bug and what needs fixing
```

### "I need to add a new feature"
```bash
/research_codebase
# Describe the feature requirements
```

### "I'm testing and something broke"
```bash
/debug thoughts/shared/plans/[your-plan].md
# Describe what went wrong
```

### "I want to pick up where I left off"
Look in `thoughts/shared/plans/` for your most recent plan file, check its progress, then:
```bash
/implement_plan thoughts/shared/plans/[your-plan].md
```

---

## Key Principles

1. **Never skip checkpoints** - They catch errors before they cascade
2. **The spec is the asset** - Plans live in `thoughts/`, code is generated from them
3. **40% context rule** - Agent stays under 40% context usage through sub-agents
4. **Human reviews specs, agent writes code** - Reverse the traditional model

---

## File Locations

- **Research**: `thoughts/shared/research/YYYY-MM-DD-description.md`
- **Plans**: `thoughts/shared/plans/YYYY-MM-DD-description.md`
- **Agents**: `.claude/agents/`
- **Commands**: `.claude/commands/`

---

## Troubleshooting

### "Research document wasn't created"
Check that `/research_codebase` command is in `.claude/commands/`

### "Agent seems confused or stuck"
Check context utilization - may need to start fresh session with plan document

### "Implementation differs from plan"
Normal! Plans are guides, not rigid scripts. Agent adapts to reality.

### "Don't know where checkpoints are"
After `/research_codebase` and after `/create_plan` - always review the generated document

---

**For more details:** See [PHILOSOPHY.md](PHILOSOPHY.md) and [advanced-context-engineering-guide.md](01-methodologies/advanced-context-engineering-guide.md)
