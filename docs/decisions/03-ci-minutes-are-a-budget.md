# CI minutes are a budget

**Status:** in force since September 2026. **Owners:** the engineering team.

## Context

By 15:22 UTC on 15 September 2026, CI had spent 4,700 billable job-minutes across
889 runs that day. Two kinds of run could not have found anything:

- A push to `development` re-ran the typecheck, build, unit suite, database suite
  and dev-tool typecheck on a tree that had already passed all of them. The merge
  is a rebase onto `development` and a squash, and the merge commit's tree is the
  tested head's tree. There were 86 such pushes that day, for 1,227 minutes.
- A pull request that changed only prose paid for a full Next build, and one that
  never touched the database paid for a database to be started.

Between 1 and 13 October the bill was 39,235 minutes. The two browser suites were
15,680 of them, running in full on every pull request and every push.

## Decision

Decide per change which expensive work can still find something, in one pure
function, and gate the steps rather than the jobs.

The rule that governs every branch of that function: **every uncertainty runs
more, never less.** A missing API answer, a failed git call, an event the function
does not know, a check that is not green, a tree that differs by one byte — each
of them plans the full run. The only way to a skip is a complete argument, and the
argument is printed in the job log.

A skip is always a step condition, never a job condition. A job-level `if:` that is
false reports "skipped", and a skipped job reads as a passing required check. The
four jobs that back required contexts always start; the function decides what they
run inside.

## Consequences

- The plan is testable without GitHub: `decide(facts)` is pure, and the tests pin
  each rule, including the fails-toward-more ones.
- A path list is a guess about what a change can affect, so the guess is measured:
  the browser suites' reads and the unit suite's scope come from recorded data,
  not intuition.
- Some runs stay expensive on purpose: a release train, a pull request into
  `main`, and anything labelled for full CI run everything.
- A skipped suite has to be recoverable. A daily scheduled run covers the work
  that a narrowed pull request did not, so nothing is permanently unchecked.

## What we would reconsider

The path lists. Every one of them is an approximation, and each should be revisited
when it misses — the recorded measurements, not the argument, are what should
change our mind.
