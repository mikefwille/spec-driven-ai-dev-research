---
description: Load complete PR context for discussion and understanding
---

# PR Context Command

Load complete context for pull request #{{arg1}} for discussion and understanding. Uses local git for efficient diff analysis.

## Steps to follow:

1. **Fetch PR metadata:** Use `mcp__github__pull_request_read` with method='get' to fetch PR details (owner: marqHQ, repo: worqshop, pullNumber: {{arg1}})
   - Save PR title, description, author, status
   - Note base branch (usually `main`) and head SHA
   - Note the number of `changed_files` for summary

2. **Ensure we have the PR commits locally:**
   - Get owner, repo, and head SHA from PR metadata (from step 1)
   - Check if we have the commit: `git cat-file -e <head_sha>^{commit}`
   - If not found, fetch from remote:
     - Detect remote with: `git remote -v | grep "github.com[:/]<owner>/<repo>"`
     - Extract the remote name from the output (first column)
     - Fetch using that remote name: `git fetch <remote_name>`
   - Get base commit with: `git merge-base main <head_sha>`
   - Save the base commit SHA for use in subsequent diff commands

3. **Get file list and stats using git:**
   - Get changed files: `git diff --name-status <base_sha>...<head_sha>`
   - Get file stats: `git diff --stat <base_sha>...<head_sha>`
   - This shows all changed files with their status (Added/Modified/Deleted/Renamed)

4. **Get diff overview:**
   - For summary purposes, get high-level diff info from git
   - Note files with significant changes
   - Identify lock files, config files, test files, source files
   - **Note:** Full diff is available via git but not loaded into context to save tokens
   - If specific file content needed for discussion: `git diff <base_sha>...<head_sha> -- <file_path>`

5. **Fetch discussion and review context:**
   - Use `mcp__github__pull_request_read` method='get_comments' for discussion comments
   - Use `mcp__github__pull_request_read` method='get_review_comments' for inline review comments
   - Use `mcp__github__pull_request_read` method='get_reviews' for all reviews and feedback
   - Use `mcp__github__pull_request_read` method='get_status' for CI/test status

6. **Summarize the complete PR context including:**
   - PR title, description, and author
   - Current status (open/closed/merged, mergeable state)
   - Total files changed and scope of changes
   - Key files modified (list most significant ones)
   - File types affected (frontend, backend, config, tests, docs)
   - Review feedback and discussions summary
   - Inline review comments (if any)
   - CI/test results
   - Any blockers or concerns mentioned

7. **Be ready to answer questions about the PR:**
   - What changed and why (use git diff for specific files as needed)
   - Review feedback and concerns
   - Testing status and coverage
   - Deployment readiness
   - Specific code changes (fetch file diffs on demand)

**IMPORTANT:**

- This is read-only - do not post anything to GitHub
- Use local git commands for all diff operations (instant, no token cost)
- Use MCP tools (mcp**github**\*) only for PR metadata, comments, reviews, and status
- Use the head SHA from PR metadata (not branch name) for all git operations
- If asked about specific code changes, use: `git diff <base_sha>...<head_sha> -- <file_path>`
- Provide summary at end:
  - ✅ PR loaded: X files changed (+Y/-Z lines)
  - 💬 Discussion: X comments, Y reviews
  - 🔍 CI Status: [status]
  - 📍 Ready to discuss: Ask questions about any file or change
