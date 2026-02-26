---
description: Analyze PR file changes with detailed summaries, code snippets, and intelligent grouping
---

# PR Changes Command

Analyze pull request #{{arg1}} and provide detailed file-by-file summaries with code snippets, intelligently grouped by related changes. Uses local git for efficient diff analysis. This command is designed for in-depth PR review and should be comprehensive enough to serve as a sole review before merge.

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

3. **Fetch CI status and checks:**
   - Use `mcp__github__pull_request_read` method='get_status' to get CI/test status
   - Parse check runs and their status (passed, failed, pending)
   - Note test coverage information if available

4. **Get file list and stats using git:**
   - Get changed files with status: `git diff --name-status <base_sha>...<head_sha>`
   - Get file stats: `git diff --stat <base_sha>...<head_sha>`
   - Categorize files:
     - Lock files to note (skip detailed analysis): `yarn.lock`, `package-lock.json`, `pnpm-lock.yaml`, `Gemfile.lock`, `composer.lock`, `Cargo.lock`, `go.sum`
     - Build artifacts to note (skip detailed analysis): `*.min.js`, `*.min.css`, files in `dist/*`, `build/*`
     - Config files: `*.config.*`, `*.json`, `.yml`, `.yaml`, `.env*`
     - Test files: `*.test.*`, `*.spec.*`, files in `testing/*`
     - Documentation: `README*`, `*.md`, `*.txt`, `docs/*`, `adr/*`, `.claude/*`, `CONTRIBUTING*`, `LICENSE*`, `CHANGELOG*`
     - Source files: Everything else

5. **Analyze and group related files:**
   - Examine file paths and categories to identify logical groupings
   - **Grouping criteria** (detect patterns like):
     - **Configuration changes**: Multiple config files for same tool
       - Example: `vitest.config.ts`, `vitest.setup.tsx`, `tsconfig.json`, `package.json` (vitest migration)
     - **Feature/component sets**: Related components and their tests
       - Example: `Header.tsx`, `Header.test.tsx`, `Header.module.css`
     - **Refactoring**: Files with similar purposes being updated together
       - Example: All files in `src/components/` being refactored
     - **Dependencies**: Package updates with related config
       - Example: `package.json` + new dependency config files
     - **Documentation**: README, docs, changelog updated together
     - **CI/Build**: GitHub Actions, Docker, build scripts
   - Group related files together, but maintain individual file summaries within each group
   - If a file doesn't fit any group, place it in its own group

6. **Process each group and generate summaries:**
   - For each group:
     - Write group heading with descriptive name (e.g., "🔧 Vitest Migration Configuration")
     - Write brief group context (1-2 sentences explaining why these files are grouped)
     - For each file in the group (excluding lock files and build artifacts):
       - Get file diff: `git diff -U3 <base_sha>...<head_sha> -- <file_path>`
       - **Extract meaningful snippets** from the diff output:
         - Focus on additions (+ lines) and key modifications
         - Capture complete logical units (functions, config blocks, imports)
         - **For modified code**: Use compact unified diff format (- lines followed by + lines)
         - **For new code**: Show only with + prefix
         - **For deleted code**: Show only with - prefix
         - **Fallback**: For very large changes (>20 lines), use separate Before/After sections
         - Limit to 10-20 lines per snippet (or per section if using fallback)
         - Include context lines for readability
         - Prefer showing 1-3 snippets per file
       - **Generate file summary**:
         - File path with emoji (📄 files, ⚙️ config, 🧪 tests, 📝 docs)
         - Change type (Added, Modified, Deleted, Renamed)
         - Summary (2-4 sentences): What changed and why (based on PR description and diff content)
         - Key changes (bullet list): 3-5 main modifications
         - Code snippets: For each snippet:
           - Show filename before the snippet
           - **Primary format**: Unified diff with +/- prefixes showing changes together
           - **Fallback format**: Separate "Before" and "After" sections for large changes
           - Format with syntax highlighting (use ```diff for unified format)
           - Include line numbers with +/- prefixes
       - **Discard the diff after processing** - we only keep the summary and snippets

7. **Analyze documentation status:**
   - Identify all documentation files in the changed files list
   - For code changes (especially new features, API changes, config changes):
     - Check if corresponding documentation exists in the repo
     - Flag documentation that may need updates based on code changes
   - List any documentation files that were modified in the PR
   - Note any missing or potentially outdated documentation

8. **Present organized output in terminal:**
   - **Header section** (with spacing between items):

     ```
     📋 PR #{{arg1}}: [title]

     👤 Author: [author]

     📊 Status: [open/closed/merged]

     📁 Files changed: X (+Y/-Z lines)
     ```

   - **Summary section**:
     - Overview paragraph (1-2 paragraphs) synthesized from PR description and changes
     - What problem does this solve?
     - What's the approach taken?

   - **Statistics**:

     ```
     📊 Statistics:
       • Total files changed: X
       • Source files: Y
       • Test files: Z
       • Config files: W
       • Documentation: V
       • Lock files/artifacts: U (skipped)
     ```

   - **Changes by group**:
     - For each logical group:
       - Group heading (emoji + descriptive name)
       - Group context (why these files are related)
       - Individual file summaries with formatted code snippets:

     **Example format for file with code snippets**:

     ````
     📄 src/components/Header.tsx (Modified)

     Updated the Header component to support dark mode theming. Adds new prop
     for theme toggle and integrates with the ThemeContext provider. This change
     is part of the site-wide dark mode feature.

     Key changes:
     • Added `onThemeToggle` prop and theme state
     • Integrated ThemeContext for global theme state
     • Updated styling to use CSS variables for theme colors

     File: src/components/Header.tsx
     ```diff
      15 | interface HeaderProps {
      16 |   title: string;
     -17 | }
     +17 |   onThemeToggle?: () => void;
     +18 | }
      19 |
     -20 | export const Header: React.FC<HeaderProps> = ({ title }) => {
     +20 | export const Header: React.FC<HeaderProps> = ({ title, onThemeToggle }) => {
     +21 |   const { theme } = useTheme();
      22 |   return (
     -23 |     <header className="header">
     +23 |     <header className={`header header--${theme}`}>
     ```

     **For large changes, use sections:**

     File: src/lib/large-file.ts

     Before:
     ```typescript
     ... (many lines of old code)
     ```

     After:
     ```typescript
     ... (many lines of new code)
     ```
     ````

     ```

     ```

   - **Test Status section:**

     ```
     🧪 Test Status:
       CI Checks: [status from PR metadata]
       • Check 1: ✅ Passed
       • Check 2: ✅ Passed
       • Check 3: ❌ Failed (if any)

       Test Changes:
       • X test files modified
       • Y new tests added
       • Z tests removed
       • Test coverage: [if available from checks]
     ```

   - **Documentation section:**

     ```
     📚 Documentation:
       Modified Documentation Files:
       • README.md: [brief description of changes]
       • CONTRIBUTING.md: [brief description of changes]
       • docs/*.md: [list any doc files changed]

       Potentially Outdated Documentation:
       • [List any doc files that might need updates based on code changes]
       • [Check if API changes are reflected in docs]
       • [Check if new features are documented]
     ```

     **Note:** Analyze code changes against existing documentation files to identify:
     - Documentation files explicitly modified in this PR
     - Documentation that may be outdated due to code changes (e.g., API changes, new features, config changes)
     - Common doc file patterns: `*.md`, `*.txt`, `docs/*`, `adr/*`, `.claude/*`, `CLAUDE.md`, `README*`, `CONTRIBUTING*`, `LICENSE*`, `CHANGELOG*`

   - **Footer section:**
     - ✅ Successfully analyzed: X files in Y groups
     - 🔒 Auto-skipped: [lock files, build artifacts - list if any]
     - 💡 View full PR: https://github.com/marqHQ/nextjs-template/pull/{{arg1}}

**IMPORTANT:**

- This is read-only - do not post anything to GitHub
- Use local git commands for all diff operations (instant, no token cost)
- Use MCP tools (mcp**github**\*) only for PR metadata
- **Process files one at a time** to minimize context usage:
  1. Get diff for file
  2. Extract snippets
  3. Generate summary
  4. Discard diff
  5. Move to next file
- Skip lock files and build artifacts in detailed analysis (just note them)
- Focus on EXPLAINING changes, not just listing them
- Intelligent grouping is key - related files grouped, each with detailed summary
- Output should be comprehensive enough to serve as sole review before merge

**Code Snippet Formatting Requirements:**

- **Always show filename** before each code block: `File: path/to/file.ext`
- **Primary format - Unified diff** (compact, shows changes together):
  - Use ```diff code blocks
  - Include line numbers with +/- prefixes: ` 15 |`, `-16 |`, `+17 |`
  - Show context lines with space prefix: ` 15 | code here`
  - Show deleted lines with - prefix: `-16 | old code`
  - Show added lines with + prefix: `+17 | new code`
  - Group related changes together (- lines followed by + lines)
- **Fallback format** (for large changes >20 lines):
  - Use separate "Before:" and "After:" sections
  - Each section in its own language-specific code block (`tsx, `ts, etc.)
  - Include line numbers without +/- prefixes
- **For new files**: Show only with + prefixes (no - lines)
- **For deleted files**: Show only with - prefixes (no + lines)
- **Keep snippets focused**: 10-20 lines max total (or per section if using fallback)

**Header Formatting:**

- Add blank lines between header items (Author, Status, Files changed) for readability
- Use emojis for visual separation (📋, 👤, 📊, 📁)

**No Impact Summaries:**

- Do NOT add impact assessment at the end of file summaries
- Keep file summaries focused on what changed, not risk/impact analysis

**Extract snippets from git diff output - look for:**

- New functions/classes: Lines starting with `+` followed by `function`, `class`, `const`, `export` (show with + prefix in diff format)
- Modified logic: Blocks with both `-` and `+` lines (show together in unified diff format with context lines)
- Config changes: JSON/YAML blocks with `+` or `-` lines (show in unified diff format)
- Import changes: Lines with `import` statements being added/modified (show in unified diff format)
- **Key**: Keep the unified diff format from git diff output - group related - and + lines together with context lines for readability
- **Only use fallback sections** when a single change block exceeds 20 lines
