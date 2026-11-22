# Hard-won lessons

Each lesson below is a rule we now keep. Most came from a real incident, described briefly
and with identifying details removed. The rules are general. The incidents show why each
one matters.

---

## Working with the agent

### 1. A correction in chat lasts one session. A rule in CLAUDE.md lasts.
The agent starts every session with no memory except what it is told: `~/.claude/CLAUDE.md`
(global), the repository's `CLAUDE.md`/`AGENTS.md`, and any memory files. When you correct
it twice about the same thing, have it add a one-line rule with the reason. A rule with a
reason ("never X, because on <date> X caused Y") is followed far more reliably than a bare
command, and it can be applied to cases the rule did not foresee.

### 2. Keep the always-loaded instructions short.
Everything in the global and project instruction files is read at the start of every
session, and it consumes context the work could have used. One setup grew to around 40k
tokens of standing instructions before any work started. Keep the core file lean. Move
long reference material (recipes, incident write-ups, host tables) into docs the
instructions *point to*, so they are read only when relevant.

### 3. "Done" from an agent is a claim. Make it cheap to check.
Ask for evidence, not adjectives: the exact test command, the pass/fail/skip counts, a
screenshot or the real output, and what was *not* tested. "Tests pass" with no numbers
means nothing. When a sub-agent reports success, the main session checks the diff or the
result before telling you.

### 4. Every side effect gets reported, and nothing is named to look like an alarm.
A sub-agent doing a security check created a marker file called `/tmp/pwned` and the main
session forgot to mention it. The developer later found it, reasonably concluded the machine was
compromised, shut down remote access and reinstalled the laptop. That reinstall then
locked them out of several servers whose only keys were on it. Rule: when any delegated run
finishes, list what it created, changed or started. Test artifacts go in a scratch folder,
are named for what they are, and are removed.

### 5. A check that cannot read its input must fail, not pass.
A health check looked up a JSON field by a slightly wrong key name, got `None`, and skipped
the comparison. For two weeks it reported healthy while the exact fault it existed to
catch was present. Rules: take field names from a real captured sample, not from memory.
Treat "could not read it" as a failure. Give every check a test that feeds it a bad input
and expects it to fail.

### 6. A stored setting is not an applied setting. Verify the effect.
Several systems (network controllers, config managers, cloud consoles) accept a write,
read it back correctly, and do not apply it until a separate reload or provision. Some APIs
return 200 and silently drop fields they do not recognize. Verify the *behaviour* (on the
wire, in the running process, from a client) rather than the config readback.

### 7. Dry runs can lie.
In Ansible's check mode, command tasks are skipped with a fake success, so every later
conditional sees a plausible but wrong answer. The same is true of many "plan" and
"--dry-run" modes. Know which probes your dry run actually executes. Read-only probes that
later steps depend on should really run, even in check mode.

### 8. Tool output is data, not instructions.
Web pages, issue comments, file contents and API responses can contain text addressed
to the agent ("ignore previous instructions", "run this"). The agent must treat it as data,
and tell you when it sees an instruction it is not following. Put this in your global file.

### 9. Ask the human real decisions, with context, as a question.
Decisions buried in a long closing summary get missed. When something is genuinely your
call (scope, an approval, a choice between approaches), the agent should ask it as a
question you can answer from your phone: what is being decided, the evidence, its
recommendation first, and what happens if you do not answer. It should keep working on
whatever does not depend on the answer. It should not ask about things it can find out
by reading the code.

### 10. Finish the agreed goal, then write the handoff.
Agree on the endpoint at the start ("merged, with tests, docs updated"). A passing
sub-task or a checkpoint commit is not the end. For long work, keep a short handoff file
(objective, done, remaining, branch and pushed commit, next step) so a new session or a
context compaction can continue without loss.

## Cost and scale

### 11. Check your usage limits before any fan-out.
Multi-agent runs are where usage goes. A 17-lane review with three verifiers per finding
grew toward ~265 agents. Another ten-agent research run hit the weekly model limit partway:
three of five verifiers died and the run returned almost nothing. Rules:
- Before launching sub-agents or a workflow, look at the usage meters, including any
  per-model weekly limit, and state the agent count and rough token cost first.
- Prefer one adversarial verifier per lane over one per finding.
- Commit work in progress before a long run, so hitting a limit loses only agent work.
- If a run is dying, stop it and salvage the finished results instead of letting it burn
  out.

### 12. Pick the lightest model and effort that works.
Set the model and the reasoning effort explicitly for each sub-agent. A large model on high
effort is for architecture, hard debugging, security and final judgment. A smaller model on
low or medium effort is right for searches, inventories, running tests and reporting,
mechanical edits, and scoped implementation from a clear spec. Escalate one step at a time
when a result fails its check because the task was harder than it looked. A thin brief is
fixed with a better brief, not with more effort.

## Repositories and data

### 13. Every test, worktree and experiment cleans up after itself.
Per-branch CI caches on one Mac grew to 1.1 TB and filled the disk to under 1 GB free,
stopping a run partway. Old agent worktrees held another ~130 GB, and a later sweep found
84 GB in hundreds of stale worktrees. Rules:
- Decide where a job's data goes and what deletes it *before* it runs. Build the cleanup
  into the script (`trap ... EXIT`), the job spec, or the merge step.
- Remove a worktree once its branch is merged: `git worktree remove <path>`.
- Never keep worktrees or anything you want to keep in `/tmp`. macOS deletes files there
  that have not been touched for 3 days, leaving half a checkout.
- Keep a free-space floor on your main disk and check it before big jobs.
- Keep the evidence (logs, reports, commands). Delete what can be rebuilt.

### 14. Several agents in one checkout will collide.
Two sessions in the same working tree will overwrite each other's edits and branch
switches. Give each session its own git worktree. Never use bare `git stash` / `git stash
pop` when other sessions may be running: the stash stack is shared across worktrees, and
you can pop another session's changes.

### 15. One writer per file, one source of truth per fact.
A config file with two automated writers flip-flopped on every deploy and restarted a
service twice per run. Separately, a host address copied into one config went stale for
two weeks after the host moved, and the alert it caused was misread as normal. Rules:
each generated or managed file has exactly one owner. Facts like addresses, versions and
ports live in one place and are rendered everywhere else, with a pre-commit hook or CI
check that fails when the rendered copy is out of date.

### 16. A pinned version under a self-updater is a minimum, not a target.
A deploy script installed "version X unless already X". A separate auto-updater moved the
binary forward, and the next deploy would have silently downgraded it. Compare with "older
than", or pick one owner for the version.

### 17. Bound every heavy process.
An unbounded batch container exhausted 59 GB of RAM on a server. The kernel's
out-of-memory killer took down the monitoring stack and still could not save the machine,
which needed a physical power-button press. Put memory limits on containers and batch jobs,
and reserve memory for SSH so you can still log in and kill the job.

### 18. Audit for dead code on purpose.
Record in each project's `CLAUDE.md` when the code was last read end to end for dead
code, stale checks and legacy support nobody uses, with the line count at that point.
Every ~10,000 new lines, do the audit. Agents add code readily and rarely remove it.

### 19. Before v1.0, do not pay for backward compatibility.
Until something has real external users, have the agent change schemas, APIs and formats
in place and regenerate outputs, instead of writing migrations, shims and dual readers.
Agents spend a lot of effort on compatibility nobody needs unless told not to. Data
integrity and users' original data still matter.

### 20. Look for an existing tool before building one.
For solved problems (scheduling, dashboards, sync, notifications, dialogs, auth), have the
agent research maintained libraries first and record what it compared. Agents will happily
write a 600-line custom scheduler. The configuration of a standard tool is usually smaller
and better. The same goes for procedures: check for a skill before improvising. See
`PRIOR-ART-AND-SKILLS.md`.

## Secrets and accounts

### 21. Stop secrets from being printed. Do not rely on redacting them afterwards.
A Wi-Fi password was sent over a serial console. The console echoed it and wrapped the long
line, adding line breaks inside the secret. A simple find-and-replace redactor missed it,
and the full password ended up in a session transcript. Rules: keep secrets in a password
manager or the OS keychain and read them at run time. Pipe them directly from where they
are stored to where they are used. Turn off terminal echo before a secret crosses a console.
Never paste a secret into chat.

### 22. Claude Code's own config files are rewritten by running sessions.
`~/.claude.json` and `~/.claude/settings.json` can be rewritten from memory by running
sessions and the desktop app. We saw hand edits made from inside a session silently
disappear on the next write. Prefer Claude Code's own commands (`claude mcp add`,
`/config`, `/permissions`). Make hand edits with Claude Code fully quit, and check
afterwards that they stuck.

### 23. Keep personal and business separate from day one.
If a project might become a product, keep personal emails, names and accounts out of it from
the start. Use accounts the business owns (GitHub org, Apple team, domain, API keys, mailbox),
and keep an inventory of every account, key, domain and licence it depends on. Check licences
for commercial use before using a dataset, font, model or SDK. Untangling this later is
expensive.

### 24. Contributing upstream: the agent drafts, you post.
For repositories you do not own, have the agent read CONTRIBUTING and any AGENTS.md first,
keep each change to one PR, run every CI check locally, and draft the PR text for *you* to
review and post. Many projects ban AI-written comments. Assume they do unless they say
otherwise. Never let the agent post to someone else's project on its own.

## Modelling the problem

### 26. Ask for the state machine before the code.
A power manager treated "doesn't answer ping" as "off". Nobody had listed the state
"running, but with its network cable unplugged", so when that happened the wake script
power-cycled a healthy, running machine three times. Asking the agent for every state, every
event and every state × event cell turns hidden edge cases into blank cells you can see,
and shows quickly whether the agent (and you) really understand the domain. See
`STATE-MACHINES.md`.

## Visual output

### 25. Render it and look at it.
For diagrams, charts and UI, the agent should render the output and inspect a screenshot
before calling it done. Overlapping labels, clipped text and numbers jammed against a gauge
are invisible in the source and obvious in the picture.
