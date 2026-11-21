# Measurement, baselines, and limits

## Three different meanings of "baseline"

| Baseline type | What it answers | Evidence available here |
|---|---|---|
| Initial graph, same runtime model | Did graph development improve the measured controller? | Telecom and retail/airline training; telecom v2 -> v3 test rerun |
| Historical non-graph agent, same named model | What did another recorded agent setup score? | Telecom mini and Qwen, on a different test split |
| Matched no-graph control | What is the graph's effect under otherwise matched conditions? | **Not measured for these graph cohorts** |

Do not call an initial graph a "without graph" baseline. Do not call the
historical base agent byte-identical to the stock benchmark agent.

### Historical non-graph references

The historical base harness subclassed the benchmark's generic `LLMAgent`.
It supplied the public policy and tools to the model; skills, middleware,
and sub-agent lists were empty and memory contained no evolved guidance.
The model, rather than the policy-state graph, selected the next action.

| Model | Historical successes | Test tasks x trials | Concurrency |
|---|---:|---:|---:|
| `gpt-5.4-mini` | 49/138 (35.51%) | 46 x 3 | 10 |
| `Qwen/Qwen3-30B-A3B-Thinking-2507` | 44/138 (31.88%) | 46 x 3 | 10 |

Both used the baseline workspace hash
`3ce98ba138600f130e4a816edba829e940a576e3300c4ef7a10b1ed4f821c70b`.
The historical Qwen checkpoint revision matched the v3 graph evaluation.
Both historical runs used `gpt-5.4` customers.

However, the historical split was **57 train / 11 validation / 46 test**,
not the graph experiment's **74 train / 40 test**. Runner seeds, wall-clock
conditions, serving configurations, and controller protocols differed.
Historical model-identity enforcement was also weaker than the later graph
adapters' fail-closed checks.

The historical mini reference is the unchanged checkpoint selected in the
second sequential AHE run. The Qwen reference is its separately completed
post-hoc base-agent evaluation. Neither was rerun for this publication.
The raw score summaries, not only narrative reports, were checked for the
published counts.

No cross-protocol difference is presented as a causal graph uplift.

## Count outcomes before computing percentages

For every frozen cohort, retain:

```text
planned = all scheduled task/trial pairs
attempted = pairs that started in that run
scored = pairs with valid official reward
successes = scored pairs whose official reward is 1
valid failures = scored - successes
unscored = planned - scored
```

Also distinguish output/adapter errors, transport failures, interrupted work,
and never-started pairs. The number of **episode attempts including recovery**
can exceed the number of unique planned pairs.

Use both:

```text
scored success rate = successes / scored
planned success fraction = successes / planned
```

The second describes pipeline success over the intended workload. It does not
retroactively assign an official zero reward to an unscored error.

For example, retail/airline pass 7 had 70 successes, 33 valid failures, and
12 output errors:

- Scored-only rate: **70/103 = 67.96%**.
- Planned success fraction: **70/115 = 60.87%**.

Both belong in the report. A partial run, such as pass 4, must not be presented
as a completed cohort using only the early scored denominator.

## Repeated-trial metrics

For a complete grid of `N` tasks and `K` trials:

```text
mean episode success = sum of all binary rewards / (N * K)
at-least-one success = tasks with >= 1 successful trial / N
all-trials success = tasks with K successful trials / N
```

These correspond to the reports' empirical `pass@1`, `pass@K`, and `Pass^K`
labels. The first numbered trial is not the repeated-trial `pass@1` mean.
At-least-one success is not consistency.

For telecom v3, both models passed all 40 tasks at least once. Mini passed
39/40 in every trial; Qwen passed 36/40 in every trial. Those consistency
figures are more informative than reporting only 100% at-least-once success.

No independence assumption or confidence interval is added to these small
repeated-task measurements. Trial order, shared tasks, and customer/server
variation matter.

## Reward is not the graph's terminal state

Always cross-tabulate official reward against graph terminal and customer stop.
An evaluator may find the required backend effect before the controller's
final confirmation, or accept an escalation. Conversely, a confident terminal
message does not establish an official task success.

The [telecom case study](../experiments/telecom.md) gives the actual
cross-tabulation. Its 99.17% mini test score is **not** a 99.17% graph-solved
terminal rate.

## Recovery is not score optimization

Preserve the original attempt and exact task/trial identity. Retry only the
declared eligible unscored infrastructure failures. Do not replace valid
low rewards or change prompts until a hard task passes.

Telecom Qwen recovery retained all 76 scored originals, including five zero
rewards, and recovered 44 customer-path errors once. That changes the time
and server instance of those conversations, so it is disclosed even though
graph source and model selection remained fixed.

In retail/airline, exhausted empty/malformed interpreter output was not
eligible disconnection recovery. A historical broad `infrastructure_error`
label does not override the underlying HTTP-200 evidence.

## Development and test boundaries

Telecom v2 was evaluated on the then-held-out 40 test tasks. The results and
failures were subsequently inspected before v3 development. V3 is therefore
a **known-test rerun**, not a new blind generalization result.

Retail/airline reserved 49 tasks but did not execute them. Its seven training
passes cannot establish held-out performance. Selecting the sixth as "best
observed" after seeing training results is descriptive, not a validation-based
generalization guarantee.

No hidden task IDs, expected arguments, or evaluator state were added to the
runtime graph to improve these results.

## Models, serving, and compute

Separate offline builder, runtime interpreter, customer simulator, and judge.
The strong customer/judge models are benchmark roles, not hidden planners
inside the evaluated graph.

The measured Copilot extension selected an exact requested model object and
generated the response's model label from that object. Matching labels and
negative controls check the observable routing contract; they do not
independently prove which upstream weights served the response.

The measured extension also buffered chat output, collapsed instruction
roles into user messages, and reported synthetic zero usage. Configured
parameters were not necessarily forwarded. In the telecom LiteLLM route,
renamed GPT output-token caps were ignored by the extension.

Consequences:

- Same named model does not imply independently attested identical weights.
- Identical graph code does not imply identical serving or equal compute.
- Usage/cost is **unavailable**, not free or tokenless execution.
- A graph-owned decision can still require an interpreter call every turn.
- SDK, proxy, interpretation, and episode retries are different layers.

No claim of token reduction, cost savings, latency superiority, or rare
runtime model use follows from this release.

## What a stronger future comparison would require

Before collecting new results, predefine a matched baseline/graph protocol:
same public policy/tools, task membership, model/customer/judge settings,
budgets, serving stack, error treatment, and trial scheduling. Freeze both
controllers before evaluating an untouched set. Report failures, uncertainty,
and full denominators for both.

That is a proposed design standard, **not an experiment performed here**.
