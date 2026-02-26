---
description: Prepare branch for pr creation, clean up, etc
---

You are performing a PR cleanup pass on the current branch. Execute each step in order and stop immediately if any step fails with unresolvable errors. Report a summary at the end.

## 1. Remove extraneous log statements

Search the `src/` directory for `console.log`, `console.debug`, `console.info`,
`console.warn`, and `console.error` calls that are clearly debug/development
artifacts (not part of intentional logging infrastructure like a logger utility).
Remove them. Do NOT remove:

- Calls inside a dedicated logger/logging module
- console.error in catch blocks that are part of intentional error handling
- Any statement with a comment explicitly marking it as intentional (e.g. "// keep")

The ESLint config already has `"no-console": "error"`, so anything the linter
flags after this step is something you missed.

## 2. Run formatter

Run `yarn format` and stage any changes it produces.

## 3. Run linter

Run `yarn lint` (ESLint with --fix). If there are errors that --fix cannot
resolve automatically, list them and fix them manually. Re-run until clean.

## 4. Run typechecker

Run `yarn typecheck`. Fix any TypeScript errors. Re-run until clean.

## 5. Run tests

Run each test suite sequentially. Use limited parallelism to avoid resource
contention:

```bash
yarn test:unit -- --pool-options.threads.maxThreads=5
yarn test:integration -- --pool-options.threads.maxThreads=5
yarn test:acceptance -- --pool-options.threads.maxThreads=5
PLAYWRIGHT_WORKERS=5 yarn test:e2e
```

If any test fails, investigate and fix. Re-run failing suites until green.

6. Audit commit history

Examine all commits on this branch that are not on main:

git log main..HEAD --oneline

6a. Flag NOMERGE

Search all commit messages AND changed files for the string "NOMERGE"
(case-insensitive):

git log main..HEAD --grep="NOMERGE" -i --oneline
git diff main..HEAD | grep -i "NOMERGE"

If any are found, stop and report them. These require human review before
the PR can proceed. List each occurrence with its commit hash, file, and line.

6b. Squash WIP commits

Identify commits whose messages start with "WIP", "wip", "work in progress",
"fixup", "squash", or "tmp" (case-insensitive). List them and suggest an
interactive rebase plan to squash them into their logical parent commits.
Do NOT perform the rebase automatically — present the plan for human approval.

6c. Verify Jira ticket IDs

Check that every commit message on this branch includes a Jira ticket ID
(pattern: one or more uppercase letters followed by a hyphen and one or more
digits, e.g. PROJ-123, WRK-456). Flag any commits that are missing a
ticket ID. Suggest amended messages where possible.

7. Summary

Print a summary with:

- Number of console statements removed
- Formatter/linter/typecheck status (pass/fail)
- Test results per suite (pass/fail, number of tests)
- NOMERGE occurrences found (blocking: yes/no)
- WIP commits that need squashing (count + suggested plan)
- Commits missing Jira ticket IDs (count + list)

```

```
