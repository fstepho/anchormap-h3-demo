# AnchorMap PR Demo

This branch adds the self-contained AnchorMap preview workflow for demo pull requests.

The workflow uses the draft Action branch:

```text
fstepho/anchormap-action@task/gha-1-composite-action
```

It pins the CLI package to `anchormap@1.2.2` and supplies an explicit baseline scan at `.anchormap/baseline.scan.json`. The workflow does not infer a baseline from Git refs, does not post PR comments, and does not upload source to any SaaS service.

## Scenarios

Demo PR branches target this branch, not `main`:

- clean PR: no AnchorMap policy failure expected;
- unmapped anchor PR: adds an observed anchor without a stored mapping;
- stale mapping PR: removes an observed anchor that still has a stored mapping;
- degraded analysis PR: introduces an unsupported local edge that degrades analysis.

Only real GitHub Actions runs and artifacts should be used as evidence.
