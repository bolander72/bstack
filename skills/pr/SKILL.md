---
name: pr
description: End-to-end GitHub pull request workflow for creating, publishing, reviewing, monitoring, fixing, and finishing PRs. Use whenever Codex is asked to create or open a PR, choose or change its base branch, publish local work, keep a PR in draft or mark it ready, request Codex review, watch checks and review feedback, address PR comments or failed CI, resolve merge conflicts, or babysit a PR until it is review-clean or merge-ready. Apply draft-aware CI and deployment rules, and continuously detect and surface migrations, backfills, environment-specific data changes, and other manual rollout operations that could otherwise be lost during repeated review loops.
---

# PR

Handle the full pull-request lifecycle without losing the distinction between a
cheap draft review loop and a fully validated ready PR.

## Establish the PR contract

Before changing Git or GitHub state:

1. Read repository instructions and inspect the worktree, current branch,
   remotes, existing PRs, and relevant CI/deployment configuration.
2. Identify the exact user-owned change. Preserve unrelated local changes and
   use a separate worktree when that is the safest way to isolate the PR.
3. Honor the user's requested base branch exactly. Never silently substitute
   `main` for `staging` or another explicit base.
4. Derive the base from repository conventions or the current branch only when
   the user did not specify it. State any consequential assumption.
5. Use the environment's required branch naming convention; in Codex Desktop,
   default to the `codex/` prefix.
6. Determine whether the requested outcome is:
   - **Draft review-clean:** suitable for continued Codex/fix iteration, with
     only checks intended for drafts complete.
   - **Ready/merge-clean:** full configured CI, deployments, and review are
     complete.
7. Create a rollout ledger whose state is **none**, **suspected**,
   **required**, or **completed**. Audit database, schema, configuration,
   external-resource, and historical-data impact before concluding that no
   manual operation is needed.

Default a newly created PR to draft unless the user explicitly asks for a
ready PR or repository policy clearly requires otherwise.

## Implement and publish deliberately

1. Inspect the complete diff and run validation proportional to the change.
2. Stage only relevant paths. Never sweep unrelated dirty files into a commit.
3. Commit with an intentional message, push the exact head branch, and create
   or update the PR against the confirmed base.
4. Write a PR body that records:
   - what changed and why;
   - user-visible or operational impact;
   - important design or root-cause details;
   - validation performed;
   - intentionally deferred full checks or deployments;
   - a dedicated `## Rollout / Manual Operations` section with the rollout
     ledger state and any required secrets, configuration, migrations,
     backfills, or manual follow-up.
5. Re-read the remote PR after publishing to verify base, head, draft state,
   description, head SHA, and checks belong to the expected change.

Do not mark a PR ready merely because the implementation is pushed. Readiness
starts the expensive/full-confidence lane and should match user intent.

Never let an absent rollout section imply that no manual work exists. Write an
explicit audited-none entry when none is required.

## Track backfills and manual rollout work

Treat deployment correctness as part of the PR, not as knowledge a human must
reconstruct from the diff. Perform the rollout audit on the initial change and
again after every review-driven commit. A small review fix can create a new
data invariant even when the original PR did not require a backfill.

Look for at least:

- new columns, constraints, indexes, defaults, enums, or state transitions;
- code that reinterprets existing rows or now requires metadata older rows do
  not have;
- identity, ownership, authorization, registry, contract, or collection
  mappings;
- renamed or removed values, tables, queues, jobs, webhooks, caches, or search
  indexes;
- environment-specific addresses, IDs, secrets, feature flags, provider
  resources, and deployment configuration; and
- one-time reconciliation, replay, repair, seeding, or cleanup work.

For each affected environment, compare the new invariant with existing data
and configuration rather than testing only a fresh database. Use authorized
read-only inspection when available. If live inspection is unavailable, mark
the ledger **suspected**, identify the exact unknowns, and do not infer safety
from staging, fixtures, or local seed data.

When manual work is needed, record:

- affected environments and exact records/resources;
- why the code change requires the operation;
- whether it runs before merge, before deploy, after deploy, or in a bounded
  compatibility window;
- exact run order, command or script, expected counts/effects, and owner;
- dry-run or read-only preflight, idempotency and rerun behavior;
- verification queries or observable success criteria; and
- rollback, recovery, or a clear statement that the operation is irreversible.

Prefer a reviewed, idempotent, dry-run-first script when the operation is
repeatable or touches more than a trivial number of records. A script does not
have to ship in the application PR when it is safer to run separately, but the
PR must link or contain the exact runbook and retain an auditable record of
what will be run in each environment. Never guess environment-specific IDs or
silently bake production data into a migration.

Keep the disclosure durable:

- Always maintain the `## Rollout / Manual Operations` section in the PR
  body, including an explicit audited-none result.
- When the state becomes **suspected** or **required**, immediately post one
  non-duplicate top-level PR comment beginning
  `⚠️ MANUAL ROLLOUT REQUIRED`. Update or supersede it when the plan changes.
  Pin it when the review surface supports pinning.
- Repeat the warning in the final chat handoff even after Codex gives a
  thumbs-up. Do not rely on intermediate commentary that may be collapsed.

Authorization to implement a PR does not automatically authorize production
data writes, secret changes, or external-resource mutations. Ask before those
actions unless the user explicitly included them in scope. When authorized,
resolve exact targets with read-only checks, execute in the documented order,
record results per environment, and update the ledger to **completed**.

## Apply draft-aware review and CI behavior

Treat Codex review, GitHub Actions, Vercel, Supabase, and other external checks
as separate integrations. Disabling a deployment or heavy job must not be
treated as disabling Codex review.

### Draft PR

- Use draft as the inexpensive implementation and review lane.
- Expect only repository-designated lightweight checks such as lint or
  typecheck. Heavy unit/component suites, integration tests, contract tests,
  production builds, database previews, and Vercel previews may intentionally
  skip.
- Codex automatic review does not run for a draft under this workflow. After
  opening the draft, post exactly one non-duplicate top-level comment whose
  content is `@codex review`.
- After pushing a fix that needs another review, inspect current comments and
  review state, then post exactly one new `@codex review` only when an
  equivalent request is not already outstanding for the current head.
- Do not wait for checks that repository policy intentionally suppresses in
  draft mode.

### Ready-for-review PR

- Treat opening a non-draft PR or changing a draft to ready as the start of the
  full validation/deployment lane.
- Rely on automatic Codex review when the PR is opened ready, marked ready, or
  receives new commits. Do not add a duplicate `@codex review` comment.
- Expect `ready_for_review` to start full configured checks and previews.
- Expect each later branch push while ready to emit a `synchronize` event and
  retrigger full configured checks and previews for the new head SHA.
- If the PR is converted back to draft, expect later pushes to return to the
  cheap lane. A correctly configured concurrency policy may cancel an active
  ready-only deployment.

If repository or organization settings differ, inspect the actual event,
review, branch-protection, and integration configuration. Do not guess or spam
manual review requests when an automatic review may already be queued.

### Recognize intentional skips

Classify a skipped or cancelled check as policy-neutral only after confirming
all of the following:

- the result belongs to the current head SHA;
- the PR state is draft, or another documented skip condition applies;
- the workflow job condition, ignored-build rule, or integration configuration
  explicitly accounts for that result; and
- the result is not a required check that leaves the PR unmergeable.

Examples include heavy jobs skipped by a `pull_request.draft == false`
condition or a native Vercel check reported as cancelled by the repository's
Ignored Build Step while a GitHub workflow owns ready-only deployments.

Never generalize from one repository. Inspect `.github/workflows`, provider
configuration, branch protection, and reported status details. In a
draft-aware setup, verify that:

- `opened` supports PRs created ready;
- `ready_for_review` starts the first full run;
- `synchronize` reruns it on every push while ready;
- `reopened` restores the expected run;
- `converted_to_draft` skips/cancels ready-only work when configured;
- duplicate `push` and `pull_request` matrices are not charging twice for the
  same PR commit;
- native provider auto-deployments are gated separately from GitHub Actions;
- direct deployment branches such as `main` or `staging` remain enabled when
  repository policy requires them; and
- workflow prerequisites such as a scoped `VERCEL_TOKEN` are documented
  before the PR is marked ready or merged.

External integrations may not honor GitHub job conditions. A Supabase Preview
or native Vercel check must be inspected and configured at its own integration
surface. Do not claim savings or suppression based only on a workflow edit.

## Start or reuse the PR watcher

After creating a PR or when asked to watch one, use the product's recurring
automation mechanism when available:

1. Record the repository, PR number and URL, base, exact head branch, current
   head SHA, draft state, and absolute worktree path.
2. Search for a watcher for that PR and update or reuse it. Never create a
   duplicate.
3. Use a 30-second thread heartbeat named `pr-<number>-review-watcher`.
4. Make its prompt self-contained; do not depend on collapsed chat context.
5. Include phase-aware completion rules, exact branch/worktree boundaries,
   permission to implement only unambiguous in-scope fixes, and the current
   rollout ledger.

Use this prompt shape, replacing every placeholder:

```text
Watch <PR URL> against <base> on branch <head> at <absolute worktree>. On the
current head SHA, inspect unresolved inline threads and replies, conversation
comments, review submissions, Codex reactions, every GitHub check/status,
external deployment or preview status, required checks, and mergeability.
Fetch every page of top-level issue/conversation comments on every scan and
parse any Codex `Reviewed commit` marker; review submissions, threads, and
reactions are not substitutes for that surface. Do not infer that Codex review
is pending merely because reviews or reactions are empty when a current-head
success comment exists. Classify Codex's result semantically rather than by one
exact sentence: any unambiguous positive completion message for the current
head counts as the success signal.
Recognize intentional draft-policy skips only after verifying the configured
condition; do not require ready-only checks for draft review-clean status.
Maintain a current Rollout / Manual Operations ledger in the PR body. On the
initial diff and after every code, SQL, configuration, or review-fix change,
re-audit whether existing data or any environment needs a migration, backfill,
reconciliation, script, secret/config update, or other manual operation. If
the ledger becomes suspected or required, update the PR body and post one
non-duplicate top-level warning beginning `⚠️ MANUAL ROLLOUT REQUIRED` with
the affected environments, timing, exact runbook, verification, and recovery
plan. Never infer that no backfill is needed from fresh tests alone. For each
new unambiguous in-scope comment, conflict, or failure, inspect the full
context, implement the smallest correct fix, validate, stage only related
files, commit, push, and reply at the original review surface with the commit
SHA and validation. Avoid duplicate work and replies. Never resolve a review
thread unless explicitly requested. After a push, post one non-duplicate
`@codex review` only if the PR is draft; if ready, rely on automatic re-review.
Stop and delete this watcher when the PR reaches its requested phase: for
draft review-clean, require a Codex success signal, no newer actionable
feedback, no conflict, every draft-required check successful, and a current
rollout ledger; for ready/merge-clean, additionally require all required/full
checks and previews successful or explicitly policy-neutral. If review is
clean while manual operations remain, report `review-clean, rollout pending`
and emit the full warning in the final notification before stopping; never
report an unqualified done. Also stop if merged or closed. Ask the user about
ambiguous feedback, product/security tradeoffs, out-of-scope failures, or
authority-expanding actions. Otherwise report a concise no-op.
```

## Inspect every review and status surface

Use thread-aware GitHub data. A flat comment list is insufficient.

- Read unresolved inline review threads with their complete reply chains,
  resolution/outdated state, file, and line.
- Read top-level conversation comments and the complete bodies of every review
  submission, including `COMMENTED` reviews. Codex can place an actionable
  finding only in its review summary/body (with a source URL or line link)
  rather than creating a separate inline review-thread comment; treat that as
  actionable feedback on the reviewed commit and inspect its cited code.
- On every watcher scan, fetch every page of top-level PR conversation comments
  through the issue-comments API or an equivalent paginated endpoint. Do not
  replace this with only `gh pr view`, review submissions, reactions, timeline
  summaries, or inline-thread queries. A Codex result can exist solely as a
  top-level issue comment.
- Recognize any unambiguous positive current-head result from the connector,
  even when it creates no review submission or reaction. Examples include:

  ```text
  Codex Review: Didn't find any major issues. :+1:
  Looks good to me.
  No issues found.
  No actionable feedback.
  LGTM.
  **Reviewed commit:** `<short SHA>`
  ```

  Interpret meaning instead of maintaining a fixed phrase whitelist or
  requiring a particular sentence, emoji, capitalization, or template. Count
  a message when its overall meaning is that review completed successfully and
  Codex found no remaining issue requiring action. Do not count progress,
  acknowledgement, thanks, a promise to review, mixed/qualified approval with
  a remaining concern, or a message that merely says a fix was received.
  Require the author to be the recognized Codex bot/connector and bind the
  result to the current head. Prefer a review `commit_id` or parse a non-empty
  `Reviewed commit` marker and require it to prefix-match the full head SHA. If
  neither exists, accept the message only when its GitHub context
  unambiguously ties it to the current-head review request after the latest
  push and no later push occurred. Record the comment/review ID, creation time,
  and head-binding evidence as durable scan state. Empty reactions or reviews
  must not override this success signal.
- On every watcher scan, enumerate all newly submitted Codex reviews since the
  prior scan and compare their bodies, not only their state, author, or
  presence of inline-thread nodes. Keep a durable review ID/submitted-at/head
  record so an older review is not handled twice while a new review body cannot
  be missed.
- Inspect reactions on the PR body and relevant Codex request/comment. If a
  connector omits reactions, use an authenticated GitHub API fallback.
- Inspect check runs and commit status contexts on the current head SHA,
  including GitHub Actions and external CI, security, database, and deployment
  integrations.
- Inspect required checks and mergeability against the actual base branch.
- Inspect the PR body's rollout ledger and re-audit it against the current
  diff. Check historical/live data and environment configuration with
  authorized read-only access when relevant; a passing fresh-schema test does
  not prove existing rows are compatible.
- Read failed GitHub Actions step logs. Follow an external check's details URL
  when possible and say when its logs are unavailable.
- Re-read remote state immediately before making changes or posting replies to
  avoid races and duplicates.

Recognize Codex by its GitHub bot/connector identity, not merely by wording
that sounds like a review.

## Address feedback, failures, and conflicts

For actionable feedback:

1. Read the whole thread and current code.
2. Determine whether a later reply or commit already addresses it.
3. Implement the smallest correct in-scope fix. Ask instead of guessing when
   feedback is ambiguous, conflicting, or requires a product/security choice.
4. Run proportional validation, stage only related paths, commit, and push.
5. Re-read the remote thread, then reply exactly once at the correct surface:
   inline for inline feedback, top-level for top-level feedback.
6. Include the commit SHA, concise fix summary, and validation performed.
7. Re-run the rollout audit against the resulting diff. Update the PR body's
   ledger and durable warning before requesting re-review.
8. Do not resolve threads unless explicitly requested.

For failed checks, verify the check belongs to the current head and distinguish
a code regression from flaky, provider, quota, secret, or policy behavior.
Change only failures caused by the PR and within the user's authorized scope.

For merge conflicts, fetch the current base, resolve every hunk in context,
preserve both sides' intended behavior, validate the result, push, and recheck
the new head. Ask when resolution requires a product or security tradeoff.

After every pushed fix, apply the draft-versus-ready Codex review rule again.

## Classify completion

A recognized Codex success signal is one of:

- a thumbs-up reaction from Codex on the PR body or relevant review comment;
- an `APPROVED` Codex review; or
- any unambiguous positive Codex completion message whose overall meaning is
  that the reviewed change is good to go with no remaining actionable issue.
  Examples such as “looks good,” “no issues,” “no actionable feedback,”
  “didn't find any major issues,” and “LGTM” are illustrative, not an
  exhaustive or exact-match list.

Require that signal to apply to the current head with no newer actionable
feedback. CI success alone, another user's reaction, an outdated comment, or
the author's own fix reply is not Codex approval.

For a top-level Codex result without a GitHub review `commit_id`, first parse
its `Reviewed commit` marker and prefix-match it against the current full head
SHA. If the marker is absent, require unambiguous current-head linkage from the
GitHub event/request chronology: the positive result must answer the latest
review request after the latest head push, and no later push may exist. Treat a
matching or unambiguously linked positive result as conclusive even if the
review-submissions and reactions endpoints are empty. Treat malformed,
non-matching, stale, or ambiguously linked results as non-current rather than
guessing.

Report **draft review-clean** only when:

- the Codex success signal exists for the current head;
- no newer actionable feedback or merge conflict remains;
- every check intended to run in draft has succeeded; and
- every skipped/cancelled check has been verified as intentionally neutral;
  and
- the rollout ledger is present and current for the final diff.

Report **ready/merge-clean** only when all draft criteria hold and every
required/full check, status, preview, and deployment for the current head is
successful or explicitly policy-neutral. Pending, queued, in-progress,
failing, timed-out, action-required, stale, or missing-required checks block
this state. Any operation required before merge must also be completed.
Deploy-time or post-merge operations may remain only when their exact runbook,
ordering, environment, owner, verification, and recovery plan are documented.

Never report an unqualified **done**, **review-clean**, or **merge-clean** when
the rollout ledger is missing, stale, suspected, or required. Use
**review-clean, rollout pending** when code review and checks are clean but a
documented manual operation remains.

When the rollout state is **suspected** or **required**, end the final chat
handoff and watcher completion notification with this prominent block:

```text
⚠️ MANUAL ROLLOUT REQUIRED
- Status:
- Environments:
- Why:
- Timing and run order:
- Command/script or runbook:
- Expected records/effects:
- Verification:
- Rollback/recovery:
```

When the state is **none**, include a concise
`Rollout/manual-data audit: none required` statement and name the surfaces
audited. When it is **completed**, report the per-environment result and
verification evidence. A Codex thumbs-up approves the reviewed code; it does
not prove that manual rollout work has been executed.

Delete the watcher when its requested completion state is reached or the PR is
merged or closed. Before deleting it, persist the final ledger in the PR body
and emit any pending rollout warning. Do not merely pause it.

## Route model effort sensibly

Use a balanced coding model for normal implementation, CI fixes, and repeated
review-comment loops. Escalate to the strongest coding model for architecture,
security-sensitive changes, subtle cross-package behavior, or after repeated
attempts fail to converge. Reserve lightweight models for mechanical edits,
formatting, straightforward tests, and triage.

Treat hosted Codex PR review as a separate reviewer from the interactive model
used to implement fixes. Exhaustive review changes review depth, not draft
trigger behavior; use it for final hardening when its extra time and usage are
justified. If an experimental smart trigger is enabled, verify whether a new
review actually started before requesting one manually.
