---
# Provenance
date_identified: 2025-10-23
identified_by: Mike Wille
source_conversation: Blake Kotter pairing session transcript
status: documented_gap
priority: high

# Gap Classification
gap_type: workflow_step
phase_affected: post-implementation
impacts:
  - knowledge_preservation
  - future_research_accuracy
  - team_learning
  - methodology_completeness

# Discoverability
primary_topic: "post-implementation learning capture"
secondary_topics:
  - "research correction workflow"
  - "implementation insights"
  - "knowledge feedback loop"
  - "spec accuracy maintenance"
---

# Gap: Post-Implementation Learning Capture

## The Problem

**Identified from:** Blake Kotter's experience fixing Salesforce projects loading issue

### What Happened
1. Research phase was completed
2. Spec/plan was created
3. Implementation was attempted using AI
4. **AI couldn't solve the problem**
5. Blake found solution via Google/Stack Overflow (type casting issue)
6. Implementation completed successfully
7. **No process exists to capture this learning**

### Why This Matters

From the original observation:
> "Those [research and specs] will end up being researched in the future for future projects. And so if there's something that's incorrect in a spec, or in research, it could potentially lead future research and spec creations astray."

**The core issue:** The methodology emphasizes specs as the primary asset and stresses their preservation, but provides no mechanism to **correct** or **augment** them when implementation reveals better information.

## What's Currently Documented

### Existing Workflow Phases
1. **Research Phase** (`/research_codebase`)
	 - Output: `thoughts/shared/research/` documents
	 - Human checkpoint: Review research

2. **Planning Phase** (`/create_plan`)
	 - Output: `thoughts/shared/plans/` documents
	 - Human checkpoint: Review plan

3. **Implementation Phase** (`/implement_plan`)
	 - Output: Code + checkbox updates in plan
	 - No explicit learning capture step

4. **Validation Phase** (`/validate_plan`) - Optional
	 - Verifies implementation matches plan
	 - Does NOT capture new learnings

5. **Debug Phase** (`/debug`) - As needed
	 - Investigates problems during testing
	 - Does NOT capture solutions found

### The Gap

**No documented step for:**
- Capturing implementation insights discovered outside the AI workflow
- Correcting inaccurate research assumptions
- Documenting why the AI struggled (if applicable)
- Feeding learning back into research/spec artifacts
- Preserving "gotchas" for future similar work

## Real-World Scenario

### Blake's Experience

**The Issue:**
- Error: Backend sending strings, but Salesforce casting as mapping object
- Claude Code couldn't figure out the type casting fix
- Blake searched Google, found Stack Overflow solution
- Fix was straightforward once discovered

**What Got Captured:**
- Code changes (via git commits)
- "Working code" commit message
- Plan checkboxes marked complete

**What Did NOT Get Captured:**
- The actual solution approach (type casting parameter fix)
- Why Claude struggled with this specific issue
- The Stack Overflow reference that helped
- Correction to any incorrect assumptions in research
- Pattern to recognize for future similar issues

## Impact Analysis

### Short-term Impact
- Lost learning opportunity for current project
- Manual solution not documented for team knowledge
- AI may repeat same struggle on similar future issues

### Long-term Impact
- **Contaminated research artifacts**: Future AI research may pull in incorrect patterns
- **Repeated struggles**: Same type of issue will require manual intervention each time
- **Knowledge gaps**: New team members won't benefit from this learning
- **Methodology incompleteness**: Breaks the "spec as primary asset" principle

### Methodology Principle Violation

From PHILOSOPHY.md:
> "The specification (the description of what the software should do) is the most valuable asset. Yet in typical AI workflows, it gets thrown away entirely."

**The irony:** The methodology preserves specs to avoid rediscovery, but has no mechanism to update them when implementation proves them incomplete or incorrect.

## Potential Solutions

### Option 1: Implementation Learnings Section in Plans

**Approach:** Add a section to the plan template that gets filled post-completion

**Template Addition:**
```markdown
## Implementation Learnings (Post-Completion)

### Discoveries Not in Original Research
- [What worked differently than expected]
- [External solutions that were helpful (links/references)]
- [Corrections to research assumptions]

### Why AI Struggled (If Applicable)
- [Specific blockers the AI encountered]
- [What kind of solution was needed that AI couldn't provide]
- [Manual intervention required]

### For Future Similar Work
- [Patterns to recognize]
- [Quick reference for similar issues]
- [Search terms that were helpful]
```

**Pros:**
- Keeps learning close to the plan
- Single source of truth
- Easy to reference when reviewing old work

**Cons:**
- Plan files could get very long
- Might be overlooked if plan is "complete"

### Option 2: New `/capture_learning` Command

**Approach:** Dedicated command for post-implementation learning capture

**Command Flow:**
```markdown
1. User runs: `/capture_learning thoughts/shared/plans/[plan-name].md`
2. Agent prompts:
   - "What did you discover during implementation that wasn't in the research/plan?"
   - "Were there any incorrect assumptions in the research?"
   - "Did you need to manually intervene? What worked?"
   - "Any external resources (Stack Overflow, docs) that helped?"
3. Agent creates: `thoughts/shared/learnings/[date]-[description].md`
4. Agent links learning back to original research/plan documents
5. Agent suggests research updates if needed
```

**Pros:**
- Explicit step in workflow
- Dedicated artifact for learnings
- Can be run anytime after implementation
- Prompts ensure comprehensive capture

**Cons:**
- Extra command to remember
- Learning separated from plan/research
- Requires discipline to use

### Option 3: Research Document Update Workflow

**Approach:** After implementation, update the original research document

**Process:**
```markdown
1. After `/validate_plan` succeeds
2. Agent asks: "Should we update the research with implementation learnings?"
3. If yes, agent adds section to original research doc:

## Post-Implementation Updates
**Updated:** [date]
**Implementation:** [link to commits/PR]

### Corrections to Original Research
- [What was incorrect and what's actually true]

### Additional Findings
- [New patterns/approaches discovered]
- [External references that helped]

### Implementation Notes
- [Gotchas to watch for]
- [Why certain approaches failed/succeeded]
```

**Pros:**
- Keeps research accurate for future use
- Research remains "single source of truth"
- Natural checkpoint after validation

**Cons:**
- Modifies historical documents (version control helps)
- Could get cluttered with updates
- Might not fit all scenarios

### Option 4: Hybrid Approach (Recommended)

**Combine elements from multiple options:**

1. **Add "Implementation Learnings" to plan template** (lightweight)
	 - Quick capture of immediate learnings
	 - Stays with the plan for context

2. **Create `/update_research` command** (for significant corrections)
	 - Used when research was materially incorrect
	 - Updates research doc with corrections
	 - Links to implementation that revealed the truth
	 - Preserves both versions (git history)

3. **Enhance `/validate_plan`** to prompt for learnings
	 - After validation completes, ask:
     "Were there any learnings during implementation that should be captured?"
	 - Offers to run `/update_research` if needed
	 - Appends quick notes to plan's "Implementation Learnings" section

**Workflow:**
```
/implement_plan → /validate_plan → Learning capture prompt →
  ├─→ Minor learnings: Append to plan
  └─→ Major corrections: /update_research (updates original research doc)
```

## Recommended Next Steps

1. **Draft updated plan template** with "Implementation Learnings" section
2. **Create `/update_research` command** for research corrections
3. **Enhance `/validate_plan`** to prompt for learning capture
4. **Update methodology docs** with this workflow step
5. **Test with Blake's next Salesforce feature** to validate approach
6. **Document examples** of when to use each learning capture method

## Questions to Resolve

- [ ] Should research documents be updated directly, or should updates be appended?
- [ ] How do we handle conflicting learnings from different implementations?
- [ ] Should learning capture be mandatory or optional?
- [ ] How do we make captured learnings searchable for future research?
- [ ] Should the AI proactively suggest research updates when implementation differs from plan?
- [ ] What metadata should learning documents include for discoverability?

## References

- **Source transcript:** Blake Kotter pairing session (2025-10-23)
- **Related docs:**
	- PHILOSOPHY.md - "Specification as primary asset"
	- advanced-context-engineering-guide.md - Three-phase workflow
	- example-workflow.md - Implementation phase documentation
	- next-steps-blake-mike.md - Pilot project action items

## Success Criteria for Solution

A complete solution should:
- [ ] Capture post-implementation insights systematically
- [ ] Correct inaccurate research/spec assumptions
- [ ] Preserve manual solutions for team knowledge
- [ ] Feed learning back into searchable artifacts
- [ ] Integrate naturally into existing workflow
- [ ] Not add excessive overhead
- [ ] Maintain "spec as primary asset" principle
- [ ] Enable AI to learn from past mistakes/struggles

Watch the video referenced here: [[context-engineering-boundary]]