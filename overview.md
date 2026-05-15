# Claude Certified Architect — Foundations
## 4-Week Team Study Plan

This plan follows a **narrative learning order** — concepts build on each other rather than mirroring the guide's chapter sequence. Each week has a central theme, with daily sessions designed for ~60–90 minutes of focused study. Each week has its own detailed guide with per-concept exercises, extension questions, and a capstone project.

---

## How to Use This Study Guide

### Learning order

The guide's chapters are organized by technology (API, tools, Agent SDK, MCP, etc.), which makes a good reference but a poor learning sequence. This study plan reorders the material so that each concept builds directly on what came before — starting with how to talk to Claude, then how to give it abilities, then how to build agents, and so on. Resist the urge to skip ahead; the later weeks assume the earlier ones.

### Session format

Each daily session covers one topic and is designed for **60–90 minutes**. A suggested structure for each session:

1. Read the day's section individually before the group meets (~20 min)
2. Work through the Quick Checks and Exercises together (~30–40 min)
3. Debrief as a group, focusing on where answers diverged (~15–20 min)

### Question types

Each day contains several types of questions, each with a different purpose:

**Quick Checks** are short comprehension questions attached to individual concepts. They're designed to be answered mentally or aloud in under a minute — the goal is to catch misunderstandings before they compound. If a Quick Check stumps you, re-read the relevant concept before moving on.

**Exercises** ask you to produce something: write a tool definition, sketch a message array, design a schema. They're meant to take 10–20 minutes and work best when done individually first, then compared with the group. There's rarely one correct answer — the discussion of different approaches is the point.

**Extension Questions** appear at the end of each day. These are open-ended and deliberately harder than what the exam will ask. They're optional — use them if your session has time to spare, or as individual take-home prompts for anyone who wants to go deeper. Don't feel obligated to get through all of them.

### Review days (Days 5, 10, 15, 20)

Every fifth day is a review session built around exam-style practice. Each review day in Weeks 1–3 includes:

- **Sample Exam Questions** — a set of multiple-choice questions in the same format as the real exam, presented without answers. Try these individually and commit to an answer before discussing.
- **Answers** — the correct answer and a full explanation of why each wrong answer fails. The reasoning matters as much as the answer itself; focus especially on questions where you picked a plausible-sounding wrong answer.
- **Domain Notes review** — the guide's Part II has focused notes for each exam domain. The review day specifies which to read.

Week 4's Day 19 is the full 60-question practice test from the guide. Treat it like the real exam: work individually and time yourself (roughly 90 minutes for 60 questions).

### Weekly capstones

Each week ends with a multi-part capstone exercise that ties the week's themes into a single realistic scenario. Capstones are longer and more open-ended than the daily exercises — they're designed to be worked through over 60–90 minutes, either as a group or individually. There are no single correct answers; the goal is to practice making and defending architectural decisions.

### Exam logistics

- Passing score: **720 out of 1000**
- No guessing penalty — answer every question
- 4 of 8 possible scenarios are randomly selected on the real exam
- Format: multiple choice, one correct answer out of four

---

## Overview

| Week | Theme | Exam Domains | Detail |
|---|---|---|---|
| 1 | Foundations — Talking to Claude and Getting Reliable Output | Domain 4 (20%), Domain 5 (15%) | [Week1.md](Week1.md) |
| 2 | Building Agents — Systems That Act Autonomously | Domain 1 (27%), Domain 2 (18%) | [Week2.md](Week2.md) |
| 3 | Developer Workflows and Reliability | Domain 3 (20%), Domain 5 (15%) | [Week3.md](Week3.md) |
| 4 | Advanced Architecture and Exam Preparation | Domain 1 (27%), Domain 4 (20%), Domain 5 (15%) | [Week4.md](Week4.md) |

---

## Week 1: Foundations
**[→ Full week guide](Week1.md)**

Before building anything complex, master how Claude receives instructions, decides what to do, and returns structured, usable results.

| Day | Topic | Guide chapters |
|---|---|---|
| 1 | How Claude Actually Works (API Fundamentals) | Ch 1 |
| 2 | Giving Claude Abilities: Tools and `tool_use` | Ch 2 (2.1–2.3) |
| 3 | Prompt Engineering: Making Claude Do It Well | Ch 6 (6.1–6.4) |
| 4 | Structured Output: Getting Reliable Data Back | Ch 2 (2.4–2.5), Ch 6 (6.5–6.6) |
| 5 | Review and Practice | Exam Qs 1–3, Domain 4 notes |

**Capstone:** End-to-end invoice extraction pipeline — schema design, tool definition, few-shot examples, retry prompts, and failure analysis.

---

## Week 2: Building Agents
**[→ Full week guide](Week2.md)**

Move from single API calls to multi-step autonomous agents and multi-agent systems using the Agent SDK, hooks, and MCP.

| Day | Topic | Guide chapters |
|---|---|---|
| 6 | The Agentic Loop | Ch 3 (3.1–3.2) |
| 7 | Multi-Agent Systems: Coordinator and Subagents | Ch 3 (3.3–3.4) |
| 8 | Hooks: Deterministic Control Over Agent Behavior | Ch 3 (3.5), Domain 1 (1.5) |
| 9 | Connecting Claude to the World: MCP | Ch 4 |
| 10 | Review and Practice | Exam Qs 7–9, Domain 1 & 2 notes |

**Capstone:** Multi-agent customer support system — agent design, context passing, hook implementation, escalation logic, and failure analysis.

---

## Week 3: Developer Workflows and Reliability
**[→ Full week guide](Week3.md)**

Apply the Week 1–2 foundations to real engineering workflows with Claude Code, then tackle escalation, error handling, and context degradation.

| Day | Topic | Guide chapters |
|---|---|---|
| 11 | Claude Code: Configuration and Developer Workflows | Ch 5 (5.1–5.6) |
| 12 | Claude Code: Planning Mode, Session Management, and CI/CD | Ch 5 (5.6–5.10) |
| 13 | Escalation and Human-in-the-Loop | Ch 9 |
| 14 | Error Handling in Multi-Agent Systems | Ch 10 |
| 15 | Review and Practice | Exam Qs 4–6, 10–11, Domain 3 & 5 notes |

**Capstone:** Production-grade CI/CD code review pipeline — configuration design, CI commands, false positive management, re-review logic, and uncertainty routing.

---

## Week 4: Advanced Architecture and Exam Preparation
**[→ Full week guide](Week4.md)**

Context management at scale, task decomposition strategy, batch processing — then consolidate with the full 60-question practice exam.

| Day | Topic | Guide chapters |
|---|---|---|
| 16 | Context Management in Production Systems | Ch 11, 12 |
| 17 | Task Decomposition: Designing the Shape of Workflows | Ch 8 |
| 18 | Scale and Optimization: Batch Processing and Built-in Tools | Ch 7, 13 |
| 19 | Full Practice Exam (60 questions) | Practice Test section |
| 20 | Final Review and Exam Readiness | Full guide, targeted by Day 19 results |

**Capstone:** AI-powered research platform — full architecture design covering multi-agent topology, decomposition strategy, provenance, batch processing, and error handling.

---

## Concept-to-Chapter Quick Reference

| Concept | Chapter | Exam Domain |
|---|---|---|
| API request structure, `stop_reason` | Ch 1 | Domains 1, 5 |
| Tool definitions and `tool_choice` | Ch 2 | Domains 2, 4 |
| JSON schemas and structured output | Ch 2 | Domain 4 |
| Agentic loop and `AgentDefinition` | Ch 3 | Domain 1 |
| Coordinator/subagent and `Task` tool | Ch 3 | Domain 1 |
| Hooks (`PreToolUse`, `PostToolUse`) | Ch 3 | Domains 1, 5 |
| MCP: tools, resources, servers | Ch 4 | Domain 2 |
| CLAUDE.md hierarchy and `@path` | Ch 5 | Domain 3 |
| Skills, slash commands, planning mode | Ch 5 | Domain 3 |
| Claude Code CI/CD (`-p` flag) | Ch 5 | Domain 3 |
| Few-shot prompting and explicit criteria | Ch 6 | Domain 4 |
| Prompt chaining and retry loops | Ch 6 | Domains 1, 4 |
| Message Batches API | Ch 7 | Domain 4 |
| Fixed pipelines vs. dynamic decomposition | Ch 8 | Domain 1 |
| Escalation and handoff protocols | Ch 9 | Domains 1, 5 |
| Error categories and propagation | Ch 10 | Domains 2, 5 |
| Context management and scratchpads | Ch 11 | Domain 5 |
| Provenance and conflicting data | Ch 12 | Domain 5 |
| Built-in tools (Grep, Glob, Read, Edit) | Ch 13 | Domain 2 |
