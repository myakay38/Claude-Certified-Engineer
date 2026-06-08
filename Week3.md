# Week 3: Developer Workflows and Reliability

**Theme:** Apply everything from Weeks 1–2 to real engineering workflows using Claude Code. Then tackle the hard problems: what happens when things go wrong, when users need a human, and when context degrades over long sessions.

**Exam domains covered:** Domain 3 (Claude Code Configuration and Workflows, 20%), Domain 5 (Context Management and Reliability, 15%)

### Quick Navigation Links
**Obsidian:** [Day 11](#Day%2011%20—%20Claude%20Code%20Configuration%20and%20Developer%20Workflows), [Day 12](#Day%2012%20—%20Claude%20Code%20Planning%20Mode,%20Session%20Management,%20and%20CI/CD), [Day 13](#Day%2013%20—%20Escalation%20and%20Human-in-the-Loop), [Day 14](#Day%2014%20—%20Error%20Handling%20in%20Multi-Agent%20Systems), [Day 15](#Day%2015%20—%20Week%203%20Review)

**Markdown:** [Day 11](#day11), [Day 12](#day12), [Day 13](#day13), [Day 14](#day14), [Day 15](#day15)

---

<a id="day11"></a>
## Day 11 — Claude Code: Configuration and Developer Workflows
**Guide reference:** Chapter 5 (sections 5.1–5.6)

Claude Code is where these concepts become daily tools for engineers. This session covers the configuration system that makes Claude Code consistent and team-friendly — and the most common mistakes teams make when setting it up.

---

### Key Concepts

**The CLAUDE.md hierarchy: three levels, three scopes**
CLAUDE.md files exist at three levels: user-level (`~/.claude/CLAUDE.md`, personal and not version-controlled), project-level (`.claude/CLAUDE.md` or root `CLAUDE.md`, shared via VCS), and directory-level (CLAUDE.md in subdirectories, scoped to that folder's conventions).

> **Quick check:** A new engineer joins the team, clones the repository, and reports that Claude Code isn't following any of the project's coding standards. The senior engineer insists the instructions are in CLAUDE.md. What is the most likely cause of the discrepancy, and how do you verify it?

---

**`@path` syntax: modular configuration**
CLAUDE.md can reference external files with `@path`, which lets you pull in coding standards, testing requirements, or architecture docs without duplicating them. Relative paths are resolved from the file containing the import. Maximum nesting depth is 5.

> **Exercise:** Your project has three separate standards documents: `coding-style.md`, `testing-requirements.md`, and `api-conventions.md`, all in a `./standards/` directory. Write the relevant lines of a project-level CLAUDE.md that imports all three, and explain why this is better than copying the content directly into CLAUDE.md.

---

**`.claude/rules/` with path-scoped YAML frontmatter**
Instead of one large CLAUDE.md, you can create topic-focused rule files in `.claude/rules/`. Each file can include YAML frontmatter with `paths` — a glob pattern that causes the rule to load *only* when Claude is editing a matching file.

> **Exercise:** You have two sets of conventions: one for API route files (in `src/api/**/*`) and one for test files (in `**/*.test.ts`, scattered throughout the codebase). Write the frontmatter and first two convention lines for each of the two `.claude/rules/` files. Explain why path-scoped rules are better than a directory-level CLAUDE.md for the test conventions.

---

**Custom slash commands and skills**
Project commands in `.claude/commands/` (or `.claude/skills/`) are version-controlled and available to everyone who clones the repo. Personal commands in `~/.claude/commands/` are private. Skill frontmatter controls isolation, tool access, and UX.

> **Quick check:** Your team wants a `/security-review` command that runs a focused security audit on any file without polluting the main conversation context. Which frontmatter field makes this work, and why does it matter for context management?

---

**Skill frontmatter: `context`, `allowed-tools`, `argument-hint`**
- `context: fork` runs the skill in an isolated subagent so verbose output doesn't fill the main session's context window.
- `allowed-tools` restricts which tools the skill can use — a security boundary.
- `argument-hint` prompts the developer for a required input when the command is invoked without arguments.

> **Exercise:** Write the complete SKILL.md frontmatter for a `/analyze-module` skill that: runs in an isolated context, can only read files (not write or execute), and asks for a directory path if none is provided. Then write the first two lines of the skill's prompt content.

---

### Extension Question
A company decides to put all Claude Code configuration in the root CLAUDE.md — coding standards, API conventions, testing requirements, deployment rules, and React patterns all in one 800-line file. What are the practical problems with this approach, and how would you restructure it using the tools covered today?

---

<a id="day12"></a>
## Day 12 — Claude Code: Planning Mode, Session Management, and CI/CD
**Guide reference:** Chapter 5 (sections 5.6–5.10)

The second half of Claude Code covers strategic decisions: when to plan vs. act, how to manage context across long sessions, and how to wire Claude into automated pipelines.

---

### Key Concepts

**Planning mode vs. direct execution**
Planning mode lets Claude explore and plan without making changes. It uses Read, Grep, and Glob to understand the codebase, then produces a plan for human approval. Direct execution is for single-file fixes where the change is unambiguous.

> **Quick check:** For each task below, choose planning mode or direct execution and explain why:
> 1. Fix a NullPointerException in `OrderController.java` based on a stack trace.
> 2. Migrate a REST API from v1 to v2, affecting routing conventions across 45 files.
> 3. Add a missing null check to a validation function.
> 4. Restructure a monolith into microservices.
> 5. Rename a variable in a single utility file.

---

**`/compact` and `/memory`: managing long sessions**
`/compact` summarizes prior history to free context window space, but risks losing specific numeric values and dates. `/memory` opens CLAUDE.md for editing so you can persist findings across sessions.

> **Quick check:** You've been investigating an authentication bug for 45 minutes. Claude's context is filling up. You know three critical facts: (1) the bug is in `src/auth/session.ts` at line 142, (2) it only affects users with expired OAuth tokens, and (3) a migration from December 2024 introduced the issue. Before running `/compact`, what should you do with these facts, and why?

---

**`--resume` and `fork_session`: session management**
`--resume <session-name>` continues a named session with its prior context. `fork_session` creates an independent branch from shared context, useful for exploring two approaches in parallel without them interfering.

> **Exercise:** You're investigating whether to refactor a payments module using Redux or the Context API. Describe how you would use `fork_session` to explore both approaches. What would each fork's starting context look like, and at what point would you merge your findings?

---

**When to start a new session instead of resuming**
If files have changed significantly since a session was saved, or if the context has degraded to the point where Claude is referencing "typical patterns" instead of specific file contents, a new session with a structured summary is more reliable than resuming.

> **Quick check:** You resume a session from two weeks ago to continue debugging a performance issue. Claude immediately references a function that was renamed in last week's refactor. What does this tell you, and what is the correct next step?

---

**The `-p` flag for CI/CD: non-interactive mode**
`claude -p "..."` processes a prompt, prints to stdout, and exits without waiting for user input. This is the only correct way to run Claude Code in automated pipelines. Combined with `--output-format json` and `--json-schema`, it produces structured results that CI scripts can parse.

> **Exercise:** Write the shell command to run Claude Code in a CI pipeline that:
> 1. Reviews a pull request diff for security issues
> 2. Outputs structured JSON
> 3. Validates against a schema where findings have `file`, `line`, `severity`, and `description` fields
>
> Then explain why you would use a *separate* Claude instance for this review rather than the instance that generated the code.

---

**Session isolation for review quality**
The same Claude session that generated code retains its reasoning context and is less likely to challenge its own decisions. An independent instance — one that only sees the code, not how it was written — produces more objective reviews.

> **Quick check:** A team argues that using the same session for code generation and review is efficient because "Claude already understands the context." What is the specific failure mode this creates, and what does the data in the guide say about its prevalence?

---

### Extension Question
Your CI pipeline re-reviews a PR after every new commit, but developers complain that Claude keeps flagging the same issues that were already addressed in the previous review. What is causing this, and how do you fix it?

---

<a id="day13"></a>
## Day 13 — Escalation and Human-in-the-Loop
**Guide reference:** Chapter 9

Agents must know when to stop trying and hand off to a human. Escalation design is one of the most nuanced judgment areas on the exam — the line between "try to resolve" and "escalate immediately" depends on specific signals, not gut feel.

---

### Key Concepts

**Clear escalation triggers**
Reliable escalation triggers are specific and unambiguous: the customer explicitly asks for a human, the request falls outside policy, the agent cannot make progress after a reasonable attempt, or a financial threshold is exceeded. Each trigger requires a different response.

> **Quick check:** For each situation below, identify the correct escalation trigger and whether escalation should be immediate or after an attempt to resolve:
> 1. Customer says: "I want to speak to your manager, now."
> 2. Customer says: "This is unacceptable, I'm very upset."
> 3. Customer requests a price match against a competitor — your policy covers your own site only.
> 4. Customer has called three times this week about the same unresolved issue.
> 5. The agent has called `lookup_order` three times and received error responses each time.

---

**Unreliable escalation proxies**
Sentiment analysis (customer mood ≠ case complexity), model self-rated confidence (the model can be confidently wrong), and automatic classifiers (requires training data, often overengineered) are all unreliable as primary escalation triggers.

> **Quick check:** An engineer proposes this escalation logic: "If the customer uses more than three exclamation marks or the words 'furious,' 'unbelievable,' or 'lawsuit,' escalate immediately." Evaluate this proposal. What cases does it handle correctly? What cases does it get wrong in each direction (escalates when it shouldn't, or doesn't escalate when it should)?

---

**Three escalation patterns**
1. **Immediate**: explicit request for a human → escalate at once, no attempt to resolve
2. **Attempt-then-escalate**: issue is within scope → try to resolve, escalate if customer remains unsatisfied
3. **Acknowledge-resolve-escalate on reiteration**: customer expresses frustration → acknowledge, offer resolution, escalate only if they re-request a human

> **Exercise:** Write out the exact sequence of agent actions (including tool calls) for this scenario:
> *Customer: "My package was supposed to arrive Monday. It's Thursday. This is ridiculous — I need this for an event tomorrow. Can you please just sort this out or get me someone who can?"*
>
> Is escalation immediate, or does the agent attempt resolution first? Justify your answer with reference to the three patterns above.

---

**Structured handoff protocols**
When escalating, the agent must pass a complete, self-contained summary to the human operator. The operator has no access to the conversation transcript — the summary is all they have.

> **Exercise:** Write the JSON handoff summary for this case:
> - Customer: Marcus Webb, CUST-9921
> - Issue: Item arrived damaged (coffee maker, $189.99), order ORD-5541
> - Agent actions taken: verified identity, confirmed order, offered replacement — declined, customer wants refund
> - Escalation reason: customer said "I want to talk to a real person"
> - Recommended action: approve full refund
>
> Make the summary complete enough that a human operator with no prior context could handle the call confidently.

---

**Multiple matches and identity ambiguity**
When a search returns multiple customer records for the same name or email, the correct action is to ask for additional identifying information — not to guess, not to proceed with the most likely match.

> **Quick check:** `get_customer(email="j.smith@email.com")` returns two records: one in New York and one in Chicago. What does the agent say to the customer, and what additional information does it request? Why is heuristic selection (e.g., "pick the one with more recent activity") the wrong approach?

---

### Extension Question
An agent is configured with this escalation instruction: "Escalate to a human if you are not confident you can resolve the issue." After two weeks in production, the escalation rate is 60% — far above the 20% target. What is wrong with this instruction, and how would you rewrite it?

---

<a id="day14"></a>
## Day 14 — Error Handling in Multi-Agent Systems
**Guide reference:** Chapter 10

When a subagent fails, what the coordinator knows determines whether the system recovers gracefully or collapses entirely. This session is about designing for failure from the start.

---

### Key Concepts

**The four error categories**
| Category | Examples | Retryable | Agent action |
|---|---|---|---|
| Transient | Timeout, 503, network failure | Yes | Retry with exponential backoff |
| Validation | Invalid input format, missing required field | No (fix input) | Modify request and retry |
| Business | Policy violation, threshold exceeded | No | Explain to user; propose alternative |
| Permission | Access denied | No | Escalate |

> **Quick check:** Categorize each error below and describe what the agent should do next:
> 1. `get_customer` returns a 503 after a 10-second timeout.
> 2. `process_refund` fails because the order is outside the 30-day return window.
> 3. `lookup_order` fails because `order_id` was passed as a string instead of an integer.
> 4. `delete_record` returns "Access denied — admin role required."
> 5. A web-search MCP tool times out after 30 seconds on the second attempt.

---

**Why generic errors are an anti-pattern**
A response of `"Operation failed"` gives the coordinator nothing to work with — it can't decide whether to retry, change its approach, or escalate. Structured errors include the type, what was attempted, partial results if any, and possible recovery paths.

> **Exercise:** Rewrite this generic error as a structured error response:
> ```json
> { "isError": true, "content": "Search failed" }
> ```
> The context: a web search subagent timed out while searching for "Q4 2024 AI regulation changes in the EU." It found 2 relevant results before timing out, and two alternative queries might succeed.

---

**"No results" vs. "search failed": a critical distinction**
An empty result set (valid search, no matches) and a failed search (tool error, no results) require completely different coordinator responses. Conflating them causes the coordinator to silently miss coverage.

> **Quick check:** A web search returns `{"results": []}`. Is this an error? How should the coordinator interpret it differently from `{"isError": true, "content": "timeout"}`? Write the coordinator logic (in pseudocode) for handling each case.

---

**Anti-patterns: silent suppression, infinite retries, aborting on one failure**
Three common mistakes:
1. Returning an empty success result on failure (masks errors)
2. Infinite retries inside a subagent (wastes resources, blocks the coordinator)
3. Aborting the entire workflow when one subagent fails (throws away all partial results)

> **Quick check:** A 5-subagent research pipeline runs. Subagent 3 fails with a transient error. Describe what each of the three anti-patterns looks like in this scenario, and describe the correct behavior instead.

---

**Coverage annotations in final output**
When a subagent fails and the coordinator continues with partial results, the final output must be honest about what's missing. Annotations let the consumer of the report know where to trust the findings and where there are gaps.

> **Exercise:** A research report on "AI in Healthcare 2024" was produced by three subagents. The web search subagent fully succeeded. The document analysis subagent fully succeeded. The regulatory data subagent timed out and returned partial results covering the US only (EU and UK data is missing). Write the coverage annotations for the final report's table of contents.

---

### Extension Question
A colleague argues: "We should just wrap every subagent in a try-catch and return an empty result on any failure. It keeps the coordinator simple." Explain specifically why this is an anti-pattern and what it costs you in practice.

---

<a id="day15"></a>
## Day 15 — Week 3 Review
**Guide reference:** Exam Questions 4–6, 10–11, Domain 3 and Domain 5 notes

This is the Friday morning group session. Complete the Sample Exam Questions individually before meeting, then work through the reasoning together.

**Agenda:**
1. Work through Exam Questions 4–6 (Claude Code scenarios) and 10–11 (CI/CD scenarios) as a group.
2. Review Domain 3 (Claude Code Configuration and Workflows) and Domain 5 (Context Management and Reliability) notes in Part II of the guide.

---

## Sample Exam Questions

Work through these on your own before the Friday morning meeting. Commit to an answer for each before checking the answers section below.

### Question 4 — Scenario: Code Generation with Claude Code

**Situation:** You need a custom `/review` command for standard code review that must be available to the entire team automatically when they clone the repository.

**Where should you create the command file?**

- A) `.claude/commands/` in the project repository
- B) `~/.claude/commands/` on your local machine
- C) The root `CLAUDE.md` file
- D) `.claude/config.json`

---

### Question 5 — Scenario: Code Generation with Claude Code

**Situation:** You need to restructure a monolith into microservices. The task involves dozens of files and requires making non-obvious service boundary decisions.

**What approach should you use?**

- A) Planning mode: explore the codebase, understand dependencies, then design an approach for approval
- B) Direct execution, working incrementally file by file
- C) Direct execution with detailed up-front instructions covering all the files
- D) Start with direct execution and switch to planning mode when it gets difficult

---

### Question 6 — Scenario: Code Generation with Claude Code

**Situation:** A codebase has distinct conventions for React components, API routes, and database layers. Tests are co-located with source files throughout the repo. You want Claude Code to apply the right conventions automatically based on which file is being edited.

**What approach should you use?**

- A) `.claude/rules/` files with YAML frontmatter `paths` glob patterns
- B) Put all conventions in the root `CLAUDE.md`
- C) Create skills in `.claude/skills/` for each convention set
- D) Add a `CLAUDE.md` file inside every directory

---

### Question 10 — Scenario: Claude Code for CI

**Situation:** A CI pipeline runs `claude "Analyze this pull request for security issues"` but hangs indefinitely waiting for interactive input.

**What is the correct fix?**

- A) Use the `-p` flag: `claude -p "Analyze this pull request for security issues"`
- B) Set the environment variable `CLAUDE_HEADLESS=true`
- C) Redirect stdin from `/dev/null`
- D) Use the `--batch` flag

---

### Question 11 — Scenario: Claude Code for CI

**Situation:** A team runs two Claude-powered workflows: (1) a blocking pre-merge code review that developers wait on before merging, and (2) an overnight tech-debt report ready for morning review. A manager proposes moving both to the Message Batches API to save 50% on API costs.

**How should you evaluate this proposal?**

- A) Use batch processing for the tech-debt report only; keep real-time calls for pre-merge checks
- B) Move both workflows to batch processing and poll for results
- C) Keep real-time calls for both to avoid ordering issues in batch results
- D) Move both to batch processing with an automatic fallback to real-time if a batch exceeds a time limit

---

## Answers

### Question 4 — Correct answer: A

Project commands in `.claude/commands/` are version-controlled and automatically available to every contributor who clones the repository — no manual setup required. `~/.claude/commands/` (B) is for personal commands that are not shared via version control. Root `CLAUDE.md` (C) contains instructions, not command definitions. `.claude/config.json` (D) does not exist as a valid command location.

---

### Question 5 — Correct answer: A

Planning mode is designed precisely for this situation: large changes spanning many files, multiple viable approaches to service boundaries, and architectural decisions with long-term consequences. Direct execution (B, C) risks expensive rework if the wrong decomposition is chosen before the codebase is fully understood. Starting with direct execution (D) is reactive — by the time you realize planning is needed, partial changes may already be inconsistent with each other.

---

### Question 6 — Correct answer: A

`.claude/rules/` files with `paths` glob patterns load conventions **only** when Claude is editing a file that matches the pattern. This handles test conventions (`**/*.test.ts`) that apply regardless of directory, and different rules for API routes vs. React components, all automatically. A monolithic root `CLAUDE.md` (B) loads all conventions always, wasting context tokens. Skills (C) require manual invocation — they're on-demand, not automatic. A `CLAUDE.md` in every directory (D) doesn't work well for conventions like test rules that apply across many scattered locations.

---

### Question 10 — Correct answer: A

The `-p` (or `--print`) flag is the documented mechanism for running Claude Code non-interactively: it processes the prompt, prints to stdout, and exits without waiting for user input. `CLAUDE_HEADLESS=true` (B) is not a real environment variable. Redirecting stdin from `/dev/null` (C) is a Unix workaround that may cause other issues and is not the intended approach. `--batch` (D) does not exist as a Claude Code flag.

---

### Question 11 — Correct answer: A

The Message Batches API offers 50% cost savings but provides no latency SLA — processing can take up to 24 hours. This makes it entirely unsuitable for the pre-merge check, where developers are actively waiting and even a 30-minute delay is unacceptable. The overnight tech-debt report has no such constraint and is a perfect fit for batch processing. Moving both to batch (B, D) breaks the pre-merge workflow. Keeping both synchronous (C) forgoes the available cost savings on the overnight workload.

---

## Week 3 Capstone Exercise

**Scenario: Production-Grade Code Review Pipeline**

Your engineering team ships 15–20 pull requests per day. You want to integrate Claude Code into your CI/CD pipeline to automate code review. The goals are: catch security vulnerabilities, flag logical bugs, identify missing tests, and post structured inline comments on PRs — without drowning developers in false positives.

**Part A — Claude Code Configuration**
Design the CLAUDE.md and `.claude/rules/` setup for this project. Your codebase has:
- TypeScript API routes in `src/api/**/*`
- React components in `src/components/**/*`
- Test files co-located with source (`.test.ts` / `.test.tsx`)
- Shared utilities in `src/lib/**/*`

Specify what goes in the project-level CLAUDE.md, what goes in path-scoped rule files, and what (if anything) goes in personal user configuration.

**Part B — CI Pipeline Command**
Write the shell command(s) for your GitHub Actions workflow. The review should:
- Run in non-interactive mode
- Output structured JSON with fields: `file`, `line_start`, `line_end`, `category` (one of `security`, `logic_bug`, `missing_test`, `style`), `severity` (`critical`, `high`, `medium`, `low`), `description`, and `suggested_fix`
- Use a separate Claude instance from the one that generated the code

**Part C — False Positive Management**
After two weeks, developers report that 35% of `style` findings are flagged on code that already matches the project style guide. Describe:
1. The root cause (be specific)
2. The prompt engineering fix you'd apply
3. How you'd measure whether the fix worked

**Part D — Re-review Logic**
A PR is updated with a new commit addressing three of five prior findings. When Claude re-reviews, it re-flags all five issues, including the two that were already fixed. Write the updated CI step that prevents this, and explain the mechanism.

**Part E — Escalation Design**
Some findings in your reviews are ambiguous — Claude isn't sure whether a pattern is a bug or intentional design. Design a structured output field that captures this uncertainty, and describe the workflow that routes ambiguous findings to a human reviewer rather than posting them directly as PR comments.

---

*← [Back to Week 2](Week2.md) | [Continue to Week 4 →](Week4.md)*
