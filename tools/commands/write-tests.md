---
description: Write failing tests based on research and behavioral requirements (TDD RED phase)
model: opus
---

# Write Tests

You are tasked with writing failing tests that define desired behavior, based on completed research and user-provided behavioral requirements. This is the RED phase of TDD — tests are written before any implementation code exists.

## CRITICAL: TESTS DEFINE BEHAVIOR, NOT IMPLEMENTATION

- Tests should target public interfaces, APIs, and user-observable behavior
- Tests should NOT prescribe internal implementation details
- Tests should fail because the behavior doesn't exist yet (or a bug exists), not because of test errors
- You are defining WHAT the system should do, not HOW it should do it internally

## Initial Response

When this command is invoked:

1. **Check if parameters were provided**:
   - If a research document path was provided, read it FULLY immediately
   - If behavioral requirements or a ticket were provided, read them FULLY
   - Begin the process

2. **If no parameters provided**, respond with:

```
I'll help you write failing tests that define the behavior you want to implement or fix.
These tests will drive the implementation plan.

Please provide:
1. The research document (from /research_codebase)
2. What behaviors you want to verify (bug to fix, feature to add, or behavioral requirements)
3. Any specific test preferences (level, framework, scope)

Tip: You can invoke with a research doc directly:
`/write_tests path/to/research/2025-01-08-auth-flow.md`
```

Then wait for the user's input.

## Process Steps

### Step 1: Absorb Research Context

1. **Read the research document FULLY**:
   - Use the Read tool WITHOUT limit/offset parameters
   - This is your primary source of truth about the system

2. **Test-relevant information**:
   - Test framework: Vitest with jsdom environment for unit tests
   - Test location: Tests live alongside source files (e.g., button.test.ts next to button.ts)
   - What test patterns exist? (factories, fixtures, helpers, page objects, mocks)
   - What test utilities/helpers are already available?
   - What conventions are followed? (naming, organization, assertion style)
   - What interfaces/APIs exist that tests could target?
   - Test commands: yarn test:unit for unit tests

3. **If research doesn't cover test infrastructure sufficiently**:
   - Spawn a focused sub-agent to find test configuration files (e.g., vitest.config.ts, playwright.config.ts, pytest.ini, jest.config.js)
   - Spawn a sub-agent to find 2-3 existing test files as pattern references
   - Read the results FULLY before proceeding

4. **Read any ticket, bug report, or additional context files FULLY**

### Step 2: Understand Behavioral Requirements

1. **Present what you found about test infrastructure**:

   ```
   From the research, I can see:
   - Test framework: [framework]
   - Test location: [directory structure]
   - Existing patterns: [factories, helpers, etc.]
   - Available test utilities: [what exists]

   Now, describe the behaviors you want to verify.
   ```

2. **Gather behavioral requirements from the user**:
   - For bug fixes: What's broken? What should happen instead?
   - For new features: What should the system do? What are the inputs and expected outputs?
   - For refactoring: What existing behavior must be preserved?

3. **Clarify scope**:
   - Are we defining tests for the entire feature or just the first phase?
   - What's the boundary of what we're testing? (one component, a full flow, an API endpoint)

### Step 3: Design Test Approach

The test ordering directly influences how `/create_plan` will phase the implementation. This step is a design decision, not clerical work.

1. **Classify each behavior by test level**:
   - **Unit**: Pure logic, single function/method, no external dependencies
   - **Integration**: Multiple components working together, API calls, database interactions
   - **E2E**: Full user flows through the actual interface

2. **Order tests following TDD progression**:
   - **Start with the simplest behavior** — this test establishes the basic interface (function signature, API shape, component structure). The equivalent of "0 returns empty string" in the Roman numeral kata.
   - **Add tests that force generalization** — each test should require the implementation to handle one more case, becoming slightly more general.
   - **Save edge cases and error handling for later** — these refine the implementation after the core behavior works.
   - **Group naturally** — if several tests belong in the same test file, keep them together but ordered simple to complex.

3. **Present the test design for approval**:

   ```
   Here's my proposed test approach:

   **Test Level**: [unit/integration/E2E — with reasoning]
   **Framework**: [from research]
   **Location**: [following existing conventions]

   **Tests in progression order:**

   1. [Test name] — [what behavior it verifies]
      Why first: [establishes basic interface / simplest case]

   2. [Test name] — [what behavior it verifies]
      Why next: [forces generalization from case 1]

   3. [Test name] — [what behavior it verifies]
      Why here: [handles additional case / edge condition]

   4. [Test name] — [what behavior it verifies]
      Why last: [error handling / complex edge case]

   This ordering is designed so each test builds on the previous,
   guiding implementation from simple to general.

   Does this progression make sense? Should I adjust the scope or ordering?
   ```

4. **Get user approval** before writing any test code

### Step 4: Write the Tests

After approval:

1. **Write actual test files** following the project's existing conventions:
   - Use the same framework, assertion style, and patterns found in research
   - Follow existing directory structure and naming conventions
   - Use existing test utilities, factories, and helpers where available
   - Create new helpers only if necessary for these specific tests

2. **For each test, ensure it**:
   - Has a clear, descriptive name stating the expected behavior
   - Tests ONE behavior per test case
   - Uses arrange-act-assert structure
   - Targets the public interface, not internal implementation
   - Is independent — no test depends on another test's execution

3. **For new features where interfaces don't exist yet**:
   - Write the test as if the interface exists — import the function, call the API, render the component
   - The test defines the interface contract: function name, parameters, return type, behavior
   - Note which imports/references will fail until implementation exists
   - If the framework requires it, create minimal stubs (empty files with type exports only) so tests can at least RUN and fail on assertions rather than on missing modules — but do NOT implement any logic in stubs

4. **For bug fixes**:
   - Write a test that reproduces the bug using existing interfaces
   - The test asserts the CORRECT behavior, not the current broken behavior
   - The test should pass once the bug is fixed

5. **Use table-driven tests** when multiple cases follow the same pattern:
   - Group related inputs/outputs into a data table
   - One parameterized test covers many cases
   - Makes adding future test cases trivial

### Step 5: Verify RED State

1. **Run the tests**:
   - Execute the test suite using the project's standard test command
   - All new tests should FAIL

2. **Verify tests fail for the expected reason**:
   - For new features: tests should fail because the function/endpoint/component doesn't exist, or because it returns wrong/default values
   - For bug fixes: tests should fail because the bug is still present
   - Tests should NOT fail because of:
     - Syntax errors in the test code
     - Wrong import paths for existing modules
     - Test framework misconfiguration
     - Typos or incorrect test setup

3. **Fix any tests that fail for the wrong reason**:
   - If a test has a typo, fix the typo
   - If an import path is wrong for an existing module, fix the path
   - Do NOT fix tests by changing the expected behavior — the assertions define what we want

4. **Report RED state**:

   ```
   All [N] tests written and verified:

   FAILING (expected):
   - [test name]: [why it fails — module not found / returns undefined / assertion mismatch]
   - [test name]: [why it fails]

   These tests define the behavior that /create_plan will plan the implementation for.
   ```

### Step 6: Write Test Summary Document

Create a summary document that serves as the handoff artifact for `/create_plan`.

**Filename**: Follow the project's documentation conventions. If no convention exists, place alongside the research document using the pattern `YYYY-MM-DD-test-summary-description.md`.

**Structure**:

```markdown
# Test Summary: [Feature/Bug Description]

## Overview

[What behaviors these tests define and why]

## Research Reference

- Research document: `[path to research doc]`
- Ticket/issue: `[path or reference if applicable]`

## Test Infrastructure

- **Framework**: [what's being used]
- **Test location**: [where tests were written]
- **Existing utilities used**: [helpers, factories, etc.]

## Tests Written

### [Test File 1]: `path/to/test/file`

#### Test Progression:

1. **[test name]** — [behavior defined]
   - Expected failure: [why it currently fails]
   - Interface defined: [function signature / API shape / component props]

2. **[test name]** — [behavior defined]
   - Expected failure: [why it currently fails]
   - Builds on: [how this extends test 1]

### [Test File 2]: `path/to/test/file`

[Same structure...]

## Interfaces Defined by Tests

[Functions, endpoints, components, or APIs that tests reference but don't exist yet.
This tells /create_plan exactly what needs to be built.]

- `functionName(param: Type): ReturnType` — defined in [test file:line]
- `POST /api/endpoint` — expected request/response in [test file:line]
- `<ComponentName prop={value} />` — expected rendering in [test file:line]

## Suggested Implementation Order

[Based on test progression, the order in which behaviors should be implemented.
This maps directly to implementation phases in /create_plan.]

1. Make [test 1] pass — requires [minimal what, not how]
2. Make [tests 2-3] pass — requires [extending to handle more cases]
3. Make [tests 4-5] pass — requires [edge case handling]
```

### Step 7: Present and Confirm

1. **Present summary to user**:

   ```
   Tests written and verified (RED state confirmed):

   Files created:
   - [test file paths]

   Summary document:
   - [summary doc path]

   Next step: Provide both the research document and this test summary
   to /create_plan so it can plan the implementation to make these tests pass.

   Would you like to adjust any tests before proceeding to planning?
   ```

2. **Handle adjustments**:
   - If the user wants to modify tests, update them and re-verify RED state
   - If scope needs to change, return to Step 3 and redesign

## Important Guidelines

1. **Tests Define Behavior, Not Implementation**:
   - Test the WHAT, not the HOW
   - Target public interfaces, not internal methods
   - If you find yourself testing implementation details, you're constraining `/create_plan` unnecessarily
   - Exception: if research shows a specific internal pattern must be followed for architectural reasons

2. **Follow Existing Conventions**:
   - The project already has test patterns — use them
   - Don't introduce new test libraries or patterns unless nothing exists
   - Match naming, file structure, and assertion style to what research found

3. **Progression Matters**:
   - Test ordering is a design decision — it shapes implementation phases
   - Simple -> general -> edge cases -> errors
   - Each test should be meaningful on its own, not just a stepping stone

4. **Scope Control**:
   - Write enough tests to define the current scope of work
   - Don't try to test every possible future case
   - You can always run `/write_tests` again for additional phases
   - Over-testing upfront fights the emergent nature of TDD

5. **RED Means RED**:
   - Every new test must fail before handing off to `/create_plan`
   - Tests that pass already are not defining new behavior — they test existing behavior (only useful as regression protection, flag this to the user)
   - Verify failure reasons, not just failure status

6. **File Reading**:
   - Always read mentioned files FULLY (no limit/offset) before proceeding
   - Read the research document completely — it contains the system knowledge you need
   - **CRITICAL**: Read files yourself in the main context before spawning any sub-tasks

7. **Be Interactive**:
   - Don't write all tests in one shot without user input
   - Get approval on test approach and progression before writing code
   - Allow course corrections on scope and ordering

8. **Stubs Are Not Implementation**:
   - If you create stubs for missing modules, they contain ONLY empty exports with correct type signatures
   - No logic, no default return values, no placeholder implementations
   - The stub exists solely so the test runner can import and execute — the test should still FAIL on assertions
