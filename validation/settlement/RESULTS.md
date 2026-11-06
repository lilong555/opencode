# Settlement Validation

PR: [anomalyco/opencode#62167](https://github.com/anomalyco/opencode/pull/62167).

| Revision | Commit |
| --- | --- |
| Baseline | `217cc811de9b0c1594924e89e48705871f1cacea` |
| Candidate | `78857fca51b85544811616edd82c830e2de22118` |

The candidate passes the root lint and both workspace typecheck commands on
Ubuntu. With ripgrep prepared before testing, all 33 workspace unit tasks pass
on both Ubuntu and Windows. The implementation head remains `78857fca`.

## Workspace Checks

[Run 42102508583](https://github.com/lilong555/opencode/actions/runs/42102508583)
uses workflow commit `8bb55e595a276dc38bf15c5c39be1ad596df60bf`. Both revisions
pass `bun run check` and `bun run typecheck:tsc` on `ubuntu-24.04`.

Each revision completes 35/35 Bun typecheck tasks and 34/34 official TypeScript
tasks, with zero task-cache hits. Oxlint reports 553 warnings and zero errors on
each revision. Tracked-source integrity checks pass.

## Cross-Platform Units

[Run 42104731792](https://github.com/lilong555/opencode/actions/runs/42104731792)
uses workflow commit `6376a4cc4cdcd3d93f1a9b516438a8045cbb391c`.

| Platform | Revision | Workspace Tasks | Core Pass | Core Skip | Core Fail |
| --- | --- | --- | ---: | ---: | ---: |
| Ubuntu 24.04 | Baseline | 33/33 pass | 6,981 | 337 | 0 |
| Ubuntu 24.04 | Candidate | 33/33 pass | 7,003 | 337 | 0 |
| Windows 2025 | Baseline | 31/33 pass | 6,208 | 1,110 | 1 |
| Windows 2025 | Candidate | 33/33 pass | 6,231 | 1,110 | 0 |

All four runners attempt all 33 workspace tasks, with zero task-cache hits.
Every task has a final exit code in its Turbo JSON summary. Core covers 314
files per revision and platform; platform-specific skips are preserved.

The Windows baseline has two timeouts:

- `@opencode/server`, `test/artifact.test.ts`: local and plugin-backed artifact
  management, 5,000 ms.
- `@opencode/core`, `test/git.test.ts`: packing loose objects and combining
  packs, 60,000 ms. The terminated Git child also produces one between-test
  error in the core summary.

Both tests pass as part of the candidate's complete workspace run. These
baseline-only observations are retained without assigning a speedup or a
causal explanation to the settlement change.

## Focused Repetition

Each revision runs its existing Form and Permission tests with
`--rerun-each 10 --randomize --seed 34853` through core's isolated-home runner.
The repetitions occur within one test invocation per platform.

| Platform | Baseline | Candidate |
| --- | ---: | ---: |
| Ubuntu 24.04 | 1,340 pass, 0 fail | 1,560 pass, 0 fail |
| Windows 2025 | 1,340 pass, 0 fail | 1,560 pass, 0 fail |

The Windows contender assertion that failed in the first run passes 20/20
targeted repetitions on each revision. Both Linux TUI case families observed in
the first run pass 70/70 targeted executions on each revision. Their complete
workspace suites also pass in the diagnostic run.

## Initial Failures

The initial run remains available with its original failures:

- Both Linux core suites fail the same 23 Ripgrep/search tests. Windows baseline
  fails 22 corresponding cases; an external-symlink case is skipped there.
  The error stacks show the test network guard rejecting the automatic
  ripgrep 15.1.0 download when no executable is available.
- Windows candidate reports one client contender assertion: three starts,
  expected two. Other tasks are terminated by Turbo's default failure policy
  before the candidate core suite completes.
- Each Linux job records a different TUI failure while other work is being
  terminated. TUI lacks a complete result in that initial run.

The diagnostic run installs the same ripgrep 15.1.0 archives and SHA256 values
pinned by `packages/core/src/ripgrep/binary.ts`, then adds the executable to
PATH. The test network guard and product code remain unchanged.

Full workspace execution uses
`GITHUB_ACTIONS=false bun turbo test --concurrency=3 --continue=always --summarize`.
Continuing after a failed task preserves independent suites' results and keeps
the final exit code nonzero when any task fails.

## Environment And Evidence

All jobs check out the exact tested commit before installing dependencies.
They use the repository's frozen Bun installation, Bun 1.4.4 baseline,
Node 24.22.2, four CPUs and approximately 16 GiB RAM per runner.

Diagnostic image versions:

- Ubuntu: `20261101.360.1`, identical for baseline and candidate.
- Windows: `20261030.294.1`, identical for baseline and candidate.

The first run's Windows candidate uses image `20261103.298.1`; its baseline uses
`20261030.294.1`. Comparisons retain that image difference. Both diagnostic
Windows jobs use `20261030.294.1`.

The six initial artifacts and four diagnostic artifacts contain source
identity, runtime details, raw test output and tracked-source integrity logs.
Diagnostic artifacts also contain ripgrep provenance, per-step outcomes and
Turbo JSON summaries. All ten source-integrity steps pass. The ten downloaded
ZIPs pass CRC verification and match GitHub's SHA256 artifact digests; source
identities agree with both the job metadata and the runtime records.
[Artifact manifest](ARTIFACTS.json) records the verified identities and digests.

These runs provide supplemental checks of fixed source commits. Upstream
workflow authorization and review remain with the maintainers; browser E2E,
packaging, release builds and generated-client/OpenAPI checks belong to the
upstream workflow.
