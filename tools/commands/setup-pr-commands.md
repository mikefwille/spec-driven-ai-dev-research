---
description: Create or update the four PR interaction commands for this repository
---

# Setup PR Commands

Idempotently create or update the four PR interaction commands (pr-context, pr-review, pr-respond, pr-changes) for this repository. All commands use local git for efficient diff analysis, dramatically reducing token usage and improving performance.

## Steps:

1. **Detect repository name:**
   - Run `git remote get-url github` to get the GitHub remote URL
   - Extract the repo name from the URL (e.g., `marqHQ/worqshop` → `worqshop`)
   - If no github remote exists, ask user for the repo name

2. **Define common configuration:**
   - Owner: `marqHQ`
   - Repo: (detected repo name)
   - **New architecture:** Use local git for all diffs, API only for metadata/comments/reviews
   - Git workflow:
     - Get owner, repo, and head SHA from PR metadata
     - Check if commit exists locally: `git cat-file -e <head_sha>^{commit}`
     - If not, detect remote and fetch: `REMOTE=$(git remote -v | grep "github.com[:/]<owner>/<repo>" | head -1 | awk '{print $1}'); git fetch $REMOTE`
     - Get base: `BASE=$(git merge-base main <head_sha>)`
     - Get diffs: `git diff $BASE...<head_sha>`
   - Files to skip in detailed analysis:
     - Lock files: `yarn.lock`, `package-lock.json`, `pnpm-lock.yaml`, `Gemfile.lock`, `composer.lock`, `Cargo.lock`, `go.sum`
     - Build artifacts: `*.min.js`, `*.min.css`, files in `dist/*`, `build/*`

3. **Create/update `.claude/commands/pr-context.md`:**
   - Purpose: Read-only PR context loading for discussion
   - Steps:
     1. Fetch PR metadata via API (title, description, author, status)
     2. Fetch PR branch locally via git
     3. Get file list and stats via git diff
     4. Fetch comments, reviews, and status via API
     5. Summarize PR context
     6. Be ready to answer questions (use git diff for specific files on demand)
   - Important notes:
     - Read-only - do not post anything
     - Use git for all diffs (instant, no tokens)
     - Use MCP tools only for metadata, comments, reviews, status
     - Summary format:
       - ✅ PR loaded: X files changed (+Y/-Z lines)
       - 💬 Discussion: X comments, Y reviews
       - 🔍 CI Status: [status]
       - 📍 Ready to discuss

4. **Create/update `.claude/commands/pr-review.md`:**
   - Purpose: Comprehensive PR review with analysis and GitHub posting
   - Steps:
     1. Fetch PR metadata and existing reviews via API
     2. Fetch PR branch locally via git
     3. Get file list via git diff, categorize files
     4. Analyze changes file-by-file (get diff for each file via git)
     5. Review for: code quality, bugs, security, performance, tests, docs, breaking changes
     6. Provide comprehensive feedback in terminal
     7. Ask user for confirmation
     8. If confirmed: Post review to GitHub via API
   - Important notes:
     - Use git for all diffs (process one file at a time)
     - Use MCP tools for metadata, existing reviews, status, and posting review
     - Skip lock files and build artifacts
     - Summary format:
       - ✅ Reviewed: X files (Y source, Z tests, W config)
       - 🔒 Skipped: [lock files, build artifacts]
       - 🔍 High-risk areas: [list if any]
       - 📊 CI Status: [status]
       - 💡 Ready to post: [APPROVE/REQUEST_CHANGES/COMMENT]

5. **Create/update `.claude/commands/pr-respond.md`:**
   - Purpose: Load PR review comments and implement requested changes
   - Steps:
     1. Fetch PR metadata and ALL review feedback via API (reviews, review_comments, comments)
     2. Fetch PR branch locally via git
     3. Get file list via git diff
     4. For files with review comments: get diff via git
     5. Analyze and categorize feedback (requested changes, questions, suggestions, blockers)
     6. For each: propose implementation approach
     7. Ask user for confirmation
     8. If confirmed: Post responses to GitHub via API
   - Important notes:
     - Use git for all diffs (only for files mentioned in reviews)
     - Use MCP tools for fetching comments and posting responses
     - Process by priority: blockers, requested changes, questions, suggestions
     - Summary format:
       - ✅ Loaded: X review comments across Y files
       - 📋 Action items: [list by priority]
       - 💬 Responses drafted: X comments
       - 🔧 Code changes needed: [list files]
       - 💡 Ready to post responses

6. **Create/update `.claude/commands/pr-changes.md`:**
   - Purpose: Detailed file-by-file analysis with code snippets, intelligently grouped
   - Steps:
     1. Fetch PR metadata via API (title, description, author, status)
     2. Fetch PR branch locally via git
     3. Get file list and stats via git diff, categorize files
     4. Analyze and group related files (configs, features, refactoring, dependencies, docs, CI)
     5. Process each group:
        - For each file: get diff via git, extract snippets, generate summary, discard diff
        - File summary: path, change type, description, key changes, code snippets, impact
     6. Present organized output with groups
     7. Final readiness check (high-risk, tests, docs, breaking changes, merge recommendation)
   - Important notes:
     - Read-only - do not post anything
     - Use git for all diffs (process one file at a time to minimize context)
     - Use MCP tools only for PR metadata
     - Skip lock files and build artifacts in detailed analysis
     - Extract snippets from git diff output (functions, logic changes, config, imports)
     - Summary format:
       - ✅ Successfully analyzed: X files in Y groups
       - 🔒 Auto-skipped: [lock files, build artifacts]
       - 💡 View full PR: [GitHub URL]

7. **Verify command creation:**
   - Check that all four `.md` files exist in `.claude/commands/`
   - Confirm repo name is correct in all files
   - Report success with list of created/updated commands

**Template structure for each command:**

Each command should follow this pattern:

- Header with command name and purpose (mention "uses local git for efficient diff analysis" where applicable)
- "## Steps to follow:" section with numbered steps
- **Git-based workflow:**
  1. Fetch PR metadata via API (lightweight)
  2. Fetch PR branch locally via git
  3. Use git diff commands for all diff operations
  4. Fetch comments/reviews via API (if needed)
  5. Process and analyze
  6. Post to GitHub via API (if not read-only)
- **IMPORTANT** section at bottom with:
  - Read-only status (or confirmation before posting)
  - Use git for diffs, API for metadata/comments/posting
  - Processing strategy (file-by-file, progressive, etc.)
  - Summary format specific to that command
- Use `mcp__github__pull_request_read` with `method` parameter for API calls
- Use `git` commands via Bash tool for all diff operations
- Repo references: `owner: marqHQ, repo: {detected_repo_name}, pullNumber: {{arg1}}`

**Key Architecture Benefits:**

- **Token savings:** 95% reduction (5-15k vs 50-150k tokens)
- **Speed:** 10-20x faster (git diffs are instant)
- **Reliability:** No API pagination failures
- **Scalability:** Works with PRs of any size
- **No caching needed:** Git is fast enough, no filesystem clutter

**Idempotency:**

- If files already exist, update them with latest template
- Preserve the repo name detection logic
- Show summary of changes if files already existed
