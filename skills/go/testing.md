---
name: go-testing
description: Use when a Go work item writes or changes a test of concurrent code, a benchmark, or a test that blocks a production goroutine
---
# Go: Concurrent Tests and Benchmarks

Sub-file of `skills/go.md`; its running-tests and universal writing rules apply first.

- **A benchmark's parallelism is part of its claim.** `-cpu` and `b.RunParallel` set the concurrency
  the result is *about*. Go weights mutex profiles by the number of blocked goroutines precisely
  because a lock with 100 waiters dominates one with 1 — so a change to lock scope measured at tens
  of goroutines says nothing about thousands, and "neutral in the benchmark" has shipped a change
  that stopped a system's connect path completing at all. Measure at the concurrency the system
  reaches, or record that you did not.
- **Concurrent test harnesses need production-grade discipline.** Collectors, fakes, and recorders shared between the code under test and the test itself must be locked like any shared state. A single green `-race` run is one schedule, not evidence that none races; `-race -count=5`, at the points `skills/go.md` names, finds a race but does not measure how often one happens. The detector reports each pair of racing locations once per process and `-count=5` runs in one process, so a race hit on every iteration reads as 1 FAIL and 4 PASS. To measure a rate, run separate processes (`-count=1`, N times). Source: WI 1713.
- **Force the contention you want to detect.** The detector reports only interleavings a test actually produces, so a concurrent path needs a test that drives many goroutines at it — a happy-path call proves nothing about a lock you forgot. This is the design half; repetition is the other, and it can find only the interleavings the design produces.
- **A test that blocks a production goroutine must be able to release it from both paths.** Bind the release first — `unblock := sync.OnceFunc(func() { close(release) })` — then `t.Cleanup(unblock)` *and* call `unblock()` where the assertion needs the goroutine to move. Ordering has a precondition most tests get wrong: every `defer` in the test body runs **before** any `t.Cleanup`, so a blocking teardown written as `defer wg.Wait()` can never be outrun. (`srv.Close()` is the wrong example to reach for here: it does not block on hijacked connections either, which is its own trap.) Register the blocking teardown with `t.Cleanup` too, then register the release after it — cleanups are LIFO, so "after" means it runs first. `OnceFunc` makes the extra registration free. Releasing only at teardown hangs the package run when an assertion fails; closing the channel in both places panics. A hung package prints nothing, which is strictly less diagnosable than a failure — pass `-timeout` so a hang becomes a goroutine dump.
