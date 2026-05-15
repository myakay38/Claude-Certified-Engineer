# Week 1: Foundations — Talking to Claude and Getting Reliable Output

**Theme:** Before building anything complex, master the fundamentals of how Claude receives instructions, decides what to do, and returns structured, usable results. These skills underpin every other topic in the course.

**Exam domains covered:** Domain 4 (Prompt Engineering and Structured Output, 20%), Domain 5 (Context Management and Reliability, 15%)

---

## Day 1 — How Claude Actually Works (API Fundamentals)
**Guide reference:** Chapter 1

The starting point: Claude is a stateless function. Every call is independent, and you must send the full conversation history every time. This session reframes Claude from "chatbot" to "API endpoint with reasoning capabilities."

---

### Key Concepts

**The anatomy of an API request**
Every call to Claude includes `model`, `max_tokens`, `system`, `messages`, `tools`, and `tool_choice`. Understanding what each field does — and what happens when you omit one — is foundational.

> **Quick check:** Look at the request below. What will happen, and why?
> ```json
> { "model": "claude-sonnet-4-6", "max_tokens": 1024, "messages": [{"role": "user", "content": "What's the order status for customer 42?"}] }
> ```
> *(The model has no system prompt defining its role, no tools to look up order data, and no conversation history. It will respond in plain text with a guess or a request for more information — it cannot actually retrieve anything.)*

---

**Message roles: `user`, `assistant`, and tool results**
The `messages` array tells Claude the full history of the conversation. Claude uses all of it to decide what to say next. Tool results appear as `tool_result` content blocks within a `user`-role message.

> **Quick check:** You call Claude, it responds with a tool call, and you execute the tool. What does the next message you send to Claude look like? Sketch the `messages` array at that point, including both the assistant's tool call and your tool result.

---

**The `stop_reason` field**
Every Claude response includes a `stop_reason`. This is the signal your code must inspect to control agent behavior. The four values are `end_turn`, `tool_use`, `max_tokens`, and `stop_sequence`.

> **Quick check:** Match each scenario to the correct `stop_reason` and describe what your code should do next.
> 1. Claude finishes answering the user's question in prose.
> 2. Claude decides to look up a customer record.
> 3. Claude is mid-sentence when the response cuts off.
> 4. You passed `"stop_sequences": ["###"]` and Claude wrote `###`.

---

**The system prompt**
The system prompt defines Claude's role, constraints, and behavioral rules. It is passed separately from `messages` and takes priority over user input. Crucially, the exact wording of a system prompt can cause unintended side effects.

> **Quick check:** A system prompt says: *"Always verify the customer's identity before taking any action."* Data shows the agent now calls `get_customer` even when the user just asks a general FAQ question. What caused this, and what is a more precise instruction that avoids the problem?

---

**The context window and the lost-in-the-middle effect**
The context window holds everything: system prompt, full message history, tool definitions, and tool results. Claude reliably processes content at the beginning and end of long inputs, but tends to miss content in the middle.

> **Quick check:** You have a long aggregated report to send Claude. You need it to prioritize three critical security findings and then follow up with action items. Where in the message should each piece go, and why?

---

### Extension Question
If Claude has no memory between calls, what are the implications for building a multi-turn customer support agent? What does your application code need to do that a human agent's brain does automatically?

---

## Day 2 — Giving Claude Abilities: Tools and `tool_use`
**Guide reference:** Chapter 2 (sections 2.1–2.3)

Tools are how Claude takes action in the world. The mechanics of tool definitions are straightforward — the critical insight is that **tool descriptions are the primary selection mechanism**, not routing logic in your code.

---

### Key Concepts

**What `tool_use` is — and isn't**
Claude does not execute code. When it wants to use a tool, it generates a structured call request. Your code receives that request, executes it, and returns the result. Claude never touches your database directly.

> **Quick check:** True or false, and explain: "If I define a `delete_record` tool but Claude's system prompt says it should never delete anything, the record is safe."
> *(False. Claude might avoid calling it most of the time, but prompt instructions are probabilistic. For destructive operations, use a hook or precondition at the code level — not just the prompt.)*

---

**Anatomy of a tool definition**
Each tool has a `name`, a `description`, and an `input_schema`. The schema enforces the structure of Claude's call; the description determines *whether* Claude calls it at all.

> **Exercise:** Write a tool definition for a `send_notification` tool that sends a push notification to a user. It takes a `user_id` (integer), a `message` (string, max 160 characters), and an optional `priority` (one of `"low"`, `"normal"`, `"urgent"`). Write the description as if Claude will use it alongside a `send_email` tool — make sure the description clearly tells Claude when to prefer one over the other.

---

**Why descriptions are the primary selection mechanism**
An LLM picks tools based on their descriptions. When descriptions are vague or overlap, the model picks inconsistently. There is no routing layer between Claude and your tools — the description *is* the router.

> **Quick check:** You have two tools: `analyze_content` and `analyze_document`. Both descriptions say "analyzes provided content and returns a summary." Users report Claude picks them seemingly at random. Without changing any code, what single change would most improve reliability?

---

**The `tool_choice` parameter**
`tool_choice` gives you control over whether and how Claude picks tools. `auto` lets Claude decide; `any` forces a tool call of Claude's choosing; `{"type": "tool", "name": "X"}` forces a specific tool.

> **Quick check:** For each scenario, choose the correct `tool_choice` value and explain why:
> 1. You want Claude to extract structured data from a document, and you don't care which of three extraction tools it uses — but you need a tool call, not prose.
> 2. You're building a customer lookup flow where `get_customer` must always run first, before any other tool.
> 3. You want Claude to answer simple questions in plain text when no tool is needed, but use tools when appropriate.

---

### Extension Question
You have a customer support agent with four tools: `get_customer`, `lookup_order`, `process_refund`, and `escalate_to_human`. How would you write the description for `get_customer` to make it clear when it should and shouldn't be used, and how would you distinguish it from `lookup_order`?

---

## Day 3 — Prompt Engineering: Making Claude Do It Well
**Guide reference:** Chapter 6 (sections 6.1–6.4)

You can't build reliable systems without knowing how to instruct Claude precisely. This session covers the core techniques that separate professional implementations from fragile demos.

---

### Key Concepts

**Few-shot prompting: examples over explanations**
Including 2–4 concrete input/output examples in your prompt is almost always more effective than a lengthy description. Claude generalizes the *pattern* demonstrated by the examples to new inputs.

> **Exercise:** You're building a support ticket classifier. Without few-shot examples, Claude classifies "my account won't let me log in" as `billing` 30% of the time. Write two few-shot examples (one clear case and one ambiguous case with a rationale) that would help Claude correctly classify authentication issues.

---

**The five types of few-shot examples**
Different problems need different example types: ambiguous scenarios, output formatting, acceptable vs. problematic patterns, extraction from varied document formats, and informal/non-standard data.

> **Quick check:** A recipe extraction system needs to handle both "2 cups of flour" and "a generous handful of cheese." Which type of few-shot example is most useful here, and what would a good example look like?

---

**Explicit criteria vs. vague instructions**
"Be conservative" is an instruction; "flag a comment only if it contradicts the actual code behavior" is a criterion. The difference in reliability is enormous. Severity definitions with code examples are more effective than adjective-based rubrics.

> **Exercise:** Rewrite this vague instruction as an explicit set of criteria with at least two concrete examples:
> *"Review the code carefully and flag any security concerns. Be thorough but avoid false positives."*

---

**Prompt chaining: sequential steps for complex tasks**
Breaking a complex task into a chain of focused prompts — each with one job — prevents attention dilution and produces more consistent quality than a single prompt trying to do everything.

> **Quick check:** You need to review a pull request for (a) local bugs in each file, (b) cross-file type consistency, and (c) whether the change matches the ticket description. Why is a single prompt over all files worse than a chain of three focused prompts? What specifically goes wrong with the single-prompt approach?

---

**The interview pattern**
Before acting on an underspecified request, Claude asks targeted clarifying questions. This is especially valuable for tasks with non-obvious implications or multiple viable approaches.

> **Quick check:** You ask Claude to "add caching to the API." Name three clarifying questions it should ask before writing any code, and explain what could go wrong if each question isn't answered first.

---

### Extension Question
A colleague argues: "Few-shot examples just waste tokens — Claude already knows how to format JSON." When would you agree with them, and when would you push back? What's the specific failure mode that few-shot examples address that Claude's training alone doesn't?

---

## Day 4 — Structured Output: Getting Reliable Data Back
**Guide reference:** Chapter 2 (sections 2.4–2.5), Chapter 6 (sections 6.5–6.6)

This session ties tools and prompt engineering into a complete data extraction pattern. The goal: structured, validated JSON that your downstream code can actually rely on.

---

### Key Concepts

**JSON schemas as the structured output mechanism**
Using `tool_use` with a JSON schema is the most reliable way to get structured output. The schema guarantees syntactic validity — Claude cannot return malformed JSON or missing required fields.

> **Quick check:** A JSON schema marks `category` as required and `notes` as optional with type `["string", "null"]`. Claude extracts from a document that contains no notes. What does Claude return for the `notes` field, and why is this better than making `notes` required?

---

**Schema design rules**
Good schemas are explicit about what's required, use nullable types for absent data, and include escape hatches like `"other"` and `"unclear"` enum values so Claude doesn't hallucinate a category that doesn't fit.

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
A schema can be designed to extract both a stated value and a computed value, allowing your code to detect inconsistencies automatically without needing a separate validation pass.

> **Exercise:** Design a JSON schema for an expense report where the document contains a stated `total_amount` and a list of `line_items` with individual prices. How do you structure the schema so that your code can detect if the stated total doesn't match the sum of line items — without Claude needing to do the arithmetic itself?

---

### Extension Question
A data extraction system achieves 97% accuracy overall — but when you break it down by document type, accuracy on handwritten invoices is only 58%. The schema is correctly designed. What does this tell you about where to focus your improvement efforts, and what techniques from this week would you apply?

---

## Day 5 — Week 1 Review
**Guide reference:** Exam Questions 1–3, Domain 4 notes (sections 4.1–4.4)

Consolidate the week's material by working through exam-style scenarios together. Focus on the *reasoning* behind each answer, not just the correct choice.

**Agenda:**
1. Work through Exam Questions 1–3 from the guide as a group. For each, have someone argue for the wrong answers before revealing the correct one.
2. Review Domain 4 notes (Prompt Engineering and Structured Output) in Part II of the guide.

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
After running your system on 200 invoices, you find it fails consistently on one type. Describe the diagnostic approach you'd use to identify which document type is failing, what information you'd need, and what technique from this week you'd apply to fix it.

---
*Continue to [Week 2 →](Week2.md)*
