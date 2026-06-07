# Week 2: Building Agents — Systems That Act Autonomously

**Theme:** Move from single API calls to multi-step autonomous agents and multi-agent systems. This week introduces the Agent SDK, the coordinator/subagent pattern, hooks for deterministic enforcement, and MCP for connecting Claude to external systems.

**Exam domains covered:** Domain 1 (Agent Architecture and Orchestration, 27%), Domain 2 (Tool Design and MCP Integration, 18%)

### Quick Navigation Links
**Obsidian:** [Day 6](#Day%206%20—%20The%20Agentic%20Loop), [Day 7](#Day%207%20—%20Multi-Agent%20Systems%20Coordinator%20and%20Subagents), [Day 8](#Day%208%20—%20Hooks%20Deterministic%20Control%20Over%20Agent%20Behavior), [Day 9](#Day%209%20—%20Connecting%20Claude%20to%20the%20World%20MCP), [Day 10](#Day%2010%20—%20Week%202%20Review)

**Markdown:** [Day 6](#day6), [Day 7](#day7), [Day 8](#day8), [Day 9](#day9), [Day 10](#day10)

---

<a id="day6"></a>
## Day 6 — The Agentic Loop

**Guide reference:** Chapter 3 (sections 3.1–3.2)

The agentic loop is the core pattern that makes Claude autonomous. This session builds the correct mental model for how agents run — before adding the complexity of multiple agents, hooks, and external systems.

---

### Key Concepts

**The loop in detail**
The agentic loop has four steps: send a request to Claude with tools, receive a response, check `stop_reason`, and act accordingly. If `stop_reason == "tool_use"`, execute the tool, append the result to the message history, and loop. If `stop_reason == "end_turn"`, the task is complete.

> **Exercise:** Sketch the full `messages` array after a two-tool-call agent loop. The user asks "What's the refund status for order 99?" The agent calls `lookup_order(99)`, gets a result, then calls `check_refund_eligibility(99)`, gets a result, and finally answers the user. Draw out every message in sequence, including roles and content types.

---

**`stop_reason == "end_turn"` is the only reliable completion signal**
Two common anti-patterns: (1) parsing the assistant's text for a phrase like "Task complete" to detect when to stop, and (2) using `max_iterations = 5` as the primary stopping condition. Both fail silently in ways that are hard to debug.

> **Quick check:** Your agent uses `max_iterations = 5` as its stopping condition. A task requires 6 tool calls. What happens? What should your stopping logic look like instead, and when would you still use an iteration limit as a _secondary_ safeguard?

---

**`AgentDefinition`: configuration over code**
Agents in the Claude Agent SDK are defined with `name`, `description`, `system_prompt`, and `allowed_tools`. The `allowed_tools` list is a security boundary — it enforces the principle of least privilege.

> **Quick check:** You have an agent for analyzing documents. It currently has access to 12 tools including `process_refund`, `delete_record`, and `send_email`. The agent only ever needs `Read`, `Grep`, and `Glob`. What is the risk of leaving the extra tools in `allowed_tools`, and what is the fix?

---

**Model-driven decision making vs. hard-coded decision trees**
In an agentic system, Claude decides which tool to call next based on context and prior results. This is fundamentally different from a state machine where your code routes between steps.

> **Quick check:** Describe one advantage and one risk of model-driven decision making compared to a hard-coded decision tree. In what kind of task would you prefer the decision tree, and why?

---

**Anti-patterns to memorize**
Three patterns that seem reasonable but reliably fail: parsing assistant text for completion signals, using arbitrary iteration limits as the primary stop condition, and checking whether the assistant produced textual content as a signal that the task is done.

> **Quick check:** An engineer defends an iteration limit: "We set `max_iterations=10` because our tasks should never need more than 10 steps — it's a safety net." Evaluate this. Is the limit itself wrong, or is it the way it's being used as the _primary_ stop condition?

---

### Extension Question

Your agent is stuck in a loop: it keeps calling `lookup_order`, getting a result, and then calling `lookup_order` again with the same arguments. What are three possible root causes, and how would you diagnose each?

---

<a id="day7"></a>
## Day 7 — Multi-Agent Systems: Coordinator and Subagents

**Guide reference:** Chapter 3 (sections 3.3–3.4)

One agent is powerful. A team of agents — each specialized, each with isolated context — is how you tackle complex real-world tasks. This session covers the hub-and-spoke architecture and the most commonly misunderstood property of subagents: they have no automatic access to the coordinator's context.

---

### Key Concepts

**Hub-and-spoke topology**
The coordinator is the hub: it owns all routing, error handling, result aggregation, and communication with the user. Subagents are the spokes: specialized workers that receive a task, execute it, and return a result. All inter-agent communication flows through the coordinator.

> **Quick check:** In a hub-and-spoke system, why does routing all communication through the coordinator matter for debugging and observability? What would you lose if subagents could communicate directly with each other?

---

**Subagents have isolated context**
This is the most commonly misunderstood property. Subagents do not automatically inherit the coordinator's conversation history. If the coordinator has spent five turns gathering customer information, that information does not exist in a subagent's context unless explicitly included in the prompt.

> **Exercise:** The coordinator has gathered the following by the time it calls a subagent:
>
> - Customer ID: CUST-4492
> - Issue: Refund request, order ORD-8821
> - Prior resolution attempted: replacement offered and declined
> - Customer sentiment: frustrated, has requested a manager once
>
> Write the subagent prompt for a `draft_escalation_summary` subagent. Include everything it needs to complete its task without access to any prior conversation.

---

**The `Task` tool for spawning subagents**
Subagents are spawned by the coordinator using the `Task` tool. The coordinator's `allowed_tools` must include `"Task"`. The difference between a good and bad subagent prompt is entirely in how much context is explicitly provided.

> **Quick check:** Compare these two coordinator calls:
>
> - `Task: "Analyze the competitive landscape"`
> - `Task: "Analyze the competitive landscape for AI-powered customer support tools. Focus on pricing models, integration capabilities, and escalation features. The report will be used to justify a budget request to our CTO. Required output format: [schema]"`
>
> What can the second subagent do that the first cannot? What specific failure modes does the first version produce?

---

**Parallel subagent spawning**
A coordinator can call multiple `Task` tools in a single response. The subagents run concurrently, which dramatically reduces total latency for independent subtasks.

> **Quick check:** You're building a research system that needs to: search the web for recent news, search an internal document database, and check a competitor pricing API. All three are independent. How do you structure the coordinator's response to run them in parallel? What must be true about the tasks for parallel execution to be safe?

---

**The coordinator's role in task decomposition**
The coordinator is responsible for breaking a high-level goal into well-scoped subtasks. Decomposition that is too narrow misses coverage; decomposition that is too broad produces poor quality.

> **Quick check:** A coordinator decomposes "research AI impact on creative industries" into three subtasks: "AI in digital art," "AI in graphic design," and "AI in photography." Reports consistently miss music, literature, and film. Is the problem with the subagents, the synthesis step, or the coordinator? What change fixes it?

---

### Extension Question

Your multi-agent research system produces a final report that covers five of the seven required topics. The web search subagent, document analysis subagent, and synthesis subagent all report success. Where in the system do you look first, and what would tell you whether the problem is decomposition, context passing, or something else?

---

<a id="day8"></a>
## Day 8 — Hooks: Deterministic Control Over Agent Behavior

**Guide reference:** Chapter 3 (section 3.5), Domain 1 (section 1.5)

Hooks are the mechanism for guaranteeing behavior — not hoping for it. This distinction between deterministic enforcement and probabilistic guidance is one of the most important concepts on the exam.

---

### Key Concepts

**`PostToolUse` hooks: transform results before Claude sees them**
A `PostToolUse` hook intercepts a tool result after your code executes the tool but before Claude receives it. This is useful for normalizing inconsistent data formats from different MCP servers or trimming verbose results.

> **Exercise:** Three MCP tools in your system return dates in different formats: one returns Unix timestamps, one returns `"Mar 5, 2025"`, and one returns `"2025-03-05"`. Write pseudocode for a `PostToolUse` hook that normalizes all three to ISO 8601 format before Claude sees them. What problem does this solve, and what would happen without the hook?

---

**`PreToolUse` hooks: intercept and block outgoing calls**
A `PreToolUse` hook intercepts a tool call _before_ it executes. This is the right place to enforce business rules that must never fail — financial thresholds, compliance checks, safety constraints.

> **Exercise:** Write pseudocode for a `PreToolUse` hook that:
>
> 1. Intercepts all calls to `process_refund`
> 2. Blocks any refund above $500 and redirects to `escalate_to_human` with a reason
> 3. Allows refunds of $500 or less to proceed normally
>
> What would happen if this rule were enforced via a prompt instruction instead? Under what circumstances would the prompt instruction fail?

---

**The fundamental rule: hooks vs. prompt instructions**
| | Hooks | Prompt instructions |
|---|---|---|
| Guarantee | Deterministic (100%) | Probabilistic (>90%, not 100%) |
| When to use | Financial operations, compliance, safety | General preferences, formatting, style |

> **Quick check:** For each item below, decide: hook or prompt instruction?
>
> 1. "Summarize tool results concisely before presenting them to the user"
> 2. "Never process a medical record deletion without a supervisor approval code"
> 3. "Prefer to resolve customer issues before escalating"
> 4. "Block all API calls to external services after 11pm UTC"

---

**Choosing hooks over prompts for compliance**
The rule is simple: when failure has financial, legal, or safety consequences, use a hook. When failure is merely inconvenient or stylistic, a prompt is fine.

> **Quick check:** Your system prompt says: _"Always call `get_customer` before `process_refund` to verify identity."_ Audit data shows this is violated 6% of the time — Claude sometimes skips `get_customer` when it's confident about the customer's identity from conversation context. Is 6% acceptable? What is the correct fix?

---

### Extension Question

A team proposes using hooks for everything — every business rule, every preference, every formatting guideline — arguing that determinism is always better than probability. What is the practical downside of this approach, and how would you decide where to draw the line?

---

<a id="day9"></a>
## Day 9 — Connecting Claude to the World: MCP

**Guide reference:** Chapter 4

MCP (Model Context Protocol) is the standard for connecting Claude to external systems — databases, APIs, internal tools, content catalogs. This session covers the protocol's structure and the configuration decisions that affect security and team workflows.

---

### Key Concepts

**MCP's three resource types**
MCP servers expose three types of resources: **Tools** (functions the agent can call to take actions), **Resources** (data the agent can read for context without taking action), and **Prompts** (reusable prompt templates). Each has a distinct purpose.

> **Quick check:** For each of the following, identify whether it should be an MCP Tool, Resource, or Prompt:
>
> 1. A list of all open Jira tickets assigned to the current sprint
> 2. A function to create a new Jira ticket
> 3. A template for writing a standup summary from a list of completed tasks
> 4. The database schema for your orders table
> 5. A function to query order records by customer ID

---

**Project vs. personal MCP configuration**
`.mcp.json` at the project root is version-controlled and shared with all team members. `~/.claude.json` in the user's home directory is personal and not shared. Secrets go in environment variables (`${GITHUB_TOKEN}`), never in the config file itself.

> **Exercise:** Your team uses a GitHub MCP server (shared) and you're personally testing a new internal analytics MCP server (not ready for the team). Where does each server's configuration go, and what does each config file look like? Write both configuration snippets.

---

**All connected tools are available simultaneously**
When you connect to multiple MCP servers, all their tools are discovered automatically and available to Claude at once. This means tool descriptions across servers must be distinct — Claude has no other way to choose between them.

> **Quick check:** You connect two MCP servers: one from GitHub (with a `search_issues` tool) and one internal (also with a `search_issues` tool). Both descriptions say "searches for issues." What will Claude do when it needs to search for issues, and how do you fix it?

---

**The `isError` flag and structured error responses**
When an MCP tool fails, it sets `isError: true` in its response. A generic error message (`"Operation failed"`) gives Claude nothing to work with. A structured error includes `errorCategory`, `isRetryable`, and enough context for the coordinator to decide how to recover.

> **Exercise:** A document parsing MCP tool fails because a PDF is password-protected. Write a structured error response (JSON) that gives the coordinator enough information to decide: should it retry with a different approach, skip this document, or escalate to a human?

---

**MCP Resources as "maps"**
Resources let the agent read context without taking action. A content catalog or database schema provided as a Resource means the agent doesn't need exploratory tool calls to understand what data exists — it already has a map.

> **Quick check:** Without an MCP Resource for the Jira project structure, what sequence of tool calls would an agent need to make to understand which epics and sprints exist before it could create a properly categorized ticket? How does a Resource eliminate this overhead?

---

### Extension Question

Your team has built a custom internal MCP server for a unique HR workflow. A community MCP server for Slack also exists. How do you decide which to build vs. which to adopt, and what are the maintenance implications of each choice?

---

<a id="day10"></a>
## Day 10 — Week 2 Review

**Guide reference:** Exam Questions 7–9, Domain 1 and Domain 2 notes

This is the Friday morning group session. Complete the Sample Exam Questions individually before meeting, then work through the reasoning together.

**Agenda:**

1. Work through Exam Questions 7–9 as a group. For Question 9 in particular, walk through why each wrong answer is wrong — the reasoning is as important as the correct choice.
2. Review Domain 1 (Agent Architecture) and Domain 2 (Tool Design and MCP) notes in Part II.

> **Note — Domain 2 coverage gap:** Questions 7–9 all test multi-agent orchestration (Domain 1). None of the sample exam questions specifically test Domain 2 (Tool Design and MCP Integration, 18% of the exam). The MCP concepts from Day 9 — tool vs. resource vs. prompt, `.mcp.json` vs. `~/.claude.json`, structured `isError` responses, and duplicate tool names across servers — are not covered by any sample question. Pay extra attention to the Domain 2 notes in Part II, and treat Day 9's exercises as your primary exam practice for this domain.

---

## Sample Exam Questions

Work through these on your own before the Friday morning meeting. Commit to an answer for each before checking the answers section below.

### Question 7 — Scenario: Multi-Agent Research System

**Situation:** A system researches "AI impact on creative industries," but every report covers only visual art. Subagent logs show the coordinator assigned three subtasks: "AI in digital art," "AI in graphic design," and "AI in photography." All subagents completed successfully.

**What is the most likely root cause?**

- A) The synthesis agent lacks instructions to detect coverage gaps
- B) The coordinator's task decomposition is too narrow, missing entire domains
- C) The web search agent's queries are not broad enough
- D) The document analysis agent is filtering out non-visual sources

---

### Question 8 — Scenario: Multi-Agent Research System

**Situation:** A web-search subagent times out while researching a complex topic. You need to design how error information flows back to the coordinator.

**Which error propagation approach best enables intelligent recovery?**

- A) Return structured error context: failure type, the attempted query, any partial results, and suggested alternative approaches
- B) Implement automatic retries with exponential backoff inside the subagent, then return a generic "search unavailable" status
- C) Catch the timeout inside the subagent and return an empty result set marked as success
- D) Propagate the raw timeout exception to a top-level handler that terminates the entire workflow

---

### Question 9 — Scenario: Multi-Agent Research System

**Situation:** When the synthesis agent needs to verify a claim, it hands control back to the coordinator, which calls the web-search agent, then re-runs synthesis. This adds 2–3 round trips per task and increases latency by 40%. Analysis shows 85% of these checks are simple fact verifications (dates, names, statistics) while 15% require deeper investigation.

**How do you reduce overhead while maintaining reliability?**

- A) Give the synthesis agent a limited `verify_fact` tool for simple checks; continue routing complex verification through the coordinator
- B) Accumulate all verification needs and return them to the coordinator as a batch at the end of synthesis
- C) Give the synthesis agent full access to all web-search tools
- D) Proactively cache additional context around each source at research time

---

## Answers

### Question 7 — Correct answer: B

All subagents completed successfully — they executed their assigned tasks correctly. The problem is what they were assigned. The coordinator decomposed "creative industries" exclusively into visual art subtopics, leaving music, literature, film, and other domains completely uncovered. The synthesis agent (A) cannot synthesize what was never researched. The web search agent (C) and document analysis agent (D) performed correctly within their given scope.

---

### Question 8 — Correct answer: A

Structured error context gives the coordinator the information it needs to make an intelligent recovery decision: retry with a modified query, use an alternative source, continue with partial results, or escalate? A generic "search unavailable" status (B) hides all of this behind a useless label. Returning an empty success result (C) masks the failure entirely — the coordinator thinks the search found nothing, rather than that it failed. Terminating the whole workflow (D) discards all partial results from other subagents unnecessarily.

---

### Question 9 — Correct answer: A

This is the principle of least privilege applied to agent design: give the synthesis agent exactly the capability it needs for the 85% common case (simple fact checks via `verify_fact`), while preserving the coordinator-mediated path for the 15% requiring deeper investigation. Batching all verifications (B) creates a blocking dependency — later synthesis steps may depend on facts verified earlier. Full web-search access (C) breaks the synthesis agent's specialization and makes it harder to reason about its behavior. Speculative caching (D) cannot reliably predict which context will be needed.

---

## Week 2 Capstone Exercise

**Scenario: Multi-Agent Customer Support System**

You are architecting a customer support system for an e-commerce company. The system must handle returns, billing disputes, and account issues. The business has strict rules: no refund above $500 may be processed without manager approval, and customer identity must be verified before any financial action. The target is 80%+ first-contact resolution.

**Part A — Agent Design**
Design the coordinator and at least two specialized subagents. For each agent, specify:

- Its role and responsibilities
- Its `allowed_tools` list (and justify each inclusion)
- A system prompt (2–5 sentences) that defines its behavior
- What it must receive in its context to do its job

**Part B — Context Passing**
Write out the full subagent prompt the coordinator would send to a `draft_resolution` subagent, given:

- Customer: Elena Reyes, CUST-7734
- Order: ORD-2251, a $320 blender purchased 8 days ago
- Issue: Item arrived damaged, customer wants a full refund
- Prior actions: `get_customer` verified identity, `lookup_order` confirmed eligible for return within 30-day window
- Customer tone: polite but firm

**Part C — Hook Design**
Write pseudocode for two hooks your system needs:

1. A `PreToolUse` hook enforcing the $500 refund limit
2. A `PostToolUse` hook that trims the `lookup_order` result from 40 fields to the 5 your agents actually use: `order_id`, `status`, `total`, `items`, `return_eligible`

**Part D — Escalation Scenario**
Walk through the exact sequence of agent actions and tool calls for this scenario:

> _Customer: "My order ORD-2251 arrived broken. I want a full refund. And honestly, if you can't sort this out, I want to speak to a manager."_

At what point, if any, does escalation occur? Is it immediate or after an attempt to resolve? Justify your answer against the escalation rules from Chapter 9 (which you'll cover fully in Week 3, but preview here).

**Part E — Failure Mode Analysis**
After two weeks in production, data shows the system escalates 40% of cases — well above the 20% target. The most common escalation reason is "unable to make progress." What are the three most likely root causes, and what change would you investigate first?

---

_← [Back to Week 1](Week1.md) | [Continue to