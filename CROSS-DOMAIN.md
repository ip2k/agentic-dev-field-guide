# Borrow solutions from other fields: lexicons, analogies and TRIZ

Every field has its own vocabulary, its **terms of art**. A pilot says "read-back", a nurse
says "triage", a railway engineer says "token", an accountant says "reconciliation". The
words hide something useful: **many of them name the same few underlying problems**, solved
independently, sometimes decades apart.

Learn to see past the jargon to the general problem, and you can take a solution that
another industry has already proven and bring it into your own. People who do this look
like inventors. Usually they are good borrowers.

## The names for this

There are several overlapping names for "solving a problem in my field with a solution
from another one":

- **TRIZ** (a Russian acronym for "theory of inventive problem solving"). Genrich
  Altshuller and colleagues studied a very large number of patents from many industries.
  They found that most inventions reuse a small set of general principles: about 40
  "inventive principles", plus a "contradiction matrix" for choosing between them. The
  method: restate your problem in general terms ("I need it stronger without making it
  heavier"), look up how other fields resolved that contradiction, and translate the
  answer back.
- **Analogical transfer** (or analogical reasoning): the cognitive-science term. The key
  finding is the difference between a *surface* resemblance (both involve water) and a
  *structural* one (both have a bottleneck feeding a queue). Only the structural ones
  transfer.
- **Technology brokering**, the **Medici effect**, cross-pollination: business-book terms for
  people and firms that carry ideas between industries, and for the innovation that
  happens where fields meet.
- **Pattern language**: Christopher Alexander's catalogue of recurring solutions in
  architecture. Software borrowed both the idea and the name, as "design patterns",
  which is itself an example of the thing.

## The same problem under many names

The quickest way to see it is in the vocabulary itself. Software especially is built from
borrowed words, and each one marks a solution that crossed over:

| general problem | what different fields call their solution |
|---|---|
| Don't let a fault spread | electrical **circuit breaker**; stock-market **circuit breaker** (trading halts); software **circuit breaker** pattern; ship **bulkheads**, which software also borrowed |
| Test on a small, expendable group first | the miner's **canary**; medicine's **phase 1 trial**; software's **canary deploy**; retail's **test market** |
| Something must not react to every tiny change | physics' **hysteresis**; electronics' **debounce**; games' **cooldown**; APIs' **rate limiting**; thermostats' **deadband** |
| Only one user of a shared resource at a time | single-track railways' **token** (a physical staff the driver must hold); computing's **mutex** and **token ring**; a meeting's talking stick |
| Slow the producer when the consumer is full | plumbing's **back pressure**; networking's **flow control**; manufacturing's **kanban** WIP limits |
| Decide what to handle first when overloaded | medicine's **triage**; incident **severity levels**; support **priority queues**; load shedding on the power grid |
| Confirm the instruction was understood | aviation and the military's **read-back** / **hear-back**; a waiter repeating your order; asking an agent to restate its brief before it starts |
| Make it impossible to skip a critical step | aviation **checklists**; the WHO surgical safety checklist (borrowed from aviation); software **pre-commit hooks** and **CI gates**; Toyota's **poka-yoke** (mistake-proofing) |
| Anyone may stop the line when they see a defect | Toyota's **andon cord**; aviation's **crew resource management**; failing CI blocking a merge |
| Keep an append-only record so mistakes can be found and fixed | accounting's **double-entry ledger** and **reconciliation**; databases' **write-ahead log**; **event sourcing**; filesystem **journaling** |
| Hand work over without losing anything | hospitals' shift **handover** (one children's hospital redesigned its ICU handover after watching a Formula 1 pit crew); a **handoff** note before an agent compacts |
| Spread information by contact | epidemiology's **SIR models**; distributed systems' **gossip** (or "epidemic") protocols; marketing's "viral" spread |
| State the goal so people can adapt the plan | the military's **commander's intent**; a good sub-agent brief ("the goal is X; if the plan breaks, optimise for X") |
| Prepare everything before the rush | a kitchen's **mise en place**; a pilot's pre-flight; warming caches and pre-building images before a deploy |
| Practise in a realistic but safe setting | theatre's **dress rehearsal**; flight **simulators**; software **staging** environments; fire **drills** and **game days** |

Every row is a solution that one field paid for, in accidents, failures and lost money,
and another field reused for free once someone noticed the resemblance.

## Why AI models are good at this

The hard part of borrowing is knowing that the other field exists, and what it calls its
version of your problem. A human expert knows one or two fields deeply. A general model
has read the textbooks, papers, manuals and forum threads of hundreds of fields, so it
knows the jargon on both sides and can translate between them. Used deliberately, that
breadth is one of the most valuable things these models offer, and most people never ask
for it.

What you bring is the problem, the judgment about which analogies are structural and which
are only on the surface, and enough familiarity with a few vocabularies to recognise a
good lead.

## How to ask

> Restate my problem in domain-neutral terms: what is the underlying contradiction or
> constraint? Then list 8–10 other fields that face a structurally similar problem. For
> each, give the term of art they use, how they solve it, and how that solution would
> translate to my case, including where the analogy breaks. Name any TRIZ inventive
> principles that apply. Finish with the two most promising ideas and the cheapest way to
> test each.

Other prompts that work:

- "What would an air-traffic controller / a pharmacist / a logistics planner / a casino
  call this problem, and what do they do about it?"
- "Give me the terms of art for this problem in three other fields, so I can search their
  literature." (The vocabulary is the search key. Once you know a field calls it
  "reconciliation", you can find fifty years of work on it.)
- "Which field has the most at stake if this goes wrong, and what have they learned?"
  Safety-critical fields (aviation, medicine, nuclear, rail) have usually paid most for
  their lessons.

## Cautions

- **Check that the term is real.** Models can produce plausible-sounding jargon. Search
  for any term you plan to rely on, and read one primary source from that field.
- **Surface versus structure.** "Both are about flow" is not enough. Ask what the
  constraints, costs and failure modes are on each side. If those don't match, the
  solution probably won't transfer.
- **Context differs.** A hospital checklist works partly because of the hospital's culture
  and accountability. Copy the mechanism, and check you have the conditions it depends on.
- **Credit and licences still apply.** Borrowing an idea is free. Borrowing someone's
  code, data or text is not (see [PRIOR-ART-AND-SKILLS.md](PRIOR-ART-AND-SKILLS.md)).

## Build the habit

- When you meet an unfamiliar term of art, ask what general problem it names. Write the
  answer down.
- Read a little outside your field: postmortems from aviation, medicine and finance are
  short and full of transferable lessons.
- When you are stuck, ask "who else has this problem, and what do they call it?" before
  "how do I solve this?".

## The rule (paste into your global CLAUDE.md)

> **Look across fields before inventing.** When a problem is hard or novel, restate it in
> domain-neutral terms and check how other fields (especially safety-critical ones) solve the
> same structure. Give me their terms of art so I can search them, explain where each
> analogy breaks, and flag any term you are not sure is real.
