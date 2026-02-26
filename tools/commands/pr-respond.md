---
description: Load PR review comments and implement requested changes
---

# PR Respond Command

Load pull request #{{arg1}} with all review comments and implement requested changes using local git for efficient diff analysis.

## Steps to follow:

1. **Fetch PR metadata:** Use `mcp__github__pull_request_read` with method='get' to fetch PR details (owner: marqHQ, repo: worqshop, pullNumber: {{arg1}})
   - Save PR title, description, author, status
   - Note base branch (usually `main`) and head SHA

2. **Fetch ALL review feedback and comments:**
   - Use `mcp__github__pull_request_read` method='get_reviews' to get all review summaries
   - Use `mcp__github__pull_request_read` method='get_review_comments' to get inline review comments with file/line references
   - Use `mcp__github__pull_request_read` method='get_comments' to get discussion comments
   - Parse all feedback and identify:
     - Files mentioned in reviews
     - Specific line references
     - Type of feedback (requested changes, questions, suggestions, blockers)

3. **Ensure we have the PR commits locally:**
   - Get owner, repo, and head SHA from PR metadata (from step 1)
   - Check if we have the commit: `git cat-file -e <head_sha>^{commit}`
   - If not found, fetch from remote:
     - Detect remote with: `git remote -v | grep "github.com[:/]<owner>/<repo>"`
     - Extract the remote name from the output (first column)
     - Fetch using that remote name: `git fetch <remote_name>`
   - Get base commit with: `git merge-base main <head_sha>`
   - Save the base commit SHA for use in subsequent diff commands

4. **Get context for files with review comments:**
   - Get full file list: `git diff --name-status <base_sha>...<head_sha>`
   - For each file mentioned in review comments:
     - Get file diff: `git diff -U3 <base_sha>...<head_sha> -- <file_path>`
     - Match review comments to specific changes in the diff
     - Understand context around commented lines

5. **Analyze ALL review feedback and categorize:**
   - **Requested changes:** Specific code modifications required (highest priority)
   - **Questions:** Clarifications needed from reviewers
   - **Suggestions:** Optional improvements to consider
   - **Blockers:** Issues that must be resolved before merge
   - **Praise/approval:** Positive feedback (acknowledge but no action needed)

6. **For each piece of actionable feedback:**
   - **Understand the concern:** What is the reviewer asking for and why?
   - **Locate the code:** Find exact location in the diff
   - **Suggest implementation approach:**
     - Specific code changes if applicable
     - Alternative solutions if better approach exists
     - Questions for user if approach is unclear
   - **Identify dependencies:** Note if changes affect other parts of code
   - **Estimate impact:** Breaking changes, test updates needed, etc.

7. **Present implementation plan in terminal:**
   - **Summary:**
     - Total review comments: X (Y actionable, Z informational)
     - Files affected: [list]
     - Priority order: [critical → major → minor]
   - **For each actionable item:**
     - Comment reference (file, line, reviewer)
     - What needs to change
     - Proposed implementation
     - Files to modify
     - Any concerns or questions
   - **Questions for reviewers:**
     - List any clarifications needed
     - Proposed responses

8. **Ask user for confirmation before responding:**
   - Show planned responses for each comment
   - Confirm implementation approach
   - User can make modifications before posting

9. **If confirmed:**
   - Use `mcp__github__pull_request_review_write` with method='create' to create a pending review
   - Use `mcp__github__add_comment_to_pending_review` for each response
   - Use method='submit_pending' with event='COMMENT' to post all responses

**IMPORTANT:**

- Use local git commands for all diff operations (instant, no token cost)
- Use MCP tools (mcp**github**\*) only for fetching comments and posting responses
- Focus on files mentioned in review comments - no need to analyze entire PR
- Process comments by priority:
  1. Blockers (must fix)
  2. Requested changes (should fix)
  3. Questions (must answer)
  4. Suggestions (consider and respond)
- Provide summary at end:
  - ✅ Loaded: X review comments across Y files
  - 📋 Action items: [list by priority]
  - 💬 Responses drafted: X comments
  - 🔧 Code changes needed: [list files]
  - 💡 Ready to post responses and implement changes
