# Models, effort and context: right-sizing the work

Three settings decide most of what a Claude Code session costs and how good its work is:

- **which model** runs each piece of work;
- **how much effort** (reasoning) it spends;
- **how much context** it carries, and when that context is compacted.

They are separate controls. Using one when you needed another is the most common way to
waste money or get worse results.

Facts here were checked against Anthropic's docs on 2026-11-21, with Claude Code 2.1.3xx.
Model names, prices and defaults change every few months. The links at the bottom are
the source of truth. The **patterns** are what last.

---

## 1. Models: pick by how much the task has to *decide*

| model | Anthropic's one-liner | price, input / output per 1M tokens | use it for |
|---|---|---|---|
| **Haiku 5.5** | "high-volume, latency-sensitive tasks such as classification, extraction, and routing" | from $0.10 / $0.50 (more above 100K-token prompts) | well-specified work where the model decides little and the output is easy to check: pull fields out of files, classify, reformat, grep-and-summarise, run a command and report |
| **Sonnet 5.5** | "the best combination of speed and intelligence" | $2 / $10 | most scoped engineering: implement from a clear spec, write tests, triage logs, ports that follow a reference, routine research |
| **Opus 5.5** | "long-running agentic coding and knowledge work" | $4 / $20 | critical work that needs deep expertise: architecture, security, concurrency, hard debugging, reviews whose mistakes would be expensive to find later |
| **Fable 5.2** | "demanding reasoning and long-horizon agentic work" | $8 / $40 | the hardest problems: novel or creative work nobody has done before, long multi-step projects, and (in our practice) the advisor / project-manager role above everything else |

The question to ask is **"how much does this step have to decide?"**, not "how important
is this project?". Extracting 400 function signatures matters a lot, but it decides almost
nothing, so it is Haiku work. Choosing a database schema for a two-week project decides a
lot, so it is Opus or Fable work, even if it takes ten minutes.

What Anthropic's own measurements say, and why it matters:

- **Start with Opus and lower the effort, or start with Haiku and move up.** Both are
  documented approaches. Anthropic's model guide suggests Opus 5.5 for most workloads, and
  Fable 5.2 for demanding long-horizon work or when Opus at `xhigh`/`max` still falls
  short. Its pricing and Claude Code cost pages suggest Sonnet for most coding. Measure on
  your own work.
- **Haiku sub-agents are a trade, not a free win.** Under an Opus orchestrator they cut
  cost per attempt by about a third, and the pass rate fell about 4 points. Use Haiku where
  a wrong answer is cheap to detect: the output is checked by a test, a schema or the
  orchestrator.
- **A cheaper worker beats a lower effort only sometimes.** An Opus orchestrator with
  Sonnet workers cost about a third less for about the same score. Anthropic measured
  delegation winning in only two cases: routine work with a long cost tail, and work
  larger than one context window. For a single chain of dependent steps, or work that fits
  in one context, the orchestrator model alone at lower effort won every case measured.

## 2. Effort: how hard the model thinks, not which model thinks

On current models, thinking is **adaptive**. The model decides at each step whether and
how much to reason. **Effort** is the dial that biases that decision. It affects every
output token: thinking, tool calls and text. Lower effort means fewer and terser tool
calls and less verification. Higher effort means more checking, more edge cases
considered, and more tokens. It is a signal, not a hard token budget.

| level | what it is for (Anthropic's guidance, condensed) |
|---|---|
| `low` | quick exchanges, small edits, simple single-shot sub-agent tasks. Anthropic warns that on Haiku, `low` may skip searches or checks in long agent prompts |
| `medium` | **the Claude Code default for the current 5.x models.** Day-to-day scoped engineering |
| `high` | work where verification matters or edge cases are likely. A sensible floor for critical work |
| `xhigh` | long agentic or coding runs (more than about 30 minutes) where deeper reasoning pays for itself |
| `max` | measuring the ceiling on the hardest problems. Anthropic notes for Fable that it is slower and costlier than `xhigh` for a small gain, and Claude Code's docs say it can overthink. Test before you adopt it |

How to set it in Claude Code, from highest precedence to lowest:

1. The `CLAUDE_CODE_EFFORT_LEVEL` environment variable.
2. The session pick: `/effort <level>` or the `/model` picker. Typing `ultrathink` in a
   prompt raises that one turn to `xhigh` (2.1.311 and later, in the terminal).
3. The `--effort` flag at launch.
4. `settings.json`: `effortLevel`, or per model under `modelSettings`. These accept `low`
   to `xhigh` only. **`max` cannot be saved here.** In the current docs a top-level
   `effortLevel` does not apply to Opus 5.5, so use the per-model setting or `/effort` for
   it.
5. The model's own default.

For sub-agents, the `effort:` frontmatter in an agent or skill definition, or the Agent
tool's per-call `effort` parameter (2.1.292 and later), overrides the session's effort, but
not the `CLAUDE_CODE_EFFORT_LEVEL` environment variable, which wins everywhere. If you set
neither, the sub-agent runs at the session's effort (our observation; check the sub-agents
page for your version). A session at `xhigh` that sends out ten file searches spends
`xhigh` effort, and the extra tokens that come with it, on every one of them.

**Effort is often a better lever than switching models** (Anthropic's words). Before
moving a task up a tier, try the current tier one effort level higher. Before moving a
task down a tier, try one effort level lower.

## 3. Context: size it, compact it on purpose

### How big the window is

All four current models have a **1M-token** window, and Claude Code uses it by default.
Bigger is not free:

- **Every turn re-sends the whole context.** Cached tokens are cheap but not free, and an
  uncached turn (after a break longer than the cache lifetime: one hour for the main
  conversation on a subscription, five minutes otherwise) reprocesses everything at full
  price.
- **Quality drifts as the window fills.** In our experience the agent pays less attention
  to instructions and findings from early in a very long session. It repeats work, forgets
  a decision, or picks up a stale fact.
- **The fixed part is bigger than you think.** Tool definitions, MCP servers, skill and
  agent lists, and your CLAUDE.md files are loaded before you type anything. Our sessions
  start at around 100–140K tokens. `/context` shows the live breakdown. Trim what you
  don't use.

### What we landed on

We cap the **compaction window at 300K tokens** (`"autoCompactWindow": 300000` in
`~/.claude/settings.json`) instead of letting sessions run toward 1M. With our ~136K fixed
start, that leaves roughly 130–160K of working room before compaction (the trigger sits a
little below the window, to keep room for output). That matched the
behaviour we liked in another agent client, and it keeps sessions sharp and cheap.

Lower it (minimum 100K) if your fixed context is small. Raise it for work that has to
hold a lot at once, such as a big refactor plan or a long log investigation. **Never set
it below your fixed starting context.** If you do, compaction fires on every turn and
cannot help. Out-of-range values behave differently depending on where you set them: the
docs describe clamping for the environment variable, and reading the Claude Code 2.1.284
code we found an out-of-range settings value is dropped, not clamped. After changing it, check `/context`.

Other ways to set it (the environment variable takes precedence over all of them):

- the `CLAUDE_CODE_AUTO_COMPACT_WINDOW` environment variable. A bare number counts
  tokens, so `200000`, not `200`;
- the `--autocompact` flag, for one launch;
- `/autocompact <value>`, saved per model under `modelSettings`. Here `200` means 200K,
  and `/autocompact auto` restores the default;
- the `autoCompactWindow` setting, for all models.

### What compaction actually does

Compaction asks the model to **write a structured summary of the conversation so far**,
then **replaces the history with that summary**.

These are re-injected, so they survive:
- the system prompt and output style;
- your root CLAUDE.md, unscoped rules and auto-memory;
- skill descriptions, plus up to about 5K tokens per skill already invoked (25K in total);
- a fresh `git status`;
- the current plan;
- a few recently edited files (large ones come back as a path only).

Background commands and sub-agents keep running.

These are **lost**:
- exact tool output: the error message you were staring at, a diff, a log excerpt;
- the reasoning behind decisions that never made it into a file;
- anything the summary judged unimportant.

The summary is written by a model, so it is lossy and occasionally wrong, like any
summary.

Compaction also throws away the conversation's prompt cache. While the cache is warm
that is cheap. After a long break it means one full-price turn.

### When to compact, and how

- **Compact at natural breaks, not mid-thought.** Good moments: after a fix lands and
  before verification, after research and before implementation, after a PR merges. Bad
  moments: mid-debugging, when the exact error text matters.
- **Write the handoff first.** Before `/compact`, have the agent write the objective,
  constraints, branch and pushed SHA, what is done, what is left and the next step into a
  file or the PR. Then the summary can be lossy and nothing important is lost.
- **Steer it.** `/compact keep the failing test names and the decision to drop the cache
  layer` focuses the summary. A `# Compact instructions` section in CLAUDE.md does the
  same every time.
- **`/clear` between unrelated tasks.** A fresh session reloads only the fixed context
  and carries nothing stale. Sub-agents are the other way to keep the main context clean:
  send the large reads out, and get a conclusion back.
- **Don't fight auto-compaction.** If a session keeps hitting the threshold, the fix is
  usually smaller tasks, more delegation, or a trimmed fixed context, not a bigger window.

## 4. Advisors and orchestrators: a smarter mind above cheaper hands

There are two related patterns. Both put a stronger model where judgment matters and
cheaper models where the work is routine.

### The orchestrator (project manager) pattern

The **main session** plans, briefs sub-agents, judges their results and integrates them.
**Sub-agents** do the scoped work, each on a model and effort chosen for its task. The
main session should, in our experience, be at least as capable as anything it judges. A
Sonnet session often misses an Opus sub-agent's subtle mistake.

What we do:
- Run the main session on Opus or Fable for long or ambiguous projects, especially
  creative or never-been-done work, where most of the value is in judgment: what to try,
  what to drop, what "done" means.
- Send searches and inventories to Haiku at `medium` or Sonnet at `low`. Single-shot
  extraction and test runs with checked output can go to Haiku at `low`.
- Send scoped implementation from a clear brief to Sonnet at `medium`.
- Keep Opus at `high` for security, architecture, adversarial review and final synthesis.

### The advisor tool

Claude Code has an experimental **advisor**. While a cheaper executor model runs the
session, it can consult a stronger model that sees the whole transcript. Enable it with
`/advisor <model>`, the `advisorModel` setting or `--advisor`. The advisor must rank at
or above the model it advises. For example, Fable can advise Opus, Opus can advise
Sonnet, and Fable can only be advised by Fable. Sub-agents inherit the advisor setting.
Advisor tokens bill at the advisor's rate.

Anthropic's measurements carry a warning:
- **An advisor helps only when the executor asks it.** Haiku and Sonnet executors on a
  hard benchmark never consulted their advisor at all.
- Opus at `low` with a Fable advisor scored **worse** than Opus alone, because low effort
  meant it rarely stopped to ask.
- Opus at `high` with a Fable advisor scored 1.7 points higher, within noise, at about
  2.1 times the cost. Anthropic calls these internal, directional numbers.

So: keep the executor's effort at `medium` or higher, check that it actually consults
the advisor, and measure again on each model release.

Our practice is **Fable in the advisor and project-manager seat** for creative and novel
work, where a second, stronger opinion before committing to an approach pays off most.
This is our judgment, not Anthropic's published recommendation. Anthropic's docs position
Fable for "demanding reasoning and long-horizon agentic work", and in Anthropic's own
orchestrator test a Fable 5.2 orchestrator cost 19% more than an Opus one for about the
same score. Our case for Fable is novel, open-ended work that benchmark did not cover.

We make the advisor a **conscious, per-project choice**. When a project starts, the agent
says whether it looks complicated or novel enough to benefit from a Fable advisor. When
the answer isn't clear, it asks rather than deciding. The choice goes in the project's
CLAUDE.md.

## 5. Give Claude the authority to right-size

Most sub-agents default to the session's model and effort. Tell them they are allowed, and expected, to choose. Paste
something like this into your global CLAUDE.md, and adjust the tiers to your plan and
budget:

> **Right-size every agent.** You may choose the model and effort for each sub-agent,
> workflow step and side session, and you should set both explicitly rather than inherit.
> Pick the lightest model that can do the step, then the lowest effort at which it does it
> reliably:
> - **Haiku**, `medium` (`low` only for single-shot, checked steps): extraction,
>   classification, reformatting, searches and inventories, running a command and
>   reporting. Use it when the step decides little and its output is checked.
> - **Sonnet**, `low`/`medium`: scoped implementation from a clear brief, tests, log
>   triage, ports with a reference, routine research and summaries.
> - **Opus**, `high`: architecture, security, concurrency, hard debugging, adversarial
>   review, and anything whose mistakes would be expensive to find later.
> - **Fable**, `high`: the hardest and most novel problems and creative problem-solving.
>   For long, novel projects, the main session (or the advisor) runs on Fable or Opus: it
>   plans and judges, sub-agents do scoped work.
>
> Escalate on evidence, one step at a time. When a result fails its check because the task
> was harder than it looked, rerun it one effort level up, or one tier up if it is already at
> `high`, and say why in one line. Don't start high "to be safe": fix a thin brief with a
> better brief. Use `xhigh`/`max` only when I ask, or after `high` failed for lack of
> reasoning. State each agent's model and effort when you announce a fan-out. The
> orchestrator or advisor may change these choices mid-run when the evidence says so.

## Sources

- Claude Code model and effort configuration: <https://code.claude.com/docs/en/model-config>
- Context window and compaction: <https://code.claude.com/docs/en/context-window>
- Cost management: <https://code.claude.com/docs/en/costs>
- Advisor: <https://code.claude.com/docs/en/advisor>
- Sub-agents: <https://code.claude.com/docs/en/sub-agents>
- Effort (API): <https://platform.claude.com/docs/en/build-with-claude/effort>
- Models overview (the one-line descriptions): <https://platform.claude.com/docs/en/about-claude/models/overview>
- Choosing a model: <https://platform.claude.com/docs/en/about-claude/models/choosing-a-model>
- Cost vs intelligence (the advisor and orchestrator measurements): <https://platform.claude.com/docs/en/about-claude/models/optimizing-for-cost-and-intelligence>
- Prompt caching in Claude Code: <https://code.claude.com/docs/en/prompt-caching>
- Pricing: <https://platform.claude.com/docs/en/about-claude/pricing>
