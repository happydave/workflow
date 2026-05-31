# Log Evidence Reliability

## Core Principles

**A single decisive log can be sufficient evidence.** If a log message explicitly records the event under investigation (e.g., an error message with a stack trace, a state transition log, a rejection reason), that log entry alone may answer the question. Not everything requires corroboration — sometimes one log says exactly what happened.

**Log absence is never definitive.** Logs get lost. They are dropped by buffering, sampling, rate limiting, and ingestion failures. The absence of an expected log entry is worth noting but never proves the event didn't happen. Frame it as: "No log found for X in the examined window" — not "X did not occur."

## Volume Analysis

**Reduced log volume requires comparative analysis.** If an investigation observes fewer logs than expected, this could mean the events genuinely decreased OR that logs are being lost. To distinguish:
- Compare against a known-good baseline period.
- Check for evidence of log dropping (ingestion errors, pipeline metrics, gaps in otherwise continuous streams).
- If log dropping cannot be ruled out, state the ambiguity explicitly.

## Usage
Reference this guideline during investigations involving application logs, system logs, or audit trails.
