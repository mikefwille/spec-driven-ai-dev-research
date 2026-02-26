---
type: log
date: 2026-02-10
area: personal
category: ai-dev
---

# TDD Integration and `/write_tests` Command Design

Session designing a new `/write_tests` command to integrate TDD methodology into the existing research-plan-implement workflow for AI coding agents.

## Starting Point: Reading the Source Materials

Reviewed the following documents in the ai-dev repo:

### TDD Workshop Guide (`00-sources/tdd-workshop-guide.md`)
Jonathan Turner's TDD workshop at Marq (Jan 8, 2026). Key concepts absorbed:
- TDD is a design practice, not just "writing tests"
- Three Rules (Uncle Bob): no production code without a failing test, only enough test to fail, only enough code to pass
- Red-Green-Refactor cycle as the core rhythm
- Three activities: Obvious Implementation, Fake It, Triangulate
- "Do the simplest thing that could possibly work" (Ward Cunningham)
- Test ordering matters — simple to complex, building toward generalization
- Roman numeral kata as the canonical example of how tiny steps lead to elegant solutions (3-line recursive + mapping table vs nested loops)
- Key insight: duplication is a signal to refactor, and tests give you the safety net to do it

### Testing Skill (`tools/skills/testing/SKILL.md`)
Playwright/Vitest testing guide for a workflow builder app. Covers E2E patterns, page object model, test data factories, DRY principles. Oriented around writing tests for existing code, not test-first development.

### Systematic Debugging Skill (`tools/skills/systematic-debugging/SKILL.md`)
4-phase debugging methodology: Root Cause Investigation, Pattern Analysis, Hypothesis & Testing, Implementation. Iron law: no fixes without root cause investigation. Notable alignment with TDD — Phase 4 requires creating a failing test case before fixing. If 3+ fixes fail, stop and question the architecture.

### Implement Plan Command (`tools/commands/humanlayer/implement_plan.md`)
Executes pre-approved technical plans phase by phase. Reads plan fully, tracks progress with checkboxes, runs automated verification, pauses for human manual testing between phases. Supports resuming from existing checkmarks.

### Create Plan Commands (3 variants)
- `create_plan.md` — HumanLayer-specific with thoughts directory, specialized agents, `humanlayer thoughts sync`
- `create_plan_generic.md` — Same process stripped of HumanLayer tooling
- `create_plan_nt.md` — For projects without a thoughts directory

All three follow the same 5-step interactive process: context gathering with sub-agents, research & discovery, plan structure approval, detailed plan writing (phases with automated + manual success criteria), sync and review. Key principles: no open questions in final plan, be interactive (don't dump full plan), be skeptical (verify with code), success criteria always split into automated vs manual.

### Research Codebase Commands (3 variants)
- `research_codebase.md` — HumanLayer-specific
- `research_codebase_generic.md` — Stripped of HumanLayer tooling
- `research_codebase_nt.md` — No thoughts directory

Pure documentation commands — explicitly forbidden from suggesting changes, evaluating code, or recommending improvements. "Document what IS, not what SHOULD BE." Spawn parallel sub-agents (locator, analyzer, pattern-finder), synthesize findings into structured research documents with file:line references.

### Advanced Context Engineering Guide (`01-methodologies/advanced-context-engineering-guide.md`)
The overarching methodology document. This is where TDD is already specified but not fully operationalized. Key sections:
- Tests replace code review: "If the research accurately reflects how the system works, and the plan is architecturally sound, and the tests verify the right behaviors — then the implementation is trustworthy without reading every line."
- Test design happens during Planning, informed by Research
- Tests written first during Implementation (red-green-refactor)
- Plan template includes "Tests to Write" per step
- Two categories: automated verification (agent-runnable) and manual verification (human testing)
- Hierarchy of error prevention: bad line of research > bad part of plan > bad line of code

## The Core Tension Identified

The methodology doc says TDD should happen, but the actual commands don't implement it:

1. **create_plan** — The plan template has `Changes Required` with code blocks but NO `Tests to Write` field per step. The methodology doc's plan template has it, but the command template doesn't.

2. **implement_plan** — Says "implement phase, run verification, pause for manual testing" but doesn't enforce red-green-refactor within each phase. The methodology doc says "write the tests first, then the implementation code" but the command doesn't operationalize this.

3. **testing skill** — Oriented around writing tests for an existing app, not structured as TDD-first practice.

The methodology doc is ahead of the commands that implement it. The TDD intent is documented but not operationalized in the tooling.

## Discussion: Where Do Tests Come From?

### Do you provide tests as input to create_plan?
Initial assumption was maybe the user provides tests. But the methodology doc is clear: test design belongs as an OUTPUT of planning, not an input. Research has to happen first to reveal architecture, integration points, and edge cases.

### Correction: Research is a separate prior step
Research is its own command (`/research_codebase`) that runs before `/create_plan`. By the time you invoke create_plan, research is already done. So the question becomes: after research is done, should you provide tests to create_plan?

### So should tests be input to create_plan after research?
The answer is still no for tests as a finished artifact — if create_plan produces both the tests and the implementation plan, then tests aren't driving the design. They're just part of the plan alongside the code. The plan already determined both.

However, the user should provide **behavioral requirements** (what to test) to create_plan. The distinction: you say what behaviors you want verified, and the plan specifies the concrete tests + implementation. But this still means tests don't drive the design — the plan determines both simultaneously.

### Design path considered and rejected: behavioral requirements as input to create_plan
The idea: provide behavioral requirements to `/create_plan` ("I want to verify X handles Y"), and have it produce the entire plan including both test specifications AND step-by-step implementation code. The plan would contain everything — what tests to write and what code to write to pass them.

**Why this was rejected:** Even though the user specifies behaviors up front, if the same command produces both the tests and the implementation plan in one pass, the tests aren't genuinely driving the design. The plan determines both simultaneously. The implementation isn't shaped by failing tests — it's shaped by the planner's idea of what should exist. Tests become documentation of the plan's intent rather than constraints that force the implementation into a specific shape. This is test-first execution of a predetermined design, not TDD.

This rejection is what led directly to proposing `/write_tests` as a separate step — the tests need to exist as real code BEFORE the plan is written, so the plan is genuinely constrained by them.

## The Key Insight: A New Step Between Research and Planning

The proposal: insert a `/write_tests` command between research and create_plan:

1. `/research_codebase` — understand the system
2. `/write_tests` (new) — write actual failing tests (RED)
3. `/create_plan` — plan the implementation to make tests pass (plan for GREEN)
4. `/implement_plan` — execute the plan, tests go green (GREEN + REFACTOR)

This solves the tension cleanly:
- Tests are real executable code, not specifications in a document
- The plan is reverse-engineered from making tests pass
- The human review checkpoint between write_tests and create_plan is high-leverage — reviewing test design before any implementation planning
- For bug fixes: write a test that reproduces the bug, plan the fix
- For new features: tests define the API/interface before it exists

### Why not a `/create_test_plan` before `/write_tests`?
Considered adding a planning step before test writing. Decided against it — too many steps (research, create_test_plan, write_tests, create_plan, implement_plan = 5). The `/write_tests` command itself can be interactive (propose test structure, get feedback, then write), handling the planning internally. The user's behavioral input ("I want to verify X handles Y") is effectively the test plan.

## Discussion: What Does the User Provide to `/write_tests`?

Not test code. Not test specifications. The user provides:
- The research document (mandatory — tells the command about frameworks, patterns, directory structure, existing interfaces)
- Behavioral requirements — what the system should do ("I want to verify that X handles Y correctly")
- Optionally: a ticket or bug report

The command translates behaviors into concrete test code, informed by what research discovered about the system.

## Discussion: Does Pre-Specifying Implementation Alongside Tests Conflict with TDD?

### The tension with create_plan as it exists today
If `/create_plan` specifies both exact tests AND exact implementation code per step, tests aren't driving design — the plan determined both. The TDD part becomes mechanical: write pre-planned test, write pre-planned code. That's test-first execution of a predetermined design, not TDD.

### Two possible approaches identified
1. **Test-first execution of a predetermined design** — plan specifies both tests and code, agent writes tests first then pre-planned code. Safer, more predictable for AI agents.
2. **Actual TDD** — plan specifies only the tests/behaviors, agent writes them first and implements whatever makes them pass. More emergent, riskier with agents.

### The `/write_tests` command resolves this
By writing tests BEFORE the plan exists, the plan becomes genuinely constrained by the tests. `create_plan` receives failing tests and asks "what code makes these pass?" — not "what tests and code should we write?" The tests drive the plan, which drives the implementation. That's real TDD adapted for an AI agent workflow.

## Design Decisions for `/write_tests`

### Test progression ordering is a first-class concern (Step 3)
The order tests are written in directly maps to how `/create_plan` will phase the implementation. Following the TDD workshop's principles:
- Start with the simplest behavior (establishes the basic interface)
- Add tests that force generalization
- Save edge cases and error handling for last
- Group naturally by test file, ordered simple to complex within each

This mirrors the Roman numeral kata progression: test 0, test 1, test 5, test 10 (Fake It) -> test 2 (Triangulate/generalize) -> test 4, 9 (subtractive/edge cases).

### RED state verification is mandatory (Step 5)
Not just "do tests fail" but "do they fail for the expected reason." A test that fails because of a typo or wrong import path is not useful — it's a broken test. A test that fails because the function doesn't exist or returns the wrong value is defining behavior. This distinction matters because broken tests contaminate the planning phase.

### Tests define behavior, not implementation
Tests should target public interfaces (function signatures, API endpoints, component rendering). They should NOT test internal implementation details, because that would pre-determine the internal design and defeat the purpose of letting tests drive it. Exception: if research shows a specific internal pattern must be followed for architectural reasons.

### Stubs are allowed but constrained
Some test frameworks won't run at all if imports fail (e.g., TypeScript with strict module resolution). Minimal stubs — empty files with type exports only, NO logic — let tests execute and fail on assertions rather than on module resolution. The stub exists solely so the test runner can import. The test still fails on the actual behavior assertion. "Stubs are not implementation" is called out as an explicit guideline.

### Scope control prevents over-testing
Writing a comprehensive test suite upfront fights the emergent nature of TDD. The command should write enough tests to define the current scope of work. The user can run `/write_tests` again for additional phases. This keeps the incremental discovery that makes TDD valuable.

### The test summary document is the handoff artifact
The tests themselves are code in the repo. The summary document explains:
- What behaviors are being tested and why
- What each test expects (plain language)
- Expected failure mode for each test
- Interfaces defined by tests that don't exist yet (critical for create_plan)
- Suggested implementation order based on test progression

This document, alongside the research document, becomes the input to `/create_plan`.

### Interactive design, same as other commands
Following the pattern from create_plan: don't dump all tests at once. Present the test approach (what to test, at what level, in what order), get approval, then write. Allow course corrections before committing to test code.

## How This Changes the Full Workflow

### Before (current state)
1. `/research_codebase` -> research document
2. `/create_plan` (takes research + requirements) -> phased plan with code changes + verification commands
3. `/implement_plan` -> executes plan, runs verification after each phase

Tests appear as verification commands in the plan's success criteria. They're written during or after implementation. They verify but don't drive design.

### After (with `/write_tests`)
1. `/research_codebase` -> research document (what exists, how it works)
2. `/write_tests` (takes research + behavioral requirements) -> failing test files + test summary document
3. `/create_plan` (takes research + test summary + failing tests) -> phased plan where each phase makes specific tests pass
4. `/implement_plan` -> executes plan, tests go green phase by phase

Tests are real code written before the plan. The plan is shaped by what the tests require. Implementation is constrained by both.

## What Still Needs to Change

### create_plan needs updating
The create_plan command needs to accept test files and a test summary as input alongside the research document. Its plan template should reference specific tests per phase ("this phase makes tests X, Y, Z pass") rather than specifying its own tests. The plan becomes "what code makes these failing tests pass" rather than "what code and tests should we write."

### implement_plan may need adjustment
Currently it implements phases and runs verification. With the new workflow, verification is more concrete — run the specific tests that the phase targets and confirm they go green. The implement_plan command could be more explicit about the red-to-green transition per phase.

### testing skill may need rethinking
The testing skill is oriented around writing tests for an existing workflow builder app. With `/write_tests` as a command, the skill might become more of a reference for test patterns and conventions rather than a standalone test-writing guide.

## Artifacts Created

- `tools/commands/write_tests.md` — The new command, placed in root commands directory (not humanlayer subdirectory) since this is a generic command applicable to any project.
