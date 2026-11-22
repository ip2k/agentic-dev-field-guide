# GitHub the way we use it

Mostly private repositories, GitHub Flow, every change through a PR the agent writes and
you read, and GitHub's free CI as the gate.

## Setup

```bash
brew install gh          # or your platform's package
gh auth login            # you run this; the agent never handles your credentials
gh auth setup-git
```

The agent uses `gh` for everything: creating repos (`gh repo create --private`), PRs,
checks (`gh pr checks`), CI logs (`gh run view --log-failed`) and releases.

Turn on, per repo (Settings): branch protection or a ruleset for `main` with "require
status checks to pass", "require a PR", and "block force pushes". Also turn on secret
scanning and push protection where your plan offers them, and Dependabot alerts.
**On the Free plan, branch protection and rulesets are not enforced on private repos**
(they need Pro or Team). Without them, the rule "merge only when CI is green" is a
convention: put it in your `CLAUDE.md` and have the agent check `gh pr checks` before any
merge.

## Branches

GitHub Flow: `main` is always releasable, and every stream of work gets a short-lived branch.

`{type}/{YYYY-MM-DD}@{short-slug}`, e.g. `feature/2026-11-21@csv-export`, `fix/...`,
`chore/...`. Branches sort by date and show their age at a glance. `@` is legal in git branch
names. Use `.` instead if a tool chokes on it.

Each agent session works in its own **git worktree** (`.claude/worktrees/<name>`), so
parallel sessions do not trample one checkout. Claude Code's desktop app can create one
per session. In the CLI, ask the agent to `git worktree add` one. Remove worktrees after
merge.

## The PR is the record

The agent writes the PR description, and it is the document someone reads a year later. It
has three parts (`templates/github/PULL_REQUEST_TEMPLATE.md`):

1. **Intent**: the problem, the change, the decisions and the alternatives rejected.
2. **Release notes**: what users see, or "No user-facing change".
3. **Test results**: the commands run, with pass/fail/skip counts, plus manual checks, plus
   what was *not* tested. "Tests pass" without numbers is not accepted.

Merge with a **merge commit**, so the branch stays one visible unit in history. Never
force-push. Delete the branch after merge.

How much the agent may do alone is your call. A sensible ramp:
1. It commits on branches and opens PRs. You review and merge.
2. Once you trust it: it merges its own PRs when CI is green and nothing needs your eyes.
   Changes you must look at, listen to or approve still wait for you.

## CHANGELOG.md

Every repo has one, in [Keep a Changelog](https://keepachangelog.com) format, newest
first. Each PR adds its release notes under `## [Unreleased]`. At release time those
entries move under the version heading, and the GitHub release body is that section, not
just the auto-generated list of PR titles.

## Free CI in private repos

As of 2026-11, GitHub Actions is free for public repos on standard runners. Private repos
on the Free plan get an included monthly allowance (documented as 2,000 Linux minutes;
Pro gets more). macOS and Windows minutes are billed at higher rates, so they use the
allowance faster. GitHub changed Actions pricing several times in 2025–2026. Check
[Actions billing](https://docs.github.com/en/billing/concepts/product-billing/github-actions)
and your Settings → Billing usage page rather than trusting any number here.

Making the allowance last (the first four are in `templates/github/workflows/ci.yml`):
- Linux runners by default. Add macOS or Windows only for jobs that need them.
- `concurrency: cancel-in-progress` so a new push cancels the stale run.
- `timeout-minutes` on every job: a hung test should not burn the month.
- Dependency caching (`actions/setup-*` with `cache:`).
- `paths:` / `paths-ignore:` filters so a docs-only change does not run the full matrix.
- Heavy or long jobs (GPU, big builds) go on a **self-hosted runner** on a machine you
  already own. As of this writing those minutes are free. GitHub announced a per-minute
  platform charge for self-hosted runners in late 2025 and then postponed it, so check.
  **Use self-hosted runners only on private repos.** On a public repo, anyone's PR can
  run code on your machine.

Hardening defaults for every workflow:
- `permissions: contents: read` at the top. Widen per job only when needed.
- `pull_request`, never `pull_request_target`, for anything that runs PR code.
- `actions/checkout` with `persist-credentials: false`.
- Secrets only in jobs that need them, never in jobs that run untrusted code.
- A secret scan on every push plus a weekly full-history scan
  (`templates/github/workflows/security.yml`).

Let the agent own CI: when a check fails, it reads the log (`gh run view --log-failed`),
fixes the code (not the test, unless the test is wrong) and pushes again.

## Before going public

Before the first push of a repo to a **public** remote, scan the whole history, not just
the current tree, for secrets, personal data, internal hostnames and private notes:
`gitleaks git .`, plus a grep for your own email, name and home path. A secret removed in
a later commit is still in history.

## Repos you don't own

Fork, branch, and follow their CONTRIBUTING.md and AGENTS.md literally. One change per PR.
Every CI check run locally first. The agent drafts the PR text and **you** post it. Many
projects reject AI-written comments. Some CLA bots reject commits with AI co-author
trailers. Check before pushing.
