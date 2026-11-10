---
name: plainflow
description: How to write code. Avoid avoidable complexity, hide unavoidable complexity behind a simple interface, and make behaviour explicit and traceable through structured logs that work in any environment. Use when writing or changing implementation code, adding a module, class or function, designing an interface, or deciding what to log. For judging someone else's pull request, use diffluent-review instead.
---

# Plainflow

Three rules:

1. **Avoidable complexity is avoided.**
2. **Unavoidable complexity is hidden** behind a simple interface.
3. **Behaviour is explicit** and traceable through logs, in any environment.

Where they conflict, "When the rules collide" at the end resolves it by situation, usually with a third option rather than one rule beating another.

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

**Simple does not mean small.** Splitting a coherent unit into several thin ones buys an interface for every split while hiding nothing, and each interface costs the reader more than it saves. Prefer one unit that does a whole job to three that each do a third of it.

## 2. Hide what is left

When complexity is essential, decide who carries it: this module, or every caller forever. Carry it here.

- The functionality is the benefit; the interface is the cost. Keep the interface narrower than the implementation, and judge a module by that ratio rather than by its length.
- Each module keeps one secret: a format, a protocol, a schema, an algorithm, a vendor quirk. If a caller must know the secret to use the module, it has leaked.
- A caller should not have to call three things in a fixed order, pass flags that only mean something internally, or handle errors that only mean something internally — translate, never swallow.
- Place the seam where change is likely, not at the steps of the process.
- State the contract at the boundary: what it requires, what it guarantees, what it does on failure. Write the why, not the what.

## 3. Make behaviour explicit and traceable

Explicit:

- Return outcomes. Never route control flow through shared mutable state.
- No action at a distance. Someone reading the call site should be able to predict what happens.
- Name the decision, not the mechanism: `requiresManagerApproval`, not `flag2`.

Traceable: **log the rules, not the plumbing.** If you wrote this unit out as one-line business rules, each rule belongs in the logs and none of the plumbing does.

Log two things, and nothing else — a log line is code, and logging everything is the avoidable complexity rule 1 exists to prevent:

- **One event per unit of work**, emitted at its boundary, carrying the outcome and the values that drove each decision as fields.
- **Every failure**, including every error returned by something you called, with enough context to act on. This is the highest-value category — in field studies, failures that logged an error were diagnosed far faster than those that logged nothing.

Never log loop iterations, getters, "entering function", anything already carried on that event, or secrets, credentials, tokens and personal data. Decide the last one as you write the line, not in review.

To work in any environment: structured key-value fields rather than interpolated prose; one stable event name with what varies as fields (`payment.failed` with `reason=...`, not a message per reason); the request or trace identifier, so one unit of work can be followed across services; and this repository's existing field names and levels, or OpenTelemetry semantic conventions for new ones.

A log that no longer matches the code is worse than no log. Change a rule, change its log line in the same edit.

## When the rules collide

Each line is a situation you will be in, and what to do. Not extra rules — these are the four places the three rules pull against each other.

**An error only means something inside this module** → translate it into an outcome the caller can act on. Never swallow it to keep the interface clean, and still log it.

**One unit of work makes five decisions worth recording** → one event, five fields. Not five log lines.

**An error crosses several layers** → log it once, where it is first detected. Do not log it again as it rises.

**Two secrets always change together** → they are one secret. Keep them in one module; a seam there has no change running through it.

**A unit of work fails** → the failure line carries the vendor or exception detail, at the layer that can still see it. The event carries the business outcome. No field on both.

**A field would help diagnosis but describes a person** → opaque internal identifiers go in. Request bodies, credentials, contact details and the names of people stay out.

**You could put an interface over the one implementation you have** → do not. Rule 1 decides whether the machinery exists at all; rule 2 only decides where its seams go. Declining to anticipate a feature is not the same as placing a boundary badly.

**Anything else** → prefer the option that costs you and saves the next reader. Never trade away a failure someone can diagnose.

## Before you finish

- Can anything be deleted without losing behaviour?
- Could someone diagnose a failure here from the logs alone — no debugger, no reproduction?
- Did this change leave any log line stale?

The ideas behind these rules: information hiding (Parnas, 1972), deep modules (Ousterhout, *A Philosophy of Software Design*), and empirical work on logging and production failure diagnosis (Yuan et al., 2012).
