# Retail and airline: independent construction and seven training passes

## Why this is a separate experiment

After telecom, the general procedure was applied to two different policy
domains. This was **not** a copied telecom controller or a transfer evaluation
of its learned rules.

Construction used the generic playbook, permitted structural/setup references,
and the public retail/airline policies and interfaces. It explicitly excluded
training runs, solutions, grading results, and trained domain controllers from
the other experiment repository. Subsequent repairs used only this project's
own approved training evidence.

This publication reports both projects together after development; it does
not retroactively mix their training evidence or introduce new graph repairs.

## Data boundary and model roles

Public benchmark commit:
`fc0055dc4e0a316c3f83133267fbd6faaa770992`.

The split was constructed independently per domain:

```text
IDs = sorted string task IDs
shuffle IDs with a fresh random.Random(42)
training size = floor(0.7 * number_of_tasks + 0.5)
remaining IDs = reserved test set
```

| Domain | Corpus | Training | Reserved test |
|---|---:|---:|---:|
| Retail | 114 | 80 | 34 |
| Airline | 50 | 35 | 15 |
| **Total** | **164** | **115** | **49** |

This is a task-level split, not a family-disjoint split. Task IDs were
mechanically projected for membership. Hidden task/database contents were not
graph-development inputs; the official runtime components received their
designated private information.

**All 49 reserved test tasks remain unexecuted. No test accuracy is reported.**

The runtime interpreter was `gpt-5.4-mini` throughout. The official customer
was `gpt-5.4`. Retail's natural-language judge also used `gpt-5.4`, an explicitly
approved substitute for the unavailable upstream default
`gpt-4.1-2025-04-14`. Official judge prompts and scoring were retained, with
strict rejection of malformed/incomplete judge output.

## Initial architecture

There were separate retail and airline graphs, sharing a small executor,
interpretation/transport infrastructure, and tracing conventions.

The design had four important boundaries:

1. Customer statements established intent, preferences, reasons, and scoped
   assent; public tools established ownership, status, prices, availability,
   balances, and actual effects.
2. Model outputs were typed observations, never tool choices or backend
   state mutations.
3. Every database update required an exact user-visible offer and approval.
4. Selection, authorization, dispatch, and result verification were distinct
   stages with recorded transitions.

Compared with telecom, these domains placed much more pressure on **reference
resolution and multi-goal state**. A phrase can refer to an owned source
address, a target order, a physical unit, a replacement variant, or a remembered
payment method. Treating all of those as generic slots was insufficient.

## The complete training history

Each complete pass planned one episode for each of the same 115 training
tasks. Unscored failures are not removed from the planned denominator.

| Pass | Retail success/scored | Airline success/scored | Combined success/scored | Success/planned | Unscored or incomplete |
|---|---:|---:|---:|---:|---|
| 1: initial graph | 18/70 | 23/35 | 41/105 (39.05%) | 41/115 (35.65%) | 10 adapter errors |
| 2: first repairs | 16/71 | 22/35 | 38/106 (35.85%) | 38/115 (33.04%) | 9 empty-output errors |
| 3: contract-first | 18/75 | 23/34 | 41/109 (37.61%) | 41/115 (35.65%) | 6 output errors |
| 4: evidence-led, **partial** | 19/29 | 14/14 | 33/43 (76.74%) | 33 successes observed; incomplete cohort | 5 output errors, 5 timeouts, 12 interrupted, 50 not started |
| 5: deterministic binding | 41/73 | 24/35 | 65/108 (60.19%) | 65/115 (56.52%) | 7 output errors |
| 6: scoped selection, **best observed** | 49/77 | 27/32 | 76/109 (69.72%) | **76/115 (66.09%)** | 6 output errors |
| 7: recoverable selection, **latest** | 45/70 | 25/33 | 70/103 (67.96%) | **70/115 (60.87%)** | 12 output errors |

Pass 4 is not a completed 76.74%-accuracy run. A separately authorized
67-episode continuation never started scoring because its live gate passed
only 34/36. Nothing from that attempted continuation was merged into later
passes.

Pass 2's original runner labeled the nine empty HTTP-200 responses
`infrastructure_error`. Their actual evidence did **not** establish a
disconnection. The original label is retained in the machine-readable
evidence, alongside the corrected interpretation. They were not retried.

## What changed, and why

### Passes 1-2: discover contradictions at the language boundary

The first pass had ten retail catalog-string decoding errors. Initial repairs
addressed adapter and source-attribution problems, but the aggregate result
regressed.

The second pass revealed a deeper contradiction: a narrow question could
silently expose a broad schema that asked the interpreter to restate already
known identifiers. The source validator then correctly rejected identifiers
that were absent from the current reply. Most rejected attempts concerned
this literal-identifier boundary, not inability to understand the actual
payment or reason answer.

**Lesson:** the displayed question, actual schema, accepted fields, and
source validator must describe the same job.

### Pass 3: redesign around exact contracts

The contract-first revision introduced or strengthened:

- Exact per-question answer schemas without hidden extra fields.
- Stable goals and typed preferences rather than reparsing concatenated history.
- Distinct answer, correction, clarification, replacement, and withdrawal paths.
- Atomic partial patches that preserve unmentioned accepted values.
- Source-literal identifiers and deterministic descriptive resolution.
- Pure selectors and explicit logged internal transitions.
- Verification/readback precedence over unrelated new work.
- Material-progress accounting that preserves distinct live obligations.

A fixed live suite covered 12 mechanisms with three paraphrases each.
The final candidate passed 36/36, but training still reached only 41 successes.
Structural correctness and a small live gate did not eliminate missing
domain-resolution capabilities.

### Passes 4-5: add evidence-backed resolution and computed answers

Repairs focused on the first broken state/contract distinction:

- Resolve targets only against owned records.
- Distinguish a saved source address from the address being changed.
- Resolve saved passenger/payment references instead of demanding every field
  be restated.
- Compute requested summaries from authoritative records rather than dumping
  raw records or trusting model arithmetic.
- Represent explicit goal order and dependencies.
- Reconcile same-order work before an irreversible one-shot item operation.
- Preserve concrete price, payment, itinerary, and scope constraints.

Pass 4 was interrupted by customer-model timeouts. The later fifth pass was
a new full evaluation of a frozen revision, not a selective recovery of
failed tasks. It produced 65/115 planned successes.

### Pass 6: make selection target- and unit-specific

The sixth revision separated original-item selection from replacement
selection, scoped alias and capacity resolution to each owned order, and
added typed deselection and explicit order subsets. It also tightened
passenger/target invalidation, financial wording, and flight-stop preferences.

An item variant ID does not necessarily identify one purchased physical
unit. Likewise, withdrawing one selection is not withdrawing the whole goal.
Those distinctions had to become graph state and transition rules.

State semantics advanced to version 7. The frozen pass achieved
**76/115 planned successes**, the best observed combined training result.

### Pass 7: repair recovery paths, but preserve the regression

The seventh revision addressed a new trap: a multi-request clarification
could require at least two requests forever, rejecting a later single-request
correction or withdrawal. It also added or refined:

- Current-offer protection against false "additional request" intake.
- Original purchased-item ranking and per-unit replacement choices.
- Complete-option repair and explicit keep-original choices.
- Qualified saved-address sources.
- Direct original-payment wording and refund-rule explanations.
- Exact time-string normalization.

The final candidate passed all 36 live gates; an earlier 34/36 preparation
remained preserved. State semantics advanced to version 8.

The complete pass nevertheless **fell from 76 to 70 successes**:
nine gains, 15 regressions. Six previously successful retail tasks became
output errors. Among the 100 tasks scored in both passes, successes fell
from 70 to 69. The regression is not merely a percentage-denominator artifact.

No further repair/evaluation was silently folded into this final result.

## Baseline, best, and latest by domain

| Domain | Initial graph, planned rate | Best combined snapshot, pass 6 | Latest snapshot, pass 7 |
|---|---:|---:|---:|
| Retail | 18/80 (22.50%) | 49/80 (61.25%); 77 scored | 45/80 (56.25%); 70 scored |
| Airline | 23/35 (65.71%) | 27/35 (77.14%); 32 scored | 25/35 (71.43%); 33 scored |
| **Combined** | **41/115 (35.65%)** | **76/115 (66.09%); 109 scored** | **70/115 (60.87%); 103 scored** |

The retail scored-only percentage rose slightly from pass 6 to pass 7 while
its number of successes fell. Reporting only successes/scored would obscure
that deterioration.

These are comparisons between **initial and revised graphs using the same
interpreter**, not against an unassisted/default agent. There is no matched
no-graph baseline for this training cohort and no reserved-test result.

## Execution differences and error accounting

| Setting | Passes 1-3 | Pass 4 | Passes 5-7 |
|---|---|---|---|
| Concurrency | 10 | 20 | 20 |
| Client timeout | 600 s | 600 s | 1,200 s |
| Episode timeout | 1,800 s | 1,800 s | 1,800 s |
| Interpreter | `gpt-5.4-mini` | Same | Same |
| Customer / retail judge | `gpt-5.4` | Same | Same |

Reasoning was intentionally provider-default, with buffered chat completions
and no LLM cache. Limits included 200 framework steps, 100 graph dispatches,
10 tool errors, two interpretation attempts per source/contract job, and
three identical no-progress dispatches before unresolved termination.

Passes 5-7 had no recorded timeouts, disconnections, aborted/unstarted
episodes, or episode retries. Their unscored outputs were retained. Pass 7
had 33 valid zero rewards and 12 unscored output errors; its 2,295 recorded
model calls all returned HTTP 200 with matching observable model labels.
HTTP success did not guarantee usable model text.

The retail judge substitution, concurrency/timeout changes, stochastic
dialogue, and bundled revisions preclude a graph-only causal interpretation.
Token usage and cost were unavailable.

## Remaining limitations

Observed failures still involved composite conditional requests, cross-order
scope, physical-unit corrections, unavailable order chronology, one-shot tool
constraints, source-address ambiguity, and operation ordering. Airline also
exposed interactions between operation order and computed entitlements.

The graph did not infer missing chronology, execute consent withheld by the
customer, rewrite the public tools, or copy expected evaluator arguments.
Some post-run observations suggest missing planning capabilities; they do not
establish that adding a planner would solve them.

This is evidence that the method can improve a difficult controller, but also
that refinement can regress and fixed synthetic gates can miss real dialogue
failures. It is not evidence of solved general retail/airline automation.

## Frozen identities

| Snapshot | Evaluated commit |
|---|---|
| Initial graph | `ac69c0761166ded92c8859b291245fab6b7d038d` |
| Best observed, pass 6 | `003ed020f813074170afc958130c936a2e4852c2` |
| Latest evaluated, pass 7 | `2bc60a2c6920b0451079e08ec089b6ad84fd7210` |
| Final reporting HEAD | `c6b440a` |

Reporting HEAD is not the evaluated source snapshot.
See [aggregate evidence](../evidence/results.json) for all seven identities
and [reproduction scope](../docs/reproduction.md) for archive limitations.
