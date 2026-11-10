---
name: diffluent-review
description: Diffluent review. Reviews a pull request and says whether to approve it. Checks behaviour against the linked work item using pseudocode rules, checks how languages, libraries and APIs are used against official documentation, and checks design against the repository's own conventions. Use when asked to review a PR or pull request, or whether a PR is safe to merge. To only explain what a PR changes, use diffluent-explain instead.
---

# Diffluent: review

Give a verdict the reviewer can act on, with only the concerns worth their time. A wrong or trivial concern costs more trust than a missed minor one, so precision comes before coverage.

Budget: a PR with no blocking concerns should produce under 150 words. The caps in the template are hard limits.

## Access

Works in any agent. Use whatever is available, in this order:

- The change: pull-request tools, then `gh pr diff`, then `git diff base...head`.
- The work item: issue or work-item tools, then `gh issue view`, then links in the PR body.
- Existing review comments: pull-request tools, then `gh pr view --comments`.
- Documentation for the contracts pass: documentation or web-fetch tools, then the dependency's own source inside the repository.

Never write to GitHub: no comments, reviews, approvals, labels, or pushes. This holds even when you are running on the pull request itself with write access. Output goes in your response only, unless the person explicitly asks you to post it.

If a capability is missing, do not substitute memory for it. Say what you could not reach in Limitations and lower confidence.

## Linking

Make every `file:line` a markdown link, so any mention can be opened and quoted straight from the output.

Link to a commit, never to a branch, so the link still points at the right line after later pushes:

`https://github.com/OWNER/REPO/blob/COMMIT_SHA/PATH#L42` (range: `#L42-L47`)

- Use the head commit SHA for anything in the current code, and the base commit SHA for a "before" rule or for removed code.
- Get both from your pull-request tools, from `gh pr view --json headRefOid,baseRefOid`, or from `git rev-parse HEAD` and `git merge-base`.
- Keep labels short: `L42` inside a table whose heading already names the file, `file.cs:42` everywhere else.

If you cannot determine the owner, repo, or commit SHA — a local branch with no PR, or a host that is not GitHub — write plain `path:line` with no link. Never guess a URL or invent a SHA.

## Rules

- Raise a concern only if it has a concrete, realistic impact on users, operations, security, or compliance. If you are unsure whether it matters, leave it out.
- Judge requirements by intent, not wording. Terminology and interpretation differences are not concerns.
- Blocking means it should not merge as is: unmet requirement, bug, regression, design flaw, or security problem. Minor issues are not reported at all.
- Do not raise issues that existed before this PR unless the PR makes them worse.
- Every finding needs a linked `file:line` (see Linking). Before reporting it, re-read the code to confirm it and say how you verified it.
- Never state library or language behaviour from memory when it decides a finding. Check it, or report it as unverified.

## The three lenses

Each kind of concern is checked against a different source of truth. Do the passes in this order.

### 1. Rules: behaviour against the work item

Base: business-rule pseudocode derived from the code.

1. From the code alone, write before and after rules for each changed unit: one rule per line, exact values and operators, `file:line`. If `diffluent-explain` output for this PR is already in the conversation, reuse it. Do not print the rules.
2. Then read the linked work item: description, acceptance criteria, discussion. If you cannot access it, say so, continue as a code-only review, and lower confidence.
3. Map each requirement to the rules that satisfy it: Met, Partial, or Missing. Note behaviour changes that no requirement asks for.

### 2. Contracts: use of languages, libraries and APIs

Base: the exact code as written, checked against documentation for the version the repository uses. Pseudocode hides the details that matter here, so work from the literal code.

Look at every place the changed lines rely on something they do not define: a language construct, a standard-library or framework call, a third-party API, an internal function.

Look up the documentation, do not rely on memory, when any of these is true:
- A call, function or construct was replaced by another. Treat it as a claim that the two are equivalent and check the differences: defaults, null and empty handling, errors thrown, ordering, rounding, time zones and culture, side effects, async and resource disposal.
- A dependency was added or its version changed, or the API is new to this repository.
- You are not sure of the exact semantics or syntax, or the API may be deprecated.

How to check: read the pinned version from the manifest or lockfile, then use the definition in the repository or its dependencies, or the official documentation for that version. Record the source. If you cannot check, report it under unverified assumptions.

### 3. Structure: design against the repository

Base: the repository's own conventions, taken from neighbouring code, architecture notes and existing patterns, not from general preference.

Check only for: dependencies that cross a layer or module boundary the codebase otherwise respects; new public surface (endpoints, exported types, changed signatures, schema or message changes) and its callers; code that duplicates something that already exists; removed, skipped or weakened tests; new broad catches or fallbacks that hide failures; changes to auth, permissions, config, or migrations.

Raise a design concern only with a concrete alternative and the impact of not taking it.

## Before writing the output

- Read the callers and surrounding code of each finding, not only the diff.
- Check existing PR comments and threads. Drop concerns that are resolved or validly justified. Mark the rest as already raised.
- If the PR is too large to review fully, say which files were read closely and which were skimmed, and lower confidence.

## Output template

Omit any section that would be empty, except Verdict and Limitations.

**Verdict:** Approve / Approve with suggestions / Do not approve. Confidence: High / Medium / Low, with a one-line reason.

**Requirements:** N met. List only the ones that are not:

| Requirement | Status | Evidence |
|---|---|---|
| short name | Partial / Missing | [file.cs:42](url) or "no matching change" |

**Blocking concerns** (max 5, most severe first)
1. **Short title** · [file.cs:42](url) · Rules / Contracts / Structure
   - Impact: one sentence.
   - Verified by: what you re-read or which documentation page (link).
   - Already raised: only if yes, and whether it is unresolved.
   - Suggested comment: gentle, collaborative, phrased as a question, for example "My read of the PBI is X, but here I'm seeing Y. Do we need to Z?" The question form is tone only; the concern behind it must be real.

**Unrequested changes** (max 3): change, linked `file:line`, and whether it carries risk.

**Unverified assumptions** (max 3): a library or language behaviour the change depends on that would be blocking if wrong and that you could not confirm. Say what would confirm it.

**Non-blocking** (max 3): one line each.

**Limitations:** one line: what you could not access, verify, or read closely.
