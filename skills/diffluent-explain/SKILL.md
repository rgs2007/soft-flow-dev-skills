---
name: diffluent-explain
description: Diffluent explain. Explains what a pull request changes without judging it. Shows the behaviour change as a before/after comparison in business-rule pseudocode, plus a short neutral list of technical changes. Use when asked to explain, summarise or walk through a PR, or to show a PR as pseudocode, usually before reviewing it. For a verdict or concerns, use diffluent-review instead.
---

# Diffluent: explain

Help a human reviewer understand a pull request fast enough to form their own opinion. Describe, never judge: no verdict, no concerns, no advice. That is the job of `diffluent-review`, and mixing the two anchors the reader on the AI's opinion.

Budget: the default output must be readable in about two minutes. The caps in the template are hard limits. Anything beyond them is offered in the last line, not printed.

## Access

Works in any agent. Use whatever is available, in this order:

- The change: pull-request tools, then `gh pr diff`, then `git diff base...head`.
- The PR description: pull-request tools, then `gh pr view`, then ask for it.

Never write to GitHub: no comments, reviews, approvals, labels, or pushes. This holds even when you are running on the pull request itself with write access. Output goes in your response only.

If something is unreachable, never substitute memory for it. Name it in the last line.

## Rules

- Code first, description last. Write everything from the code before reading the PR description or work item. Reading them first makes you describe what was intended instead of what the code does.
- Exact values and operators. `>` and `>=` are different rules. Never round or paraphrase thresholds, limits, status values, or error codes.
- Every line carries `file:line`.
- Never fill a gap with an assumption.

## Steps

1. Identify the PR from a URL, `owner/repo#number`, or branch plus base. If it is unclear, ask.
2. Get the changed files and the diff.
3. For each changed function, method, endpoint, or query, read the complete before and after versions. Hunks alone are not enough to state a rule correctly.
4. Sort the files. Convert source code with logic. Give tests, config, migrations and dependency changes one line each. Skip generated files, lockfiles, formatting-only changes and pure renames.
5. Write before and after pseudocode per unit (style below), align them rule by rule, and mark each rule Changed, Added or Removed.
6. List the technical changes neutrally (see template).
7. Only now read the PR title and description, and compare them with what you wrote.
8. Fill the template and cut to the caps, keeping the most significant items.

## Pseudocode style

One rule per line, plain language, present tense.

Keep: conditions, thresholds, calculations, validations, state changes, external effects (database writes, messages, emails, API calls), error outcomes, defaults, ordering when it affects the result.

Drop: logging, null and type plumbing, dependency wiring, mapping boilerplate, framework ceremony.

Do not silently drop anything that can change behaviour (a new broad catch, a changed default, a retry, a timeout). State it as a rule.

Example: `If order total >= 500, require manager approval [OrderService.cs:42]`

## Output template

Omit any section that would be empty, except the first and the last line.

**What this PR does:** two or three sentences, plain language.

**Behaviour changes** (max 7, most significant first)
1. Before: X. After: Y. `file:line`

If behaviour does not change, say so in one line and skip the next section.

**Comparison** (max 5 units, only units whose behaviour changes)

`path/to/file` · `unitName` (New / Changed / Deleted)

| | Before | After |
|---|---|---|
| Changed | rule [line] | rule [line] |
| Added | | rule [line] |
| Removed | rule [line] | |

Unchanged rules are not listed. Add one only when a neighbouring change cannot be understood without it.

**Technical changes** (max 5, one line each, facts only)
- Replaced calls or language constructs: `old` → `new` `file:line`
- New, removed or upgraded dependencies, with versions
- Structural changes: code moved between modules, new public endpoints or types, changed signatures, schema or contract changes
- Tests: added, removed, skipped, or asserting less than before

**Description vs code** (only if they differ)
- Claimed but not found in the code:
- Changed but not mentioned:

**Read these closely** (max 5): `file:line` and a few words on why. Choose the places where pseudocode is least able to stand in for the code: boundary conditions, error handling, concurrency, money or date arithmetic, security checks.

**Not covered:** one line naming what was skipped, not fetched, or only skimmed, and what you can expand on request (for example "ask for the full comparison of the 4 remaining units").

## Large PRs

Do not thin out every unit to fit everything in. Cover the behaviour-changing units properly up to the caps, name the rest in the last line, and offer to continue.
