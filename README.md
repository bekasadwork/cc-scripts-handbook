# Claude Code Skills & Commands Handbook

A collection of reusable prompts, custom skills, and slash commands for [Claude Code](https://docs.claude.com/en/docs/claude-code). Originally drafted with Gemini, refined for Claude Code workflows.

---

## Table of Contents

1. [Where things live in Claude Code](#where-things-live-in-claude-code)
2. [Prompt — Real-time Tic-Tac-Toe demo](#1-prompt--real-time-tic-tac-toe-demo)
3. [Skill — `/session-copy` (handoff between sessions)](#2-skill--session-copy-handoff-between-sessions)
4. [Project rules — Java code conventions for `CLAUDE.md`](#3-project-rules--java-code-conventions-for-claudemd)
5. [Skill — `/write-tutorial` (deep-dive tutorial generator)](#4-skill--write-tutorial-deep-dive-tutorial-generator)
6. [Skill — `/summarise-project` (architecture document generator)](#5-skill--summarise-project-architecture-document-generator)

---

## Where things live in Claude Code

Claude Code reads three kinds of files from two scopes (global and per-project):

| Type | Global path | Project path | Loaded |
|---|---|---|---|
| Custom skill | `~/.claude/skills/<name>/SKILL.md` | `.claude/skills/<name>/SKILL.md` | On demand via `Skill` tool / trigger words |
| Slash command | `~/.claude/commands/<name>.md` | `.claude/commands/<name>.md` | When user types `/<name>` |
| Project rules | — | `CLAUDE.md` (repo root) | Every turn, always |

**Pick which one you want:**
- One-off prompt → paste into chat. No install.
- Reusable workflow with trigger phrases → **skill** (`SKILL.md` with YAML frontmatter).
- Quick invocation with `/name` → **slash command** (`.md` in `commands/`).
- Codebase-wide rules → `CLAUDE.md`.

A single `SKILL.md` can be invoked both ways if you also drop a thin wrapper into `commands/`.

After dropping files in, restart Claude Code or run `/reload` so the new skill/command is discovered.

---

## 1. Prompt — Real-time Tic-Tac-Toe demo

**What it is:** A one-shot prompt for generating a React + Spring Boot WebSocket Tic-Tac-Toe demo. Useful as a live demo of concurrent WebSocket sessions.

**How to use in Claude Code:** Paste the prompt block into a fresh session. No install needed. If you want to reuse it often, save it as `~/.claude/commands/tictactoe-demo.md` and invoke with `/tictactoe-demo`.

````markdown
Act as an expert frontend engineer, UX designer, and Java backend architect. I am preparing a live tech demonstration for a developer audience, and I need you to generate the code for a real-time Multiplayer Tic-Tac-Toe game.

The goal of this app is to cleanly demonstrate concurrent WebSocket sessions by running two browser windows side-by-side.

Please generate the code using the following stack and design constraints:

### 1. Tech Stack
* Frontend: React, Tailwind CSS, Lucide React (for icons), and Framer Motion (for micro-animations).
* Backend: Java / Spring Boot 3.x using WebSockets (STOMP over SockJS).

### 2. UI / UX Design System
* Visual Theme: Minimalist, premium dark mode by default (inspired by tools like Linear or Vercel). Background should be a deep, rich black/gray (e.g., `bg-zinc-950`).
* The Board: A subtle, glassmorphism or soft-bordered grid. No harsh, solid white lines.
* Typography & Colors: Crisp sans-serif font. Player `X` is neon blue, Player `O` is neon orange/pink.
* Layout: Clean lobby screen first (Create Room / Join Room via short Room ID), then the game board.

### 3. Interaction & Animation Requirements
* Smooth piece placement: X/O pops in with a quick spring animation (Framer Motion).
* Win state: Glowing SVG line across the winning row/column/diagonal. Dim the rest of the board.
* Status indicators: Header showing whose turn it is, plus a pulsing green "Live" dot for WebSocket status.
* Toast notifications for "Opponent Connected", "Opponent Disconnected", "Opponent is typing/thinking".

### 4. Deliverables
1. Spring Boot WebSocket config class + `GameController` (moves, room creation, win logic).
2. React components (`Lobby`, `GameBoard`, `App`) with Tailwind classes and Framer Motion variants.
3. Brief explanation of the WebSocket message payload (JSON structure both directions).

Write modular, production-ready code. Skip boilerplate explanations and focus strictly on architecture and UI implementation.
````

**Tip:** If you want Claude Code to actually scaffold the project on disk (not just print blocks), append: *"Create the files under `./tictactoe-demo/` using your Write tool. Don't paste them in chat."*

---

## 2. Skill — `/session-copy` (handoff between sessions)

**What it is:** Saves the current session's state to a Markdown file so you can resume in a fresh session without losing context. Solves the long-running-session problem.

**Install:**

```bash
mkdir -p ~/.claude/skills/session-copy
# paste the block below into ~/.claude/skills/session-copy/SKILL.md
```

For per-project install, use `.claude/skills/session-copy/SKILL.md` instead.

**Triggers:** "save state", "create handoff", "pause", "resume from handoff", or `/session-copy`.

```markdown
---
name: session-copy
description: Creates and loads comprehensive handoff documents for seamless session transfers. Triggered when the user asks to "save state", "create handoff", "pause", or "resume from handoff".
---

# Session Handoff

Save the current session's context into a Markdown file, or load the latest one in a new session. Use built-in file tools (`Write`, `Read`, `Bash`) — no external scripts.

## 1. CREATE WORKFLOW (save state)
When the user asks to save, pause, create a handoff, or context is filling up:

1. Create the storage directory: `mkdir -p .claude/handoffs`
2. Generate a summary structured **exactly** like this:
   - **Current State Summary** — overarching goal of the session + what was just completed.
   - **Critical Context & Decisions** — hard-won knowledge the next session needs (e.g., "Server is UTC+5 but DB stores UTC, adjust offsets in SQL", "Strictly Spring Boot 3.x with constructor injection").
   - **Modified Files** — every file changed or created this session.
   - **Immediate Next Steps** — exact, actionable items for the next session.
3. Write the file to `.claude/handoffs/handoff-YYYY-MM-DD-HHMMSS.md` (real date/time).
4. Confirm the handoff was created, print the exact path, and recap next steps.

## 2. RESUME WORKFLOW (load state)
When the user asks to resume, load context, or continue:

1. `ls -la .claude/handoffs/`
2. Read the most recent file.
3. Output a brief "Welcome back" summary.
4. State the "Immediate Next Steps" verbatim and ask: *"Should I begin executing Step 1?"*

## 3. PROACTIVE SUGGESTION
After a complex architectural change, 5+ file edits, or a deep debug session, suggest:
> "We've made significant progress. If you plan to switch tasks or close the session, you can ask me to 'create a handoff' to preserve this context."
```

**Why this works well in Claude Code specifically:** Claude Code's context is automatically compacted near the limit, but a compaction summary is lossy. An explicit handoff file gives you full control over what survives.

---

## 3. Project rules — Java code conventions for `CLAUDE.md`

**What it is:** Per-project rules that Claude Code reads on every turn. Drop this in the **repo root** as `CLAUDE.md` (or append to the existing one).

**Install:** Save as `CLAUDE.md` at the project root. For team-wide use, commit it. For personal-only, add to `.gitignore` or use `CLAUDE.local.md`.

```markdown
# Role and Identity
You are an expert Java engineer adhering strictly to Oracle Java Code Style and the project conventions below. Code must be clean, deterministic, and highly readable.

# Java Code Style & Architecture Rules

## 1. Immutability by Default
- **Method parameters:** MUST be `final`.
- **Local variables:** MUST be `final` unless reassignment is strictly required.

## 2. Control Flow and Guard Clauses
- **Early returns:** Avoid deep nesting (arrow code). Use guard clauses at the top of methods to handle invalid states, empty `Optional`s, or nulls.
- **Null checking:** NEVER use `== null` or `!= null`. Use `java.util.Objects.isNull()` and `java.util.Objects.nonNull()`.

## 3. Formatting and Syntax
- **Indentation:** 4 spaces, no tabs.
- **Chained calls:** When chaining (Streams, Optionals, Builders), each call on a new line, indented one extra level (8 spaces from block start).
- **Annotations:** Method-level annotations (`@Transactional`, etc.) on their own line directly above the signature.

## 4. JavaDoc Conventions
Every public method has a JavaDoc block with this structure:
1. **Description:** Concise active-voice summary, ending with a period.
2. **Spacing:** Exactly one blank line between description and the first tag.
3. **Tags alignment:** `@param`, `@return`, `@throws` names and descriptions vertically aligned with spaces (not tabs).

## 5. Reference Example
When generating or refactoring method bodies, mirror this structure exactly:

```java
/**
 * Locks the device session for the given user and refreshes its TTL.
 *
 * @param userId    the authenticated user identifier
 * @param sessionId the device session identifier
 * @return          the refreshed session, never null
 * @throws SessionNotFoundException if no session matches sessionId
 */
@Transactional
public DeviceSession refreshSession(final UUID userId, final UUID sessionId) {
    if (Objects.isNull(sessionId)) {
        throw new IllegalArgumentException("sessionId must not be null");
    }

    return sessionRepository.findById(sessionId)
            .filter(s -> s.belongsTo(userId))
            .map(DeviceSession::refresh)
            .orElseThrow(() -> new SessionNotFoundException(sessionId));
}
```
```

**Tip:** Keep `CLAUDE.md` short and surgical. Claude Code reads it on every turn — bloat costs tokens on every message. If a rule only applies to one folder, put a smaller `CLAUDE.md` inside that folder; Claude Code picks up nested ones automatically.

---

## 4. Skill — `/write-tutorial` (deep-dive tutorial generator)

**What it is:** Generates a structured, visual, production-grade technical tutorial as a Markdown file with Mermaid diagrams and realistic code examples.

**Install:**

```bash
mkdir -p ~/.claude/skills/write-tutorial
# paste the block below into ~/.claude/skills/write-tutorial/SKILL.md
```

**Triggers:** `/write-tutorial`, "create a tutorial", "document this concept", "explain this for the team".

```markdown
---
name: write-tutorial
description: Generates a comprehensive, visually rich technical tutorial as a Markdown file. Triggered when the user asks to "/write-tutorial", "create a tutorial", "document this concept", or "explain this for the team".
---

# Write Tutorial

When invoked with a topic or prompt, generate a deep-dive tutorial as a `.md` file in `docs/tutorials/`. **Do not dump the tutorial into chat** — write the file with the `Write` tool.

## 1. PREPARATION
1. Identify the core technical concept from the user's prompt.
2. `mkdir -p docs/tutorials`
3. Filename: `docs/tutorials/YYYY-MM-DD-topic-slug.md`.

## 2. CONTENT GENERATION RULES
Required sections, in order:

### A. The Mental Model & First Principles
Before any code, explain *why* this concept exists.
- **First Principles:** strip the concept to its core truth (e.g., "Kafka is an append-only log", "Docker is a wrapper around Linux cgroups and namespaces").
- **Mental Model:** real-world analogy that lets an engineer grok it intuitively.

### B. Visual Architecture (Mermaid)
At least one well-structured Mermaid diagram. Pick the right type:
- `sequenceDiagram` for request lifecycles, OAuth flows, microservice traffic.
- `flowchart TD` / `graph LR` for system architecture, data pipelines, decision trees.
- `stateDiagram-v2` for entity states (e.g., payment lifecycle).
- Group components with `subgraph`. Keep it readable.

### C. Real-World Examples
- **No Foo/Bar.** Use realistic names (`UserDeviceSession`, `PaymentTransaction`, `OrderFulfillment`).
- Show the naive/bad approach first, then the optimized one.

### D. Edge Cases & Gotchas
Where this technology or pattern breaks in production. Race conditions, partial failures, scaling cliffs.

## 3. EXECUTION
Write the complete tutorial to the file with `Write`. Then respond with:
1. The exact path of the generated file.
2. A 2-sentence summary of the mental model used.
3. A nudge to review the Mermaid diagrams and code samples for accuracy.
```

**Tip for Claude Code:** If the project already has a docs convention (e.g., `mkdocs.yml` or a `README` linking to `/docs`), add a line to your project `CLAUDE.md` telling the skill where to write — it will override the default `docs/tutorials/` path.

---

## 5. Skill — `/summarise-project` (architecture document generator)

**What it is:** Scans the current workspace, infers architecture, and writes a high-level system document with Mermaid diagrams.

**Install:**

```bash
mkdir -p ~/.claude/skills/summarise-project
# paste the block below into ~/.claude/skills/summarise-project/SKILL.md
```

**Triggers:** `/summarise-project`, "document the architecture", "give me a project overview".

```markdown
---
name: summarise-project
description: Scans the codebase and generates a high-level architecture document with Mermaid diagrams. Triggered when the user asks to "/summarise-project", "document the architecture", or "give me a project overview".
---

# Summarise Project

When invoked, scan the workspace, infer architecture, and write a deep-dive architecture summary as a `.md` file in `docs/architecture/`. **Do not dump it in chat** — write the file.

## 1. PREPARATION & ANALYSIS
1. Use `Read`, `Glob`, `Grep`, and `Bash` to inspect:
   - Build files: `build.gradle`, `pom.xml`, `package.json`, `Cargo.toml`, etc.
   - Infrastructure: `docker-compose.yml`, `Dockerfile`, `.gitlab-ci.yml`, `.github/workflows/`, `k8s/`.
   - App config: `application.yml`, `application.properties`, `.env.example`.
   - Domain code: top-level entities, main controllers/services, core config classes.
2. Identify: tech stack, primary services/modules, datastores, observability (Loki/Tempo/Grafana/Prometheus), CI/CD targets.
3. `mkdir -p docs/architecture`
4. Filename: `docs/architecture/YYYY-MM-DD-system-architecture.md`.

## 2. CONTENT GENERATION RULES
Required sections:

### A. Executive Summary & First Principles
- **The "Why":** business problem this system solves.
- **First Principles:** the core architectural philosophy (e.g., "event-driven microservices prioritizing eventual consistency", "centralized observability config library").

### B. High-Level Architecture (Mermaid flowchart)
- `flowchart TD` or `flowchart LR`.
- Show external actors, main apps, DBs (PostgreSQL, Redis), brokers (Kafka, RabbitMQ), CI/CD targets.
- Group infrastructure with `subgraph` (e.g., LGTM observability stack, GitLab→Nexus pipeline).

### C. Tech Stack & Core Conventions
- Frameworks, languages, runtime versions.
- Key libraries (MapStruct, Lombok, MyBatis, etc.).
- Conventions discovered in code (constructor injection, custom exceptions, centralized caches).

### D. Domain Model (Mermaid erDiagram)
- `erDiagram`.
- 4–6 most critical entities only (e.g., `User`, `DeviceSession`, `Order`).
- Include cardinality and primary keys.

### E. Infrastructure & Deployment
- Environments (dev/stage/prod), CI/CD stages (build/test/push/deploy), artifact repos, deployment targets (VPS, k8s, Docker).

## 3. EXECUTION
Write the document with the `Write` tool. Then respond with:
1. The exact file path.
2. A 2–3 sentence summary of the architecture discovered.
3. Suggest the user review Mermaid charts and offer to refine the ERD or system flowchart.
```

**Tip:** On large monorepos, scope the scan first. Tell the skill: *"Limit analysis to `services/payments/` and its dependencies."* Otherwise it may spend the whole context budget reading peripheral modules.

---

## Quick reference — what to do when

| You want to... | Use |
|---|---|
| Try a one-off prompt | Paste into chat |
| Reuse a workflow with trigger phrases | Custom skill (`~/.claude/skills/<name>/SKILL.md`) |
| Invoke a workflow with `/name` | Slash command (`~/.claude/commands/<name>.md`) |
| Enforce coding rules in a repo | Repo `CLAUDE.md` |
| Enforce personal rules across all repos | `~/.claude/CLAUDE.md` |
| Survive a long session | `/session-copy` (see [section 2](#2-skill--session-copy-handoff-between-sessions)) |

---

## Contributing

Add new skills/commands as separate sections following the pattern above:
1. One-line description of what it does.
2. Install path.
3. Trigger phrases.
4. The full skill body in a fenced code block.
5. A short "Tip for Claude Code" note about how to use it well.