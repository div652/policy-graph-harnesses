# Reproduction scope and provenance

## What this repository contains

This release publishes original explanatory prose, a reusable builder prompt,
curated aggregate measurements, and source/evidence identities. It does not
contain a runnable graph implementation or a complete raw benchmark archive.

| Included | Not included |
|---|---|
| General construction and refinement procedure | Original experimental controller source/history |
| Architecture and contract descriptions | Dependency environments or serving extension source |
| Telecom and retail/airline development histories | Task text, customer personas, environment databases |
| Aggregate counts and protocol metadata | Raw trajectories, grader output, provider logs |
| Source commits and SHA-256 provenance | Model weights or credentials |

Consequently, cloning this repository does **not** reproduce the reported
accuracy by running a bundled evaluation command. The evidence permits
checking arithmetic and understanding provenance; without the retained
original artifacts, it does not independently replay the official grades.

## Public upstream identities

Benchmark source:

```text
repository: https://github.com/sierra-research/tau2-bench
commit: fc0055dc4e0a316c3f83133267fbd6faaa770992
```

Qwen interpreter:

```text
model: Qwen/Qwen3-30B-A3B-Thinking-2507
revision: 144afc2f379b542fdd4e85a1fcd5e1f79112d95d
weights used: original BF16
```

These are historical pins, not instructions to substitute the latest
benchmark or model release. The benchmark's current default branch is not
the protocol against which these results were collected.

Third-party policy, software, and model artifacts remain subject to their
own licenses; this repository does not redistribute or relicense them.

## Retained experiment source identities

These identify original local archives. They are **not** links to source
commits shipped in this repository.

### Telecom

| Role | Identity |
|---|---|
| V3 implementation | `a5febf8b3ea2b9f839212cf1c693a66f79728090` |
| Workspace including later documentation | `dd21242564055dd4ab44c27500bbfa736eb32bef` |
| Frozen evaluation runner after setup-only repair | `0259bbdfa95d7c3bec22ac4a76ac9d9a35ad8b65` |
| Infrastructure-only recovery freeze | `58779721e2ced7017c6a1ce023c770245937c579` |
| Final report and compact evidence | `38fbef7da85406f1c2689d0b62751d1f6752dade` |
| V3 workspace SHA-256 | `357ee6d7b68dc7f8722c15d0a43e50735cb1b0704b24533758bf56c82e1cc10b` |

V1/v2 were originally identified by frozen workspaces and manifests.
Later Git checkpoints for them were reconstructed retrospectively. V3 and
its evaluation/recovery proposals used prospective commits.

The original Qwen cohort summary remains incomplete by design; the final
merged score is in the separate `recovery-01/summary.json`. Likewise, v1's
final aggregate is its `recovery-timeouts-01/summary.json`, not the earlier
incomplete summary.

### Retail/airline

| Training pass | Evaluated commit |
|---|---|
| 1 | `ac69c0761166ded92c8859b291245fab6b7d038d` |
| 2 | `8250044c78ee6bd1afe636e07e24c03f831046c9` |
| 3 | `a1321101864f285126d4ac6732797bf42ea559ef` |
| 4, partial | `0592c70d85c7320b884d31ef7c54150948c212d5` |
| 5 | `e5fdfc08a9f06b90302187272c0adac4081cc986` |
| 6, best observed | `003ed020f813074170afc958130c936a2e4852c2` |
| 7, latest | `2bc60a2c6920b0451079e08ec089b6ad84fd7210` |

Final reporting commit:
`c6b440a247535d0a7d79c368f8445056b6a2a04c`.
Every training pass used its frozen protocol, rather than this later reporting
HEAD. Earlier attempts and rejected preparations were preserved separately.

## Evidence files

- [`results.json`](../evidence/results.json): selected aggregate fields from
  original score summaries, with explicit protocol/comparability labels.
- [`results.csv`](../evidence/results.csv): flat cohort counts for spreadsheet
  inspection. Partial-run and historical-reference rows retain their status.
- [`provenance.json`](../evidence/provenance.json): source-relative archive
  paths, SHA-256 hashes, tracking status, and source repository identities.

Evidence is extracted through field allowlists. It excludes task-level
identifiers, descriptions, transcripts, environment values, and local host or
account metadata.

The historical non-graph narrative reports were untracked local files at
publication. Their headline counts were independently checked against saved
score summaries. The provenance manifest marks whether each source file was
tracked; a repository HEAD is not presented as proof that an untracked result
was committed.

Hashes establish which retained file a claim refers to. They are not a
signature, a substitute for publishing that file, or independent validation
of a benchmark's result.

## Recreate the methodology on a new policy task

1. Start with [the builder prompt](../prompts/frontier-builder.md) and
   [the playbook](methodology.md). Supply only the policy, permitted public
   interfaces, framework contract, and declared training access.
2. Pin the sources and freeze task membership without inspecting reserved
   task contents. Specify which model serves each role.
3. Construct the rule ledger, typed knowledge state, operators, independent
   authorization, exact interpretation schemas, and graph-owned messages.
4. Add invented interface-faithful tests and a model-free trace. Commit the
   proposal and code before validation and approved live probes.
5. Evaluate a frozen initial graph. Preserve every outcome; diagnose the first
   broken state/contract distinction and implement general repairs.
6. Freeze each new revision and compare it on the same permitted training
   cohort. Track regressions and output failures, not just successes.
7. Before untouched evaluation, freeze the complete protocol and baseline.
   Report full denominators, test exposure, role-specific model evidence,
   recovery, and terminal-versus-reward differences.

Those steps are reproducible methodology, not a guarantee of the same scores
on another domain.

## Replay the original numerical experiment

Exact reruns additionally require the retained controller snapshot, manifests,
dependency locks, public-source hashes, original split/schedule, and original
runner adapters. Obtain those artifacts separately with appropriate
redistribution permission before attempting an exact replay.

For telecom, reproduce the unchanged graph first, then the sequential
mini-train, mini-test, and Qwen-test stages with their declared concurrency.
The historical outage is not a required experimental condition. A new run's
failures must follow its own frozen recovery rule, not the original run's
44-pair selection.

For retail/airline, select the actual evaluated commit, not reporting HEAD.
Use the matching frozen protocol: concurrency and client timeout changed
between passes. The reserved test tasks are not part of the recorded training
reproduction and have no reported result.

Even with all source artifacts, scheduling seeds cannot reproduce unseeded
model text or guarantee identical rewards. Fresh results must be labeled
as new measurements.
