---
name: live-stack-testing
description: Use when writing, fixing, or choosing the tier of a test that drives a running multi-process system — a cluster simulation, a live server, a multiplayer end-to-end harness — or when such a test fails intermittently.
---

# Live-Stack Testing

A test that acts on a running system from outside and waits for it to answer. The Go forms of the
in-process rules are in `skills/go.md` and `skills/go/testing.md`; negative controls are `Test.md` step 3.

## Choose the tier first

The first row that matches a claim decides where it is judged. A change whose claims fall on
different rows is split, each claim tested on its own row.

| The change is… | Test it with… |
|---|---|
| a decision — a calculation, a parse, a predicate over state | a unit test; extract the decision first if it is inline |
| an interaction inside one process — a state machine, a protocol exchange, a lock | an in-process test or simulation |
| behaviour that needs real processes, real sockets, or a scale the simulation cannot reach | a batch on a shared test host, run by its runbook |
| a judgement or a device — how it looks, how it plays | the owner, by hand |

- The batch runs only after a nominal-scale run on the cheaper tier has passed, so it has nothing
  left to fix, and it is sized before it runs to decide its question (`Plan.md`, *Size the batch
  from the variance already seen*).
- A defect found on an expensive tier is pinned afterwards by a test on the cheapest tier that fails
  on it.
- A test in the gate finishes in minutes, on the stack as shipped; a harness that assumes one host's
  tuning fails on the next. Anything longer is a batch, run by its runbook, not by the gate.

## Arm, act, wait

```go
before := c.Delivered(sub)              // arm: the baseline, read before acting
c.Publish(topic, msg)                   // act
waitUntil(t, 10*time.Second, func() bool { // wait: past the baseline, bounded
	return c.Delivered(sub) > before
})
```

- **Arm before acting.** A wait set up after the action can be met by a step already queued, or by
  state from before the action, and then it measures nothing.
- **Wait on a condition, never a fixed sleep.** A sleep sized on one machine fails under a race
  detector or load. A short sleep between polls of the condition is the wait, not a substitute for
  it. The timer's only job is to bound the wait so a hang fails by name; a duration the scenario
  itself needs is commented as one.
- **Wait for what the next step needs, not for "started".** After an asynchronous command, wait until
  the system will do what the next step assumes — a node that accepts a publish, not one that is
  serving — probing it on something the test does not judge. Wait for everything the next step reads
  to have settled, on every participant it reads it from, never a part that an intermediate state
  also matches.
- **Waiting past a baseline proves arrival, not that nothing more came.** Where the exact count is
  the claim, assert it once, after every source that could add to it has stopped (`skills/go.md`).
- **A time oracle is a bounded window with a stated threshold**, derived from measured runs, starting
  at the event it times rather than at a reading taken before it.

## Judge the behaviour

- **The oracle is what the system did**: the reason code, the delivered message, the acknowledged
  count. "No error" or "any reply" passes a system that refuses everything. A failure prints the state
  it judged — each participant's view, the identifiers — so diagnosing it needs no re-run.
- **A setup that never reached the state under test is a precondition failure**, reported as one:
  not a pass and not a skip that reads as a pass. Its message names the state that was not reached
  rather than a cause, since a setup built from product behaviour can fail because of the product. A
  test that skips when it cannot build its case has never tested that case, and a run in which the
  condition never arose is not a sample. A measurement refuses to start on an unsettled machine
  (`Plan.md`, *Assert starting conditions*).
- **A test for a defect still open** is disabled whole under the work item's number, and re-enabling
  it is part of that fix: it fails on the build without the fix and passes with it. Never a soft pass,
  never an exemption no work item owns.

## Keep concurrent runs apart

- Every port the live system binds, and anything else it claims by name, comes from a per-process record
  that hands each value out once, registers cleanup at creation, and fails by name when the pool is
  exhausted. The record covers one process; a collision with another process still needs a bounded
  re-draw.
- A batch reserves its host whole, and the gates do not run on that host meanwhile.

## Intermittent failures

- Against a correct product, an intermittent failure is a harness defect until shown otherwise.
- Read every one and classify it — environment, timing, collision, leftover state, or product race.
  Repeating it to classify it is fine; repeating it until it goes green is not.
- Never retry on failure, and never raise a timeout to pass: both hide the defect the flake is
  reporting.

Source: WIs 1205, 1273, 1293, 1308, 1488, 1498, 1500, 1533, 1565, 1570, 1661, 1699, 1707.
