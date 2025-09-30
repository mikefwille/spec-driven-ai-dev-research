---
# Provenance
source_type: article
source_url: "https://www.walturn.com/insights/principles-for-building-an-llm-powered-software-tool"
source_author: "Dexter Horthy"
date_captured: 2025-05-08
published_date: 2025-05-08

# Document Status
doc_type: article
maturity: raw
implementation_status: not_started

# Relationships
parent_doc: ""
child_docs: []
related_concepts:
  - 12-factor-agents
  - llm-architecture
  - production-ai
  - stateless-design
  - human-in-loop
  - tool-calls

# Discoverability
primary_topic: "12-factor AI agents framework"
secondary_topics:
  - "LLM application architecture"
  - "production AI principles"
  - "agent design patterns"
  - "reliability and scalability"
applicable_to:
  - "LLM-powered applications"
  - "AI agent architecture"
  - "production AI systems"
  - "enterprise AI development"

# Article Metadata
authors:
  - "Dexter Horthy"
  - "Flavia Trotolo"
  - "Abdullah Ahmed"
  - "Hashim Hayat"
  - "Daheem Hayat"
publication: "Walturn Insights"
article_type: "framework/methodology"
---


May 8, 2025

[Flavia Trotolo](https://www.linkedin.com/in/flavia-trotolo-0baa04259/), [Abdullah Ahmed](https://www.walturn.com/author/abdullah-ahmed), [Hashim Hayat](https://www.walturn.com/author/hashim-hayat), [Daheem Hayat](https://www.walturn.com/author/daheem-hayat)

Artificial Intelligence

LLMs

SaaS

![](https://framerusercontent.com/images/937JflsYiGDqwpAEoX8xgXuLzyw.png?width=800&height=800)

##### Summary

Dexter Horthy proposes a “12-Factor” model for LLM-based software, adapting proven app architecture practices to AI agents. The approach ensures production-grade reliability, observability, adaptability, and control by treating prompts, tools, context, and execution as modular, testable software components.

##### Key insights:

- **LLMs as Command Generators**: LLMs should output structured tool calls, not just text, enabling deterministic code execution.
    
- **Prompt & Context Control**: Prompts and context windows must be explicitly managed and versioned like code.
    
- **Structured, Stateless Design**: Agents should operate as pure functions with all state externalized to logs or stores.
    
- **Modular, Focused Agents**: Specialized agents reduce complexity and increase reliability over monolithic designs.
    
- **Human-in-the-Loop by Design**: Structured calls for human input enhance resilience and trust.
    
- **Scalable Interfaces**: Statelessness and API-accessible design make agents easier to test, scale, and integrate.
    

## Introduction

Building applications with large language models (LLMs) has quickly become a major trend in tech. Developers are racing to include LLMs into products, such as AI chatbots and coding assistants, in order to produce more intelligent and natural user experiences. The promise of software that can comprehend and produce language that is similar to that of humans, as well as quickly evolving models (such as GPT-4 and beyond), are what are driving this rush.

It takes considerable engineering and design to incorporate an LLM into a practical device, though. An LLM agent was frequently viewed as a magic box in earlier methods, which involved giving it a prompt and letting it perform tasks repeatedly until it achieved a desired outcome. In reality, it is often unreliable to simply let the AI "figure it out." The majority of so-called "AI agents" that are now in use are actually deterministic software with a small amount of LLM-driven processes. This reality check emphasizes the importance of developing LLM-powered software that genuinely benefits end users: we need ways to capitalize on LLMs' strengths while making up for their shortcomings.

Proven software engineering knowledge is useful in this situation. Heroku's Twelve-Factor App model, which was developed ten years ago, standardized best practices for creating scalable web services. These initial 12 characteristics, which ranged from handling settings to considering logs as event streams, assisted teams in developing software-as-a-service applications that are scalable, portable, and maintainable by design. 

This has motivated developer Dexter "Dex" Horthy to adapt those timeless ideas to the realm of artificial intelligence agents with his proposal of 12 Factors for LLM-based applications. Making sure our LLM-powered tools are dependable, scalable, and simple to maintain in production is the same objective. The remainder of this insight will go over the goals of an LLM-driven software tool before delving into Dexter's 12 fundamental ideas, also known as the "12-Factor" technique for LLMs, which can help us create AI products of the highest caliber.

## Objective and Features

When designing an LLM-powered tool, it’s important to define what “robust and production-ready” means. In general, we want the software to have several high-level qualities:

**Reliability:** The program should handle the unpredictable nature of AI outputs with elegance and carry out its job consistently. This entails reducing mistakes or absurd outcomes from the LLM, learning from mistakes, and bringing in people or backups in case the AI runs into trouble.

**Scalability:** Without requiring significant overhaul, the system must manage increasing utilization, such as more users or requests. In order to scale out effectively, this frequently calls for stateless architectures and appropriate usage of cloud infrastructure (much like the original 12-factor programs scaled via process models).

**Observability:** An AI's reasoning is opaque, thus the tool should make what is going on visible. It is crucial to provide robust logging, monitoring, and traceability of the LLM's decisions (such as which tools were used, which prompts were utilized, and which outputs were produced). This facilitates troubleshooting and gradual system improvement.

**Adaptability:** As models advance or requirements change, the product should be simple to adapt. For instance, you may want to change a question, add a new "skill" for the AI, or replace the LLM. Key configurations (such as model parameters or prompt templates) are externalized by a well-designed system, allowing for experimentation and tweaking without affecting the system as a whole. According to Dexter, you want the freedom to experiment with different triggers and behaviors.

Product teams (developers and product managers) make sure that an LLM-powered application is more than just a nice demo; it is a sustainable software product by striving for these qualities. Let us now examine the 12 variables, which are a collection of engineering concepts, that support the attainment of these objectives for LLM-based tools

## Key Principles for LLM-Powered Applications

We may list twelve guidelines for creating dependable LLM-driven software by using Dexter's 12-Factor Agents modification and the original 12-Factor App methodology. Every "factor" refers to a best practice that tackles a certain facet of operation or design. The meaning of the 12 factors in relation to LLM-based tools is as follows.:

### 1. Natural Language to Tool Calls

Let the AI output high-level commands, not just free-form text. A user's simple request should be transformed by an LLM into an organized action that your system can carry out. In actuality, this entails translating inputs in plain language as tool calls. For instance, the AI should not merely reply with text if a user requests, "Create a $10 payment link for Max." Rather, it ought to generate a structured command (such as a function call or JSON file) like  {"action": "create_payment_link", "amount": 10, "recipient": "Max"}. 

This tool call can then be used by the remainder of your code to do the actual functionality (such as contacting the payment API). By bridging the gap between software commands and human language, this method makes the AI's output instantly useful. The LLM determines what should be done, and your deterministic code handles how to achieve it. This is a big change from having the LLM do everything.

### 2. Own Your Prompts

Treat prompt design as a first-class part of your codebase. Prompts, the instructions and context you give the LLM, largely determine how it behaves.You must have complete control over these prompts in order to create a reliable system. Owning your prompts in practice entails creating and updating the prompt text yourself (do not cloak it in someone else's black-box library), keeping it under version control, and allowing for customization. This allows you to explore and gradually refine prompts. Since nobody can foresee the ideal prompt right away, you want to be able to experiment with multiple strategies and easily adjust instructions. You can make sure the AI's "script" can change like any other piece of code by externalizing prompts (for instance, by keeping them in files or databases instead of hard-coding them) and monitoring modifications.

### 3. Own Your Context Window

Manually manage what information the LLM sees at each step. The constrained amount of input text that an LLM takes into account when generating output is known as its context window. Take control of the context instead of depending on a preset conversation history or hope the model would remember everything. To do this, previous messages, data that has been retrieved, and other information must be arranged in an understandable manner and fed into the model explicitly each time. You can also be creative in how you assemble the input; you are not required to utilize a typical chat transcript structure. You can use tactics like summarizing previous exchanges, inserting pertinent database findings, or organizing the prompt as a sequence of events by controlling the context window. As a result, the AI operates with a carefully selected memory, producing responses that are more precise and effective.

### 4. Tools Are Just Structured Outputs

Use the LLM to decide actions, and represent those actions in a simple, structured way. In an LLM-powered tool, the output of the AI calls upon "tools" (or external functions/APIs). A tool invocation should simply be a prepared output from the model, usually a JSON object or other machine-readable snippet, that your code can parse and act upon, according to the design principle. Fundamentally, tools are basically structured output from your LLM that initiates deterministic code; they do not require intricate integration.

The LLM's duty is to determine what needs to be done (represented as a data structure), and your program's job is to execute that decision using dependable, hard-coded logic. The next step is planned by the AI and carried out by your software when you regard tools as structured outputs. The system becomes more predictable as a result. For instance, the AI may produce a JSON file like {"action": "lookup_order", "order_id": 1234} which your code knows how to handle. This factor keeps the boundary between AI and software clean and testable.

### 5. Unify Execution State and Business State

Keep a single source of truth for what’s happening in the system. Conventional apps frequently distinguish between "business state" (such as user information or order status in a database) and "execution state" (such as workflow step or the current task of the AI). It can be tempting to keep a distinct, hidden state for the agent's progress in LLM-based workflows (much like variables in memory that indicate the step we are on). This idea proposes to combine these stages instead, recording all occurrences as events in a single timeline or log. For example, a single history can include events like the user's request, the AI's call to a tool, the tool's fault, etc. appended to it

Because of this unified history, you can always reconstitute the agent's actions and the current situation. This same event log can also be used as context by the AI (instead of transitory internal variables). There are two advantages: developers can use the combined log to determine the next course of action, and the AI itself can be programmed to read the log and make decisions. A unified state guarantees that the AI and the rest of the program remain in sync regarding the "story so far" and facilitates debugging (one record of truth).

### 6. Launch/Pause/Resume with Simple APIs

Make the agent’s loop controllable and breakable into segments. In contrast to a conventional function call, an AI agent may need to wait for outside input or complete a multi-step activity that takes a lengthy time. This component suggests creating an interface for your agent that makes it simple to start, pause, and resume operations. In actuality, that might imply that your agent takes one step at a time and then comes back, allowing the calling code to determine whether to proceed or not.

For instance, your system should be able to halt the AI's loop, transfer a request (to a database or to a human), then feed the response back and continue where it left off if the agent needs a piece of data it needs to retrieve or needs human permission at step 3. In order for it to pick up later, this usually entails assigning an ID to each agent execution and saving its context (keep in mind Factor 5's unified log). You allow the rest of your application to coordinate the AI workflow by providing basic APIs or functions to start, stop, or resume an agent. By, for example, permitting a user to interrupt or approve at specific moments, this enhances user experience and increases system resilience by preventing lengthy or halting procedures from stopping the entire process. In essence, the agent stops being an unmanageable monolith and starts to resemble a state machine that your code may walk through.

### 7. Contact Humans with Tool Calls

Involve human feedback as another explicit step when needed. Even the finest AI will occasionally be unclear or need direction (e.g., a manager's consent, or clarification of a vague request). Rather than reaching a dead end or speculating, a well-designed LLM tool should seek assistance. Treating human engagement as just another tool that the AI can use is what Factor 7 entails. Every time the agent requires user input or confirmation, it can produce a structured "AskHuman" action, much like it might call an API.

Your system can detect that and forward it to an individual through a notification (for example, sending an email or Slack message with the query). The agent starts working again after the human answers and the response is fed back into its context. You can make sure the AI is aware that it has the choice to defer to a human by formalizing human engagement in this way. Because the agent will not make important decisions without approval, this increases dependability and credibility. In essence, it incorporates a human-in-the-loop checkpoint into the AI's workflow, which is crucial for error recovery and securely managing unfamiliar circumstances.

### 8. Own Your Control Flow

Don’t let the LLM “run the show” indefinitely – your code should drive the high-level logic. This idea is about not giving the AI complete control over whether to loop or stop. Once the prompt is issued, the LLM will continue to generate ideas and act in a loop until it determines it is finished, as is the case with many agent frameworks. That may result in loops that are unpredictable or never-ending. Owning your control flow, on the other hand, entails putting the loop into your code. After every AI step, you have the option to either pause, ask a human (as mentioned above), try an action again, or proceed.

For instance, you may establish a rule that limits the number of tool calls the agent may make before needing human review, or you may terminate the loop once the answer confidence score exceeds a certain level. The orchestration logic surrounding the LLM is written in the host language (such as Python) with explicit if/else conditions, but the LLM itself is essentially treated as a subroutine that suggests the next step. You have consistent control over the workflow as a result. Timeouts can be managed, infinite loops may be avoided, and domain-specific logic on the termination of a work can be integrated. As a result, the LLM stops being a free agent that could veer off course and instead becomes a part of a broader, controlled process.

### 9. Compact Errors into Context Window

Handle errors by feeding them back into the AI’s input, in a concise form. Errors can occur in any software; for example, an API call may fail or a tool may produce an error (such as "database timeout" or "invalid input"). A basic strategy for an AI agent could be to either retry the same step blindly or stop completely on failure. A better pattern is suggested by Factor 9: record the error details and include them into the context of the subsequent AI question.

Basically, you record the event (according to Factor 5, unified timeline) and give the LLM the error message on the subsequent iteration whenever a tool call fails or returns an exception. The AI can then modify its plan based on that understanding. For example, the AI now learns to avoid that language or request a different input if a "translate_text" API returns the error "Unsupported language." The phrase "compact" is crucial; in order to avoid filling the prompt window, the mistake data should be condensed or structured concisely. You might include just ERROR: Unsupported language for translation as a system message or part of the context. Then, allow the AI to attempt a fix or alternative. Naturally, you would restrict the number of tries before elevating to a person, thus this should not loop indefinitely either. The agent becomes more resilient and frequently self-corrects minor errors rather than giving up right away by looping in error context.

### 10. Small, Focused Agents

Don’t build one giant AI that does everything – build many narrow AIs. This principle borrows the idea of microservices or the Unix philosophy (“do one thing well”) and applies it to AI agents. It is usually preferable to have several specialized agents that each handle a specific task rather than a single large agent that manages a wide variety of tasks or a very lengthy complex workflow. Instead of trying to conduct all of those tasks with a single, enormous prompt, you might, for instance, have one LLM agent that is devoted to data extraction from documents, another that manages meeting scheduling, and another that composes summaries. Smaller agents typically provide shorter cues and simpler reasoning, which reduces the likelihood that the LLM would veer off course or become confused. 

Additionally, they facilitate the testing and upkeep of every component. You can update one agent without worrying about upsetting the others if they are all attentive. These agents can be combined to create a product, with your control logic orchestrating the output of one agent into the input of another as necessary. Although it may appear to the end user as a single, seamless assistant, there is actually a network of collaborating mini-AI services at work. Because each component is exerting less effort, this lowers complexity and frequently speeds up response times. To put it briefly, break down your large AI problem into smaller ones.

### 11. Trigger from Anywhere, Meet Users Where They Are

Make the AI’s capabilities accessible through multiple channels and services. Instead of being limited to a single user interface, your LLM-powered tool should be made to be invoked or triggered by several front-ends or events. For instance, a consumer may communicate with your AI through an email, a Slack bot, or a web application; all of these should be feasible without requiring constant innovation. Decoupling the fundamental agent logic from any one interface is the focus of this component.

Concretely, you could make the agent available as a function that can be called with the right context or as an API endpoint. The agent logic is the same regardless of whether the trigger is a cron job, an HTTP request from your web frontend, or a Slack webhook. Deliver the output back to the original context in the same way (e.g., email the result, publish the answer in Slack). Integrating with current workflows is another way to meet consumers where they are. For example, if your AI tool arranges meetings, let it start when a user adds a keyword to a calendar invite. When you build for interoperability, your LLM agent becomes a modular component that may be used in a variety of situations. The fact that your tool is not platform-specific adds to its value. The AI basically becomes a ubiquitous service that can be accessed from any location where it can be helpful.

### 12. Make Your Agent a Stateless Reducer

Design the agent like a pure function: it takes input (context + query) and returns an output, without carrying over hidden state. Many of the advantages of the previous principles, particularly those 3, 5, and 8, are summed up in this last one. Similar to a Redux reducer, a stateless reducer is a concept from functional programming that says it does not depend on outside side effects and always produces the same output given the same input state. Making the agent stateless in the context of LLM implies that it should only know what you explicitly specify in the prompt context each time, without implicitly remembering anything from past executions. All of the "state"—discussion history, intermediate outcomes, etc.—is kept outside (perhaps in that single event log) and entered when required. 

The current state is only reduced to the subsequent action by the agent function. Cloud software has demonstrated the benefit of stateless processes, which include easy horizontal scaling and recovery in the event of a failure. You can manage numerous users in parallel by spinning up multiple instances of your AI agent if it is stateless; each instance simply receives the user's context and generates a result without any cross-talk. Additionally, it makes testing and debugging easier because you can feed the agent a recorded history of events and consistently replicate what the agent would do next. There’s no hidden memory that could differ. In summary, treat the agent like a pure function next_action = AI(brain_state, new_event) where **all** necessary information is in brain_state (and that comes from a store or database). This will make your AI far more scalable and maintainable in the long run.

These 12 elements provide a thorough framework for designing an application that uses LLM. To handle the particular difficulties of LLM integration, they incorporate new AI-specific rules (such tool usage and prompt management) along with ideas from the traditional 12-factor app (like statelessness, clear configuration, and logs). By following these guidelines, an AI product's quality can be considerably raised. To show one of the concepts in action, let us now examine a little example.

```
## Example

To cement the ideas, consider a simplified code example demonstrating **stateless execution** (Factor 12) and how it ties together a few of the principles. In this Python-like pseudocode, we have an agent_step function that represents one cycle of an LLM agent interacting with the world. It takes in the current context (history of events) and a new user input, and it returns the next action for the agent to perform:

# Example: Stateless agent step as a pure function

def agent_step(event_history, user_message):

&nbsp;&nbsp;# 1. Own your context window: build prompt from event history + new message

&nbsp;&nbsp;prompt = format_prompt(event_history, user_message)

&nbsp;&nbsp;# 2. Call the LLM to decide next action (natural language -> tool call)

&nbsp;&nbsp;model_output = llm_api.generate(prompt)

&nbsp;&nbsp;# 3. Parse structured output (tool call) from the LLM's response

&nbsp;&nbsp;next_action = parse_tool_call(model_output)

&nbsp;&nbsp;return next_action

# Usage:

shared_history = []&nbsp; # unified log of events (empty at start)

user_msg = "Create a $10 payment link for Max"

action = agent_step(shared_history, user_msg)

print("Agent decided on action:", action)

# Suppose action = {"action": "create_payment_link", "amount": 10, "recipient": "Max"}

# The deterministic code would then execute this tool call (e.g., call payments API).

# We would append the user_msg and action result to shared_history, and repeat as needed.
```

In this snippet, the function agent_step does not rely on any hidden global state, everything it needs (past events and the latest user query) is passed in. It formats the prompt from the history (illustrating Factors 3 and 5 by controlling exactly what context the model sees), then calls an llm_api to get the model’s output. The output is expected to be a structured representation of the next step (illustrating Factor 1 and 4: the model output is a **tool call** in JSON or similar). We parse that into next_action, which the calling code can handle. After executing the action, the event (and any result or error) would be recorded in the shared_history for the next cycle.

This design makes the agent behave like a stateless service – each call is a pure function of the inputs. We could easily scale this by running multiple agent_step instances in parallel (for different users or tasks), since none of them hold persistent internal state. Also, testing this in isolation is straightforward: provide a fake history and user message, and check if the returned action is correct. If we wanted to illustrate externalizing configuration, we could imagine that format_prompt reads a template from a file or environment variable (so that prompt phrasing can be tweaked without changing code), and that llm_api uses an API key from an environment variable rather than a hardcoded string. Those practices align with traditional config management and make our agent more adaptable.

## Conclusion

As AI-powered software transitions from demos to production, following these principles can make the difference between a flaky prototype and a reliable product. Disciplined LLM integration design results in systems that scale to actual user loads, function reliably, and can be maintained or enhanced over time. In essence, Dexter Horthy's list of 12 criteria for LLM-powered applications is a distillation of lessons learned over the years. It encourages us to combine the advantages of classical software engineering (deterministic logic, modular design, and clear state management) with the strengths of LLMs (flexible reasoning, understanding natural language). As a result, rather than being merely intelligent black boxes, AI agents feel solid and "engineered."

In practice, adhering to these principles leads to AI tools that are more reliable, more scalable, and easier to maintain, the very qualities any product manager or startup founder would want in a core software tool. The modified 12 factors for LLMs help guarantee that our AI systems can manage demands in the real world, much like the original Twelve-Factor App principles produced more robust online applications. They encourage agility (through modularity and configuration), resilience (through error handling and human fallback), and clarity (in the way the AI is prompted and behaves). These recommendations provide a road map for anyone working with large language models today to steer clear of problems and create long-lasting, superior AI software solutions that will benefit users for years to come. Adopting these guidelines is an architectural investment that will pay off as your LLM-powered solution develops and expands in tandem with AI's growing capabilities.