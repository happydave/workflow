# Metric Evidence Reliability

## Core Principles

**Metric absence is never definitive.** Metrics can be lost due to scrape failures, exporter crashes, network partitions, or retention expiry. The absence of a metric or a gap in a time series is worth noting but does not prove the underlying system stopped doing the thing the metric measures. Check exporter health (`up` metric, scrape targets) before drawing conclusions from metric absence.

**Stuck metrics may indicate reporting failure.** A metric that shows absolutely no change over a period where change is expected (e.g., a counter that should increment, a gauge that normally oscillates) may indicate the exporter or reporter has stalled rather than that the measured quantity is truly static. Investigate the reporter's health before trusting a flatlined metric.

**Non-zero values can be decisive when zero is expected.** Some metrics are expected to be zero under normal operation (rejected connections, assertion counts, error counters for specific categories). A non-zero value in these metrics is immediate, high-confidence evidence that the condition occurred. These are among the most reliable signals available.

## Usage
Reference this guideline during investigations involving time-series data, counters, gauges, or histograms.
