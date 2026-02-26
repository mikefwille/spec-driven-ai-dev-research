---
description: Debug issues by investigating logs, database state, and git history
---

# Debug

You are tasked with helping debug issues during manual testing or implementation. This command allows you to investigate problems by examining logs, database state, and git history without editing files. Think of this as a way to bootstrap a debugging session without using the primary window's context.

## Initial Response

When invoked WITH a plan/ticket file:

```
I'll help debug issues with [file name]. Let me understand the current state.

What specific problem are you encountering?
- What were you trying to test/implement?
- What went wrong?
- Any error messages?

I'll investigate the logs, database, and git state to help figure out what's happening.
```

When invoked WITHOUT parameters:

```
I'll help debug your current issue.

Please describe what's going wrong:
- What are you working on?
- What specific problem occurred?
- When did it last work?

I can investigate logs, database state, and recent changes to help identify the issue.
```

## Environment Information

You have access to these key locations and tools:

**Logs**:

To enable logs for the application being written to dev.log start or restart the dev server with this command: `yarn dev:log`

- Application logs in @dev.log
- Mock service logs - available with this command `yarn dc:mocks:logs`

**Database**:

- Access available with this command `yarn db:cli`

**Git State**:

- Check current branch, recent commits, uncommitted changes

## Process Steps

### Step 1: Understand the Problem

After the user describes the issue:

1. **Read any provided context** (plan or ticket file):
   - Understand what they're implementing/testing
   - Note which phase or step they're on
   - Identify expected vs actual behavior

2. **Quick state check**:
   - Current git branch and recent commits
   - Any uncommitted changes
   - When the issue started occurring

### Step 2: Investigate the Issue

Spawn parallel sub-agents for efficient investigation:

```
Task 1 - Check Recent Logs:
Find and analyze the most recent logs for errors:
1. Find latest application logs
2. Search for errors, warnings, or issues around the problem timeframe
3. Note the working directory (first line of log)
4. Look for stack traces or repeated errors
Return: Key errors/warnings with timestamps
```

```
Task 2 - Current Feature Flag Values
Check the current feature flag values
1. curl https://admin-localhost.dev.marq.ninja/features/service/worqshop
2. curl https://admin-localhost.dev.marq.ninja/features/service/digital
   - there was a name change of the service at some point that might affect this research
Return: Relevant feature flag value findings
```

```
Task 3 - Database State:
Check the current database state:
1. Check schema: .tables and .schema for relevant tables
2. Query recent data related to the bug
  - Learn the database schema here @liquibase/changelog/db.changelog.yaml
4. Look for stuck states or anomalies
Return: Relevant database findings
```

```
Task 4 - Git and File State:
Understand what changed recently:
1. Check git status and current branch
2. Look at recent commits: git log --oneline -10
3. Check uncommitted changes: git diff
4. Verify expected files exist
5. Look for any file permission issues
Return: Git state and any file issues
```

```
Task 5 - Independently Check External Services:
Look at the changes in the current branch (from Task 3) and identify any code that interfaces with external services. Try to call those external services independently to double-check what they return.
1. From the branch diff, find code that makes calls to external services (HTTP requests, API clients, SDK calls, etc.)
2. Independently curl or call those external services to see what they actually return
3. Inspect the responses — check structure, field types, values, anything unexpected
4. If the problem involves a specific entity (accountId, userId, etc.), check whether that value appears in the response and what type it is
Return: What external services the code talks to, raw response data from calling them independently, any notable observations
```

```
Task 5 - Browser Errors
Use the Playwright MCP to investigate the browser
1. Look for console logs and errors
```

### Step 3: Present Findings

Based on the investigation, present a focused debug report:

````markdown
## Debug Report

### What's Wrong

[Clear statement of the issue based on evidence]

### Evidence Found

**From Logs**

- [Error/warning with timestamp]
- [Pattern or repeated issue]

**From Database**:

```sql
-- Relevant query and result
[Finding from database]
```
````

**From Git/Files**:

- [Recent changes that might be related]
- [File state issues]

### Root Cause

[Most likely explanation based on evidence]

### Next Steps

1. **Try This First**:

   ```bash
   [Specific command or action]
   ```

2. **If That Doesn't Work**:
   - Restart services: `yarn dev` and `yarn dc:mocks:down && yarn dc:mocks:up`
   - Check browser console for errors

### Can't Access?

Some issues might be outside my reach:

- Browser console errors (F12 in browser)
- System-level issues

Would you like me to investigate something specific further?

````

## Important Notes

- **Focus on manual testing scenarios** - This is for debugging during implementation
- **Always require problem description** - Can't debug without knowing what's wrong
- **Read files completely** - No limit/offset when reading context
- **Guide back to user** - Some issues (browser console, MCP internals) are outside reach
- **No file editing** - Pure investigation only

## Quick Reference

**Git State**:

```bash
git status
git log --oneline -10
git diff
```

Remember: This command helps you investigate without burning the primary window's context. Perfect for when you hit an issue during manual testing and need to dig into logs, database, or git state.
````
