# Agentic Coding Setup in 2026: Copilot vs. Claude Code

*Both promise autonomous coding at scale. The setup philosophies — and the multi-agent ambitions — couldn't be more different.*

---

The era of "just autocomplete my line" is over. Today's AI coding tools want to understand your entire project, remember your conventions across sessions, and orchestrate whole teams of autonomous agents. Both GitHub Copilot in VS Code and Anthropic's Claude Code have built structured setups for this — but they've taken meaningfully different roads.

Let's break it down.

---

## The VS Code Copilot Approach: Layered Customization Inside the IDE

Microsoft's guide for agentic Copilot setup is built around a **five-layer customization stack**, all living in a `.github/` folder committed to your repo. Run `/init` in the chat panel and Copilot auto-generates the base instructions file by analyzing your codebase — zero-friction onboarding. From there you layer file-specific rules (via `applyTo` YAML patterns), reusable slash-command prompts, custom agent personas with restricted tool access, and context-triggered skills. Commit it once, and every teammate inherits the full AI setup instantly.

```
your-project/
└── .github/
    ├── copilot-instructions.md       → auto applied standards 
    │
    ├── instructions/                 → auto-applied rules 
    │   └── python.instructions.md    → (applyTo: '**/*.py')
    │
    ├── prompts/                      → reusable prompt templates
    │   └── refactor.prompt.md   
    │     
    ├── agents/                       → custom agents/ personas
    │   ├── reviewer.agent.md         
    │   ├── python-expert.agent.md    
    │   ├── coding.agent.md           
    │   └── supervisor.agent.md       
    │
    ├── skills/                       → context-triggered
    │   └── update-readme/            
    │       └── SKILL.md              → skill + optional scripts 
    │
    └── hooks/                        → auto triggered agent hooks
        ├── security.json             → block unsafe commands
        └── approval.json             → require tool approval 
```

**Verdict:** Elegant, approachable, and deeply integrated with the VS Code UI. A developer who's never configured an AI agent before can follow the guide in an afternoon and end up with something genuinely useful.

---

## The Claude Code Approach: A Filesystem-First Memory Architecture

Claude Code is a terminal-based agent that can interact with your codebase as an autonomous process. The setup philosophy reflects this: more powerful, more opinionated, and yes, slightly more humbling to configure.

**CLAUDE.md is the beating heart** — loaded into Claude's system prompt at the start of every session. The memory model has three levels: a project-level file committed with the repo, a personal user-level file that stays local, and a `MEMORY.md` that Claude *writes itself* after sessions, accumulating its own notes about your codebase over time. Subagents are Markdown files with YAML frontmatter — each gets its own system prompt, tool restrictions, and model selection (e.g., Haiku for a cheap read-only reviewer), running in a fully isolated context window.

```
your-project/
├── CLAUDE.md                       → team-shared project memory
├── .claude/
│   ├── agents/                     → custom agents
│   │   ├── reviewer.md              
│   │   └── backend-architect.md    
│   │ 
│   ├── skills/                     → sematically triggered 
│   │   └── update-readme/          → skill + optional scripts 
│   │       └── SKILL.md            
│   ├── rules/                     → semantically triggered
│   │   ├── code-style.md              
│   │   └── testing.md  
│   │ 
│   ├── commands/                   → reusable slash commands
│   │   ├── fix-issue.md   
│   │   └── review.md  
│   │ 
│   └── hooks/                      → event driven (pre/post)
│       └── validate-bash.sh                

~/.claude/                          → personal, NOT committed
├── CLAUDE.md                       → machine-specific preferences
└── projects/<hash>/
    └── MEMORY.md                   → self-written session notes
```


**Verdict:** More powerful, more flexible, and — honestly — more complex. The self-writing memory system is impressive once it's running. But it requires CLI comfort and deliberate architecture thinking. You're handed a blueprint and a shovel.

---

## Multi-Agent & Agent Teams: Where Things Get Interesting

This is where the two tools diverge most dramatically. Both support *some* form of multi-agent work — but the depth, maturity, and philosophy are quite different.

### Copilot's Agent Model in VS Code

VS Code **1.113** (March 25, 2026 — now on a weekly release cadence) supports **four distinct agent types**, all manageable from a unified Agent Sessions view:

- **Local agents** — run interactively inside VS Code with full workspace and tool access. Built-in modes: *Agent* (complex tasks), *Plan* (structured plans), *Ask* (Q&A over your codebase).
- **Background agents (Copilot CLI)** — CLI-based, run non-interactively using Git worktrees. Now supports slash commands, prompt files, hooks, and skills — and can be renamed for easier tracking. An *Autopilot* permission level (v1.112) lets the CLI agent write files and execute commands without per-step approval.
- **Cloud agents (Copilot Coding Agent)** — run on GitHub's remote infrastructure, produce PRs for team review. Pro+ and Enterprise users can assign GitHub issues directly to a cloud agent.
- **Claude & third-party agents** — Claude agents (via the official Claude Agent SDK) run directly inside VS Code under your Copilot subscription. As of v1.113, MCP servers configured in VS Code are now bridged to Claude and Copilot CLI agents automatically — previously they were only accessible to local in-editor agents.

**What's new in v1.113 for multi-agent work:**

- **Nested subagents** — previously blocked to prevent infinite recursion, subagents can now invoke other subagents via the opt-in `chat.subagents.allowInvocationsFromSubagents` setting, enabling genuine multi-step hierarchical workflows
- **Session forking** — branch any conversation at any point to explore a different prompt path without losing the original context; now available in both Copilot CLI and Claude agent sessions
- **Configurable thinking effort** — a new Thinking Effort selector in the model picker controls reasoning depth per request for capable models (e.g. Claude Sonnet 4.6), letting teams balance quality against quota

**Handoffs** let you chain agent types: Plan locally → implement in a background agent → open a PR via a cloud agent — conversation history carried forward throughout.

**The key limitation:** There's still no shared task list between concurrent agents and no peer-to-peer messaging. Nested subagents and session forking move things forward, but true parallel multi-agent execution with dependency tracking remains Claude Code territory for now.

### Claude Code's Agent Teams

Claude Code's multi-agent story is significantly more ambitious — and still partly experimental.

**Subagents** (stable) are defined in `.claude/agents/` as Markdown files. Each runs in its own context window with its own system prompt, tool restrictions, and optionally its own model. Built-in subagents include `Explore` (fast, read-only codebase search on Haiku) and `Plan` (planning mode). Crucially: subagents cannot spawn other subagents — preventing infinite nesting while preserving context isolation.

**Agent Teams** (experimental, enable with `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`) is the headline feature for true parallel execution:

- A **Team Lead** decomposes work and creates a shared task list
- **Teammates** are independent Claude Code instances with their own context windows, running concurrently in tmux split panes
- A **shared task list** tracks statuses (pending, in_progress, completed, blocked) with automatic dependency resolution — when backend marks its API task complete, the blocked test-writing task auto-unblocks
- **Peer-to-peer messaging** lets teammates communicate directly (e.g., backend sends the API contract straight to frontend) without routing through the lead

The result: three agents building frontend, backend, and tests simultaneously — each in an isolated Git worktree to prevent merge conflicts.

For even larger-scale work, **hierarchical orchestration** goes deeper: your orchestrator spawns "Feature Lead" agents, which in turn spawn their own specialists — mimicking how real engineering orgs decompose work through tech leads. Third-party tools like Conductor, Gas Town, and Multiclaude extend this further with visual dashboards for 3–10 parallel agents.

---

## Head-to-Head Comparison

| Dimension | VS Code Copilot | Claude Code |
|---|---|---|
| **Setup friction** | Low — GUI-driven, `/init` bootstraps | Medium — CLI-first, manual CLAUDE.md |
| **Memory persistence** | Static instruction files | Static + Claude's own evolving MEMORY.md |
| **Subagents** | Within-session, isolated context | Within-session + cross-session, own model/tools |
| **True parallel agents** | ❌ Not natively | ✅ Agent Teams (experimental) |
| **Shared task list** | ❌ | ✅ With dependency tracking + auto-unblock |
| **Peer-to-peer messaging** | ❌ | ✅ Between teammates |
| **Cloud/async agents** | ✅ Copilot Coding Agent (Pro+/Enterprise) | ✅ Claude Code Web (browser-based) |
| **Multi-provider in one UI** | ✅ Claude + Codex + Copilot | ❌ Claude only |
| **Team collab via PRs** | ✅ Native GitHub integration | Manual / third-party tools |
| **IDE integration** | ✅ Deep, native | ❌ Terminal-based, IDE agnostic |

---

## Who Should Use What?

**Choose Copilot in VS Code if:** You want agentic features without leaving the IDE, your team runs on the GitHub/Microsoft stack, you need cloud agents that produce reviewable PRs, or you want to mix Claude + Codex + Copilot under one subscription without juggling separate tools.

**Choose Claude Code if:** You need genuine parallel multi-agent orchestration today, want agents that *learn* from your project over time via self-written memory, require fine-grained subagent customization (per-agent model, tool restrictions, hooks), or you're comfortable in the terminal and want the most powerful agentic setup currently available.

---

## The Broader Picture

Both approaches share one core truth: **the quality of agentic coding is directly proportional to how well you've structured the AI's context.** A good `CLAUDE.md` or `copilot-instructions.md` isn't busywork — it's institutional memory for an agent that otherwise wakes up with amnesia every session.

There's also cross-tool convergence on the horizon. `AGENTS.md` — a shared standard backed by the Linux Foundation and supported by Claude Code, Copilot, Cursor, Windsurf, and others — is emerging as a universal context layer that could eventually make these comparisons moot.

For now: Copilot gives you a well-lit, beautifully furnished room. Claude Code hands you an architectural blueprint, a shovel, and the ability to run an entire construction crew in parallel.

Both are genuinely useful. One scales further. The choice, as always, depends on how deep you want to go.

---
## Interesting Community Examples:

- https://skills.sh
- skill evaluation: https://skills.sh/anthropics/skills/skill-creator
- https://github.com/github/awesome-copilot
- https://github.com/anthropics/skills
- https://github.com/pamelafox/office-hours-writeups/tree/main/.github
- https://github.com/Azure-Samples/python-agentframework-demos/tree/main/.github
- Copilot multi-agent orchestration: https://youtu.be/-BhfcPseWFQ?si=8orl4QpsdbGSkjpP

---
![claude_folder_structure](claude_folder_structure.png)

