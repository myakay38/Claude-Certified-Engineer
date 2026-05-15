# Claude Certified Architect — Overview

## How to Use This Study Guide

### Learning order

The guide's chapters are organized by technology (API, tools, Agent SDK, MCP, etc.), which makes a good reference but a poor learning sequence. This study plan reorders the material so that each concept builds directly on what came before — starting with how to talk to Claude, then how to give it abilities, then how to build agents, and so on. Resist the urge to skip ahead; the later weeks assume the earlier ones.

### Weekly rhythm

**Monday–Thursday** — work through one daily session per day on your own. Each session covers one topic and is designed for **60–90 minutes** of individual study. Read the day's section, work through the Quick Checks and Exercises, and note any Extension Questions you find particularly interesting or challenging.

**Friday morning** — the group meets to go through that week's Sample Exam Questions together. Commit to your answers individually before the meeting, then work through the reasoning as a group. Bring any Extension Questions from the week that you think are worth discussing — not all of them need to be covered, just the ones that sparked questions or disagreement.

**Friday afternoon** — complete the week's Capstone exercise individually. If you get stuck or want a second opinion on an approach, message or call a teammate. There are no single correct answers; the goal is to practice making and defending architectural decisions.

### Question types

**Quick Checks** are short comprehension questions attached to individual concepts. They're designed to be answered in under a minute — the goal is to catch misunderstandings before they compound. If one stumps you, re-read the relevant concept before moving on.

**Exercises** ask you to produce something: write a tool definition, sketch a message array, design a schema. They're the core of each daily session and typically take 10–20 minutes. There's rarely one correct answer — note where you're uncertain so you can raise it on Friday.

**Extension Questions** appear at the end of each day. These are open-ended and deliberately harder than what the exam will ask. They're optional during your individual sessions, but flag any that seem particularly interesting for the Friday morning discussion.

### Friday morning: exam question review

Each review day in Weeks 1–3 includes a set of **Sample Exam Questions** followed by an **Answers** section. Work through the questions on your own before Friday — commit to an answer for each option without peeking. During the group meeting, focus on the reasoning behind each answer, especially questions where people disagreed. The explanation of why each wrong answer fails is as important as identifying the correct one.

Week 4's Day 19 is the full 60-question practice test from the guide. Work through it individually during the week and time yourself (roughly 90 minutes for 60 questions). Use the Friday meeting to compare results and identify weak spots before the exam.

### Exam logistics

- Passing score: **720 out of 1000**
- No guessing penalty — answer every question
- 4 of 8 possible scenarios are randomly selected on the real exam
- Format: multiple choice, one correct answer out of four

---

## Overview

| Week | Theme                                                       | Exam Domains                                   | Detail               |
| ---- | ----------------------------------------------------------- | ---------------------------------------------- | -------------------- |
| 1    | Foundations — Talking to Claude and Getting Reliable Output | Domain 4 (20%), Domain 5 (15%)                 | [Week1.md](Week1.md) |
| 2    | Building Agents — Systems That Act Autonomously             | Domain 1 (27%), Domain 2 (18%)                 | [Week2.md](Week2.md) |
| 3    | Developer Workflows and Reliability                         | Domain 3 (20%), Domain 5 (15%)                 | [Week3.md](Week3.md) |
| 4    | Advanced Architecture and Exam Preparation                  | Domain 1 (27%), Domain 4 (20%), Domain 5 (15%) | [Week4.md](Week4.md) |

---

## Week 1: Foundations

**[→ Full week guide](Week1.md)**

Before building anything complex, master how Claude receives instructions, decides what to do, and returns structured, usable results.

| Day | Topic                                         | Guide chapters                 |
| --- | --------------------------------------------- | ------------------------------ |
| 1   | How Claude Actually Works (API Fundamentals)  | Ch 1                           |
| 2   | Giving Claude Abilities: Tools and `tool_use` | Ch 2 (2.1–2.3)                 |
| 3   | Prompt Engineering: Making Claude Do It Well  | Ch 6 (6.1–6.4)                 |
| 4   | Structured Output: Getting Reliable Data Back | Ch 2 (2.4–2.5), Ch 6 (6.5–6.6) |
| 5   | Review and Practice                           | Exam Qs 1–3, Domain 4 notes    |

**Capstone:** End-to-end invoice extraction pipeline — schema design, tool definition, few-shot examples, retry prompts, and failure analysis.

---

## Week 2: Building Agents

**[→ Full week guide](Week2.md)**

Move from single API calls to multi-step autonomous agents and multi-agent systems using the Agent SDK, hooks, and MCP.

| Day | Topic                                            | Guide chapters                  |
| --- | ------------------------------------------------ | ------------------------------- |
| 6   | The Agentic Loop                                 | Ch 3 (3.1–3.2)                  |
| 7   | Multi-Agent Systems: Coordinator and Subagents   | Ch 3 (3.3–3.4)                  |
| 8   | Hooks: Deterministic Control Over Agent Behavior | Ch 3 (3.5), Domain 1 (1.5)      |
| 9   | Connecting Claude to the World: MCP              | Ch 4                            |
| 10  | Review and Practice                              | Exam Qs 7–9, Domain 1 & 2 notes |

**Capstone:** Multi-agent customer support system — agent design, context passing, hook implementation, escalation logic, and failure analysis.

---

## Week 3: Developer Workflows and Reliability

**[→ Full week guide](Week3.md)**

Apply the Week 1–2 foundations to real engineering workflows with Claude Code, then tackle escalation, error handling, and context degradation.

| Day | Topic                                                     | Guide chapters                         |
| --- | --------------------------------------------------------- | -------------------------------------- |
| 11  | Claude Code: Configuration and Developer Workflows        | Ch 5 (5.1–5.6)                         |
| 12  | Claude Code: Planning Mode, Session Management, and CI/CD | Ch 5 (5.6–5.10)                        |
| 13  | Escalation and Human-in-the-Loop                          | Ch 9                                   |
| 14  | Error Handling in Multi-Agent Systems                     | Ch 10                                  |
| 15  | Review and Practice                                       | Exam Qs 4–6, 10–11, Domain 3 & 5 notes |

**Capstone:** Production-grade CI/CD code review pipeline — configuration design, CI commands, false positive management, re-review logic, and uncertainty routing.

---

## Week 4: Advanced Architecture and Exam Preparation

**[→ Full week guide](Week4.md)**

Context management at scale, task decomposition strategy, batch processing — then consolidate with the full 60-question practice exam.

| Day | Topic                                                | Guide chapters |
| --- | ---------------------------------------------------- | -------------- |
| 16  | Context Management in Production Systems             | Ch 11, 12      |
| 17  | Task Decomposition: Designing the Shape of Workflows | Ch 8           |
