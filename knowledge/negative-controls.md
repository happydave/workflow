# Negative controls — the sweep, and the cases behind the rules

`Test.md` step 3 says which controls every Test runs and when a full sweep runs. This file says how
a sweep runs — the rules under **Running a sweep** bind whenever one does — and records the cases
that produced them.

## Running a sweep

The scope is the plan's behaviors: every behavior in its **Required Behaviors & Verifications** and
**Edge Cases**, not every new test and not every line of the change. Read the first column of the
table under **Record** before breaking anything, and again after each run, since some rows are
keyed to a result: a matching row adds to the obligations here, and a behavior that matches no row
owes only these.

### Plan the set

- **Name where each observable can be seen**: the fixture feature that would differ if the behavior
  were violated — a channel, a field, a state — and the moment the assertion samples it. An
  observable the fixture cannot carry is untested however many controls fail, because a control
  trips only assertions the fixture lets differ. An effect that ends before the assertion samples —
  a goroutine that starts and returns, a flag a later step clears, input that stops before the
  broken state is reached — is asserted through the decision that produces it, with a test still
  reaching the call site, or the input is extended until the effect holds when the assertion
  samples. When a fixture changes, every assertion reading it is named again and its control re-run,
  not only the new ones.
- **One break per observable.** Read the observables from what the behavior claims rather than from
  the count of its verification bullets: split a claim wherever a plausible weaker implementation
  could satisfy one part of it and not the other.
- **Where another site checks the same rule**, break each site alone and then all of them together.
  A site an earlier refusal makes unreachable is one finding, not a split repeated for every
  observable whose path crosses it.
- **Two breaks are exempt from one-per-observable**: the combined break just described, and a break
  re-expressed because the first attempt failed before the assertion that targets the behavior (the
  table's row *fails at an earlier assertion, or before any assertion runs*). A re-expression and the attempt it replaces are both recorded, each as its own
  row, naming the other.

### Run them one at a time

- **Start from a committed tree and restore to it between controls** — a combined break is one
  control. `git status` shows the tree clean before the first control, because a break restored by
  checkout reverts every uncommitted edit in its file (WI 1828); a test fixed during this step is
  committed before the next control, so the next restore keeps it. After the last control, a diff
  against that commit shows nothing: a control left applied forges every result after it.
- **Run with any compiled-code cache off** — for Python, `-B` with `__pycache__` removed; Go's
  caches key on content, so `-count=1` is enough there. A cache revalidated by timestamp and size
  runs the previous break after a same-size edit within the timestamp's resolution, while the
  source still reads as the new break (WI 1732).

### Read what came back

- **Read which cases failed, not only that something did.** A break that fails more cases than
  expected, or fewer, has located a gap.
- **A break that hangs or panics ends its test binary**, and no case after it in that binary ran.
  Run controls with a timeout, read the log for the panic or the timeout's dump, and re-run each
  case the break targets on its own (Go's `-run`, or the runner's equivalent) before reading which
  cases failed. A targeted case that panics again on its own failed before any assertion (the row
  *fails at an earlier assertion, or before any assertion runs*) (WI 1275).
- **Where a behavior is reached by more than one entry point**, say which of them the suite covered
  it at, whether the break failed something or nothing.
- **Show the break is live** before reading a nil result as missing coverage: a temporary test that
  reads the mutated state, or its first effect, directly, deleted once it has answered and before
  the restore is checked, because a
  break that never executed reports the same nothing. Where the break is unobservable by
  construction — two checks of one rule that refuse identically — the combined break discharges
  this, and the unobservability is itself the finding.
- **A break that fails nothing is a finding for step 5**, with four readings to separate: the
  behavior has no test; the break ran but its effect ended before the assertion sampled, which the
  liveness probe tells apart from the last reading; another check already does the broken one's
  work; or nothing reaches what you broke. Name which one it was — or which ones, since a masked
  path is both unreached and covered by whatever masks it.
- **Read the multi-site split**: every single break passing while the combined one fails means the
  sites are redundant, unless no case reaches one site alone — a fixture that trips two sites at
  once hides each behind the other, which is a fixture gap; a single break that fails on its own is the load-bearing one, and the sites
  that passed are either its redundant copies or unreachable, which the applied diff tells apart; a
  combined break that also fails nothing means the behavior is untested.

### Record

- **Each control is a row in `test.md`**, in these columns: the behavior, the break as written — the exact expression — and where it was applied, the
  fixture feature and moment its assertion reads, the applied-state check, the tests it failed, the
  assertions that tripped, anything it failed or spared unexpectedly — including a failure at an
  assertion belonging to another behavior — the runs, and the reading where nothing failed, left
  blank where something did. One run completes a deterministic control, such as an error versus
  nil.
- **Where a control could not be run**, name the behavior and the reason, so an omitted control is
  never indistinguishable from one that passed. A plan verification with no test is not an omitted
  control: it is a finding for step 5 in its own right. An observable the fixture cannot carry is recorded
  the same way, as untested, and is a finding for step 5.

| When the behavior or its control… | Then… | Source |
|---|---|---|
| guards a destructive, irreversible, or outward-facing operation | run the control against the predicate that decides, never by driving the operation with arguments the test did not create — *A destructive operation decides in a pure predicate*, `AGENTS.md`. A control runs with the guard removed, so a control that reaches the operation performs it. Where the decision is not separable from the act, do not run the control: the non-separability is the finding, and the restructuring comes first. | WI 1378; the 2026-09-08 loss |
| could pass or fail by chance — a choice among waiters, scheduling, a random port, a race | one red run is a coin. Run the control repeatedly and record failed-of-total; the passes in that split, over the total, are the rate at which the test misses this defect, and the remedy is to restructure the control so a wrong implementation cannot pass it by luck — typically by extracting the decision and driving it directly, where one run then decides. The remedy is owed even at 0 misses in N: N runs bound the rate, they do not remove the chance. Where the test can also fail by chance on the unbroken tree, run that tree the same number of times, or a red control is a coin of its own. | WI 1500 |
| fails at an earlier assertion, or before any assertion runs | that is not coverage of this behavior: the tripped assertion must be the one that targets it. The test has other teeth; this behavior may have none. Re-express the break to reach the assertion that targets this behavior — holding the earlier claim constant — and record both attempts. | WI 1486 |
| is broken by deleting a block | express the break so the artifact still builds — flip the whole condition (`if false && (…)` — a bare `false &&` prefix binds only to the first operand of an `||`), assign the zero value, discard the result, return early — and delete only where the deletion leaves nothing unused. A deletion that strands a variable or an import fails the build, and a build failure reads as *the tests did not notice* while proving nothing about the suite. | WIs 1644, 1646, 1667–1669, 1275 |
| is broken by moving a derived value — a hash, a timestamp, an identifier the view under test does not carry | move the value the property is about, and confirm the test can tell the difference: a timestamp shifted by a second under a millisecond assertion, or a key changed so the record leaves the view entirely, reports *did not fail* about something else. | WIs 1648, 1670 |
| mutates a target that is not unique in the file, is applied by a replace helper or script, or runs against an artifact a stale build could answer for | verify the applied state — the diff, a fresh build of the mutated tree — before reading the result, and record that check in its column. The result is exactly what a broken control forges. | WIs 1401, 1403 |

This is `skills/evidence.md`'s "actively seek contradictory evidence" applied to the suite itself: a control that *should* fail is what distinguishes a test with teeth from a test that agrees with whatever it is given.

## Worked cases

### A guard tested by driving the operation (2026-09-08; WI 1378)

A failover driver's store remover was guarded by an inline path check and tested by calling the remover with `/`, `/etc` and the home directory — live arguments, on the theory that the guard would refuse them. Running the negative control means removing the guard, and with the guard removed the remover ran. It destroyed a host's home directory.

The rebuilt shape: `validateStoreRoot` decides and touches nothing; `removeStore` calls it before `os.RemoveAll`. The guard's test asserts on `validateStoreRoot` with the dangerous paths, which a predicate cannot act on. A second test hands `removeStore` only a directory it created itself. Breaking the guard fails the first test and removes nothing; that was verified by running exactly this control.

### A mutation that passed the whole suite (md-mcp WI 1169)

A mutation removed the invariant the plan had singled out as the subtle one and the entire suite stayed green. Every other case was byte-identical either way, so the assertions that appeared to cover the invariant did not; it had no test at all. A control that fails nothing has located a behavior without coverage.

### A control that failed 3 runs in 20 (hoardmq WI 1500)

The behavior: a SUBACK settles the subscribe that sent it, not whichever subscribe is waiting. The first control removed the packet-identifier lookup and ran a two-subscribe concurrency test, which failed — once. Run twenty times it failed about three times: with two waiters, a wrong implementation picks correctly half the time, and the test's timing narrowed that further.

The rebuilt control extracted the dispatch into `settleSuback` and drove it with sixteen waiters, which a packet-blind implementation would have to guess right sixteen times running. That control fails 10 of 10. The concurrency test was kept as coverage of the real path, not as the control.

### A control that failed for the wrong reason (hoardmq WI 1486)

A control removed a behavior and the test failed — at an assertion earlier than the one targeting the behavior. The row was nearly recorded as evidence. The test had other teeth; the behavior under control had none proven.

### Two controls whose mutation never landed (WIs 1401, 1403)

In one, the mutation did not compile, so the previously built artifact answered in its place and the control "passed". In the other, the mutated string appeared twice in the file and a replace-first-occurrence helper exercised one site twice, leaving the second uncontrolled. Both produced exactly the expected output; both were caught by reading the mutation mechanism, not the result.

### Seven controls that failed, over a fixture that could not carry the property (WI 1611)

An image-token suite asserted size, format and the absence of a baked border, and ran seven negative controls; every one failed the test that targeted it. Every token it shipped had its alpha inverted. The fixture was an RGB image with no alpha channel, so no assertion could observe polarity and no break could reveal the gap — each control tripped an assertion the fixture could satisfy. It was found by the next work item that used the output. Replacing the fixture (WI 1636) then silently invalidated an existing border check that had assumed a flat image, which is why a changed fixture re-opens every assertion reading it.

### A break that ran, over an effect that ended before the assertion (WIs 1705, 1618)

A control disabled a refusal so that a stopped component would start a background goroutine and hold its wait group; the test asserted the group was not held over a two-second window. The break was live — the goroutine started — but for an unreachable peer it returned almost at once, so the window saw nothing and the control failed nothing. Read as *nothing reaches the break*, it would have closed as a pass. The remedy was a test asserting the refusal itself. In the second case the test fed a single reading after the state it meant to observe had settled; the remedy was to extend its input.

### Two controls that ran the previous break (WI 1732)

Two Python controls edited same-length tokens ("fail" to "pass", "skip" to "fail") within a second of the previous restore. CPython revalidates a cached `.pyc` by the source's modification time, at one-second resolution, and its size, so the previous break's bytecode ran. One control failed nothing; the other reported a different control's failures. The applied-state check passed both times, because the source was right; the executed code was not. Re-run with `python3 -B` and the cache removed, both failed the tests that target them.

### A break the fixture routed around, and one that hid its target (WI 1275)

A refusal written as one condition over several keys was broken by prefixing `false && `. `&&` binds tighter than `||`, so only the first key's clause was disabled, and the fixture removed a key that a surviving clause still refused: the control failed nothing and read as missing coverage until the whole condition was wrapped. In the same suite, removing a length check made the first subtest panic with an index out of range. The panic ended the binary before the sibling subtest that trips the targeted assertion; re-run alone with `-run`, the sibling failed at its assertion.
