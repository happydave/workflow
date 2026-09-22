# Negative controls — worked cases

Reference for `procedures/Test.md` step 3. Non-normative: the rules live in the procedure; these are the cases that produced them.

## A guard tested by driving the operation (2026-09-08; WI 1378)

A failover driver's store remover was guarded by an inline path check and tested by calling the remover with `/`, `/etc` and the home directory — live arguments, on the theory that the guard would refuse them. Running the negative control means removing the guard, and with the guard removed the remover ran. It destroyed a host's home directory.

The rebuilt shape: `validateStoreRoot` decides and touches nothing; `removeStore` calls it before `os.RemoveAll`. The guard's test asserts on `validateStoreRoot` with the dangerous paths, which a predicate cannot act on. A second test hands `removeStore` only a directory it created itself. Breaking the guard fails the first test and removes nothing; that was verified by running exactly this control.

## A mutation that passed the whole suite (md-mcp WI 1169)

A mutation removed the invariant the plan had singled out as the subtle one and the entire suite stayed green. Every other case was byte-identical either way, so the assertions that appeared to cover the invariant did not; it had no test at all. A control that fails nothing has located a behavior without coverage.

## A control that failed 3 runs in 20 (hoardmq WI 1500)

The behavior: a SUBACK settles the subscribe that sent it, not whichever subscribe is waiting. The first control removed the packet-identifier lookup and ran a two-subscribe concurrency test, which failed — once. Run twenty times it failed about three times: with two waiters, a wrong implementation picks correctly half the time, and the test's timing narrowed that further.

The rebuilt control extracted the dispatch into `settleSuback` and drove it with sixteen waiters, which a packet-blind implementation would have to guess right sixteen times running. That control fails 10 of 10. The concurrency test was kept as coverage of the real path, not as the control.

## A control that failed for the wrong reason (hoardmq WI 1486)

A control removed a behavior and the test failed — at an assertion earlier than the one targeting the behavior. The row was nearly recorded as evidence. The test had other teeth; the behavior under control had none proven.

## Two controls whose mutation never landed (WIs 1401, 1403)

In one, the mutation did not compile, so the previously built artifact answered in its place and the control "passed". In the other, the mutated string appeared twice in the file and a replace-first-occurrence helper exercised one site twice, leaving the second uncontrolled. Both produced exactly the expected output; both were caught by reading the mutation mechanism, not the result.

## Seven controls that failed, over a fixture that could not carry the property (WI 1611)

An image-token suite asserted size, format and the absence of a baked border, and ran seven negative controls; every one failed the test that targeted it. Every token it shipped had its alpha inverted. The fixture was an RGB image with no alpha channel, so no assertion could observe polarity and no break could reveal the gap — each control tripped an assertion the fixture could satisfy. It was found by the next work item that used the output. Replacing the fixture (WI 1636) then silently invalidated an existing border check that had assumed a flat image, which is why a changed fixture re-opens every assertion reading it.

## A break that ran, over an effect that ended before the assertion (WIs 1705, 1618)

A control disabled a refusal so that a stopped component would start a background goroutine and hold its wait group; the test asserted the group was not held over a two-second window. The break was live — the goroutine started — but for an unreachable peer it returned almost at once, so the window saw nothing and the control failed nothing. Read as *nothing reaches the break*, it would have closed as a pass. The remedy was a test asserting the refusal itself. In the second case the test fed a single reading after the state it meant to observe had settled; the remedy was to extend its input.
