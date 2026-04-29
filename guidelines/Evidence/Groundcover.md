# Groundcover & Trace Evidence

## Trace Reliability

**Traces are sampled.** Most distributed tracing systems sample rather than capture every request. The absence of error traces does not mean errors didn't occur. The presence of error traces is reliable evidence that errors did occur. Quantitative conclusions from trace counts should account for the sampling rate.

## Groundcover Specifics

*Note: Groundcover-specific evidence assessment rules (e.g., eBPF-based capture reliability, Issues vs. Traces) are under development.*

Groundcover leverages eBPF to capture data with minimal overhead. While more exhaustive than some library-based tracing, it is still subject to the principles of sampling and system-level visibility constraints.

## Usage
Reference this guideline during investigations involving distributed traces or Groundcover-specific telemetry.
