---
youtube_title: "Advanced context engineering for coding agents: 🦄#17"
descriptive_title: "Advanced Context Engineering for Coding Agents - Three-Phase Workflow"
content_type: "Workflow/Automation Demo"
ai_primary_topic: "context engineering for coding agents"
source: youtube
video_id: 42AzKZRNhsk
url: https://youtube.com/watch?v=42AzKZRNhsk
youtube_channel: "Boundary"
youtube_channel_id: "UCL2VtT75x0fPLLn2YWIDTIg" 

summary_long: >-
  Detailed methodology for maximizing AI coding agent performance through three-phase workflow: Research (understanding problem and codebase with sub-agents), Planning (creating phased implementation plans), and Implementation (test-driven development with automated verification). Focuses on context window management, strategic use of sub-agents for parallelization, and human checkpoints at critical stages to ensure quality outcomes.

summary_short: >-
  Three-phase workflow for AI coding agents: Research (context gathering with sub-agents), Planning (phased implementation design), Implementation (test-driven development). Emphasizes context window management and human review checkpoints for quality assurance.

ai_tags:
  - context-engineering
  - ai-coding-agents
  - llm-workflow
  - software-development
  - spec-first-development
  - prompt-engineering
  - brownfield-development

---

This document outlines the detailed process for advanced context engineering for coding agents, as presented in the video "Advanced context engineering for coding agents: 🦄#17" by Boundary. The methodology focuses on maximizing the performance of AI coding agents, specifically Claude, in complex software development by meticulously managing the agent's context window.

---

## Introduction to Context Engineering for AI Coding Agents

The core principle behind this advanced workflow is **context control**. Large Language Models (LLMs) like Claude operate within a "context window." Every interaction—sending a user message, calling a tool, or receiving a response—consumes tokens within this window. As the context window fills, the "density of useful information" decreases, leading to reduced agent performance and increased costs. The goal is to keep the context window as lean and relevant as possible to enable the agent to make optimal decisions.

This documentation details a three-phased workflow: **Research**, **Planning**, and **Implementation**. Each phase is designed to manage context effectively, reduce noise, and ensure high-quality outputs.

---

## Key Concepts and Custom Tools

Before diving into the phases, understanding these concepts and tools is crucial for duplicating the process:

### 1. Context Window Management

The central idea is to prevent the context window from becoming overly large and diluted. This involves:

- **Compaction:** Actively compressing the conversational history and relevant information.
  
- **Fresh Sessions:** Starting new agent sessions with only the most critical, compacted context.
  
- **Sub-Agents:** Delegating specific, isolated tasks to new agent instances with their own dedicated context windows.
  

### 2. Custom Commands and Prompts

The team uses highly structured prompts and custom commands to guide Claude's behavior. These are often stored in open-source repositories and can be updated as best practices evolve. Examples mentioned include:

- **Research Prompt:** A multi-line Markdown prompt (e.g., 300 lines) that defines the research task, how to use sub-agents, and the desired output format.
  
- **Plan Command:** A specific command that instructs the agent to create a phased implementation plan.
  
- **`/compact` (Claude's built-in command):** While available, the speakers prefer manual compaction over this generic command, as it offers more control over the "trajectory" of the problem-solving.
  
- **`/clear` (Claude's built-in command):** This command is used to start a fresh agent session with no prior context, effectively resetting the context window.
  

### 3. Sub-Agents (Tasks)

Sub-agents (referred to as "tasks" in Claude's toolset) are not for anthropomorphizing the AI but for precise **context control** and **parallelization**.

- **How they work:** When the main agent encounters a complex sub-task (e.g., "find where we load user data"), it launches a sub-agent using a `task` tool. This sub-agent receives a new, clean context window with a user message defined by the _parent model_.
  
- **Typical Use Cases:** Primarily for read-only operations such as:
  
    - Searching the codebase for specific files or functions.
      
    - Reading documentation.
      
    - Gathering information from the web.
    
- **Output:** The sub-agent performs its task (e.g., reading, searching) and returns a concise "assistant message" as a tool response to the parent agent (e.g., "that file is in source main whatever"). This keeps the parent agent's context lean.
  
- **Parallelism:** Multiple sub-agents can be launched in parallel to tackle independent sub-tasks simultaneously, significantly improving efficiency (e.g., writing different types of unit tests).
  
- **User Instruction:** The main agent often needs to be explicitly prompted by the user to "use a sub agent and prompt it like this" to ensure effective utilization.
  

### 4. Codebase and Tooling Best Practices

Optimizing the underlying codebase and development environment is critical for agent performance.

- **Consistent Nomenclature:** Use clear, consistent naming conventions for files, functions, and concepts throughout the codebase. This helps the LLM accurately understand and navigate the code.
  
- **Trivial Test Commands:** Ensure that test commands are simple and easy for the LLM to execute and understand the results.
  
- **IDE Integration:** Using the Claude Code VS Code extension and running Claude in the integrated terminal provides access to Language Server Protocol (LSP) errors, giving the agent real-time linting and type-checking feedback.
  
- **Monorepos and Hard Links:** For large projects, monorepos are preferred. If multiple repositories are involved, using hard links (instead of symlinks) can ensure Claude's search tools traverse all necessary directories.
  
- **`settings.json` for Multi-Repo Access:** To grant Claude permissions to work and write in additional directories, specify them in `settings.json`.
  
- **Optimized Build Systems:** Tools like Turbo for TypeScript projects, or Cargo for Rust, simplify build processes and dependency checking, making it easier for agents to manage large workspaces.
  

---

## The Three-Phased Workflow Explained

This section details the step-by-step process used by the Boundary team.

### Phase 1: Research 🧐

**Goal:** To deeply understand the problem, identify relevant code locations, and gather all necessary context before attempting a solution. This phase aims to prevent the agent from making uninformed decisions or going off-scope.

**Steps:**

1. **Define the Issue:** Clearly articulate the problem or feature requirement. For instance, the video example addresses a bug in the BAML codebase where `@assertion` behavior is inconsistent.
   
2. **Initial Research Prompt (with Sub-Agents):**
   
    - The user initiates a new Claude session.
      
    - A lengthy, pre-defined "research prompt" (e.g., 300 lines of Markdown, often stored in an open-source `commands` directory) is used.
      
    - This prompt instructs Claude to use sub-agents to search the codebase.
      
    - **Example Prompt Structure (conceptual):**
      
        Markdown
        
        ```
        # Research Task: Investigate BAML Assertion Bug
        
        ## Objective
        Find everywhere in the BAML codebase (`.baml` files and relevant Rust source) related to the issue of `@assertion` attributes behaving inconsistently. Specifically, focus on:
        - How `@assertion` attributes are parsed.
        - How they are validated.
        - How their values are processed during test execution.
        - Identify the information flow from parsing to validation and execution.
        
        ## Sub-Agent Instructions
        Use sub-agents (via the `task` tool) for all search and read operations to keep the main context window clean.
        - **`search_codebase <query>`:** Use this for broad searches across files.
        - **`read_file <path>`:** Use this for reading specific file contents.
        - **`read_glob <pattern>`:** Use this for reading files matching a pattern.
        - **Constraints:** Avoid using `bash` commands as they often require manual approval and can pollute context. Prioritize `read`, `glob`, `gp_search` (fast pattern search), and `blob` [00:23:55].
        
        ## Desired Output Format
        Generate a Markdown file named `research_report.md` that includes:
        - A summary of the bug.
        - Identified files and relevant line numbers.
        - Explanation of how information flows through these files.
        - Potential areas for modification or validation.
        ```
        
    - The user then pastes the specific issue description (e.g., "Find everywhere in .baml relevant to solving this issue and figure out how the information flows").
    
3. **Agent Execution and Steering:**
   
    - Claude launches multiple sub-agents in parallel to perform searches (e.g., `gp_search`, `read_glob`).
      
    - **User Intervention (Steering):** If the initial research yields incorrect results (e.g., "no bug found"), the user must intervene. This involves:
      
        - **Reviewing the research output:** Critical human step to identify inaccuracies.
          
        - **Refining the Prompt:** Updating the initial research prompt with more specific details, failing test cases, or explicit constraints (e.g., "only use read glob gp search never bash").
          
        - **Manual Compaction/Restart:** If the context is too polluted from incorrect research, the user might manually create a new research prompt based on the _learnings_ from the failed attempt, then start a _new agent session_ using `/clear` and the refined prompt. This is a form of manual compaction, leveraging past learnings without carrying unnecessary context.
    
4. **Research Output Generation:** The agent consolidates its findings into a detailed Markdown file (e.g., `research_report.md`). This file should explicitly list files, line numbers, and an explanation of the system's behavior relevant to the issue.
   
5. **Human Review of Research:** This is a critical checkpoint. The research report is thoroughly reviewed by a human. The team uses a linear board with states like "research in progress," "research in review," and "ready to plan". Errors caught here prevent cascading issues in later phases.
   

**Outcome:** A well-understood problem space with identified relevant code, documented in a concise Markdown file.

---

### Phase 2: Planning 🗺️

**Goal:** To translate the research findings into a concrete, phased implementation plan that guides the coding agent. This plan acts as a blueprint, ensuring the agent stays focused and delivers the desired solution.

**Steps:**

1. **Initiate Planning Session:**
   
    - Start a _new_ Claude session (`/clear`) to ensure a clean context window.
      
    - Provide the agent with the **research report** generated in Phase 1. This acts as the initial context.
      
    - Use a specific "plan command" or a detailed prompt to instruct Claude to create an implementation plan.
    
2. **Plan Generation Prompt:**
   
    - The prompt instructs Claude to analyze the research report and propose a phased approach to solving the problem.
      
    - It should include directives for:
      
        - **Identifying Phases:** Breaking the work into logical, sequential steps.
          
        - **Defining Scope ("What not to do"):** Explicitly stating what functionalities or changes are _out of scope_ to prevent feature creep.
          
        - **Error Messages:** Specifying the desired quality and content of any new error messages.
          
        - **Test-Driven Development (TDD):** Crucially, the prompt dictates that phases should _always_ begin with writing a _failing test_ that demonstrates the bug, followed by the code fix. This provides immediate, automated feedback for the agent.
          
        - **Automated Verification Steps:** For each phase, include shell commands or instructions for the agent to verify its work (e.g., `cargo test`, `npm run lint`).
        
    - **Example Prompt Structure (conceptual):**
      
        Markdown

        ```
        # Planning Task: Develop Implementation Plan for BAML Assertion Bug
        
        ## Objective
        Based on the provided `research_report.md`, create a detailed, phased implementation plan to fix the BAML `@assertion` bug.
        
        ## Instructions
        1.  **Analyze Research:** Thoroughly review the `research_report.md` for understanding the problem and identified files.
        2.  **Define Phases:** Break the implementation into logical phases. Each phase *must* begin with writing a failing test.
        3.  **Phase Structure:** For each phase, describe:
            * **Objective:** What this phase aims to achieve.
            * **Changes:** Specific files and approximate line numbers to be modified.
            * **New Code/Logic:** Details on what code to add or change.
            * **Error Messages:** Craft clear, human-readable error messages for validation failures.
            * **Failing Test:** Describe the failing test to be written (and its expected output).
            * **Fix:** Describe how the code will be modified to make the test pass.
            * **Automated Verification:** List exact shell commands to run after making changes (e.g., `cargo test --package baml-parser`, `cargo check`).
        4.  **Out of Scope:** Clearly state any features or changes that should NOT be implemented as part of this plan.
        5.  **User Interaction:** Propose the phases to the user for approval before detailing the full plan.
        ```

3. **Agent-User Collaboration (Optional but Recommended):** The planning agent can be prompted to propose phases and iterate with the user to refine the plan, ensuring alignment before fully detailing each step.
   
4. **Plan Output Generation:** The agent generates a Markdown file containing the detailed, phased implementation plan.
   
5. **Human Review of Plan:** This is another crucial checkpoint. The human reviewer verifies:
   
    - **Correctness:** Does the plan address the bug accurately based on the research?
      
    - **Completeness:** Are all necessary steps included?
      
    - **Test Strategy:** Are the proposed tests robust and effective?
      
    - **Error Messages:** Are the error messages clear and helpful?
      
    - **Architecture:** Does the plan align with the codebase's architectural principles?
      
    - The linear board tracks "plan in progress," "plan in review," and "ready for dev".
      

**Outcome:** A detailed, human-reviewed, phased implementation plan, stored in a Markdown file, ready for the coding agent.

---

### Phase 3: Implementation 💻

**Goal:** To execute the approved plan, writing and testing the code with minimal human intervention, leveraging automated feedback loops.

**Steps:**

1. **Initiate Implementation Session:**
   
    - Start a _new_ Claude session (`/clear`).
      
    - Provide the agent with the **approved implementation plan** from Phase 2. This serves as its primary instruction set.
      
    - Instruct the agent to "implement this plan."
    
2. **Test-Driven Execution:** The agent strictly follows the TDD approach outlined in the plan.
   
    - **Write Failing Test:** For each phase, the agent first writes the specified failing test case.
      
    - **Run Test:** The agent executes the test command (e.g., `cargo test`). It receives feedback (test failure).
      
    - **Fix Code:** Based on the test failure and the plan, the agent modifies the codebase to fix the bug or implement the feature.
      
    - **Run Automated Verification:** The agent runs the automated verification steps (including tests, linters, type checkers) specified in the plan.
      
    - **Iterate:** This cycle (write failing test, run test, fix code, verify) continues until all tests pass and verification steps are complete.
    
3. **Leveraging IDE Integration (for Agent Self-Correction):**
   
    - If using the Claude Code VS Code extension in an integrated terminal, the agent can access LSP errors. This provides crucial feedback on linting, typing, and other code quality issues in real-time.
      
    - The agent might be prompted to "get LSP errors" if it encounters issues that built-in tests don't catch.
    
4. **Pull Request (PR) Creation:** Upon successful completion of the plan (all tests pass, all verification steps are green), the agent generates a Pull Request (PR) with the implemented changes.
   
5. **Human Code Review (Tests Focused):**
   
    - The focus of the human code review shifts dramatically. Instead of line-by-line code review, the primary focus is on the **quality and design of the tests**.
      
    - Reviewers check if the tests accurately cover the intended functionality and prevent regressions.
      
    - If the tests are robust, confidence in the agent-generated code is high, leading to quicker approvals and merges.
      
    - Example from video: A PR with research-backed planning resulted in a well-designed test and an accurate fix, leading to automatic merging. A PR without research-backed planning might have an incorrect test design, requiring human rejection and re-direction.
      
    - Critical areas (e.g., core libraries, algorithmic complexity, low test coverage areas) may still require deeper human review.
      

**Outcome:** A successfully implemented feature or bug fix, validated by automated tests, and merged into the codebase after human review of the tests and plan.

---

## Conclusion

This structured, three-phased approach to context engineering—Research, Planning, and Implementation—allows developers to leverage AI coding agents effectively for complex tasks. By consciously managing the agent's context window through intentional compaction and strategic use of sub-agents, coupled with meticulous human review at the research and planning stages, the team at Boundary achieves high-quality, efficient software development. The emphasis on test-driven development and robust automated verification further enhances the reliability of agent-generated code. This methodology transforms AI from a demo tool into a productive and integrated part of the development workflow.