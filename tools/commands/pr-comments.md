---
description: Fetch all comments left by the claude bot on the PR for the current branch, evaluate each, and produce an assessment
---

# PR Bot Comments Command

Fetch all non-outdated comments left by `claude[bot]` on the pull request associated with the current git branch, evaluate each comment using sub-agents, and synthesize an assessment report.

## Steps to follow:

1. **Determine current branch:**
   - Run `git branch --show-current` to get the current branch name

2. **Find the PR for this branch:**
   - Run `gh pr list --head <branch_name> --json number,title,url`
   - If no PR is found, inform the user and stop

3. **Fetch non-outdated comments from claude[bot]:**
   - Get review summaries: fetch the JSON first, then filter in a second step to avoid shell quoting issues:
     1. `gh api repos/marqHQ/worqshop/pulls/<pr_number>/reviews > /tmp/pr_reviews.json`
     2. Use `jq` to filter: `jq '[.[] | select(.user.login == "claude[bot]")]' /tmp/pr_reviews.json`
   - Get inline review comments, filtering out outdated ones — again, fetch first, then filter:
     1. `gh api repos/marqHQ/worqshop/pulls/<pr_number>/comments > /tmp/pr_comments.json`
     2. Use `jq` to filter: `jq '[.[] | select(.user.login == "claude[bot]") | select(.position)]' /tmp/pr_comments.json`
     - **Important:** Never use the "not equal" operator in jq expressions passed through bash. The exclamation mark triggers bash history expansion, which escapes it and breaks jq. Use truthy checks instead — for example, select(.position) filters out null values because null is falsy in jq.
     - Comments with `"position": null` are outdated (the code they reference has changed since the comment was made). The `select(.position)` filter excludes these since null is falsy in jq.
     - Also check the `in_reply_to_id` field — if a comment is a reply to an outdated comment, exclude it too
   - If no non-outdated comments remain, inform the user that all bot comments are outdated and stop

4. **Spawn a sub-agent for each comment:**
   - Use the Task tool to launch a `general-purpose` sub-agent for each inline comment, running them **in parallel**
   - Provide each sub-agent with:
     - The PR title and number
     - The full comment body
     - The file path and line number the comment targets
   - Instruct each sub-agent to:
     - Read the full file mentioned in the comment
     - Find any other relevant content (e.g., related library code, test files)
     - Assess the **priority** of the comment (Critical / High / Medium / Low)
     - Evaluate whether the comment is **valid** and **actionable** given the actual code
     - Propose a concrete **plan** to address it, or explain why it should be dismissed
     - Note any **risks** of making or not making the change

5. **Synthesize the assessment report:**
   - Collect all sub-agent results
   - Create a markdown file at `.claude-reviews/pr-<pr_number>-review-assessment.md`
     - If the file already exists update its contents
   - The report should include:
     - PR metadata (title, branch, number, reviewer)
     - A priority summary table listing each comment with its file, priority, and recommended action
     - An overall verdict (e.g., "No changes required" or "N items to address")
     - A detailed section for each comment containing the sub-agent's full analysis: priority, validity, plan, risks, and recommended PR response
   - Present a summary of the report to the user
