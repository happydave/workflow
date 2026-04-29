# Evidence Assessment

Rules for evaluating the reliability and decisiveness of evidence during investigation. This document defines the core principles and confidence framework used across all observability data.

## Core Principles

### Baseline Before Judgment
Before claiming a metric or pattern is abnormal, compare it to its own historical baseline. Absolute values without context are meaningless.
- Investigate the metric's state *before* the event.
- If baseline data is unavailable, state this as a limitation.

### Visual Verification
Summary statistics (min, max, avg) cannot reveal patterns or inflection points. 
- Interpretations remain hypotheses until confirmed visually via time-series charts.
- Flag findings as "unverified" if no visual reference is provided.

### Uncertainty Must Be Visible
Confidence labels reflect the strength of the **evidence**, not the narrative.

| Label | Meaning |
|---|---|
| **Confirmed** | Visually verified OR corroborated by multiple independent signals |
| **Supported** | Consistent with hypothesis but not independently confirmed |
| **Hypothesis** | Plausible explanation that has not been tested or visually verified |
| **Inconclusive** | Evidence examined but does not clearly support or refute the hypothesis |

### "Nothing Found" Is a Valid Output
Resist the temptation to produce a "root cause" without evidence. Documenting what was checked and found to be normal is valuable.

## Time Window Considerations
- **Baseline inclusion**: The window must start before the event.
- **Clock Skew**: Account for seconds/minutes of disagreement in distributed systems.
- **Periodicity**: Observation must span 2–3 full cycles of suspected periodic phenomena.

## Confirmation Bias
1. Ask: "What did this look like before?"
2. Actively seek contradictory evidence.
3. Check at least two independent signals before labeling a finding "Confirmed".
4. Separate data collection from interpretation.

## Specific Evidence Guidelines
For detailed rules on specific data types, refer to the following:
- [Logs Evidence](Evidence/Logs.md)
- [Metrics Evidence](Evidence/Metrics.md)
- [Groundcover & Traces](Evidence/Groundcover.md)

## Usage
Reference this hub document in all investigation plans.
