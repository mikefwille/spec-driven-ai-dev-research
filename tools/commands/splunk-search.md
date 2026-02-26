# Splunk Search Generator

You are tasked with generating Splunk search links based on log statements found in the current branch's changes.

## What This Does

Scans the git diff (current branch vs main) for log statements, extracts the log message text, and generates clickable Splunk search URLs so developers can observe their changes in staging or production.

## Steps

### 1. Identify the branch and diff

Run `git diff main...HEAD` to get all changes on the current branch. If `main` doesn't work, try `master`.

### 2. Find log statements in the diff

Look through added/modified lines (lines starting with `+`) for common log patterns:

- `logger.error(...)`, `logger.warn(...)`, `logger.info(...)`, `logger.debug(...)`
- `console.log(...)`, `console.error(...)`, `console.warn(...)`
- `log.error(...)`, `log.warn(...)`, `log.info(...)`
- Any other logging pattern visible in the codebase

Extract the human-readable message string from each log statement. For template literals or string concatenation, extract the static portions.

Examples:

- `logger.error("Store initialization failed")` -> `Store initialization failed`
- `logger.info('Retry succeeded after previous failure')` -> `Retry succeeded after previous failure`
- `logger.warn(\`Failed to connect to ${service}\`)`->`Failed to connect to`

### 3. Also suggest log statements if useful

If the diff contains error handling (try/catch, .catch, error callbacks) that does NOT have logging, mention it briefly:

```
Heads up: found error handling without log statements at:
- src/services/admin.ts:45 (catch block, no logging)
```

Keep this short. Don't be pushy about it.

### 4. Build Splunk search URLs

**Splunk base URL**: `https://marq.splunkcloud.com/en-US/app/search/search`

**Indexes**:

- Staging: `index="final-staging" service="worqshop-app"`
- Production: `index="final-prod" "worqshop-app"`

**URL construction**:
For each log message, URL-encode this query:

```
search index="final-staging" service="worqshop-app" "log message text"
```

The full URL pattern:

```
https://marq.splunkcloud.com/en-US/app/search/search?q=search%20index%3D%22final-staging%22%20service%3D%22worqshop-app%22%20%22log%20message%20text%22&earliest=-4h%40h&latest=now
```

Use `earliest=-4h@h&latest=now` as the default time range (last 4 hours).

### 5. Present the output

Format the results like this:

```markdown
## Splunk Searches for branch `<branch-name>`

### Log Statements Found

| File                     | Level | Message                                |
| ------------------------ | ----- | -------------------------------------- |
| src/services/store.ts:23 | error | Store initialization failed            |
| src/services/store.ts:40 | info  | Retry succeeded after previous failure |

### Staging Links

- **Store initialization failed** (error)
  [Open in Splunk](<staging URL>)

- **Retry succeeded after previous failure** (info)
  [Open in Splunk](<staging URL>)

### Combined Search (all log messages)

[Open in Splunk](<URL with all messages joined by OR>)

### Production Links

<same format but with prod index>
```

For the combined search, join messages with OR:

```
search index="final-staging" service="worqshop-app" ("Store initialization failed" OR "Retry succeeded after previous failure")
```

## Notes

- Keep it simple. The goal is: developer deploys to staging, clicks a link, sees their logs.
- If there are no log statements in the diff, say so and suggest where logging might be useful based on the changes.
- If the diff is large, focus on the most meaningful log statements (errors and warnings first, then info).
- The service name `worqshop-app` is specific to the BrandWagon/digital-editor team. If this command is used in a different repo, the developer should update the service name.
