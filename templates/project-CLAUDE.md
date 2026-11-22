<!--
Starter per-repository CLAUDE.md. Claude: write this AFTER reading the code. Replace every
<...>, delete sections that do not apply, and remove this comment.
If the repo is also used with Codex or other agents, keep one file and link the other name to
it: `ln -s CLAUDE.md AGENTS.md` (or keep AGENTS.md and have CLAUDE.md contain `@AGENTS.md`).
-->

# <project> — context for an agent session

<One paragraph: what this is, who uses it, what state it is in.>

**Personal or business:** <personal | business (owner: <org>)>. <If business: no personal
emails, names or accounts in the product or its metadata; inventory of accounts and keys
in docs/inventory.md.>

## Build, test, run

```bash
<install command>
<build command>
<test command>             # full suite, <N> tests, ~<T> min
<single-test command>      # how to run one test
<lint/format command>
<run locally command>
```

CI: `.github/workflows/<file>.yml` runs <what> on every PR. A PR merges only when it is green.

## Skills and tools adopted

- Skills: <names> (pre-approved for this project; ask before loading others).
- Code intelligence: <Rarefaction / Serena / graphify, with the result of the 5-minute check>.
- Key libraries chosen over custom code: <library: why; see docs/decisions.md>.

## Where the truth lives

- `docs/<...>.md`: <architecture / decisions / operations>
- `docs/HANDOFF.md`: current state, open issues, next steps (keep it updated)
- `CHANGELOG.md`: user-facing changes; each PR adds to `## [Unreleased]`

## How work is done here

- <Conventions that differ from defaults: module layout, error style, naming, test style>
- <Generated files and how to regenerate them; never hand-edit them>
- <Anything the agent must never touch: vendored code, migrations already shipped, etc.>

## Landmines

<Add one bullet each time something surprising costs time. Say what happened, why, and the
rule. Example:>
- **<Short title>.** <What happened, when.> <Why.> <The rule now.>

## Repository hygiene

Last audited for dead code, stale checks and unneeded legacy support: **<date>**, at
**<N> lines** (commit `<sha>`). Next audit due at roughly <N + 10,000> lines.
