---
description: Review PR for code quality, bugs, and security, then post review to GitHub
---

# PR Review Command

Review pull request #{{arg1}} using local git for efficient diff analysis. Analyze code quality, bugs, security, and post comprehensive review to GitHub.

## Steps to follow:

1. **Fetch PR metadata:** Use `mcp__github__pull_request_read` with method='get' to fetch PR details (owner: marqHQ, repo: worqshop, pullNumber: {{arg1}})
   - Save PR title, description, author, status
   - Note base branch (usually `main`) and head SHA
   - Note the number of `changed_files`

2. **Ensure we have the PR commits locally:**
   - Get owner, repo, and head SHA from PR metadata (from step 1)
   - Check if we have the commit: `git cat-file -e <head_sha>^{commit}`
   - If not found, fetch from remote:
     - Detect remote with: `git remote -v | grep "github.com[:/]<owner>/<repo>"`
     - Extract the remote name from the output (first column)
     - Fetch using that remote name: `git fetch <remote_name>`
   - Get base commit with: `git merge-base main <head_sha>`
   - Save the base commit SHA for use in subsequent diff commands

3. **Get file list using git:**
   - Get changed files with status: `git diff --name-status <base_sha>...<head_sha>`
   - Get file stats: `git diff --stat <base_sha>...<head_sha>`
   - Identify file types:
     - Lock files to note: `yarn.lock`, `package-lock.json`, `pnpm-lock.yaml`, `Gemfile.lock`, `composer.lock`, `Cargo.lock`, `go.sum`
     - Build artifacts to note: `*.min.js`, `*.min.css`, files in `dist/*`, `build/*`
     - Config files: `*.config.*`, `*.json`, `.yml`, `.yaml`
     - Test files: `*.test.*`, `*.spec.*`, files in `testing/*`
     - Source files: Everything else

4. **Fetch additional context:**
   - Use `mcp__github__pull_request_read` method='get_status' to check CI/test status
   - Use `mcp__github__pull_request_read` method='get_reviews' to see existing reviews

5. **Analyze changes file-by-file:**
   - For each file (excluding lock files and build artifacts):
     - Get file diff: `git diff -U3 <base_sha>...<head_sha> -- <file_path>`
     - Analyze for:
       - **Code quality:** Best practices, readability, maintainability
       - **Potential bugs:** Logic errors, edge cases, null/undefined handling
       - **Security:** Input validation, authentication, authorization, SQL injection, XSS
       - **Performance:** Inefficient algorithms, unnecessary re-renders, N+1 queries
       - **Test coverage:** Are changes tested? Are tests sufficient?
       - **Documentation:** Inline comments for complex logic, README updates
       - **Breaking changes:** API changes, config changes, migration needs
     - Identify high-risk changes:
       - Database migrations
       - Authentication/authorization code
       - Security-sensitive operations
       - Core business logic
       - API contracts

6. **Provide comprehensive review feedback in terminal:**
   - **Summary:**
     - Total files changed: X files (+Y/-Z lines)
     - Files reviewed: [list by category]
     - Files skipped: [lock files, build artifacts]
   - **Key findings by file:**
     - Group by severity (critical, major, minor, suggestions)
     - For each finding:
       - File and line number
       - Issue description
       - Recommendation
       - Code snippet if helpful
   - **Overall assessment:**
     - Code quality score (subjective but helpful)
     - Test coverage assessment
     - Security considerations
     - Merge recommendation (approve, request changes, comment)

7. **Ask user for confirmation before posting review:**
   - Show what will be posted
   - Confirm review event type (APPROVE, REQUEST_CHANGES, COMMENT)

8. **If confirmed:**
   - Use `mcp__github__pull_request_review_write` with method='create' to create a pending review
   - Add comments for specific issues if needed
   - Use method='submit_pending' with appropriate event to post review to GitHub

**IMPORTANT:**

- Use local git commands for all diff operations (instant, no token cost)
- Use MCP tools (mcp**github**\*) only for PR metadata, existing reviews, status, and posting review
- Process files progressively - get diff for one file at a time to minimize context usage
- Skip lock files and build artifacts in review (just note them)
- Focus review time on:
  - Source code (highest priority)
  - Test files (verify coverage)
  - Config files (check for security issues)
  - Documentation (ensure it's updated)
- Provide summary at end:
  - ✅ Reviewed: X files (Y source, Z tests, W config)
  - 🔒 Skipped: [lock files, build artifacts]
  - 🔍 High-risk areas: [list if any]
  - 📊 CI Status: [status]
  - 💡 Ready to post: [APPROVE/REQUEST_CHANGES/COMMENT]
