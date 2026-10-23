# Building Discipline Gates for Autonomous AI Agents

*Mergen · October 2026. We have run an autonomous agent system since late April 2026, and since mid-August as two separate agent seats. Running it taught us that autonomy and reliability are different properties. This is what we built to keep them apart, why each piece exists, and what still breaks.*

## The problem

An agent that can execute tasks is not a system that can show its claims are true. Our agents open work, run tools, write records, and report "done". Every one of those steps can be wrong in a way that looks right: a search that returns "nothing found" because it searched the wrong population, a scheduled job reported as installed that never fired, a timestamp written from a sense of the time rather than a clock. None of these are hallucinations in the usual sense. They are ordinary engineering failures, produced faster and with more confidence than a human team would produce them. The operational risk is not that the agent does nothing. It is that the agent does a great deal and signs for it.

## The mechanisms

Four controls carry most of the weight. For each we say why it exists, what it made possible, and whether the agent platforms already offer it. We also mark how far our own evidence goes: *designed*, *implemented*, *tested*, or *demonstrated in live operation*. The platform check covers the public documentation of six agent frameworks, ten pages read in October 2026; where we say "not found", that is the coverage of those pages, not proof that the feature does not exist.

**1. Graded claims, and absence as the weakest class.** Every factual statement in a record carries one of four grades: recomputed (re-derived from raw data when written), cross-checked (a second independent source agrees), single-read (one source, read once), or inferred. An ungraded claim counts as unmeasured and cannot support a decision. "Nothing found" claims get a stricter rule: the first line must say what was searched, how many items were covered, and what was not.
- *Why:* our most expensive errors were not invented facts but confident absences: a search over the wrong set, reported as "nothing there", closed a question that was still open.
- *What it made possible:* a reviewer can go straight to the weakest claims under a decision instead of re-reading everything, and a grade can be withdrawn when a re-run disagrees.
- *Platforms:* we found no built-in claim grading in any of the six frameworks' pages.
- *Evidence:* demonstrated in operation; review gates check grades on every verdict file, and grades have been withdrawn after re-runs disagreed more than once.

**2. A cap on newly opened work.** New items opened per week are capped at a fixed fraction of what was actually closed the week before, read from the closure record, not from anyone's sense of how busy the week was.
- *Why:* a bright new task kept pushing a waiting high-stakes task out of view; nothing was wrong with either task, only with the order.
- *What it made possible:* opening work now costs closing work, so the backlog has to be faced rather than outgrown.
- *Platforms:* one framework exposes task-created and task-completed events on which such a gate could be written; none of the pages we read offers a work cap itself.
- *Evidence:* implemented and tested. The gate ran one week in warning mode and now refuses. It has not yet had to refuse a real opening, so its effect on the backlog is not demonstrated.

**3. Two seats and a second-look queue.** Work is split between two independent agent seats. Every finished verdict enters a confirmation queue that the other seat answers with its own measurement, not by re-reading the first seat's report. Agreement between two readers who used the same tool on the same data version does not count as independence. A lane gate at commit time refuses any commit that contains the other seat's lines.
- *Why:* one seat's confident report used to be accepted because it read well; the second case below is what that cost.
- *What it made possible:* a verdict that drives a decision has been measured twice, by two seats that could disagree, and neither seat can publish the other's unfinished work under its own name.
- *Platforms:* all six frameworks offer something partial (a verifier step, an output guardrail, a validation function), but inside the same run; we found no independent second-agent queue.
- *Evidence:* the lane gate is demonstrated and fires routinely. The confirmation queue is demonstrated but slow, and we have not yet shown that it gets faster.

**4. Paper versus life.** The question about any rule, guard, or job is not "does it exist" but "who consumed it in the last N days". A rule with no consumer in that window is treated as dead however well it is written. The same control applies to the controls themselves: a new gate must bite its author before it counts as installed, and a recurring mistake becomes an enforcing rule only after the candidate rule is replayed on real firings, with a minimum sample that the measuring tool enforces and a false-positive share of at most ten percent.
- *Why:* we kept finding rules that were well written and never read, and jobs that were installed and never ran (the first case below).
- *What it made possible:* dead rules get found by a query instead of by accident, and rule-writing stops being a way to feel finished.
- *Platforms:* two frameworks offer tracing or event logs that could feed such a check; we found no count of which rules were actually consumed.
- *Evidence:* demonstrated for gates; designed and partly tested for rule promotion. Of the five most frequent error classes (675 lessons between them, counted 21 October 2026), 43 lessons carry a rule address and the other 632 do not. That is the control working as intended, and it is also an admission that most of our lessons are not yet rules.

## Three cases

**The scheduled job that never fired.** A data-collection job was recorded as installed. Two and a half days later its output was stale: it had never run. The liveness check had matched the scheduler's own command text, and the pattern looked for a full path while the process used a relative one. Control: paper versus life. Outcome: the check now requires a first real firing before the job is listed as live. *Observed event, demonstrated fix.*

**Same tool, different data version.** Two seats asked how a backlog counter had moved over time. Both ran the same canonical tool and both labelled the result "recomputed". One got a smooth rising series, the other a sharp drop. The first had moved the as-of date backwards but read today's ledger, which already held records that did not exist on the earlier dates, a look-ahead leak. Control: the second-look queue, which requires the second seat to measure rather than agree. Outcome: the leaking series was withdrawn; "same tool" is no longer accepted as independence. *Observed event, demonstrated fix.*

**A hundred records counted eight times.** Two public endpoints failed the same night in the same shape. One ignored its paging parameter and returned the first page every time, so a hundred records were counted eight times; the other embedded seven overlapping lists in one page. Both returned success and a valid schema. What exposed them was arithmetic: every total divisible by eight, unique keys fewer than rows. Control: graded claims, because a "recomputed" total must survive a unique-key count. Outcome: pulled data now passes a divisor and uniqueness check before any number is written. *Observed event, demonstrated fix.*

## Limitations

These controls do not make the system correct. They make unsupported claims harder to write and easier to find afterwards. They do not detect a confident wrong answer that is graded honestly as single-read. The weekly cap has not yet refused anything. The rule-promotion threshold was chosen by judgement, not derived. The lesson ledger has been kept since 5 August 2026. At the time of writing it holds 1,269 recorded lessons in 189 recurring error classes, and the case book holds over ninety worked cases (counted 21 October 2026). Of those 1,269 lessons, 131 carry the address of a rule or gate; the rest are still lessons, not gates. The platform comparison is a reading of ten documentation pages, not of the products. And every number in this article is from our own records; nobody outside has audited them.

## What a team can take from this

Grade every claim your agents write, and treat ungraded output as unmeasured. Make "nothing found" state its coverage. Cap new work against real closures. Never let two readers of the same report count as two measurements. Ask who consumed a rule, not whether it exists. And distinguish, in your own documents, between a control you designed, one you installed, one you tested, and one that has demonstrably changed an outcome.

## Closing

If your team runs autonomous workflows and wants to compare notes on how claims are verified before they are acted on, open an issue at https://github.com/open-discipline/agent-discipline-gates/issues. We will not pretend the problem is solved; we can show how we measure it.
