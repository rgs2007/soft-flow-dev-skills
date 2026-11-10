---
name: plainflow
description: How to write code. Avoid avoidable complexity, hide unavoidable complexity behind a simple interface, and make behaviour explicit and traceable through structured logs that work in any environment. Use when writing or changing implementation code, adding a module, class or function, designing an interface, or deciding what to log. For judging someone else's pull request, use diffluent-review instead.
---

# Plainflow

Three rules, in priority order:

1. **Avoidable complexity is avoided.**
2. **Unavoidable complexity is hidden** behind a simple interface.
3. **Behaviour is explicit** and traceable through logs, in any environment.

The order matters because rules 1 and 3 pull against each other. A log line is code: logging everything is avoidable complexity. Rule 3 says how to resolve that.

## 1. Avoid avoidable complexity

Cut these before writing them:

- An abstraction for a need that does not exist yet. One caller means one function, not an interface.
- A parameter, flag or option that no caller sets.
- Indirection with a single implementation behind it.
- A reimplementation of something the language, its standard library, or this repository already provides. Look before writing.
- Caching, pooling or batching added before a measurement showed it was needed.

Never cut these. They are not complexity, they are the work:

- Error handling and failure paths.
- Boundary and empty cases.
- Validation at a trust boundary.
- Concurrency correctness.

**This is not "make everything small."** Splitting a coherent unit into several thin ones buys an interface for every split while hiding nothing, and each interface costs the reader more than it saves. Prefer one unit that does a whole job to three that each do a third of it.

## 2. Hide what is left

When complexity is essential, decide who carries it: this module, or every caller forever. Carry it here.

- The functionality is the benefit; the interface is the cost. Keep the interface narrower than the implementation, and judge a module by that ratio rather than by its length.
- Each module keeps one secret: a format, a protocol, a schema, an algorithm, a vendor quirk. If a caller must know the secret to use the module, it has leaked.
- A caller should not have to call three things in a fixed order, pass flags that only mean something internally, or handle errors that only mean something internally.
- Choose the boundary by what is likely to change, not by the steps of the process.
- State the contract at the boundary: what it requires, what it guarantees, what it does on failure. Write the why, not the what.

## 3. Make behaviour explicit and traceable

Explicit:

- Return outcomes. Do not signal through shared mutable state.
- No action at a distance. Someone reading the call site should be able to predict what happens.
- Name the decision, not the mechanism: `requiresManagerApproval`, not `flag2`.

Traceable: **log the rules, not the plumbing.** The test — if you wrote this unit out as one-line business rules, each rule is a candidate for a log line and none of the plumbing is.

Log:

- One event per unit of work, at its boundary, carrying the outcome.
- Each decision that changed the outcome, with the values that drove it.
- Every failure, and every error returned by something you called, with enough context to act on. This is the highest-value category: in field studies, failures that logged an error were diagnosed far faster than those that logged nothing.

Do not log:

- Loop iterations, getters, successful routine steps, "entering function".
- Anything already carried on the event you are about to emit.
- Secrets, credentials, tokens or personal data. Decide this as you write the line, not in review.

Make it work in any environment:

- Structured key-value fields, not interpolated prose, so the same log is queryable in development, CI and production alike.
- One stable event name per event, with what varies as fields: `payment.failed` with `reason=...`, not a different message per reason.
- Carry the request or trace identifier so one unit of work can be followed across services.
- Reuse the field names this repository already uses. For new ones, follow OpenTelemetry semantic conventions rather than inventing a vocabulary.
- Make levels mean something: ERROR is someone must act, WARN is degraded but handled, INFO is a decision or outcome worth keeping, DEBUG is for development only.

A log that no longer matches the code is worse than no log. Change a rule, change its log line in the same edit.

## Before you finish

- Can anything here be deleted without losing behaviour? Delete it.
- Does any caller have to know something it should not? Move it inside.
- Could someone diagnose a failure here from the logs alone, without a debugger and without reproducing it?
- Does every log line have a stable name, structured fields, and no secrets?
- Did this change leave any log line stale?

The ideas behind these rules: information hiding (Parnas, 1972), deep modules (Ousterhout, *A Philosophy of Software Design*), and empirical work on logging and production failure diagnosis (Yuan et al., 2012).
