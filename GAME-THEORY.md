# Game theory: when the costs change, the right move changes

Game theory studies decisions whose outcome depends on **costs, payoffs and other
players' choices**. You don't need equations to use it. You need the habit of asking three
questions about any process:

1. **Who are the players, and what does each one want?** That includes you, your users,
   attackers, reviewers, vendors, and the agent itself.
2. **What does each move cost, and what does it pay?**
3. **If one of those costs changed by a factor of 100, would the best strategy change?**

The third question is the one AI agents force on you. Many of the "best practices" you
learned are the best response to **what human labour used to cost**. An agent can now
write, read, test and rewrite code for a small fraction of that cost, often in minutes
instead of days. When an input cost drops that far, the right answer can flip to the
opposite of what experience tells you.

## Strategies that flip when work becomes cheap

| old equilibrium (human time is expensive) | new equilibrium (agent time is cheap) |
|---|---|
| Write one implementation and patch it until it works | Try three approaches in parallel, compare them, keep the best one. Throwaway prototypes are nearly free now |
| Tests are a cost centre, so test the main paths | Exhaustive tests, property tests and a test for every state × event cell (see [STATE-MACHINES.md](STATE-MACHINES.md)) are cheap insurance |
| "We'll document it later" (never) | Every PR carries a full description and a changelog entry, because writing them costs seconds |
| Read the code you need, skim the rest | Have an agent read the *whole* codebase, every dependency's changelog, or every issue in the upstream tracker before deciding |
| Patch around legacy code because rewriting is too expensive | A careful rewrite behind good tests can be cheaper than the patches. Wanting a rewrite still needs justification, but the price has dropped |
| Build it ourselves: research takes days | Research takes minutes, so look for prior art first, always ([PRIOR-ART-AND-SKILLS.md](PRIOR-ART-AND-SKILLS.md)) |
| Automate a chore only if it recurs often enough to repay the script | Automate after the second time, because the script costs almost nothing to write |
| Migrations and audits happen once a year, if ever | Sweep the codebase for dead code, stale checks and drift on a schedule |

## What becomes *more* expensive

Cheap generation moves the bottleneck. It does not remove it.

- **Your attention is now the scarce resource.** An agent can produce more code, PRs and
  options than you can review. Spend agent time to save your time. Ask for summaries,
  tests that prove the claim, and a decision framed as a question. Don't ask for more
  output.
- **Verification costs more than generation.** Code that writes itself is cheap. Knowing
  it is right is not. Invest in CI, type checkers, property tests and reviewers (human or
  a stronger model) in proportion. A 10× increase in generated code with the same review
  capacity is a 10× increase in unreviewed code.
- **Mistakes scale too.** A wrong instruction in CLAUDE.md is applied in every session.
  A bad pattern gets copied across a hundred files in an hour. Cheap action raises the
  value of good defaults and guardrails.

## Incentives: the agent is a player too

Economists call it the **principal–agent problem**: you (the principal) want working
software, and the agent is rewarded, loosely, for *looking* done. Most bad agent
behaviour is a rational response to the incentives you set up:

- **Goodhart's law:** "when a measure becomes a target, it ceases to be a good measure."
  Ask an agent to "make the tests pass" and the cheapest path may be to weaken the
  tests. Make the target the real goal ("the feature works, and here is how we will check"),
  and protect the measure (CI the agent cannot edit, tests reviewed separately).
- **Reporting:** if "done" is accepted without evidence, unverified "done" is the
  cheapest output. Require the commands run and their results, and the reporting changes.
- **Asymmetric information:** the agent knows what it skipped, and you don't. Ask
  explicitly for "what you didn't test" and "what you're unsure of". Making those cheap to
  say is how you get them said.

## The adversary's economics change too

Everything that got cheaper for you got cheaper for attackers and spammers: phishing
that reads like a colleague wrote it, automated vulnerability hunting, fake reviews,
bulk account creation, prompt injection hidden in web pages your agent reads. When you
design a system, assume the attacker has an agent too:

- anything that relied on "too tedious for anyone to bother" no longer protects you;
- rate limits, authentication and verification matter more, and obscurity less;
- your own agent reading untrusted content is a new attack surface. Text from a page or
  file is data, never instructions.

## Classic patterns worth knowing by name

Asking the agent "which game-theory patterns apply here?" works well, and it helps to
recognise the names:

- **Prisoner's dilemma / tragedy of the commons:** everyone defecting is individually
  rational and collectively bad (shared CI minutes, a shared cluster, a shared
  rate-limited API key). Fix it with rules and quotas, not appeals.
- **Repeated games:** cooperation emerges when players meet again. Upstream maintainers
  remember contributors. A good first PR buys goodwill for the tenth.
- **Signalling:** costly signals are believed and cheap ones aren't. When anyone can
  generate a polished PR description, the *test results* become the signal, not the prose.
- **Mechanism design:** designing the rules so that each player's selfish best move is
  the outcome you want, such as auctions, bounties and review rotations. A good CLAUDE.md
  is a small piece of mechanism design.
- **Jevons paradox:** making something cheaper usually increases total use. Cheap agent
  work tends to raise your total spend and output, not lower it. Budget accordingly.
- **Comparative advantage:** even if the agent were better at everything, you still gain
  by each doing what you are *relatively* best at. For you that is usually judgment,
  taste, priorities and accountability.

## A prompt to try

> Analyse this decision as a game. List the players and what each wants, the moves
> available, and the costs and payoffs of each. Then tell me which of those costs changes
> by an order of magnitude when an AI agent does the work, and whether that changes the
> best strategy. Flag any incentive that rewards looking done over being done.

## The rule (paste into your global CLAUDE.md)

> **Re-price old habits.** When a convention exists because work was expensive (skip the
> tests, patch instead of rewrite, one prototype only, document later), check whether
> agent-cheap work changes the answer, and say so. Treat my review time as the scarce
> resource: prefer evidence, tests and short decisions over more output. Never satisfy a
> metric by weakening the measure (editing tests, loosening checks) without telling me.
