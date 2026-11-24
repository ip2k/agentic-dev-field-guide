# Agentic dev field guide

This packet holds the working setup that one developer arrived at after about a year of
daily AI-assisted development with Claude Code. Most of the work happens in private repos. Most of the rules
in it exist because something went wrong without them. Take what fits. Leave the rest.

It is written for two readers: **you**, and **your Claude**. Read the files once yourself,
then give the whole folder to Claude and let it adapt the templates to you
(see "Setup" below).

## What is in here

| file | for | what |
|---|---|---|
| `README.md` | you | this page |
| `START-HERE-FOR-CLAUDE.md` | your Claude | instructions for Claude: interview you, then install adapted versions of the templates |
| `LESSONS.md` | both | the hard-won lessons, each with the incident behind it |
| `templates/global-CLAUDE.md` | Claude | a starter `~/.claude/CLAUDE.md`, the instructions every session reads |
| `templates/project-CLAUDE.md` | Claude | a starter per-repository `CLAUDE.md` / `AGENTS.md` |
| `templates/github/` | Claude | a hardened GitHub Actions CI and secret-scan workflow, and a PR template |
| `PRIOR-ART-AND-SKILLS.md` | both | look before building: research existing libraries and platform features, use skills, and only then write custom code |
| `STATE-MACHINES.md` | both | ask the agent for a finite state machine of anything stateful: every state × event, so edge cases show up before users hit them |
| `MODELS-EFFORT-CONTEXT.md` | both | right-sizing: which model (Haiku, Sonnet, Opus, Fable) for which work, what effort levels do, context window size, when and how to compact, trimming plugins and MCP servers, lean custom agents, advisors and orchestrators |
| `GAME-THEORY.md` | both | look at a process as a game: cheap agent work flips old best practices, moves the bottleneck to review, and changes incentives |
| `CROSS-DOMAIN.md` | both | borrow proven solutions from other industries by mapping their jargon to the general problem (TRIZ, analogical transfer) |
| `CODE-INTEL.md` | both | Rarefaction, Serena and graphify: code-navigation tools for the agent, and how to check that they work |
| `GITHUB.md` | both | branches, PRs as the record, changelogs, free CI in private repos |
| `CLOUD.md` | both | DigitalOcean for always-on services, OpenRouter for model APIs, and how to keep secrets out of transcripts |

## Reading order

1. `LESSONS.md`, about 15 minutes. This is the part worth the most.
2. `PRIOR-ART-AND-SKILLS.md`, the habit that saves the most wasted work.
3. `templates/global-CLAUDE.md`, to see what a "constitution" for the agent looks like.
4. `MODELS-EFFORT-CONTEXT.md`, before you start sending work to sub-agents.
5. `STATE-MACHINES.md`, `GAME-THEORY.md` and `CROSS-DOMAIN.md`: three ways of thinking
   that get much more out of a model with broad knowledge.
6. Skim the rest. Come back to each when you need it.

## Setup (about 30 minutes)

1. Install Claude Code (CLI, desktop app, or IDE extension) and sign in.
2. Clone it somewhere permanent: `git clone https://github.com/ip2k/agentic-dev-field-guide ~/agentic-dev-field-guide`.
3. Start a Claude Code session **in that folder** and say:

   > Read START-HERE-FOR-CLAUDE.md and follow it.

4. Answer its questions. It will propose a `~/.claude/CLAUDE.md` and an install plan. You
   approve each step. It installs nothing that runs automatically (hooks, cron jobs)
   without a separate yes from you.
5. In your first real project, ask Claude to write that repository's `CLAUDE.md` from
   `templates/project-CLAUDE.md` after it has read the code.

## First week: things worth knowing

- **Employer code:** check your employer's policy on AI tools and on sending code to
  external services before you use any of this at work.
- `/init` drafts a project `CLAUDE.md` from the code. `/memory` shows which instruction
  files are loaded. `/compact` condenses a long session. `/clear` starts fresh.
- Permission prompts are the safety net. Allow read-only tools permanently, and keep
  prompts for writes, pushes and anything outward-facing.
- Watch your usage (`/usage` in the CLI, or the app's usage view), especially before
  asking for many parallel agents.
- Add `.claude/worktrees/`, `.claude/settings.local.json` and `graphify-out/` to your
  global git ignore (`~/.config/git/ignore`).

## Ideas behind all of it

- **Write the rule down where the agent will read it.** A correction you make in chat is
  lost when the session ends. A line in `CLAUDE.md` is applied in every later session.
  Every time you say "no, not like that" twice, it becomes a rule.
- **The agent works on a branch and proves it with tests. You read the PR.** The pull request
  description is the record of what changed, why, and how it was tested.
- **Look before you build.** An agent will happily write code for a problem that a
  maintained library, a platform feature or an existing skill already solves. Make it search
  for prior art and record what it found first (`PRIOR-ART-AND-SKILLS.md`). Give it the
  project's call graph (Rarefaction, in `CODE-INTEL.md`) so it reads code precisely
  instead of grepping.
- **Make the agent show its model of the problem.** For anything with a lifecycle or protocol,
  ask for the state machine first: every state, every event, every cell of the table
  (`STATE-MACHINES.md`). The blank cells are the edge cases, and a wrong diagram is a
  misunderstanding you can correct in one sentence, before any code exists.
- **Right-size the work.** The lightest model and effort that does a step reliably, a
  stronger model where judgment matters, and a context window kept small enough to stay
  sharp (`MODELS-EFFORT-CONTEXT.md`).
- **Trust, but make it cheap to verify.** The agent's report of success is a claim. Tests,
  CI, and a quick look at the real output turn it into a fact.

## About the details

Prices, free-tier limits and tool versions in here were checked on 2026-11-21 and change
often. Each file links to its primary source. Check before relying on a number.

Model names in the templates (Haiku, Sonnet, Opus, Fable) are the Claude model tiers available at the time
of writing. Use whatever the current equivalents are.
