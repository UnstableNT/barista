---
name: task-review
description: Review one pull request in this repository before it is merged — a task, retry or fix pull request, or a decision pull request. Use whenever a thread is started to review a pull request of this project.
---

# Task review

You decide whether one pull request may be merged. Nobody reads along and the
merge happens on your verdict, so a wrong *pass* lands on the default branch
unseen. You did not write this; review it as someone who did not.

Write the review in the language of the plan.

## What you review against

In this order of authority:

1. **CI on the pull request's head commit.** A claim in the run record that tests
   pass counts for nothing if CI says otherwise.
2. **`CLAUDE.md`** — invariants, security-critical core, build environment,
   dependency policy, which files a thread may write.
3. **The recorded decisions** the task lists, and any decision the diff touches —
   coordinator decisions count like any other.
4. **The task** — its section in `docs/plan.md`, plus the specification
   sections (`## Spezifikation`, §1–§11) it lists under **Reads**.

Not against your taste. Generic code quality is the least valuable thing you add.

## Procedure — task, retry and fix pull requests

1. **Anchor.** Note the pull request, its head SHA and its base. The review is
   valid for that SHA only. In round K > 1, also note the SHA of round K−1.
2. **Light pass** — always, before reading any implementation:
   - CI result per check on the head SHA.
   - The run record's `Result`: deviations and open points it flags come first.
   - Diffstat against the base (round K > 1: against the previously reviewed SHA).
   - **Scope check:** files outside the task or outside what `CLAUDE.md` lets
     the thread write (plan, briefs, decisions, status, other run records)?
     Invariant tests weakened, skipped or deleted? Tests the task says must stay
     red touched? New dependencies — each with a justification line?
   - Every acceptance test the task names exists and has its expected outcome in
     CI. CI-only checks: the result comes from CI logs, not from the thread.
     For Barista that is everything in the `macos` job: read the
     `scripts/integration.sh` result table and the `swift test` output there.
   - **Expected-failure markers:** the diff removes `withKnownIssue("TNNN")`
     and lines in `tests/integration/expected-failures.txt` only for the
     task under review (fix tasks `TNNN-fK`: for `TNNN`). Removing another
     task's marker, or changing a test's assertions, inputs or timeouts beyond
     what "Decision latitude" allows, is must-fix.

   Do the scope check before reading code, so good code cannot talk you out of
   noticing it was out of scope.
3. **Deep pass** — only the hunks that need it, when the task says
   `review: deep`, the diff touches the security-critical core or an invariant
   test, the light pass flagged something, or a new dependency appeared. Read a
   decision only when a specific finding needs it.
4. **Round K > 1:** re-check only the previous round's must-fix items and the
   diff since the previously reviewed SHA. Do not reopen what already passed.

## Procedure — decision pull requests

A decision thread records one design decision and adjusts the tasks it affects.
Check, without needing a deep pass:

- The change touches only `docs/plan.md`. No code, no run records.
- The decision follows the plan's decision-latitude criteria and names the
  alternative that lost.
- It does not change purpose, non-goals or the Definition of Done.
- Existing decisions are not edited — except the replaced one getting its
  `→ replaced by` marker or `superseded_by` field.
- Every task not yet merged that names the replaced decision or contradicts the
  new one is adjusted. Merged tasks are untouched.

## What to look for in code

- **Contradictions with recorded decisions** — literally, not in spirit. The most
  common failure: the task's wording is satisfied while a decision the task did
  not restate is violated.
- **Silent failure paths** — they pass CI by definition:
  - ordering or comparison running opposite to its meaning,
  - non-atomic writes (a marker written before the data it describes),
  - declared but unenforced constraints,
  - lossy conversions on data read back from disk or network,
  - tests that pass for the wrong reason: a guard that passes only because the
    feature is still a stub; an assertion a constant function satisfies. Guard
    tests need their complement.
- **New dependencies:** none are allowed (D11, `inv_no_third_party_deps`).
  Any `.package(` entry or vendored third-party code is must-fix.
- **Logic in the app layer:** a decision about *when* something happens
  (ending, suspending, choosing a mode) made in `BaristaAppKit` instead of
  `WakeEngine` violates D2, even if it works.
- **Fail-open (D5):** every path that ends or suspends a phase releases
  assertions; every error path ends in an `Event` or an `os_log` line.
- **Scope creep dressed as helpfulness**, and **boundary erosion** (a
  dependency's type leaking through an interface meant to hide it).
- **Finish-line drift** — work that serves no deliverable, exit criterion or
  Definition-of-Done item is scope, even if it is good.

## Verdict

- **pass** — CI green on the head SHA, every acceptance test of the task present
  with its expected outcome (decision pull request: every check above holds), no
  must-fix finding.
- **must-fix** — anything else. Each finding states a **target state** and an
  **acceptance test**, explained by an invariant, a decision or the task. Never a
  patch: you do not change code or plans.

Notes go in their own section; the fix thread ignores them unless you mark one
"fix in this round". Name what is done well, specifically — it tells later
threads which patterns to keep. Say what you verified and what you only read: a
review that reads like measurement but is inference gets over-trusted, and here
it gets merged.

## Recording

1. Write the review file on the pull request's branch:
   task pull request `docs/reviews/TNNN-rK.md`; decision pull request
   `docs/reviews/TNNN-dN-rK.md` (N = number of the decision for this task).
2. Task pull request only: add your line to the log in `docs/runs/TNNN.md`.
3. Commit and push. This commit may touch **only** those files; the verdict covers
   the reviewed SHA plus this commit and nothing else. If anything else changed
   after the reviewed SHA, the verdict is void.
4. End with a final message whose first line is exactly

       VERDICT: pass (reviewed <sha>)
   or
       VERDICT: must-fix (reviewed <sha>, round K)

   followed by the must-fix titles, one per line. In round 3 with must-fix, add:
   `Round 3 without pass — task is over budget per project instructions.`

## Template

```markdown
---
pr: <number>
task: TNNN
kind: task | decision
round: K
reviewed: <sha>
previous: <sha or none>
depth: light | deep
verdict: pass | must-fix
---

# TNNN — review round K

**Method:** <what you actually did: CI results read, run record read, scope
check, hunks of X and Y read. Say if you ran nothing yourself.>

## Verdict
<one paragraph>

## Verified
<what CI or a command you ran proves, distinct from what you inferred>

## What is right
<specific>

## Must fix
### MF1 — <finding>
<why, in terms of an invariant, decision or the task>
**Target state:** <what should be true>
**Acceptance test:** <named, checkable>

## Notes
### N1 — <finding>

## Not verified
<everything you could not check>
```
