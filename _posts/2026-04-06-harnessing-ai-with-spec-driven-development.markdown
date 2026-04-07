---
layout: post
title:  "Harnessing AI with Spec-Driven Development"
date:   2026-04-06 12:00:00
categories: blog ai development
description: "How I built a spec-driven framework to harness AI coding assistants — steering docs, structured workflows, and the context engineering patterns that made the biggest difference."
mermaid: true
---

I've been working with AI coding assistants for a while now. Copilot, Claude Code, Cursor — you name it, I've probably tried it. And if you've done the same, you know the pattern: it starts great, then the AI goes off the rails. It forgets what you told it three prompts ago. It hallucinates a function that doesn't exist. It rewrites half your codebase when you asked for a one-line fix.

The tool isn't the problem. The **structure** around it is.

So I built a framework at work to fix this — a spec-driven development toolkit designed to harness AI assistants into producing consistent, high-quality work. It was heavily inspired by [Kiro](https://kiro.dev/) and [spec-kit](https://github.com/spec-kit/spec-kit), two projects that showed me the power of structured, spec-driven approaches to AI coding. I can't share the proprietary details, but I want to walk through the architecture and the patterns that made the biggest difference.

## The Problem with Unstructured AI

When you drop an AI into a codebase with no guidance, it does what any new developer would do with no onboarding — it guesses. It guesses your conventions, your architecture, your testing strategy. Sometimes it guesses right. Often it doesn't.

The real issue is **context**. AI models have a limited context window, and most of the time we're wasting it. We dump entire files in, hope the model figures out what matters, and then wonder why the output is inconsistent.

What if instead of hoping the AI understands your project, you **told** it — in a structured, decomposed way?

## Two Layers of Context

The framework is built around a two-layer context model: **steering documents** and **rules**.

<div style="text-align: center;">
<pre class="mermaid">
graph TB
    subgraph Steering["Steering Docs (Project-Level)"]
        P[Product Vision]
        T[Tech Stack]
        S[Structure]
        A[Architecture]
        TS[Testing Strategy]
        PR[Principles]
    end

    subgraph Rules["Rules (Path-Scoped)"]
        R1["code-style.md<br/><small>applies to: src/**</small>"]
        R2["testing.md<br/><small>applies to: tests/**</small>"]
        R3["security.md<br/><small>applies to: api/**</small>"]
    end

    Steering -->|"loaded every session"| AI[AI Agent]
    Rules -->|"loaded conditionally<br/>based on file path"| AI

    style P fill:#3498db,color:#fff,stroke:#2980b9
    style T fill:#3498db,color:#fff,stroke:#2980b9
    style S fill:#3498db,color:#fff,stroke:#2980b9
    style A fill:#3498db,color:#fff,stroke:#2980b9
    style TS fill:#3498db,color:#fff,stroke:#2980b9
    style PR fill:#3498db,color:#fff,stroke:#2980b9
    style R1 fill:#5dade2,color:#fff,stroke:#3498db
    style R2 fill:#5dade2,color:#fff,stroke:#3498db
    style R3 fill:#5dade2,color:#fff,stroke:#3498db
    style AI fill:#2c3e50,color:#fff,stroke:#34495e
</pre>
</div>

**Steering documents** are project-level context files. Instead of one massive README, the framework decomposes your project context into ~6 focused files: product vision, tech stack, directory structure, architecture decisions, testing strategy, and core principles. These get loaded into the AI's context at the start of every session.

**Rules** are path-scoped conventions. Different parts of your codebase have different needs — your API layer has different conventions than your frontend components. Rules are loaded conditionally based on which files the AI is working with. This keeps the context window lean and relevant.

The key insight: **decompose context so the AI loads only what it needs, when it needs it.**

### The System Prompt Effect

Two of these steering documents — the **product vision** and the **principles** — are wired directly into the AI's system prompt. That means they're present in *every single conversation*, not just when you remember to mention them. This has a massive impact on output quality.

Here's a concrete example. Say you ask the AI: *"Add a user preferences endpoint."*

**Without steering docs in the system prompt:**

The AI has no product context. It might:
- Create a generic REST endpoint with a structure that doesn't match your existing API patterns
- Store preferences in a new SQLite database when your project uses DynamoDB
- Skip authentication entirely because it doesn't know your security model
- Name things inconsistently with the rest of your codebase (`userPrefs` vs your convention of `user_preferences`)

You end up spending more time correcting the AI than you would have writing it yourself.

**With product.md and principles.md in the system prompt:**

The AI knows your product's domain, your users, your constraints. It knows that your API follows a specific pattern, that all endpoints require auth middleware, that you use a specific data layer. Now the same prompt produces:
- An endpoint that follows your existing route structure and naming conventions
- Data stored in the same database and table patterns as the rest of your features
- Auth middleware applied by default because the principles say "all endpoints require authentication"
- Error handling that matches your established patterns

The difference isn't subtle — it's the difference between getting a pull request from a new hire on day one versus a teammate who's been on the project for six months.

**Another example: refactoring.** Ask the AI to *"refactor the notification service."*

Without context, the AI might extract a clean abstraction that completely breaks your event-driven architecture because it didn't know that notifications are consumed asynchronously by three other services. With the architecture and principles loaded, it respects the boundaries and refactors within the existing patterns.

To understand why this matters so much, it helps to visualize how an AI coding assistant like Claude Code actually prioritizes context:

<div style="text-align: center;">
<pre class="mermaid">
block-beta
    columns 1
    block:L1["1. Base System Prompt & System Prompt Files"]
        columns 2
        L1A["Agent instructions\nCore capabilities"]
        L1B["product.md\nprinciples.md"]
    end
    block:L2["2. Tool Definitions (~50 tools)"]
        columns 1
        L2A["Bash, file system, grep, MCP tools, etc."]
    end
    block:L3["3. User Content"]
        columns 1
        L3A["CLAUDE.md, memory\nLoaded every session"]
    end
    block:L4["4. Conversation History"]
        columns 2
        L4A["Messages, reasoning\ntool calls"]
        L4B["tech.md, architecture.md\nrules (loaded by skills)"]
    end
    block:L5["5. Attachments"]
        columns 1
        L5A["Per-turn specs, @-mentions, parameters"]
    end
    block:L6["6. Skills"]
        columns 1
        L6A["Relevant or user-specified skills appended at the end"]
    end

    style L1 fill:#2c3e50,color:#fff
    style L1B fill:#f39c12,color:#fff,stroke:#f5b041,stroke-width:3px
    style L2 fill:#34495e,color:#fff
    style L3 fill:#3498db,color:#fff
    style L4 fill:#5dade2,color:#fff
    style L4B fill:#f5b041,color:#fff,stroke:#f39c12,stroke-width:2px
    style L5 fill:#85c1e9,color:#2c3e50
    style L6 fill:#aed6f1,color:#2c3e50
</pre>
</div>

As Drew Breunig explains in his excellent breakdown of [how Claude Code builds a system prompt](https://www.dbreunig.com/2026/04/04/how-claude-code-builds-a-system-prompt.html), the system prompt alone has ~30 components assembled dynamically, with ~50 tool definitions, and about a dozen methods for compacting and summarizing the conversation as it grows.

Not all steering docs are created equal — and the framework is deliberate about where each one lives:

- **product.md and principles.md** are wired as `systemPromptFiles`, which places them at **layer 1** — the system prompt itself. They're always present, never truncated, and they shape every response the AI gives. These are your "who we are and what we never compromise on" docs.
- **CLAUDE.md and memory** live at **layer 3** — loaded every session but at a slightly lower priority. This is where project-specific instructions and persistent notes go.
- **The rest of the steering docs** (tech stack, architecture, testing strategy) and **path-scoped rules** are loaded by skills at runtime and land in **layer 4** — conversation history. They're only brought in when relevant, and they get compacted and summarized as the context window fills up.

This layering is intentional. You don't need the full testing strategy in every conversation — but you *always* need the AI to know your product vision and your hard constraints. By placing only the two most critical docs at the highest priority and loading everything else on demand, the framework stays token-efficient while keeping the AI aligned.

This is why a random prompt like "add a user preferences endpoint" produces wildly different results depending on what's loaded in those top layers. The AI isn't smarter with the framework — it just has the right context at the right priority level.

The lesson: **the system prompt is the most valuable real estate in your AI workflow.** Don't waste it on generic instructions. Fill it with your product's identity and your team's hard-won engineering principles.

## The Execution Flow

Instead of freeform prompting, the framework defines a structured workflow. Each step is a discrete "skill" — a focused operation the AI knows how to execute with clear inputs and outputs.

<pre class="mermaid">
graph LR
    A["Idea"] --> B["Brainstorm"]
    B --> C["Backlog"]
    C --> D["Spec"]
    D --> E["TODO"]
    E --> F["Implement"]
    F --> G["Review"]
    G --> H["Changelog"]

    style A fill:#2c3e50,color:#fff,stroke:#34495e
    style B fill:#5dade2,color:#fff,stroke:#3498db
    style C fill:#5dade2,color:#fff,stroke:#3498db
    style D fill:#3498db,color:#fff,stroke:#2980b9
    style E fill:#3498db,color:#fff,stroke:#2980b9
    style F fill:#2c3e50,color:#fff,stroke:#34495e
    style G fill:#f39c12,color:#fff,stroke:#e67e22
    style H fill:#2c3e50,color:#fff,stroke:#34495e
</pre>

Here's how it works:

- **Brainstorm** — Validate an idea against the product vision. Is it worth building? Does it conflict with existing plans?
- **Backlog** — A living document of discovered issues and feature requests, categorized by priority.
- **Spec** — The most important step. Every feature gets a formal specification before any code is written.
- **TODO** — A focused sprint plan pulled from the backlog, scoped to what can be done now.
- **Implement** — The AI writes code guided by the spec, steering docs, and rules.
- **Review** — Automated review against principles and conventions. Not just linting — architectural compliance.
- **Changelog** — Completed work gets archived with context for future reference.

The magic is that each skill has a **constrained scope**. The brainstorm skill doesn't write code. The review skill doesn't add features. This prevents the AI from going off-script.

## The 3-File Spec

This is probably the pattern I'm most proud of. Every feature gets decomposed into three files:

<div style="text-align: center;">
<pre class="mermaid">
graph LR
    subgraph Spec["Feature Spec"]
        REQ["requirements.md<br/><b>WHAT</b><br/><small>User story<br/>Acceptance criteria</small>"]
        DES["design.md<br/><b>HOW</b><br/><small>Technical approach<br/>Affected files</small>"]
        TSK["tasks.md<br/><b>DO</b><br/><small>Ordered checklist<br/>Atomic steps</small>"]
    end

    REQ --> DES --> TSK

    style REQ fill:#f39c12,color:#fff,stroke:#e67e22
    style DES fill:#3498db,color:#fff,stroke:#2980b9
    style TSK fill:#2c3e50,color:#fff,stroke:#34495e
</pre>
</div>

- **requirements.md** (WHAT) — The user story and acceptance criteria. What does this feature do? How do we know it's done?
- **design.md** (HOW) — The technical approach. Which files are affected? What patterns should be used? What are the trade-offs?
- **tasks.md** (DO) — An ordered checklist of atomic implementation steps. Each task is small enough to verify independently.

This mirrors how a senior engineer naturally decomposes work. The beauty is that each file serves the AI at a different phase — requirements during planning, design during implementation, tasks during execution and review.

## Harnessing Patterns That Actually Work

Beyond the structure, there are specific patterns that dramatically improved the quality of AI output:

### Deterministic Hooks

Not everything needs an LLM. The framework uses simple shell scripts to enforce preconditions — checking that a changelog was updated before committing, validating that spec files have the right sections, ensuring tests exist for new features. These run at zero token cost and catch issues before the AI even sees them.

### Dynamic Context Injection

Instead of loading everything into the context window and hoping for the best, the framework figures out which steering docs and rules are relevant to the current task and injects only those. Working on the API layer? You get the architecture doc and the security rules. Working on tests? You get the testing strategy and test conventions. This keeps the context focused and the output consistent.

### Assumption Confirmation

This one saved us from countless wrong turns. When the AI hits an ambiguous decision — which module should own this feature? should this be async or sync? — it **surfaces the decision to the human** with a recommended default instead of silently guessing. The human stays in the loop on decisions that matter, and the AI handles the execution.

### Multi-Agent Parallel Execution

For discovery tasks like scanning a codebase for issues, the framework spawns multiple specialized agents in parallel — each focused on a different domain (security, performance, code quality, etc.). Think of it like CI jobs: if two agents don't touch the same concerns, why serialize them? The results get consolidated into a single prioritized backlog.

### Skill-Based Orchestration

Instead of one mega-prompt that tries to do everything, work is broken into discrete skills with focused purposes. Each skill has defined inputs, outputs, and constraints. The scan skill discovers issues. The spec skill creates specifications. The review skill validates changes. This composability is what makes the framework reliable — each piece is small enough to be predictable.

## What I Learned

Building this framework changed how I think about AI-assisted development. A few takeaways:

**Structure beats prompting.** The best prompt in the world won't save you if the AI doesn't understand your project's architecture, conventions, and constraints. Invest in context, not cleverness.

**Decomposition is everything.** Breaking context into focused documents, breaking workflows into discrete skills, breaking specs into three files — every time I decomposed something, the output quality jumped.

**Keep humans in the loop on decisions, not execution.** The AI is great at writing code, running tasks, following patterns. It's terrible at making architectural decisions without context. The assumption confirmation pattern — where the AI surfaces ambiguity instead of guessing — was the single biggest quality improvement.

**Deterministic validation saves tokens and sanity.** Not every check needs an LLM. A shell script that validates file structure is faster, cheaper, and more reliable than asking the AI to self-check.

The best AI hack isn't a better prompt — it's a better process.
