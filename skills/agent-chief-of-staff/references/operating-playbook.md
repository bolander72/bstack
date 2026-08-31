# Operating Playbook

Read this reference for multi-step programs, unfamiliar products, recurring agent operations, or any request to increase agent autonomy.

## Task charter

Capture the operating contract before work fans out:

```markdown
Outcome:
Acceptance criteria:
Constraints and non-goals:
Authority / approval boundary:
Critical path:
Workstreams and ownership:
Verification plan:
Integration owner:
Stop conditions:
```

Keep this proportional. A small task may need one sentence per field; a program spanning repositories or external systems needs enough detail to prevent divergent interpretations.

## Useful team shapes

Choose a shape because it reduces a concrete risk or latency source.

- **Scout → implementer → reviewer:** A scout maps the relevant surface, an implementer owns the change, and an independent reviewer challenges the finished result. Use for unfamiliar or risky code.
- **Parallel components:** Assign non-overlapping modules or artifacts to separate owners, then reserve one integration pass. Use when dependencies are already clear.
- **Competing hypotheses:** Give independent investigators the same symptoms but not each other's diagnosis. Let the lead select using a decisive reproduction or measurement. Use for ambiguous debugging and research.
- **Evidence lanes:** Separate source gathering by domain, then have the lead reconcile claims against a prewritten rubric. Use for research, audits, and decisions.
- **Triage → queue:** One worker normalizes incoming reports and maps each to a feature or owner; implementation begins only after reproduction and priority are established. Use for repeated feedback or bug intake.

Avoid a ceremonial hierarchy. A worker should not spawn another worker unless the added context isolation or parallelism is worth the integration cost and the platform permits nesting.

## Feature map

A feature map gives agents the business and navigation context that source search alone cannot provide. Store it with project documentation when it will remain useful. Describe each important feature with:

```markdown
Feature / user goal:
Entry path and prerequisites:
Primary actions and stable selectors or commands:
Expected visible result:
Owning modules, services, and data:
Fixtures, accounts, or environment assumptions:
Logs, traces, metrics, or diagnostics:
Known edge cases:
Last verified:
```

Build the first map from code, actual product navigation, and runtime observation. A list of guessed filenames is not a feature map. Keep it current through verified changes rather than trying to document every implementation detail.

## Trust ladder

Trust is specific to a workflow, repository, toolchain, and risk class. It does not transfer automatically because the same model or agent succeeded elsewhere.

1. **Observed:** The agent investigates or drafts while the lead checks its evidence closely. No consequential mutation.
2. **Isolated:** The agent implements in a branch, worktree, sandbox, draft, or other reviewable boundary and runs prescribed checks.
3. **Supervised:** The agent performs routine in-scope mutations after passing automated and independent review gates; a human or lead approves the consequential boundary.
4. **Delegated:** The agent completes a narrow recurring workflow under hard controls, audit logs, bounded permissions, and a tested rollback path.
5. **Autonomous:** The workflow may merge, deploy, publish, or otherwise finalize only when the user has explicitly authorized that policy and eval history shows an acceptably low failure rate for the blast radius.

Move one level at a time. Regress a workflow when its environment, architecture, model behavior, permissions, or failure profile changes materially.

## Verification design

Prefer evidence that exercises the same surface the user depends on.

For a recurring application workflow, control CLI, or product-specific verification skill, read [verification infrastructure](verification-infrastructure.md).

- Establish a baseline before changing anything when performance or behavior is at issue.
- Make the application observable and controllable by the agent: reproducible startup, seeded data, stable navigation, inspectable logs, and scripted diagnostics.
- Verify the reported path, not merely compilation or a nearby unit test.
- Require proof artifacts in a compact form: command plus result, screenshot, trace, query output, or cited source.
- Keep authoring and high-stakes judging separate. A reviewer should receive the acceptance criteria and artifacts, not the author's desired verdict.
- Treat flaky, non-reproducible, or environment-dependent results as uncertainty, not success.

## Promote recurring feedback into guardrails

When review finds a recurring failure, ask what would make the wrong path unavailable or immediately visible:

1. Can architecture or least-privilege permissions eliminate it?
2. Can types, the compiler, schema validation, dependency boundaries, or static analysis reject it?
3. Can a focused automated test or CI check catch it?
4. Does a project rule or skill add necessary context that cannot be enforced mechanically?
5. Is human review still the only realistic gate?

Use the strongest layer justified by the evidence. Do not encode incidental history in comments or instructions that future agents will misread as universal policy.

## Evaluate and maintain the system

Treat reusable agent workflows like tested software:

1. Write a rubric before running candidates. Score observable outcomes, evidence quality, policy compliance, and unnecessary work.
2. Select representative successes, known failure modes, and a few adversarial or ambiguous cases.
3. Run candidates in isolated directories or branches with the same permissions and context they will receive in practice.
4. Avoid revealing judge-only scoring details when doing so would distort normal behavior.
5. Use an independent judge or deterministic checks where possible. The lead adjudicates disagreement and inspects failures, not just aggregate scores.
6. Compare the changed skill or workflow against the current baseline. Keep changes only when the observed tradeoff is worthwhile.
7. Record the failure pattern and the narrow correction. Avoid accumulating universal rules from isolated examples.

Do not loop blindly toward a perfect numerical score; detect overfitting, cost growth, latency, and new regressions.

## Cost and capacity

- Use the fewest workers that shorten the critical path or improve confidence.
- Route bounded search, formatting, or mechanical checks to economical workers when the platform supports model selection.
- Keep ambiguous planning, integration, policy decisions, and final risk adjudication with a sufficiently capable lead.
- Stop low-value branches early and reuse completed worker context for follow-ups.
- Judge return on verified outcomes and reduced human bottlenecks, not agent count, token volume, commit count, or PR count.
