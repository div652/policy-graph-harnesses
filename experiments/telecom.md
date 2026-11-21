# Telecom: from an initial graph to a frozen two-model evaluation

## Scope and construction

The goal was to replace open-ended runtime planning with a controller derived
from the public telecom policy and technical-support manual. The builder
inspected permitted public tool signatures, return models, tool behavior, and
framework integration. Initial fixtures were invented, not copied from hidden
task answers.

The benchmark snapshot was
`fc0055dc4e0a316c3f83133267fbd6faaa770992`. This project used the **official
74-training / 40-test split**, not the separate AHE experiment's 57/11/46
partition.

The runtime graph included reusable procedures for service, mobile data,
speed, picture messaging, payment, refueling, and roaming. It selected the
diagnostic, customer action, agent-side tool, arguments, question, and terminal.
The smaller model interpreted the current customer reply under a selected
schema. Backend results were handled by typed code.

This was manually authored, frontier-model-assisted graph construction. No
general policy compiler, BFS over all configurations, or runtime planner was
implemented.

## How the graph developed

### V1: an executable policy/interface-derived baseline

The initial controller already separated device observations from backend
records and represented identity, target, pending action, consent, and goals.
One training trial per task produced **20/74 successes (27.03%)**.

That final baseline retained 59 originally scored episodes, including 44
failures, and recovered only 15 unscored infrastructure timeouts. The recovered
subset added five successes. Earlier setup-only failures were preserved;
they were not successful scored evaluations.

The low score exposed information-contract problems rather than a lack of
tools:

- Desired outcomes could be confused with observed outcomes.
- Negative diagnostics could be treated as inability to perform the check.
- Stale or insufficiently attributed facts could influence completion.
- Generic completion could stand in for the actual requested goal.
- Mandatory follow-up could be lost across conversational changes.
- Exact quotation copying created avoidable interpretation failures.

### V2: make knowledge and evidence mean the right thing

V2 changed the state and transition contracts:

| Repair | General lesson |
|---|---|
| Separate goal intake from observed results | A request is not a measurement |
| Bind diagnostics to their pending question | Facts need actor, source, target, and request context |
| Accept unfavorable diagnostic outcomes | A negative result can advance diagnosis |
| Invalidate affected observations after changes | Old evidence cannot verify a new configuration |
| Use goal-specific predicates and contradiction guards | An intermediate improvement is not always task completion |
| Require resolution confirmation for the graph's solved terminal | Official reward and conversational completion differ |
| Preserve payment/readback and restart obligations | New dialogue cannot erase unfinished work |
| Attach provenance in code | Do not make the model reproduce bookkeeping |

Training improved to **69/74 (93.24%)**. The frozen v2 then underwent the
first evaluation on the 40 reserved test tasks, three trials each:
**105/120 (87.50%)**, with trial scores **38/40, 37/40, 30/40**.

The unexpectedly low third trial was inspected. The recorded failures were
blocked graph endings, not timeouts that could be recovered. Model-label and
available native-log inspection found no observed substitution, but did not
independently attest upstream model weights.

Earlier v1/v2 Git checkpoints were reconstructed from retained snapshots.
They must not be described as prospective pre-run commits.

### V3: repair the actual customer-facing interface

The five remaining v2 training failures reduced to two main mechanisms.

**1. A correct private argument was absent from the actual instruction.**
The graph knew the messaging application's exact tool identifier, but its
customer-facing wording was too vague. Customers repeatedly asked for that
identifier. V3 rendered the exact argument from the operator's own argument
dictionary, including `app_name="messaging"`.

This was not a request for more autonomous reasoning. It was a mismatch
between the graph's internal edge and the interface its recipient could use.

**2. The diagnostic supplied a literal name, while the schema expected an
abstract judgment.** A check reported the current APN name and a separate
MMS setting. The model repeatedly supplied the MMS fact without the APN
correctness fact. V3 extracted the literal name, mapped documented names to
their meaning in code, rejected unknown/conflicting observations atomically,
and asked only for still-missing fields.

The repair also prevented a known invalid APN from coexisting with a
success-authorizing connectivity claim. It did not infer general connectivity
or task success merely from a recognized name.

Implementation, regression tests, and the unrun proposal were committed before
validation. The core implementation commit was
`a5febf8b3ea2b9f839212cf1c693a66f79728090`.

By this point v2's test failures had already been inspected. V3's later
test measurements are therefore **known-test reruns**, even though its
documented repair hypotheses came from the five training failures.

## Final frozen evaluation

The already-built v3 was evaluated on training once, then on test three
times. The interpreter was then replaced with Qwen and the same test grid
was evaluated three times. No graph, prompt, training, or selection step
occurred between those stages.

| Cohort | Interpreter | Tasks x trials | Successes | Valid failures | Final unscored |
|---|---|---:|---:|---:|---:|
| Train | `gpt-5.4-mini` | 74 x 1 | **73/74 (98.65%)** | 1 | 0 |
| Test, known-test rerun | `gpt-5.4-mini` | 40 x 3 | **119/120 (99.17%)** | 1 | 0 |
| Test, known-test rerun | `Qwen/Qwen3-30B-A3B-Thinking-2507` | 40 x 3 | **115/120 (95.83%)** | 5 | 0 |

| Three-trial measure | V2 mini | V3 mini | V3 Qwen |
|---|---:|---:|---:|
| Trial 1 successes | 38/40 | 40/40 | 36/40 |
| Trial 2 successes | 37/40 | 39/40 | 39/40 |
| Trial 3 successes | 30/40 | 40/40 | 40/40 |
| Mean episode success | 87.50% | 99.17% | 95.83% |
| Tasks passing at least once | 40/40 | 40/40 | 40/40 |
| Tasks passing all three | 29/40 | 39/40 | 36/40 |

All five v2 training failures passed in the v3 run, but one previous success
failed. All 15 v2 test failures passed in v3, but one previous success failed.
These are descriptive paired counts for stochastic conversations, not
per-edit causal ablations.

### Model and execution protocol

| Setting | Mini interpreter | Qwen interpreter |
|---|---|---|
| Customer simulator | `gpt-5.4` | `gpt-5.4` |
| Concurrency | 20 | 6 |
| Reasoning | Intentional provider default | Thinking checkpoint, `qwen3` reasoning parser |
| API | Buffered chat completions | Buffered chat completions |
| Sampling | No overrides | Temperature 0.6, top-p 0.95, top-k 20, min-p 0 |
| Output limit | Requested 2,500; not enforced on this measured adapter path | 16,384 generation tokens, including reasoning |
| Client timeout | 600 seconds | 600 seconds |
| Ordinary SDK/episode retries | None | None |

The customer requested 4,096 output tokens, also not an enforced cap on the
measured Copilot/LiteLLM path. Episode limits were 1,800 seconds, 200 framework
steps, 10 framework errors, and the graph's 100-dispatch budget.

Qwen used original BF16 weights at revision
`144afc2f379b542fdd4e85a1fcd5e1f79112d95d`, local vLLM, a 32,768-token context,
and six active sequences. The runner seed was 42; native trial seeds were
670487, 116739, and 26225. These do not seed the customer/interpreter generations.
V1/v2 mini concurrency was 10, not v3's 20.

### Customer-model outage and restricted recovery

The initial Qwen attempt scored 76 episodes: **71 successes and five valid
failures**. Another **44 were unscored** because the GPT-5.4 customer path
recorded 36 timeouts, six cancellations, and two connection errors.

The customer server's extension host later exited after its renderer had
been disconnected too long. Qwen was healthy at the initial outage
inspection. This established the failing role and eventual disconnection;
it did not explain every earlier timeout.

After explicit recovery authorization and fresh preflight, a separate recovery
checkpoint resumed **only the 44 unscored task/trial pairs**. All 44 passed.
Every originally scored record, including all five failures, remained
unchanged. The graph and prompts were not repaired during recovery.

Final accounting: **314 retained scored pairs, 358 episode attempts**,
excluding smoke/probe calls. Qwen trial 2 includes four recovered pairs;
its entire third trial was recovered. The 40/40 third trial does not mean
Qwen learned during evaluation.

Completion was recorded on **October 7, 2026, at 16:47:29 UTC**.

## What the successes do and do not mean

### Official success is not internal graph completion

| Ending among officially successful episodes | Mini train | Mini test | Qwen test |
|---|---:|---:|---:|
| Graph `solved` | 2 | 5 | 4 |
| Graph `escalated` | 8 | 36 | 36 |
| Customer stopped while graph remained `running` | 63 | 78 | 75 |
| **Total official successes** | **73** | **119** | **115** |

The evaluator could recognize the required environment effect before the
graph's final confirmation exchange, or accept escalation. The headline
accuracy is not a claim that nearly every episode reached a solved graph node.

### The model still interpreted every consumed customer turn

| Retained-trajectory count | Mini train | Mini test | Qwen test |
|---|---:|---:|---:|
| Interpreter completions | 1,336 | 1,852 | 1,801 |
| Consumed customer turns | 1,336 | 1,852 | 1,801 |
| Turns handled without a model call | 0 | 0 | 0 |

Decision authority moved to the graph, but measured language interpretation
was not rare. Token usage/cost was unavailable, not zero. Counts exclude
probes and incomplete work in discarded infrastructure-error episodes.

### Genuine failures remained

The mini training failure ended with the customer stopping before an MMS
repair finished. The mini test failure was an unexpected out-of-scope
customer stop after a speed-test request. Both were kept as valid zero rewards.

Four Qwen failures arose when a reported **0.00 Mbps, "Very Poor"** reading was
classified as `no_connection` rather than the reported speed category. That
schema-valid semantic error selected a different deterministic branch.
One further failure involved an unanswered concern about losing photos during
a restart; the generic explanation did not resolve the obstacle.

No valid failure was retried, and no repair based on these final observations
was folded into the reported v3.

## Baseline interpretation and provenance

V1 -> v3 training and v2 -> v3 repeated testing are **same-model graph
development comparisons**. Historical non-graph references exist for mini
(**59/138 = 42.75%** in the original post-hoc evaluation, and **49/138 = 35.51%**
in a separate sequential-run evaluation) and Qwen (**44/138 = 31.88%**).
The original mini baseline's trial scores were 20/46, 20/46, and 19/46.
These historical evaluations use 46 test tasks rather than the graph's 40,
with different execution protocols. They are not matched controls for the
numbers above; both mini baseline measurements are retained.

The frozen graph workspace hash was
`357ee6d7b68dc7f8722c15d0a43e50735cb1b0704b24533758bf56c82e1cc10b`.
The repaired frozen runner was
`0259bbdfa95d7c3bec22ac4a76ac9d9a35ad8b65`;
the infrastructure-only recovery freeze was
`58779721e2ced7017c6a1ce023c770245937c579`.
The final report/evidence commit was
`38fbef7da85406f1c2689d0b62751d1f6752dade`.

See [measurement](../docs/measurement.md),
[reproduction](../docs/reproduction.md), and
[aggregate evidence](../evidence/results.json).
