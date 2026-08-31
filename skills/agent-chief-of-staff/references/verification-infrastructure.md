# Verification Infrastructure

Read this reference when agents repeatedly build, debug, measure, or triage an application, service, or CLI. The objective is to remove the human as the only person who can exercise the product and judge whether a change works.

## Build the reusable lever

Prefer a small project-local control CLI or similarly stable interface over prose-only instructions and one-off scripts. It should let an agent start the target, drive realistic paths, inspect state, capture evidence, and restore a clean environment.

Build this layer when interaction recurs, manual verification is the throughput bottleneck, or multiple workers need the same control surface. For a one-off task, use existing test and debugging tools directly rather than creating an oversized framework.

Start by identifying what the stack already exposes:

- browser or desktop debugging protocols, accessibility trees, and end-to-end test drivers;
- platform simulators, device tooling, debuggers, profilers, and system logs;
- service APIs, health endpoints, traces, metrics, database inspection, and test fixtures;
- CLI commands, structured output, exit codes, and reproducible input files.

Use the native surface with the best fidelity to the user's experience. Add custom tooling only where existing tools leave a repeated gap.

## Define the environment contract

The control surface is only reliable when its environment is reproducible. Document or automate:

- installation, build, startup, readiness, and shutdown;
- seeded data, test users, authentication, entitlements, and feature flags;
- test or staging endpoints and how external APIs are stubbed or safely exercised;
- isolated ports, databases, caches, accounts, and artifact directories for concurrent workers;
- cleanup and recovery after a failed or interrupted run.

Default to a non-production environment and refuse ambiguous destructive operations. Never print secrets into logs or proof artifacts.

## Design an agent-friendly control surface

Expose only commands justified by real workflows. Useful command families include:

- **Health:** diagnose prerequisites, readiness, versions, and environment mismatches.
- **Inspection:** return accessibility or component snapshots, state summaries, screenshots, logs, and current configuration.
- **Navigation:** reach stable user-visible destinations and reset to known starting points.
- **Interaction:** click, type, press keys, upload fixtures, invoke commands, and toggle explicitly safe test flags.
- **Performance:** record traces or profiles, collect comparable metrics, and wait for a deterministic settled state.
- **Streaming and network:** capture console output, requests, responses, errors, and concise summaries.
- **Cleanup:** stop processes, release resources, clear isolated state, and recover from partial runs.

Prefer these interface properties:

- composable subcommands with focused responsibilities;
- idempotent behavior where feasible and `--dry-run` for consequential actions;
- useful exit codes plus machine-readable output such as JSON;
- concise human output on standard output and diagnostics on standard error;
- rich help, examples, and error messages that state the likely cause and next corrective action;
- explicit timeouts and a deterministic readiness or settle check instead of arbitrary sleeps;
- evidence artifacts written to reported paths with commit, environment, and timestamp metadata.

Test the control surface itself. A stale or flaky verifier cannot be used to establish confidence in application code.

## Materialize product memory with a feature map

Use a hierarchical map rather than one giant document:

1. An overview lists major user goals and links to focused feature files.
2. Each feature file describes user-visible behavior and how to drive and verify it.

Record for each feature:

```markdown
Feature / user goal:
Entry path and prerequisites:
Entitlement, account, or data variants:
Control commands and stable accessibility selectors:
Expected visible and data outcomes:
Owning modules and services:
Evidence and diagnostics:
Gotchas and recovery path:
Last verified commit or date:
```

Treat the map as compact materialized memory, not the ultimate source of truth. Runtime behavior and current code outrank stale prose. When they disagree, reproduce the behavior, determine the intended contract, and update the map with the verified result.

## Run a closed verification loop

For implementation or debugging:

1. Capture the baseline or reproduce the report before editing.
2. Preserve the exact environment, inputs, and navigation path.
3. Make the narrowest change that tests the hypothesis.
4. Repeat the same path and capture the same evidence.
5. Run adjacent regression checks and compare against acceptance thresholds.
6. Return the commands, outcome, and artifact paths to the lead.

For performance work, compare equivalent builds and conditions, warm up when appropriate, collect multiple samples, and report the distribution or variance rather than the best run. Use independent replicas or a larger sample only when noise or blast radius justifies the cost.

## Maintain the verifier

Update and retest the verification layer when startup, navigation, selectors, feature flags, accounts, logging, or feature behavior changes. Add a periodic audit only when the observed drift rate justifies one; do not schedule a daily routine by default.

A maintenance pass should:

- run health checks and a small set of known journeys;
- detect broken commands, stale selectors, missing features, and outdated prerequisites;
- repair the narrow problem and refresh the verification timestamp;
- avoid rewriting stable documentation merely for stylistic consistency.

Assign an owner for critical verification infrastructure, whether a person, rotating role, or explicitly authorized automation. Escalate verifier failures separately from product failures so a broken test harness does not create false bug reports.

## Scale only after the loop closes

Prove that one worker can repeatedly complete and verify representative tasks before adding more workers or automation. Parallel runs need isolated runtime state in addition to isolated code.

For incoming feedback, use this sequence:

1. normalize the report and map it to a feature;
2. reproduce it in a controlled environment;
3. classify it as reproduced, already fixed, environment-specific, insufficient evidence, or not reproduced;
4. attach proof and route it to the correct owner;
5. attempt a fix only within the workflow's current trust and approval level.

Automated reproduction does not imply authorization to edit, merge, deploy, message reporters, or close tickets. Promote those actions separately after representative evals and explicit approval.
