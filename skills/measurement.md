---
name: measurement
description: Use when a work item includes a measurement (a benchmark, a scale run, a resource envelope) or builds a harness that checks properties over runs — what the plan requires of it
---
# Measurements and Property Harnesses

Named in the plan's Applicable Guidelines. Each rule below that binds becomes a requirement in the plan's Required Behaviors & Verifications, with its verification tag, and the batch sizing enters the same section as a dated amendment; Code and Test read them there.

## A measurement

- **Run it early.** The measurement runs at a nominal scale as soon as it compiles, not at the end,
  and is repeated until its run-to-run variance is visible. A measurement that first executes on the
  final day discovers its harness bugs on the final day.
- **Size the batch from the variance already seen.** The nominal runs show how often the effect
  appears and what uncontrolled condition it follows. When it appears in a fraction of runs, state
  how many runs an arm needs to see it and the smallest p the design can give; when it follows a
  condition the harness does not control, control it, or balance it across arms, before spending the
  batch. Where the nominal runs come after `plan.md`, the sizing goes into it as a dated amendment
  before the batch is spent; a reservation that cannot hold the batch is said to be so, and the
  result is reported as descriptive. A condition that was controlled is part of the claim and is
  stated with the result. A batch sized by the clock measures the clock.
- **Assert starting conditions.** The measurement checks the preconditions it depends on (a settled
  machine, available ports, an empty data directory) and refuses to run when they do not hold, naming
  what it saw — rather than assuming them and producing a number that looks like a finding.
- **The instrument reports its own state.** An instrument outside the population it measures — a
  probe, a load client, a sampler — records enough of itself (what it sent, what it received, how far
  behind it ran, what it failed to do) that each figure can be attributed to the system or to the
  instrument. A figure that could be either is not yet a result.

## A property harness

- **Violations carry evidence.** Each reported violation includes the state of the participants at
  the moment it happened, not only the fact of the violation. A one-line verdict forces a re-run with
  hand-added tracing for every diagnosis.
- **Exemptions expire.** An allowance carved out of a property names the work item that will remove
  it, and the suite reports how many times it fired. A silent exemption is load-bearing scope no one
  is tracking.
- **Properties state their non-vacuity.** For each property, say what makes it non-vacuous and assert
  that too (a run must actually deliver and acknowledge something). A property that passes over an
  idle system verifies nothing.
- **The harness's own client is tested.** The harness speaks a protocol; test its client against the
  same specification the product is judged by. When the harness reports a defect, the harness is one
  of the suspects.

A claim the measurement produces carries its task, its scale and its sample (`skills/evidence.md`).

Source: WIs 1300, 1618, 1657 (p = 0.17 from five pairs whose arms drew unlike conditions the nominal runs had already shown).
