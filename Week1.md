# Week 1: Foundations — Talking to Claude and Getting Reliable Output

**Theme:** Before building anything complex, master the fundamentals of how Claude receives instructions, decides what to do, and returns structured, usable results. These skills underpin every other topic in the course.

**Exam domains covered:** Domain 4 (Prompt Engineering and Structured Output, 20%), Domain 5 (Context Management and Reliability, 15%)

### Quick Navigation Links
**Obsidian:** [Day 1](#Day%201%20—%20How%20Claude%20Actually%20Works%20(API%20Fundamentals)), [Day 2](#Day%202%20—%20Prompt%20Engineering%20Making%20Claude%20Do%20It%20Well), [Day 3](#Day%203%20—%20Giving%20Claude%20Abilities%20Tools%20and%20`tool_use`), [Day 4](#Day%204%20—%20Structured%20Output%20Getting%20Reliable%20Data%20Back), [Day 5](#Day%205%20—%20Week%201%20Review)

**Markdown:** [Day 1](#day1), [Day 2](#day2), [Day 3](#day3), [Day 4](#day4), [Day 5](#day5)

---
<a id="day1"></a>
## Day 1 — How Claude Actually Works (API Fundamentals)

**Guide reference:** Chapter 1

The starting point: Claude is a stateless function. Every call is independent, and you must send the full conversation history every time. This session reframes Claude from "chatbot" to "API endpoint with reasoning capabilities."

---

### Key Concepts

**The anatomy of an API request**
Every call to Claude includes `model`, `max_tokens`, `system`, `messages`, `tools`, and `tool_choice`. Understanding what each field does — and what happens when you omit one — is foundational.

> **Quick check:** Look at the request below. What will happen, and why?
>
> ```json
> {
>   "model": "claude-sonnet-4-6",
>   "max_tokens": 1024,
>   "messages": [
>     { "role": "user", "content": "What's the order status for customer 42?" }
>   ]
> }
> ```
>
> _(The model has no system prompt defining its role, no tools to look up order data, and no conversation history. It will respond in plain text with a guess or a request for more information — it cannot actually retrieve anything.)_

---

**Message roles: `user` and `assistant`**
The `messages` array tells Claude the full history of the conversation. Claude uses all of it to decide what to say next. The two roles are `user` (input to Claude) and `assistant` (output from Claude). The API enforces a strict alternating structure between them.

> **Note:** Tool use introduces a third kind of message content — `tool_result` blocks. These are covered in Day 3.

> **Quick check:** What does the `messages` array look like after two turns of a conversation where the user asks a question and Claude answers in prose? Sketch the array.

---

**The `stop_reason` field**
Every Claude response includes a `stop_reason`. This is the signal your code must inspect to control agent behavior. The four core values are `end_turn`, `tool_use`, `max_tokens`, and `stop_sequence`. We'll learn more about how `tool_use` fits into the agent loop on Day 3.

> **Quick check:** Match each scenario to the correct `stop_reason` and describe what your code should do next.
>
> 1. Claude finishes answering the user's question in prose.
> 2. Claude decides to look up a customer record.
> 3. Claude is mid-sentence when the response cuts off.
> 4. You passed `"stop_sequences": ["###"]` and Claude wrote `###`.

---

**The system prompt**
The system prompt defines Claude's role, constraints, and behavioral rules. It is passed separately from `messages` and takes priority over user input. Crucially, the exact wording of a system prompt can cause unintended side effects.

> **Quick check:** A system prompt says: _"Always verify the customer's identity before taking any action."_ Data shows the agent now calls `get_customer` even when the user just asks a general FAQ question. What caused this, and what is a more precise instruction that avoids the problem?

---

**The context window and the lost-in-the-middle effect**
The context window holds everything: system prompt, full message history, tool definitions, and tool results. Claude reliably processes content at the beginning and end of long inputs, but tends to miss content in the middle.

> **Quick check:** You have a long aggregated report to send Claude. You need it to prioritize three critical security findings and then follow up with action items. Where in the message should each piece go, and why?

---

### Extension Question

If Claude has no memory between calls, what are the implications for building a multi-turn customer support agent? What does your application code need to do that a human agent's brain does automatically?

---

<a id="day2"></a>
## Day 2 — Prompt Engineering: Making Claude Do It Well

**Guide reference:** Chapter 6 (sections 6.1–6.4)

You can't build reliable systems without knowing how to instruct Claude precisely. This session covers the core techniques that separate professional implementations from fragile demos.

---

### Key Concepts

**Few-shot prompting: examples over explanations**
Including 2–4 concrete input/output examples in your prompt is almost always more effective than a lengthy description. Claude generalizes the _pattern_ demonstrated by the examples to new inputs.

> **Exercise:** You're building a support ticket classifier. Without few-shot examples, Claude classifies "my account won't let me log in" as `billing` 30% of the time. Write two few-shot examples (one clear case and one ambiguous case with a rationale) that would help Claude correctly classify authentication issues.

---

**The five types of few-shot examples**
Different problems need different example types: ambiguous scenarios, output formatting, acceptable vs. problematic patterns, extraction from varied document formats, and informal/non-standard data.

> **Quick check:** A recipe extraction system needs to handle both "2 cups of flour" and "a generous handful of cheese." Which type of few-shot example is most useful here, and what would a good example look like?

---

**Explicit criteria vs. vague instructions**
"Be conservative" is an instruction; "flag a comment only if it contradicts the actual code behavior" is a criterion. The difference in reliability is enormous. Severity definitions with code examples are more effective than adjective-based rubrics.

> **Exercise:** Rewrite this vague instruction as an explicit set of criteria with at least two concrete examples:
> _"Review the code carefully and flag any security concerns. Be thorough but avoid false positives."_

---

**Prompt chaining: sequential steps for complex tasks**
Prompt chaining is the practice of breaking a complex task into a series of focused prompts and sending them one at a time, where the output of each step feeds into the next as input. Application code drives the sequence. This is more reliable than putting numbered steps in a single prompt — when Claude is asked to do multiple things in one call, it can shortchange later steps after spending attention on earlier ones (attention dilution).

```python
# Simplified example — no model, max_tokens, system prompt, or error handling shown.
# In production, each step's output should be validated before feeding it forward.

# Step 1: per-file analysis
response1 = client.messages.create(
    messages=[{"role": "user", "content": f"Analyze each of these files for local bugs:\n\n{files}"}]
)
local_bugs = response1.content[0].text

# Step 2: cross-file consistency
response2 = client.messages.create(
    messages=[{"role": "user", "content": f"Given these local bugs:\n\n{local_bugs}\n\nNow check for cross-file type consistency issues."}]
)
cross_file_issues = response2.content[0].text

# Step 3: ticket alignment
response3 = client.messages.create(
    messages=[{"role": "user", "content": f"Given these findings:\n\n{local_bugs}\n{cross_file_issues}\n\nDoes this change match the ticket description?\n\n{ticket}"}]
)
```

> **Quick check:** A pull request needs to be reviewed for (a) local bugs in each file, (b) cross-file type consistency, and (c) whether the change matches the ticket description. Why is a single prompt over all files worse than a chain of three focused prompts? What specifically goes wrong with the single-prompt approach?
> A single prompt over all files is prone to attention dilution: Claude may only focus on a few files and skip or gloss over others.

---

**Dynamic decomposition**
Dynamic decomposition is when Claude is asked to break an open-ended task into a series of focused subtasks itself, based on what it discovers during a planning phase. The planning phase produces a task list; that list is then executed using prompt chaining. The two approaches are complementary — dynamic decomposition handles the _planning_, prompt chaining handles the _execution_.

Prompt chaining is used directly when the steps are known in advance. Dynamic decomposition is used first when the right steps can only be determined by exploring the problem.

```python
# Simplified example — no model, max_tokens, system prompt, or error handling shown.
# In production, the generated plan should be reviewed before entering the execution loop
# to ensure the subtasks are appropriate and within expected scope.

# Phase 1: planning — Claude maps the problem and generates subtasks
plan_response = client.messages.create(
    messages=[{"role": "user", "content": f"Investigate why this API is returning incorrect results. Here is the codebase:\n\n{codebase}\n\nProduce a numbered list of subtasks to investigate, based on what you find."}]
)
plan = plan_response.content[0].text

# Phase 2: execute each subtask using prompt chaining, feeding results forward
results = []
for subtask in parse_subtasks(plan):  # parse_subtasks() extracts the numbered list into an iterable
    response = client.messages.create(
        messages=[{"role": "user", "content": f"Previous findings:\n\n{results}\n\nNow complete this subtask: {subtask}"}]
    )
    results.append(response.content[0].text)
```

---

**The interview pattern**
Before acting on an underspecified request, Claude asks targeted clarifying questions. This is especially valuable for tasks with non-obvious implications or multiple viable approaches.

> **Quick check:** You ask Claude to "clean up the database." What three clarifying questions should it ask before doing anything, and what could go wrong if it acts without asking?

---

### Extension Question

A colleague argues: "Few-shot examples just waste tokens — Claude already knows how to format JSON." When would you agree with them, and when would you push back? What's the specific failure mode that few-shot examples address that Claude's training alone doesn't?

---

<a id="day3"></a>
## Day 3 — Giving Claude Abilities: Tools and `tool_use`

**Guide reference:** Chapter 2 (sections 2.1–2.3)

Tools are how Claude takes action in the world. The mechanics of tool definitions are straightforward — the critical insight is that **tool descriptions are the primary selection mechanism**, not routing logic in your code.

---

### Key Concepts

**The three categories of tools**
Not all tools work the same way. Understanding which category a tool falls into determines what your application is responsible for. ([Reference](https://platform.claude.com/docs/en/agents-and-tools/tool-use/how-tool-use-works))

- **User-defined (client-executed):** You write the schema, you execute the code, you return the result via a `tool_result` block. This is the default model and the vast majority of tool-use traffic.
- **Anthropic-schema (client-executed):** `bash`, `text_editor`, `computer`, `memory`. Anthropic publishes the schema — which is trained-in, making these tools more reliably called — but your application still handles execution and returns results the same way as user-defined tools.
- **Server-executed:** `web_search`, `web_fetch`, `code_execution`, `tool_search`. Anthropic runs everything. You enable the tool in your request; by the time a response reaches you, execution is already complete. The response contains `server_tool_use` blocks (not `tool_use`), and you never construct a `tool_result` for these. The tradeoff: less control over what data enters Claude's context window. ([Web search reference](https://platform.claude.com/docs/en/agents-and-tools/tool-use/web-search-tool))

---

**What `tool_use` is — and isn't**
Claude does not execute code. When it wants to use a tool, it generates a structured call request. Your code receives that request, executes it, and returns the result. Claude never touches your database directly.

When Claude makes a tool call, the response has `stop_reason: "tool_use"`. Your code executes the tool and returns the result as a `tool_result` content block inside a `user`-role message — there is no `"role": "tool"` in the API. Tool results ride inside a `user` message because the API maintains a strict alternating `user`/`assistant` turn structure, and tool results are external input being fed _into_ Claude.

A `tool_result` block has the following structure:

```json
{
  "role": "user",
  "content": [
    {
      "type": "tool_result",
      "tool_use_id": "<id from the assistant's tool_use block>",
      "content": "<string result, or an array of content blocks>"
    }
  ]
}
```

The `tool_use_id` ties the result back to the specific tool call Claude made — this matters when Claude calls multiple tools in one turn.

The canonical agent loop for client-executed tools looks like this:

```python
# Simplified example — no model, max_tokens, system prompt, or error handling shown.
while response.stop_reason == "tool_use":
    tool_results = execute_tools(response)   # your code runs the tools
    response = client.messages.create(
        messages=[*messages, response, tool_results]
    )
# stop_reason is now "end_turn", "max_tokens", or "stop_sequence"
```

For server-executed tools (`web_search`, `web_fetch`, etc.), a fifth `stop_reason` applies: `pause_turn`. This signals that the server-side loop hit its iteration limit mid-turn — the application should resend the full conversation to let the model continue where it left off.

> **Quick check:** You call Claude, it responds with a tool call, and you execute the tool. What does the next message you send to Claude look like? Sketch the `messages` array at that point, including both the assistant's tool call and your tool result.

> **Quick check:** True or false, and explain: "If I define a `delete_record` tool but Claude's system prompt says it should never delete anything, the record is safe."
> _(False. Claude might avoid calling it most of the time, but prompt instructions are probabilistic. For destructive operations, use a hook or precondition at the code level — not just the prompt.)_

---

**Anatomy of a tool definition**
Each tool has a `name`, a `description`, and an `input_schema`. The schema enforces the structure of Claude's call; the description determines _whether_ Claude calls it at all.

Within `properties`, all standard JSON Schema types are available: `"string"`, `"integer"`, `"number"`, `"boolean"`, `"array"` (with an `"items"` field to define element type), `"object"` (for nested structures), `"null"`, and union arrays like `["string", "null"]` for fields that may be absent. The `["string", "null"]` pattern is particularly useful for optional fields where you want Claude to explicitly return `null` rather than omitting the field entirely.

The top-level `input_schema` uses `"type": "object"` in all documented examples and the API design implies it — Claude always returns tool inputs as a named JSON object — but the docs do not explicitly state it as a hard constraint.

> **Exercise:** Write a tool definition for a `send_notification` tool that sends a push notification to a user. It takes a `user_id` (integer), a `message` (string, max 160 characters), and an optional `priority` (one of `"low"`, `"normal"`, `"urgent"`). Write the description as if Claude will use it alongside a `send_email` tool — make sure the description clearly tells Claude when to prefer one over the other.

---

**Why descriptions are the primary selection mechanism**
An LLM picks tools based on their descriptions. When descriptions are vague or overlap, the model picks inconsistently. There is no routing layer between Claude and your tools — the description _is_ the router.

> **Quick check:** You have two tools: `analyze_content` and `analyze_document`. Both descriptions say "analyzes provided content and returns a summary." Users report Claude picks them seemingly at random. Without changing any code, what single change would most improve reliability?

---

**The `tool_choice` parameter**
`tool_choice` gives you control over whether and how Claude picks tools. `auto` lets Claude decide; `any` forces a tool call of Claude's choosing; `{"type": "tool", "name": "X"}` forces a specific tool.

> **Quick check:** For each scenario, choose the correct `tool_choice` value and explain why:
>
> 1. You want Claude to extract structured data from a document, and you don't care which of three extraction tools it uses — but you need a tool call, not prose.
> 2. You're building a customer lookup flow where `get_customer` must always run first, before any other tool.
> 3. You want Claude to answer simple questions in plain text when no tool is needed, but use tools when appropriate.

---

### Extension Question

You have a customer support agent with four tools: `get_customer`, `lookup_order`, `process_refund`, and `escalate_to_human`. How would you write the description for `get_customer` to make it clear when it should and shouldn't be used, and how would you distinguish it from `lookup_order`?

---

<a id="day4"></a>
## Day 4 — Structured Output: Getting Reliable Data Back

**Guide reference:** Chapter 2 (sections 2.4–2.5), Chapter 6 (sections 6.5–6.6)

This session ties tools and prompt engineering into a complete data extraction pattern. The goal: structured, validated JSON that your downstream code can actually rely on.

> **Key insight before you start:** The reliable way to get structured output from Claude is to define a tool whose `input_schema` is the shape you want, force Claude to call it with `tool_choice: {"type": "tool", "name": "..."}`, and read the `input` field of the resulting `tool_use` block. The tool is fictional — you never execute it. You are using the tool-calling mechanism as a structured output contract. This is why Day 3 and Day 4 are closely linked: `tool_use` is not only for calling external functions, it is also the primary mechanism for guaranteeing structured JSON output. Asking Claude to "respond in JSON" in a regular message is unreliable — Claude can produce malformed JSON, omit required fields, or add commentary outside the block. A tool schema eliminates all of these failure modes at the API level.

---

### Key Concepts

**JSON schemas as the structured output mechanism**
Using `tool_use` with a JSON schema is the most reliable way to get structured output. The schema guarantees syntactic validity — Claude cannot return malformed JSON or missing required fields.

> **Quick check:** A JSON schema marks `category` as required and `notes` as optional with type `["string", "null"]`. Claude extracts from a document that contains no notes. What does Claude return for the `notes` field, and why is this better than making `notes` required?

---

**Schema design rules**
Good schemas are explicit about what's required, use nullable types for absent data, and include escape hatches like `"other"` and `"unclear"` enum values so Claude doesn't hallucinate a category that doesn't fit.

When an enum field includes an escape hatch value, the standard practice is to add a separate sibling property (e.g. `category_detail`) in the `properties` object, typed as `["string", "null"]`. It should be nullable so Claude leaves it empty for the standard enum values, and its description should explicitly state when it must be populated. The conditional validation — requiring `category_detail` when `category` is `"other"` — lives in application code rather than the schema itself.

```json
"category": {
  "type": "string",
  "enum": ["hardware", "software", "services", "other"]
},
"category_detail": {
  "type": ["string", "null"],
  "description": "Required when category is 'other'. Provide a short description of the actual category."
}
```

> **Exercise:** You're extracting invoice data. Design a JSON schema for a single line item that includes: `description` (always present), `unit_price` (always present), `quantity` (always present), `discount_percent` (sometimes absent), and `category` (one of `"hardware"`, `"software"`, `"services"`, `"other"` with a detail field if `"other"`). Apply the schema design rules correctly.

---

**Syntax errors vs. semantic errors**
A schema eliminates syntax errors (malformed JSON, wrong field types). It does not prevent semantic errors — values that are structurally valid but factually wrong, arithmetically inconsistent, or in the wrong field.

> **Quick check:** Give two examples of semantic errors that would pass a JSON schema validator but still represent incorrect extraction results. For each, describe a programmatic check that would catch it.

---

**Retry-with-feedback loops**
When validation fails, re-prompt Claude with the original document, the incorrect extraction, and the specific error. This approach fixes format and structural errors effectively — but not errors caused by missing source data.

> **Quick check:** You run a retry after a Pydantic validation error. The retry produces the same wrong value. What two things might explain this, and what would you do differently in each case?

---

**Self-correction schemas**
A schema can be designed to extract both a stated value and a computed value, allowing your code to detect inconsistencies automatically without needing a separate validation pass. The key insight is that Claude does not do the arithmetic — the schema asks it to extract what the document _states_ as the total and the individual line item prices separately. Your code then sums the line items and compares.

A `conflict_detected` boolean field is included as Claude's own soft assessment of whether the values appear consistent. Your application code should always independently verify with its own arithmetic check. The reason to have both is that they tell you different things: if both Claude and your code agree there is a discrepancy, a retry is worth attempting — Claude saw something inconsistent in the source document. If your code catches a discrepancy that Claude missed (`conflict_detected: false`), Claude likely misread a value, which a retry is less likely to fix and warrants human review instead.

```json
{
  "type": "object",
  "properties": {
    "stated_total": {
      "type": "number",
      "description": "The grand total as explicitly written in the document."
    },
    "line_items": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "name": { "type": "string" },
          "price": { "type": "number" }
        },
        "required": ["name", "price"]
      }
    },
    "conflict_detected": {
      "type": "boolean",
      "description": "Set to true if the stated_total does not appear to match the sum of line item prices. Set to false if they appear consistent."
    }
  },
  "required": ["stated_total", "line_items", "conflict_detected"]
}
```

```python
result = extract_invoice(document)

# Always compute the sum independently — do not rely on Claude's arithmetic
calculated_total = sum(item["price"] for item in result["line_items"])
discrepancy = abs(result["stated_total"] - calculated_total)

if discrepancy > 0.01:  # tolerance for floating point
    if result["conflict_detected"]:
        # Claude also noticed — likely a genuine document error or extraction issue
        # Retry with specific feedback
        retry_with_feedback(
            document,
            result,
            error=f"stated_total is {result['stated_total']} but line items sum to {calculated_total:.2f}. Re-check all values."
        )
    else:
        # Claude missed the discrepancy — more likely a misread value than a document error
        # A retry is less likely to help; flag for human review instead
        flag_for_review(result, reason="arithmetic discrepancy not detected by model")
```

> **Exercise:** Design a JSON schema to extract a social media profile summary. The document contains a `username`, a stated `total_likes` count, and a list of `tweets` each with a `content` and `likes` field. How do you structure the schema so that your code can detect if the stated total doesn't match the sum of individual tweet likes — without Claude needing to do the arithmetic itself?

---

### Extension Question

A data extraction system achieves 97% accuracy overall — but when you break it down by document type, accuracy on handwritten invoices is only 58%. The schema is correctly designed. What does this tell you about where to focus your improvement efforts, and what techniques from this week would you apply?

---

<a id="day5"></a>
## Day 5 — Week 1 Review

**Guide reference:** Exam Questions 1–3, Domain 4 notes (sections 4.1–4.4)

This is the Friday morning group session. Complete the Sample Exam Questions individually before meeting, then work through the reasoning together. Focus on the _reasoning_ behind each answer, not just the correct choice.

**Agenda:**

1. Work through Exam Questions 1–3 as a group. For each, have someone argue for the wrong answers before revealing the correct one.
2. Review Domain 4 notes (Prompt Engineering and Structured Output) in Part II of the guide.

---

## Sample Exam Questions

Work through these on your own before the Friday morning meeting. Commit to an answer for each before checking the answers section below.

### Question 1 — Scenario: Customer Support Agent

**Situation:** Data shows that in 12% of cases the agent skips `get_customer` and calls `lookup_order` using only the customer's name, which leads to incorrect refunds.

**Which change is most effective?**

- A) Add a programmatic precondition that blocks `lookup_order` and `process_refund` until a verified customer ID is obtained from `get_customer`
- B) Improve the system prompt to emphasize the correct tool order
- C) Add few-shot examples demonstrating the correct sequence
- D) Implement a routing classifier to detect order-related queries

---

### Question 2 — Scenario: Customer Support Agent

**Situation:** The agent often calls `get_customer` instead of `lookup_order` for order-related questions. Both tools have minimal, nearly identical descriptions.

**What is the first step to fix this?**

- A) Add few-shot examples showing correct tool selection
- B) Expand each tool's description with input formats, example queries, and clear boundaries for when to use each
- C) Add a routing layer that pre-classifies user intent before passing to the agent
- D) Merge the two tools into one unified lookup tool

---

### Question 3 — Scenario: Customer Support Agent

**Situation:** The agent resolves only 55% of issues against a target of 80%. It escalates simple cases and attempts to autonomously handle complex policy exceptions it shouldn't touch.

**How do you improve escalation calibration?**

- A) Add explicit escalation criteria with few-shot examples directly in the system prompt
- B) Add a self-rated confidence score (1–10) and escalate automatically below a threshold
- C) Train a separate classifier on historical escalation data
- D) Use sentiment analysis to detect frustrated customers and escalate those cases

---

## Answers

### Question 1 — Correct answer: A

A programmatic precondition provides a **deterministic** guarantee that `get_customer` runs before `lookup_order` and `process_refund`. Prompt improvements (B) and few-shot examples (C) are probabilistic — they reduce violations but cannot eliminate them, as the 12% failure rate demonstrates. A routing classifier (D) addresses intent classification, not tool ordering enforcement.

---

### Question 2 — Correct answer: B

Tool descriptions are the model's **primary selection mechanism** — the LLM picks tools based on their descriptions, not routing code. Minimal or overlapping descriptions are the root cause of misrouting, making this the highest-impact, lowest-effort fix. Few-shot examples (A) add tokens without addressing why the model is confused. A routing layer (C) is overengineering. Merging the tools (D) removes a useful distinction and requires more work than simply improving descriptions.

---

### Question 3 — Correct answer: A

Explicit escalation criteria with few-shot examples directly address the root cause: the agent has unclear decision boundaries. Self-rated confidence (B) is unreliable — models can be confidently wrong, and confidence scores are poorly calibrated for this task. A trained classifier (C) is overengineering that requires labeled historical data. Sentiment analysis (D) conflates customer mood with case complexity; a frustrated customer asking a simple question should not be escalated.

---

## Week 1 Capstone Exercise

**Scenario: Invoice Extraction System**

You are building a system to extract structured data from supplier invoices. Invoices arrive as plain text and vary significantly in format — some are formal with line-item tables, some are informal emails, some reference vague quantities like "a few units" or "half a dozen."

Your task is to design the complete extraction pipeline for a single invoice. Work through each step:

**Part A — Schema Design**
Design a JSON schema for an invoice that captures: invoice number, vendor name, invoice date, due date, a list of line items (each with description, quantity, unit price, and line total), stated grand total, and currency. Apply all schema design rules: required vs. optional, nullable fields, enum escape hatches where appropriate. Add a `conflict_detected` flag that signals when the sum of line item totals doesn't match the stated grand total.

**Part B — Tool Definition**
Write the tool definition that will wrap your schema. Write the description as if this tool will exist alongside a `validate_vendor` tool and a `flag_for_review` tool — make sure it's clear when each should be used.

**Part C — Few-shot Examples**
Write two extraction examples that demonstrate how to handle:

1. An informal quantity like "half a dozen filters"
2. A line item where the unit price is missing but the line total is present

**Part D — Retry Prompt**
Your first extraction attempt returns `grand_total: 1500.00` but the sum of line items is `1425.00`. `conflict_detected` is `true`. Write the retry prompt you would send to Claude, including everything it needs to correct the extraction.

**Part E — Failure Analysis**
After running your system on 200 invoices, you find it fails consistently on one type. Your logging system tracks failures, error messages, and input file metadata. Describe the diagnostic approach you'd use to identify the problem, and what technique from this week you'd apply to fix it.

---

_Continue to [Week 2 →](Week2.md)_
