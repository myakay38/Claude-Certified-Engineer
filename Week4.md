# Week 4: Advanced Architecture and Exam Preparation

**Theme:** Tackle the "systems thinking" topics — context management over long sessions, task decomposition strategy, and batch processing at scale — then consolidate everything through the full practice exam.

**Exam domains covered:** Domain 1 (Agent Architecture and Orchestration, 27%), Domain 4 (Prompt Engineering and Structured Output, 20%), Domain 5 (Context Management and Reliability, 15%)

### Quick Navigation Links
**Obsidian:** [Day 16](#Day%2016%20—%20Context%20Management%20in%20Production%20Systems), [Day 17](#Day%2017%20—%20Task%20Decomposition%20Designing%20the%20Shape%20of%20Workflows), [Day 18](#Day%2018%20—%20Scale%20and%20Optimization%20Batch%20Processing%20and%20Built-in%20Tools), [Day 19](#Day%2019%20—%20Full%20Practice%20Exam%20(60%20Questions)), [Day 20](#Day%2020%20—%20Final%20Review%20and%20Exam%20Readiness)

**Markdown:** [Day 16](#day16), [Day 17](#day17), [Day 18](#day18), [Day 19](#day19), [Day 20](#day20)

---

<a id="day16"></a>
## Day 16 — Context Management in Production Systems
**Guide reference:** Chapters 11 and 12

Systems that work in demos often degrade in production. This session is about what goes wrong with context at scale — and the patterns that prevent it.

---

### Key Concepts

**Progressive summarization risk**
When `/compact` or automatic summarization compresses conversation history, specific values (exact dollar amounts, percentages, timestamps, line numbers) tend to become vague ("roughly," "about," "a few"). The longer the session, the more lossy the compression.

> **Quick check:** A session has been running for 90 minutes investigating a pricing bug. Claude summarizes the history and then confidently refers to "a discount of around 15%." The actual discount was 12.5% — a number that matters for a compliance report. What should have been done before the summary was run, and how?

---

**The lost-in-the-middle effect**
Claude reliably processes content at the very beginning and very end of long inputs. Content in the middle is statistically more likely to be missed or underweighted. This has direct implications for how you structure prompts that contain large amounts of context.

> **Exercise:** You need to send Claude a 4,000-token aggregated research report along with a request to identify the three most critical findings and write three follow-up action items. The report has 12 sections. Where in the message do you put (a) the critical findings from your own prior review, (b) the bulk of the report, and (c) the follow-up action items? Justify the placement of each.

---

**Persistent "case facts" blocks**
Rather than relying on conversation history (which degrades), extract key transactional facts into a structured block that is explicitly included in every prompt — regardless of how history is summarized.

> **Exercise:** You're building a multi-turn customer support agent. By turn 7 of a conversation, the following facts have been established: customer ID CUST-4492, order ORD-8821 ($320 blender), 8 days since purchase, item arrived damaged, customer declined replacement, customer requested refund. Write the "case facts" block that should be injected into every subsequent prompt, formatted so Claude can extract the information reliably.

---

**Trimming tool results with `PostToolUse` hooks**
If `lookup_order` returns 40 fields but the agent only needs 5 for its current task, the other 35 fields consume context without contributing to the decision. A `PostToolUse` hook trims the result before Claude sees it.

> **Quick check:** Why should trimming happen in a hook rather than in the prompt? (i.e., why not just tell Claude in the prompt to "ignore irrelevant fields"?) What is the practical difference in context usage?

---

**Scratchpad files for long investigations**
In long codebase investigations, the agent can write key findings to a scratchpad file. When context degrades or a new session starts, consulting the scratchpad is far more reliable than re-running all the discovery tool calls.

> **Exercise:** You're 60 minutes into investigating why payment processing occasionally fails. You've found: the failure occurs only for international orders, it's triggered in `PaymentGateway.java` when the currency conversion service returns a null exchange rate, and the null case was introduced in PR #4451 three weeks ago. Write the scratchpad entry that captures these findings in a format that would be useful when resuming this investigation in a new session tomorrow.

---

**Preserving provenance: claim → source mappings**
When synthesizing information from multiple sources, the link between a claim and its source is easily lost during aggregation and summarization. Structured `claim → source` mappings prevent this.

> **Quick check:** A research report states: "The AI music market is estimated at $3.2B." Is this finding usable as-is? What additional fields does it need to be trustworthy and citable, and where should those fields live in the output structure?

---

**Handling conflicting data from multiple sources**
When two sources give different values for the same metric, the correct behavior is to preserve both with attribution — not to pick one arbitrarily, and not to average them. Let the coordinator (or the human) reconcile.

> **Exercise:** Two sources disagree on a key metric:
> - Spotify Annual Report (March 2024): 12% of streams are AI-generated (methodology: automated audio classification)
> - Music Industry Association Survey (July 2024): 8% of streams are AI-generated (methodology: survey of 500 labels)
>
> Write the structured JSON that a document analysis subagent should return for this data point. Include all fields needed for correct interpretation.

---

### Extension Question
Your multi-agent system produces a 20-page research report. An executive challenges a specific statistic on page 14. How does your provenance architecture let you trace that statistic back to its original source, and what would have happened to that traceability in a system that summarized rather than preserved raw subagent outputs?

---

<a id="day17"></a>
## Day 17 — Task Decomposition: Designing the Shape of Workflows
**Guide reference:** Chapter 8

With all the building blocks in place, this session takes the architect's perspective: how do you decide what shape a complex workflow should take? This is the "systems design" layer of the exam.

---

### Key Concepts

**Fixed pipelines (prompt chaining)**
Each step is defined in advance. The output of step N is the input to step N+1. This structure is stable, reproducible, and easy to test — but requires that you know the full task structure before you start.

> **Quick check:** Name two characteristics of a task that make it a good candidate for a fixed pipeline. Name one characteristic that would make a fixed pipeline the wrong choice.

---

**Dynamic adaptive decomposition**
Subtasks are generated based on intermediate results. The agent discovers the full scope as it works. This is powerful for open-ended tasks but harder to test and monitor.

> **Exercise:** You're asked to "add test coverage to a legacy codebase with no existing tests." Walk through how this task would unfold as a dynamic adaptive decomposition:
> - What does the first step discover?
> - How does that discovery shape the second step?
> - Give one example of how the plan changes mid-execution based on what the agent finds.
> - What would a fixed pipeline miss that the adaptive approach catches?

---

**Multi-pass code review: the right decomposition for large PRs**
Reviewing 14 files in a single pass produces inconsistent results: deep analysis for some files, shallow for others, missed bugs, and contradictory feedback on the same pattern. The correct decomposition is per-file local analysis first, then a separate integration pass.

> **Quick check:** A pull request touches 14 files in an inventory module. In a single-pass review, Claude provides detailed comments on the first 4 files and generic comments on the rest. Three bugs appear in files 9–14 and are missed entirely. What is the cognitive mechanism causing this, and how does the multi-pass approach prevent it?

---

**Over-decomposition and under-decomposition**
Too narrow: the coordinator assigns subtasks so specific that whole domains of the problem space go unaddressed (the "only visual art" problem). Too broad: subagents receive tasks so large they produce shallow work.

> **Exercise:** A coordinator decomposes "research the impact of AI on creative industries" into: "AI in music," "AI in visual art," "AI in film," and "AI in literature." Compared to the decomposition in the study guide ("AI in digital art," "AI in graphic design," "AI in photography"), this is better — but it's still imperfect. Redesign the decomposition to produce more thorough, well-scoped subtasks. How many subtasks do you use, and how do you prevent overlap?

---

**Fixed vs. dynamic: the decision framework**
Use fixed pipelines when the task structure is predictable, steps are known in advance, and reproducibility matters. Use dynamic decomposition when the full scope is unknown, each step depends on what the previous step found, or the task is exploratory by nature.

> **Quick check:** For each task below, choose fixed pipeline or dynamic decomposition and explain:
> 1. Weekly security audit of a production codebase (same 5 checks every week)
> 2. Investigate why a specific customer's checkout is failing
> 3. Extract structured data from 10,000 invoices using the same schema
> 4. Understand the architecture of an unfamiliar open-source codebase before contributing

---

### Extension Question
A colleague argues that dynamic decomposition is always better than fixed pipelines because it's "more intelligent" and "adapts to the situation." When would you push back on this, and what are the practical costs of dynamic decomposition that fixed pipelines avoid?

---

<a id="day18"></a>
## Day 18 — Scale and Optimization: Batch Processing and Built-in Tools
**Guide reference:** Chapters 7 and 13

Two efficiency topics that round out the architect's toolkit: the Batches API for cost-effective bulk processing, and the built-in tool reference for systematic codebase investigation.

---

### Key Concepts (Chapter 7 — Message Batches API)

**When to use batch vs. synchronous**
The Batches API offers 50% cost savings but may take up to 24 hours with no latency SLA. It is suitable for non-blocking tasks; it is unsuitable for anything a developer, customer, or downstream process is actively waiting on.

> **Quick check:** A manager proposes moving all Claude API calls to the Batches API to "cut our AI costs in half." You have five workloads: (1) pre-merge code review (developer is waiting to merge), (2) overnight tech-debt report, (3) weekly security audit, (4) real-time customer support responses, (5) monthly batch processing of 50,000 historical documents. Which of these can move to the Batches API, and which cannot? Justify each.

---

**`custom_id` for tracking and partial retry**
Each request in a batch carries a `custom_id` that correlates it to a response. This enables you to identify failed requests and re-submit only those — without re-processing the successful ones.

> **Exercise:** You submit a batch of 200 invoice extraction requests. When results come back, 18 fail with "context length exceeded" errors. Describe the recovery workflow: how do you identify which documents failed, what do you change, and how do you avoid processing the 182 successes again?

---

**SLA planning with the Batches API**
If results are needed by a deadline, you must account for the 24-hour processing window. Batch submission must happen early enough to guarantee results arrive in time.

> **Quick check:** You need results from a batch job by 9am Monday. The Batches API can take up to 24 hours. What is the latest time you can submit the batch? If you want a safety buffer of 4 hours, when do you submit?

---

**Batch API limitations**
The Batches API does not support multi-turn tool calling within a single request. Each item in a batch is one request, one response.

> **Quick check:** You want to use the Batches API to run an agentic extraction pipeline where Claude extracts metadata, then calls a validation tool, then revises if needed. Can you do this in a single batch request? What is the correct architecture if you need both cost savings and multi-turn tool use?

---

### Key Concepts (Chapter 13 — Built-in Tools)

**Tool selection reference**
| Task | Tool |
|---|---|
| Find files by name or pattern | Glob |
| Search within file contents | Grep |
| Read a file in full | Read |
| Create a new file | Write |
| Modify an existing file precisely | Edit |
| Run shell commands | Bash |

> **Quick check:** You need to find every file in the codebase that imports `PaymentGateway`. Which tool do you use, with what arguments? Then, once you have a list of files, which tool do you use to read the most important one?

---

**Incremental investigation strategy**
Rather than reading every file at once (expensive, noisy), build understanding incrementally: Grep entry points → Read found files → Grep usages → Read consumer files → repeat.

> **Exercise:** You need to understand how `process_refund()` flows through the codebase before modifying it. Write out the incremental investigation steps: what do you Grep for first, what do you Read, what do you Grep next? Stop when you have enough context to safely make the change.

---

**Edit fallback: Read + Write**
Edit requires a unique text match to make a change. When Edit fails because the target text appears in multiple places, the fallback is: Read the full file, modify the content programmatically, Write the updated version.

> **Quick check:** You're trying to Edit a function called `validate()` but there are six functions with that name across the file. Edit fails. Walk through the fallback procedure step by step.

---

### Extension Question
The incremental investigation strategy (Grep → Read → Grep → Read) seems slower than just reading all potentially relevant files at once. When is the incremental approach actually faster, and when does reading all files upfront make more sense?

---

<a id="day19"></a>
## Day 19 — Full Practice Exam (60 Questions)
**Guide reference:** Practice Test section

Work through the full 60-question practice test on your own during the week. Time yourself — the real exam paces at roughly 1.5 minutes per question, so 60 questions should take approximately 90 minutes. Bring your results to the Friday morning meeting.

**Instructions:**
- Work individually and without looking at the answers section.
- Mark questions you found difficult — these are your revision targets for Day 20.

**After completing the test:**
- Tally your scores by domain (Domains 1–5). Which domain had the most misses?
- Note which scenarios (Customer Support, Multi-agent Research, Claude Code, CI/CD, Structured Data) felt weakest.
- Return to the relevant Domain notes in Part II for targeted revision before Friday.

**Exam logistics reminder:**
- Score of 720/1000 required to pass
- No guessing penalty — answer every question, even if uncertain
- 4 of 8 possible scenarios are randomly selected on the real exam
- Scenario 8 (Agentic AI Tools) is not yet covered in the guide — if encountered, apply first-principles reasoning from Weeks 1–4

---

<a id="day20"></a>
## Day 20 — Final Review and Exam Readiness
**Guide reference:** Full guide, focus on lowest-scoring domains from Day 19

This is the Friday morning group session. Bring your Day 19 practice test results — compare scores by domain as a group, then use the remaining time to work through the key distinctions before the exam.

---

## Sample Exam Question

Work through this on your own before the Friday morning meeting. Commit to an answer before checking the answer below.

### Question 12 — Scenario: Multi-file Code Review

**Situation:** A pull request changes 14 files in an inventory tracking module. A single-pass review of all files produces inconsistent results: detailed comments for some files but superficial ones for others, missed obvious bugs, and contradictory feedback (a pattern is flagged as problematic in one file but approved in identical code in another file).

**How should you restructure the review?**

- A) Split into focused passes: analyze each file individually for local issues, then run a separate integration pass for cross-file data flows
- B) Require developers to split large PRs into submissions of 3–4 files
- C) Switch to a higher-tier model with a larger context window to review all 14 files in one pass
- D) Run three independent full-PR review passes and report only issues found in at least two runs

---

## Answer

### Question 12 — Correct answer: A

Focused passes directly address the root cause: attention dilution when processing many files at once. Per-file analysis ensures consistent depth across all 14 files, and a separate integration pass catches cross-file issues (type inconsistencies, data flow bugs) that per-file analysis would miss. Requiring smaller PRs (B) shifts the burden to developers without improving the review system itself. A larger context window (C) is a misconception — larger context does not fix attention quality, it just means more content competes for the same attention. Running three full passes and requiring consensus (D) suppresses real bugs by design: a genuine bug caught in two of three passes would still be discarded.

---

## Suggested Structure

**First 30 minutes — weak domain review**
Compare domain scores as a group. For your lowest-scoring domains, work through the questions you missed together, tracing the reasoning in the guide. Prioritize questions where you picked a plausible-sounding wrong answer.

**Final 15 minutes — exam day checklist**

Review the key distinctions that appear repeatedly across all five domains:

---

### The Must-Know Distinctions

**Hooks vs. Prompt Instructions**
Hooks give deterministic guarantees; prompts give probabilistic compliance. Any business rule with financial, legal, or safety consequences belongs in a hook. The 6% violation rate is the tell: if a prompt instruction fails even occasionally in a critical system, replace it with a hook.

**`stop_reason == "end_turn"` as the only reliable completion signal**
Not text parsing. Not iteration limits. Not whether the assistant produced text vs. a tool call. Only `end_turn`.

**Tool descriptions are a selection mechanism, not documentation**
They are the router between Claude and your tools. Vague or overlapping descriptions cause unreliable tool selection. This is the first thing to fix when an agent picks the wrong tool.

**Subagents have isolated context — always pass it explicitly**
No subagent inherits the coordinator's history. A subagent prompt that says "analyze the document" will fail because the subagent has no document. Every subagent call must include all the context it needs.

**Batch API = overnight/non-blocking only**
Never for developer-blocking workflows (pre-merge checks, real-time support). Always for cost-saving batch workloads (overnight reports, weekly audits, historical document processing).

**`.mcp.json` = team/VCS; `~/.claude.json` = personal/experimental**
Tokens go in environment variables. Never commit credentials.

**Dynamic adaptive decomposition ≠ always better**
Fixed pipelines are more testable, reproducible, and debuggable. Use dynamic decomposition for genuinely open-ended tasks where the scope is unknowable upfront. Use fixed pipelines when the structure is predictable.

**Structured errors over generic errors**
`"Operation failed"` is an anti-pattern. Structured errors include `errorCategory`, `isRetryable`, what was attempted, partial results, and alternative approaches. The coordinator needs this to recover intelligently.

**"No results" ≠ "search failed"**
An empty result set (`{"results": []}`) means the search succeeded and found nothing — a valid, informative outcome. A timeout or error means the search didn't run. The coordinator must treat these differently: accept the empty result as coverage, retry or escalate the failure.

**Aggregate accuracy can hide segment failures**
97% overall accuracy can coexist with 58% accuracy on a specific document type. Always analyze by document type and by field, not just overall.

---

## Week 4 Capstone Exercise

**Scenario: AI-Powered Research Platform**

You are the lead architect for a research platform used by analysts who prepare reports on industry trends. Reports are 10–15 pages covering 6–8 topic areas. Sources include web articles, internal PDF documents, and a proprietary database of industry statistics. Reports are produced on a weekly cadence, with some produced on-demand for urgent requests.

This is an open-ended design exercise. There are no single correct answers — the goal is to apply the full set of concepts from all four weeks coherently.

**Part A — Architecture Overview**
Design the multi-agent architecture for the report generation system. Specify:
- The coordinator's responsibilities and system prompt (key points only, not full prose)
- The specialized subagents (at least 3), their roles, and their `allowed_tools`
- How context is passed from coordinator to each subagent
- How results flow back to the coordinator for synthesis

**Part B — Task Decomposition Strategy**
For a report on "The state of AI regulation globally," decide:
- Fixed pipeline, dynamic adaptive decomposition, or a hybrid?
- What are the top-level subtasks the coordinator assigns?
- How do you prevent the "only one region" problem where the decomposition misses major geographies?
- How does the system handle discovering mid-execution that a topic area is much larger than expected?

**Part C — Context and Provenance**
Analysts will fact-check reports and need to trace every statistic to its source.
- Design the `claim → source` structure that each subagent must output
- Describe how the synthesis agent preserves this structure as it merges findings from 4 subagents
- Design the coverage annotation system for when a subagent returns partial results due to a timeout

**Part D — Scale and Cost**
Weekly reports run overnight; urgent on-demand reports need results within 2 hours.
- Which workflow uses the Batches API, and which uses the synchronous API?
- The weekly report processes 150 source documents. Design the batch submission and retry strategy, including how you handle the 8% of documents that typically fail due to encoding issues.
- Calculate the latest submission time for a weekly report that must be ready by 7am Monday.

**Part E — Reliability and Error Handling**
Design the error handling strategy for a scenario where:
- The web search subagent times out on 2 of 8 topic areas
- The proprietary database subagent returns data for only 5 of 8 regions
- The PDF parsing subagent succeeds for all documents

Specifically: what does the coordinator do, what does the final report look like, and what does the analyst see that tells them where to apply additional scrutiny?

**Part F — Reflection**
Identify the single greatest reliability risk in your architecture and describe what you would monitor in production to detect it early.

---

*← [Back to Week 3](Week3.md)*
