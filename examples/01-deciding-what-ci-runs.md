# Deciding what CI should run

An excerpt from `scripts/ci/what-to-run.mjs`. The file decides, for one event,
which expensive checks run. It is pure where it makes decisions and does the git
and API reads in a separate function that never decides anything.

Most of it is a list of facts. What makes it safe to use is the rule at the top.

```js
/**
 * ── The one rule ──────────────────────────────────────────────────────────
 *
 * EVERY UNCERTAINTY RUNS MORE, NEVER LESS. A missing API answer, a failed git
 * call, an event this file does not know, a check run that is not green, a tree
 * that differs by one byte: each of those runs everything. The only way to a
 * skip is a complete argument, and the argument is printed.
 */
export function decide(facts) {
  const event = facts?.event;
  const ref = facts?.ref;

  if (event === 'pull_request') {
    const pr = facts.pullRequest ?? {};

    if (pr.error)
      return everything(`could not list the changed files (${pr.error})`);

    // Only a draft that says so is skipped; an unknown is not one.
    // GitHub cannot merge a draft, and `ready_for_review` starts the run
    // that decides, so no draft is merged unchecked.
    if (pr.draft === true) {
      return nothingHeavy(
        'draft: the local gate ran before it was opened, and marking it ready starts the checks that count',
      );
    }

    const fullProof = fullProofReason(pr);
    if (fullProof) return everything(fullProof);

    if (!Array.isArray(pr.changedFiles) || pr.changedFiles.length === 0) {
      return everything('the changed-file list is empty or unknown');
    }

    return { ...forChangedFiles(pr.changedFiles), ...unitScopeFor(pr) };
  }

  // ... push, schedule and manual events
}
```

## What to notice

- The error paths return `everything()`, not an empty plan. An API that did not
  answer is not evidence that nothing changed.
- A draft is the only case that skips, and it is safe because GitHub cannot merge
  a draft; the pull request that matters is the one marked ready, which starts a
  new run.
- An empty changed-file list is treated as unknown. It happens when the API
  returns a partial page, and the honest reading is that we do not know.
- Every decision carries a printed reason. The job log says why a suite did not
  run, which is the difference between a measured skip and a silent one.
- The plan does not switch jobs off; each job starts and gates its own expensive
  steps. A skipped job reports success, and a required check that never ran is
  worse than no check at all.

The full file also measures what a suite reads before narrowing it, and the tests
pin the failures-toward-more cases. [The decision that produced it](../docs/decisions/03-ci-minutes-are-a-budget.md)
has the numbers.
