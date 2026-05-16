# h3 × Anchormap — structural traceability demo

## Give a 5-minute first reaction

AnchorMap flags docs-to-code drift in TypeScript PRs before merge.

You do not need to install anything to react to the preview.

Start here:

- Passing demo PR: https://github.com/fstepho/anchormap-h3-demo/pull/2
- New unmapped anchor: https://github.com/fstepho/anchormap-h3-demo/pull/3
- Stale mapping: https://github.com/fstepho/anchormap-h3-demo/pull/4
- Degraded analysis: https://github.com/fstepho/anchormap-h3-demo/pull/5
- Feedback issue: https://github.com/fstepho/anchormap/issues/5

If you only open one link, start with the passing demo PR. The other three PRs
show failure or warning-style cases:

- New unmapped anchor: a spec-like statement appears without a mapping.
- Stale mapping: a human mapping points to an anchor that is no longer observed.
- Degraded analysis: the report still renders, but analysis trust is reduced.

If you want the shortest failure case after the passing baseline, open the new
unmapped anchor PR. It is the most direct example of the review question:
a spec-like statement appears, but no mapping exists for it yet.

Useful reaction:

1. Did you understand the problem?
2. Did the PR report make sense?
3. Would you try this on a TypeScript repo?
4. What confused you?

Negative feedback is useful when it names the blocker.

## The problem

In many TypeScript projects, product/API/spec documents change separately from code.

During review, it is hard to see whether:

- a new requirement-like statement was added without code mapping;
- an old mapping points to something that no longer exists;
- a PR reduced traceability coverage;
- the report is still reliable enough to trust.

AnchorMap makes those cases visible in the PR workflow run as local artifacts
and a Markdown report, without posting a PR comment by default.

## Demo Context

This repo applies [Anchormap](https://github.com/fstepho/anchormap) to the source
of [h3](https://github.com/unjs/h3), an HTTP framework for TypeScript (MIT).
No h3 source file was modified. Four Anchormap command types were used: init, scaffold, map, and scan.

> **Anchormap** uses explicit human-defined mappings from formal spec anchors
> to source files and traces structural coverage through the local import graph
> — deterministically, with no LLM, no inference, no network.
> [github.com/fstepho/anchormap](https://github.com/fstepho/anchormap) · [npm](https://npmjs.com/package/anchormap)

---

## GitHub Action PR Preview

This repository also hosts the public AnchorMap GitHub Action preview scenario
set. The preview workflow is staged in a draft PR and uses:

- `fstepho/anchormap-action@v0-preview.3`
- `anchormap@1.2.2`
- explicit baseline scan artifact `.anchormap/baseline.scan.json`
- generated workflow artifacts, not PR comments or SaaS upload

Preview PRs:

| PR | Scenario | Expected AnchorMap signal |
| --- | --- | --- |
| [#1](https://github.com/fstepho/anchormap-h3-demo/pull/1) | Workflow base | Adds the preview workflow, policy, baseline, and demo guide. |
| [#2](https://github.com/fstepho/anchormap-h3-demo/pull/2) | Clean change | Policy pass, clean analysis. |
| [#3](https://github.com/fstepho/anchormap-h3-demo/pull/3) | Unmapped anchor | Policy exit `5` for `unmapped_anchor`. |
| [#4](https://github.com/fstepho/anchormap-h3-demo/pull/4) | Stale mapping | Policy exit `5` for `stale_mapping_anchor` and degraded analysis. |
| [#5](https://github.com/fstepho/anchormap-h3-demo/pull/5) | Degraded analysis | Policy exit `5` for degraded analysis from an unsupported local edge. |

The preview is intentionally draft-only and uses preview tag `v0-preview.3`.
No Marketplace release, merge, PR comment automation, or AnchorMap SaaS upload
is implied by these PRs.

The scenario workflow is configured with `fail-on-policy: "false"` so it can
upload artifacts and finish the GitHub job even when AnchorMap reports a policy
failure. For failure scenarios, inspect the job summary or artifact for the
AnchorMap decision; the GitHub check can still be green in preview mode.

---

## What was done

**Step 1 — init**

```bash
anchormap init --root src --spec-root specs
```

**Step 2 — scaffold**

```bash
anchormap scaffold --output specs/requirements.md
```

Anchormap parsed TypeScript exports in `src/` and generated a draft spec with
one anchor per exported API declaration, named from the module path and export name:

```
308 anchors generated
└─ e.g. UTILS.SESSION.USE_SESSION, UTILS.CORS.HANDLE_CORS,
         UTILS.JSON_RPC.DEFINE_JSON_RPC_HANDLER, ERROR.HTTP_ERROR,
         RESPONSE.TO_RESPONSE, MIDDLEWARE.CALL_MIDDLEWARE ...
```

The scaffold file keeps these 308 anchors as **draft anchors** — visible in scan, but not active and not mappable yet.

**Step 3 — promote 3 anchors and map them**

Three anchors were copied from the draft spec into an active spec file (`specs/demo.md`)
and mapped to their obvious seed files:

```bash
anchormap map --anchor UTILS.SESSION.USE_SESSION      --seed src/utils/session.ts
anchormap map --anchor UTILS.CORS.HANDLE_CORS         --seed src/utils/cors.ts
anchormap map --anchor UTILS.JSON_RPC.DEFINE_JSON_RPC_HANDLER --seed src/utils/json-rpc.ts
```

**Step 4 — scan**

```bash
anchormap scan --json > specs/scan.json
```

---

## Scan Brief

```text
AnchorMap scan brief

Scan status
- schema_version: 3
- analysis_health: clean
- product_root: src
- tsconfig_path: none
- public local aliases: none

Coverage
- product files: 58
- usable mappings: 3
- covered product files: 31 / 58
- multi-covered product files: 25

Anchors
- observed: 308
- active: 3
- draft: 305

Usable anchor reach
- UTILS.CORS.HANDLE_CORS: reached 29, unique 4, shared 25
- UTILS.JSON_RPC.DEFINE_JSON_RPC_HANDLER: reached 27, unique 2, shared 25
- UTILS.SESSION.USE_SESSION: reached 25, unique 0, shared 25

Findings
- untraced_product_file: 27

Interpretation: this is a successful clean scan. This brief is a non-contractual demo view over scan --json. It does not imply dead code, ownership, or architecture quality.
```

3 mappings. 58 source files. Anchormap traced the import graph and determined that
31 of them are **structurally reachable** from the 3 seeded files.
The other 27 are **not covered by the current explicit mappings** — not dead code,
just outside the scope of what was explicitly declared.

---

## Why 3 mappings reach 31 files

h3 has a deep internal import graph. Starting from `src/utils/session.ts`, Anchormap
follows relative imports transitively:

```
src/utils/session.ts
  → src/utils/cookie.ts
    → src/utils/internal/validate.ts
      → src/error.ts
        → src/utils/sanitize.ts
  → src/utils/internal/iron-crypto.ts
...
```

The full deepest chain found:

```
src/_entries/bun.ts → src/index.ts → src/_deprecated.ts → src/utils/base.ts
→ src/handler.ts → src/middleware.ts → src/response.ts → src/event.ts
→ src/types/handler.ts → src/types/h3.ts → src/plugin.ts → src/h3.ts
→ src/utils/request.ts → src/utils/event.ts → src/types/context.ts
→ src/utils/session.ts → src/utils/cookie.ts → src/utils/internal/validate.ts
→ src/error.ts → src/utils/sanitize.ts
```

20 files deep, all from relative imports — no inference, no heuristics.

---

## Reproduce it

```bash
git clone https://github.com/fstepho/anchormap-h3-demo
cd anchormap-h3-demo
npm install -g anchormap

anchormap scan --json | jq '{health: .analysis_health, covered: .traceability_metrics.summary.covered_product_file_count, uncovered: .traceability_metrics.summary.uncovered_product_file_count}'
```

With the same h3 commit and Anchormap version, the scan is deterministic.

---

## Files added to this repo

```
specs/
  requirements.md   ← 308-anchor draft spec generated by scaffold
  demo.md           ← 3 active anchors promoted from the draft
  scan.json         ← full anchormap scan --json output, pretty-printed
  scan-brief.txt    ← non-contractual scan brief for quick reading
  scan-summary.json ← compact JSON summary of the scan result
anchormap.yaml      ← config + explicit mappings (the only persistent state)
```

No h3 source file was modified.

---

## What this is not

`untraced_product_file` does not mean a file is unused or removable.
It means it is not reachable from any explicit mapping under the current
supported TypeScript graph rules. Anchormap makes no judgment about what
to do with those files.
