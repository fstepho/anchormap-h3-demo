# AnchorMap PR Demo

This branch adds the self-contained AnchorMap preview workflow for demo pull requests.

The workflow uses the preview Action tag:

```text
fstepho/anchormap-action@v0-preview.2
```

It pins the CLI package to `anchormap@1.2.2` and supplies an explicit baseline scan at `.anchormap/baseline.scan.json`. The workflow does not infer a baseline from Git refs, does not post PR comments, and does not upload source to any SaaS service.

The workflow base is tracked in draft PR
[#1](https://github.com/fstepho/anchormap-h3-demo/pull/1). Scenario PRs target
that branch, not `main`.

## Scenarios

| PR | Branch | Expected signal |
| --- | --- | --- |
| [#2](https://github.com/fstepho/anchormap-h3-demo/pull/2) | `demo/scenario-clean` | Policy pass with clean analysis. |
| [#3](https://github.com/fstepho/anchormap-h3-demo/pull/3) | `demo/scenario-unmapped-anchor` | Policy exit `5` for `unmapped_anchor`; analysis remains clean. |
| [#4](https://github.com/fstepho/anchormap-h3-demo/pull/4) | `demo/scenario-stale-mapping` | Policy exit `5` for `stale_mapping_anchor` and degraded analysis. |
| [#5](https://github.com/fstepho/anchormap-h3-demo/pull/5) | `demo/scenario-degraded-analysis` | Policy exit `5` for degraded analysis from an unsupported local edge. |

All scenario workflows are configured with `fail-on-policy: "false"` so the
GitHub workflow can complete and upload artifacts even when AnchorMap policy
decision is `fail`.

Only real GitHub Actions runs and artifacts should be used as evidence.
