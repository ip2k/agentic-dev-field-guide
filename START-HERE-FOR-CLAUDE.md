# Instructions for Claude: set this developer up

You are helping a software developer start AI-assisted development. Someone who has worked
this way for about a year gave them this folder. Your job is to turn it into a working
setup that fits **this** developer. Do not copy it blindly.

## Rules for this setup

- Ask before writing anything outside this folder. Show each file's contents, or a diff
  against an existing file, before you write it.
- **Never install hooks, cron/launchd jobs, background daemons, or plugins that add hooks
  without an explicit yes for that specific item.** They change every later session.
- Never ask for, type, or store a password, API key or token. If a step needs one, tell the
  developer the exact command to run themselves.
- If `~/.claude/CLAUDE.md` already exists, merge into it. Do not overwrite it.
- Keep the global `CLAUDE.md` short, about 150 lines or fewer. Every line is read at the start
  of every session and costs context. Put the optional sections in separate files that
  the main file points to.

## Step 1: read

Read every file in this folder: `README.md`, `LESSONS.md`, `CODE-INTEL.md`, `GITHUB.md`,
`CLOUD.md`, and `templates/`. Then check the machine: OS, shell, whether `git`, `gh`, `node`,
`uv`/`uvx`, and language servers (`rust-analyzer`, `clangd`, `sourcekit-lsp`, `pyright`) are
installed, whether `gh auth status` is logged in, and what is in `~/.claude/` already.

## Step 2: interview

Use your question tool (AskUserQuestion in Claude Code; otherwise a short numbered list in
chat) with at most four questions per round. Put your recommendation first in each question. Cover:

1. Languages and platforms they work in. This decides the code-intelligence setup.
2. Personal projects, work for an employer, or a business of their own? This decides the
   "personal vs business" section. If they work for an employer, tell them to check its
   AI/code policy before using any of this on employer code.
3. Where they host code (GitHub assumed), and whether they want the branch + PR workflow
   from `GITHUB.md` (recommended) or something lighter.
4. How much autonomy: may you commit on branches without asking? Open PRs? Merge
   when CI passes? Recommend: commit and open PRs freely, and merge only after they say so,
   until they trust it.
5. Cloud needs: anything always-on (a bot, an API, a scheduled job)? Any need for
   model APIs inside their own code? This decides how much of `CLOUD.md` applies.
6. Writing style preferences for docs and commit messages.

## Step 3: propose, then install

1. Draft `~/.claude/CLAUDE.md` from `templates/global-CLAUDE.md`, filled in with their
   answers. Delete sections that do not apply. Before writing, grep the draft for `<` and
   `>`: no placeholder or template comment may remain. Translate macOS-specific details
   (`/tmp` purge, `security`, `brew`) for their OS.
2. Offer `templates/project-CLAUDE.md` for their current main project, written **after**
   reading that project's code.
3. Offer the code-intelligence MCP servers from `CODE-INTEL.md` for their languages. Give
   them the install commands, or run them with their yes. Never run `graphify install`,
   `graphify claude install` or `graphify hook install`: they add hooks and edit CLAUDE.md.
   Then run the verification it describes and report the result honestly, including
   anything that did not work.
4. Offer the GitHub templates in `templates/github/` for their main repository. `ci.yml`
   ships as a Node example: rewrite the setup and commands for the real stack, or it fails.
   Point out that a required status check is matched by job name, so renaming the job or
   matrix later means updating branch protection.
5. If they want, add read-only tool permissions to `~/.claude/settings.json`, for example
   `mcp__rarefaction__status`, `mcp__rarefaction__orient`, `mcp__rarefaction__scope` and
   Serena's `find_symbol`, `find_referencing_symbols`, `get_symbols_overview`,
   `get_diagnostics_for_file` (as `mcp__serena__<name>`), so they stop getting prompted for them. Note that a
   running session may rewrite `settings.json` (see `LESSONS.md`). If an edit does not stick,
   have them make it with Claude Code quit.

## Step 4: hand off

Finish with a short summary: what was installed and where, what was skipped and why, what
they must do themselves (for example `gh auth login`), and three things to try in their
first real session. Suggest they come back to `LESSONS.md` after a week of use.
