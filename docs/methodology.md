# From policy documents to an executable state-graph harness

## 1. What we are trying to build

The objective is to move **decision-making responsibility** out of an
open-ended runtime model and into a policy-derived controller. A frontier
model helps construct and debug the controller offline. A smaller model
interprets narrowly scoped customer language during execution.

This is not just a diagram of the policy. It is an executable specification
of what the agent knows, what it is allowed to do next, what evidence each
action can produce, and when an obligation is actually complete.

The essential invariant is:

> Language interpretation proposes observations. Graph code owns state,
> decisions, authorization, tool arguments, and completion.

The two case studies implement that invariant with typed state and deterministic
hierarchical rules. They do not use a runtime frontier planner, train model
weights, enumerate the entire state space, or demonstrate a general automatic
policy compiler.

### Nodes are situations, not operation names

An operation such as `check_status` is an edge, not a complete node. These two
situations require different next decisions even if both most recently ran
the same operation:

- A requested change was verified, but the final user goal remains untested.
- The user has approved a quote, but the backend operation is still pending.

A useful state must preserve those differences. History that can change the
next action belongs in state: a previous charge, a required restart, the source
of a diagnostic, a consumed approval, or an incomplete readback cannot live
only in the transcript.

### Why an implicit graph

Enumerating every combination of facts, targets, goals, and outstanding work
quickly becomes unwieldy. Instead, define:

```text
S: typed knowledge states
A: parameterized actions and internal transitions
O: permitted observations from each actor
choose(S): deterministic next edge
authorize(S, A): independent dispatch guard
reduce(S, A or O): validated state transition
```

The state schema and rules define the possible graph. A trace records the
subgraph actually visited. Deterministic rules compute the outgoing edge
without building every node beforehand.

BFS or another planner could be added where effects, observability, and
costs are sufficiently explicit. That was not implemented here, and an
unknown observation is not a secretly known successor state.

## 2. Freeze the evidence and authority boundaries

Before reading trajectories, record what the builder and runtime may access.

| Input | Construction use | Runtime use |
|---|---|---|
| Policy and public tool specifications | Extract rules and capabilities | Compile into graph rules and contracts |
| Permitted public implementation/model source | Resolve actual argument, return, and effect semantics | Use normal public tools, not hidden object access |
| Invented fixtures | Exercise public interfaces and edge cases | Tests only |
| Approved training traces/feedback | Diagnose general mechanisms after baseline | Never use task IDs or gold outcomes to select behavior |
| Reserved evaluation data | Keep outside construction/selection | Only the official simulator/environment/evaluator receives its designated inputs |
| Hidden personas, expected actions, evaluator state | Not graph inputs | Never planner context or state initialization |

Record policy precedence. A permissive backend function does not override a
policy prohibition. If a policy cannot be enforced or a required capability
cannot be observed through the public interface, document the gap.

Distinguish three evidence levels:

1. **Policy-text-only:** decisions inferred from policy text alone.
2. **Source-assisted:** policy plus permitted public tools, models, and
   framework behavior.
3. **Training-informed:** revisions also use approved execution feedback.

The case studies are source-assisted, then training-informed. They are not
evidence that arbitrary policies can be compiled from prose alone.

**Deliverable:** a source/access manifest, allowed model roles, declared task
scope, and an unresolved-assumptions list.

## 3. Convert policy into a goal-and-rule ledger

Read the complete policy and interfaces before coding individual procedures.
For every supported goal, define the externally meaningful result and the
evidence required to establish it.

Create one record per rule:

| Field | What to write |
|---|---|
| Policy reference | Clause, precedence, and exact source version |
| Goal or obligation | What must become true, including mandatory follow-up |
| Required facts | Identity, ownership, status, amounts, preferences, eligibility |
| Permitted sources | Customer, authoritative tool, derived calculation, graph bookkeeping |
| Guard | Conditions required before an action |
| Action | Actual available operator, actor, and argument contract |
| Verification | Required result fields, identity checks, readbacks, and outcome tests |
| Alternative exits | Clarify, refuse, unresolved, or verified handoff |
| Invalidation | Which later events make these facts or approval stale |

Separate policy constants from customer preferences and backend facts.
For example, a policy can define an eligibility window, the customer can
state a reason, and a tool can establish the transaction timestamp. The
model should not invent any of these to make the predicate pass.

Define at least three kinds of predicate:

- **Eligibility:** is the proposed action permitted?
- **Effect verification:** did the requested operation actually take effect?
- **Goal completion:** does the verified state meet this customer's objective?

An intermediate improvement need not satisfy the final goal. Successful
escalation, permitted refusal, external customer stop, and solved task are
different outcomes.

**Deliverable:** a traceable rule ledger and explicit goal/terminal predicates.

## 4. Design the smallest sufficient knowledge state

"Complete" means complete for future decisions, not a copy of every historical
message or the hidden environment.

| State family | Required distinction |
|---|---|
| Goals | Stable identity, current target, status, revision, dependencies |
| Identity and ownership | Disclosed identifiers versus verified owned records |
| Preferences | Desired values versus observed current values |
| Backend facts | Validated tool records, not customer assertions |
| Observations | Value, actor/source, target, request, and relevant revision |
| Unknowns | Unknown versus false; not queried versus queried-and-empty |
| Pending action | Exact operator/question, actor, request ID, target and contract |
| Offer and consent | Complete terms, fingerprint, revision, explicit assent |
| Receipts and obligations | Attempted writes, verified effects, pending readbacks |
| Invalidation | Dependencies affected by target/parameter/effect changes |
| Execution limits | Attempts, substantive progress, remaining dispatch budget |
| Terminal | Completed, refused, handed off, blocked/unresolved, with reason |

Use typed optional fields or explicit unknown states. Never default an unknown
boolean to false merely to simplify conditionals. Likewise, an empty inventory
must say whether the inventory was actually loaded.

An observation should be associated with the request that produced it. A
customer wishing for a result cannot populate the same fact as a measured
result. A previous target's observation cannot silently apply to a new one.

Keep two identities if needed:

- A full canonical state hash for audit/replay.
- A **material-progress key** that ignores counters, timestamps, wording
  changes, and repeated log entries, but retains distinct live obligations.

A growing transcript or retry counter is not progress. Two different pending
orders are not the same obligation merely because their descriptions match.

**Deliverable:** versioned state schemas, provenance rules, invalidation
dependencies, and resume compatibility rules.

## 5. Specify operators before writing the traversal logic

An operator is a contract, not just a tool name.

```yaml
operator: apply_confirmed_change
actor: backend
requires:
  - verified_owned_target
  - policy_eligibility
  - complete_arguments
  - approval_for_exact_offer_revision
on_dispatch:
  - record_pending_request
  - consume_approval
  - mark_affected_observations_stale
on_result:
  - validate_actor_request_target_and_schema
  - distinguish_error_from_verified_effect
  - record_receipt
  - retain_required_readback
retry:
  consequential_write: never_blindly_repeat
```

This is illustrative notation, not a shipped executable format.

For each operator, enumerate useful result branches: positive, negative,
partial, ambiguous, execution error, user refusal, and no usable observation.
Do not equate a **negative diagnostic result** with failure to execute the
diagnostic. "The test found no connection" can be exactly the evidence needed
for the next branch.

Keep the lifecycle explicit:

```text
requested -> dispatched -> reported complete -> effect verified -> goal verified
```

Some steps may share authoritative evidence, but they are not interchangeable.
Sending an instruction cannot assert that the customer executed it.
Sending a write cannot assert its success.

Pending verification belongs to the operation, not just to the currently
selected goal. A new customer request cannot erase a required charge/readback
or make the graph forget an already executed action.

**Deliverable:** an operator catalog with prerequisites, arguments, outcome
branches, invalidation, verification, and retry semantics.

## 6. Build deterministic selection plus independent authorization

Organize reusable procedures around policy subproblems rather than training
task identities. Share prerequisites and verification paths where appropriate.

A useful control order is:

1. Validate and consume the event for the matching pending request.
2. Settle mandatory operation results and verification obligations.
3. Apply accepted customer corrections or goal revisions atomically.
4. Resolve missing identity, target, preferences, or prerequisite evidence.
5. Choose the next permitted diagnostic, explanation, offer, or operation.
6. Independently authorize the complete action at dispatch.
7. Execute one outward action and record the transition.
8. Reach a terminal only when its own predicate holds.

The exact precedence is policy-specific. A pending irreversible action must
not be bypassed by optional new work. Explicit user ordering must not be
silently replaced by the order that happens to suit the evaluator.

Keep `choose` pure: it returns a candidate without changing the node.
Record resource binding, goal splitting, refinement, and completion as actual
internal transitions. Otherwise a trace cannot explain how the agent reached
its next state.

Separate `authorize` from selection. Recheck ownership, eligibility, scope,
argument validity, exact consent, budgets, and duplicate-write rules where
they cross the external-action boundary.

Use explicit terminal categories. A budget limit leads to unresolved, not
success. A handoff is not complete until its actual transfer operator succeeds.

**Deliverable:** deterministic selectors/reducers and independently enforced
dispatch guards.

## 7. Make the small model a narrowly scoped language sensor

For each pending question, construct the exact interpretation job:

```text
Input:
  current customer reply
  exact pending question and permitted fields
  selected read-only context needed to interpret that reply
  bounded output schema

Output:
  proposed values/preferences
  a bounded reply meaning, such as answer/correct/clarify/decline

Forbidden output authority:
  tool selection, plans, backend updates, write permission, terminal success
```

Do not ask the model to regenerate the entire state. A reason-only question
must not secretly request the reservation ID again. A payment reply must not
be rejected because the model was also induced to copy an unrelated order ID.

### Extract words; let code apply fixed meanings

Where the domain has an explicit mapping, extract the literal observed label
and apply the documented meaning deterministically. Do not make the model
re-derive a fixed rule every time.

Reject unknown mappings rather than guessing. If both a literal label and a
claimed interpretation are returned, require agreement before accepting the
observation. Literal presence is still not proof of semantic attribution:
the model must distinguish a current report from a hypothetical or old value.

### Let code attach provenance

Code already knows which message it submitted. Attach that message identity
and request context directly. Asking a model to copy an exact quotation adds
formatting and attribution failures without creating trustworthy evidence.

For user-supplied identifiers, require complete literal tokens in the selected
source. For descriptive references, extract a typed preference and let code
resolve it against authoritative owned records. Do not pretend a backend ID
was supplied by the customer.

### Atomic updates and partial answers

Validate all proposed changes on a copy of the node. A conflicting or
out-of-scope field rejects that update without partially corrupting state.
Preserve previously accepted facts.

Useful partial information is not an error. Merge permitted field-level
patches, determine what remains unknown, and narrow both the next question
and its schema to those fields.

Distinguish ordinary answers, explicit corrections, clarification requests,
new requests, replacements, and withdrawals. A separate correction job may be
needed; log and budget it rather than hiding additional model work.

**Deliverable:** exact per-question schemas, validation, source attribution,
atomic reducers, and explicit ambiguity/recovery paths.

## 8. Treat references, consent, and outgoing language as core semantics

The graph's private argument dictionary is not the customer's interface.
Render every necessary exact value from the selected operator's bound
arguments. Test the text that the customer actually sees.

An explanation must address the obstacle. Repeating an instruction does not
answer a question about a missing identifier, price, source address, or the
consequences of an action.

Bind consent to the complete offer: actor, target, parameters, amount, effect,
and relevant revision. Conditional agreement, a question, or a request to
change the offer is not permission to execute. Changed terms revoke approval;
an old "yes" cannot authorize a new proposal.

For multi-resource domains, treat resolution as a first-class layer:

- Separate a source record from the destination being changed.
- Separate product/variant identity from purchased physical units.
- Require a unique owned match or ask a focused question.
- Preserve unmentioned accepted preferences during partial corrections.
- Invalidate target-specific facts without deleting independent preferences.
- Preserve explicit ordering and unfinished obligations across goal changes.
- Collect the entire scope before a one-shot operation makes later edits
  impossible.
- Compute amounts, allowances, capacities, and totals in code from verified
  records; do not use model-generated arithmetic as authoritative evidence.

These distinctions mattered much more in retail/airline than in telecom.
They are not optional UI polish: they determine which edge is legal.

**Deliverable:** graph-owned templates, scoped offer/consent contracts, and
deterministic reference-resolution rules.

## 9. Integrate, trace, and test the real interface

The adapter must expose only the intended customer messages and public tools.
Do not seed the graph from hidden benchmark state, expected actions, personas,
or task-ID encodings. Validate initialization and serialized-state versions.

Record, for each transition:

```text
before state -> current event -> interpretation/result -> accepted update
             -> selected rule -> authorization -> exact outward action
             -> after state / pending obligation
```

Include rejected interpretations and errors, not just successful edges.
Keep state schemas and operator catalogs alongside traces; a successful
visited path is not coverage of the entire graph.

Construct invented tests from the public interface:

| Test class | Examples |
|---|---|
| Fact meaning | Unknown/false, unloaded/empty, wish/current result |
| Lifecycle | Requested/completed/verified, wrong actor or request ID |
| Negative evidence | Executed diagnostic with an unfavorable result |
| Language | Ellipsis, correction, clarification, partial or contradictory reply |
| Scope | Ambiguous target, source/destination, repeated physical units |
| Consent | Conditional assent, stale offer, target/amount changes |
| Persistence | New goal while readback is pending; incompatible saved schema |
| Boundaries | Just below/at/above policy thresholds and documented time limits |
| Progress | Repeated question with growing logs but unchanged material state |
| Integration | Combined problems, policy refusal, and actual transfer failure |

A fake customer must not read private argument dictionaries that the real
customer cannot see. Fixtures that supply perfect abstract booleans can hide
a broken real interface that emits raw names.

Freeze the proposal, implementation, and tests **before** running checks.
Record failures and commit repairs before retesting. Then, when model calls
are approved, use a small fixed set of invented live checks before scoring.
The retail/airline project used 36 checks: 12 mechanisms x 3 paraphrases.

Passing a fixed smoke suite is a gate, not proof of benchmark performance.
Retail/airline pass 7 passed all 36 live checks and still regressed.

**Deliverable:** an immutable baseline, interface-faithful tests, a model-free
example, fixed live checks, and a frozen execution protocol.

## 10. Use training runs to find missing distinctions

Run the approved training cohort with the frozen baseline. Preserve every
planned, attempted, scored, failed, and unscored episode.

For each representative failure, reconstruct the **first consequential
divergence**, not just the final blocked message:

```text
last justified state
  -> chosen request
  -> actual response
  -> interpreted observation
  -> accepted/rejected update
  -> next decision
  -> eventual outcome
```

Classify the mechanism before changing the architecture:

| Mechanism | First repair to investigate |
|---|---|
| A wish becomes success | Separate intake from measured outcome and provenance |
| Negative test triggers blockage | Separate diagnostic result from execution failure |
| Correct reply repeatedly rejected | Align question, schema, source rules, and allowed fields |
| Exact value never reaches customer | Render the bound operator argument |
| Right operation, wrong record | Strengthen target/source/unit resolution |
| Completed action, unsolved objective | Add the missing verification or goal distinction |
| New request erases pending work | Preserve operation-level obligations |
| Clarification causes an endless loop | Add an explicit recovery transition, not unlimited retries |
| No public fact/action exists | Document capability gap; do not invent hidden access |
| Missing response or transport error | Diagnose the serving role and actual failure class |

Use a repair record:

```text
observed evidence
first broken contract or missing state distinction
portable rule change
invented regression that fails under the old behavior
invariants preserved
expected effect and possible regressions
new frozen run and paired observed outcome
```

Prefer one missing field, guard, invalidation rule, resolution contract, or
template over an exception for a benchmark task. Inspect successes too:
a helpful simulated customer may have improvised around an incomplete
instruction.

Evaluate a coherent revision on the same cohort. Count gains **and**
regressions. Bundled changes with stochastic conversations are evidence
about that bundle, not causal ablations of each edit.

Do not reflexively add a stronger planner, more retries, or weaker success
criteria. First establish whether the information contract is broken.

**Deliverable:** mechanism-level failure ledger, prospective revision,
focused regressions, and a complete paired training report.

## 11. Freeze evaluation and report what actually happened

Freeze code, policies, prompts, tool schemas, task membership, trial schedule,
model roles, settings, budgets, scoring, and recovery rules. Seeds for task
selection or scheduling do not make unseeded model generations deterministic.

Report separately:

- Planned, attempted, scored, successful, failed, and unscored episodes.
- Official reward, internal terminal, and external customer stop.
- Repeated-trial mean, at-least-one success, and all-trials success.
- Initial baseline, best observed candidate, and latest candidate.
- Training, untouched held-out evaluation, and later known-test reruns.
- Graph changes versus model, serving, concurrency, or timeout changes.
- Actual calls by role, retries, latency, and available usage.

Recover only eligible infrastructure failures under the declared procedure.
Preserve originals and valid zeros. Empty or malformed HTTP-200 model output
is not automatically a disconnection. Never rerun hard tasks until they pass.

Check the observable model contract for every role, including the customer
and judge. A wrapper's echoed label does not independently attest upstream
weights. Inspect which parameters and roles the serving adapter actually
forwards.

If test failures have been inspected, acknowledge that later work is no longer
blind to that test. Do not infer cost savings from synthetic usage or occasional
model use from an architecture that removes model planning.

Release the procedure, evidence, boundaries, and known limitations, not just a
high score. A fully runnable release would additionally need the actual frozen
controller, dependencies, and permitted reproduction inputs.

See [measurement rules](measurement.md) and
[this release's reproduction scope](reproduction.md).

## When a bounded reasoning subgraph might be justified

Some policies require genuine conditional or combinatorial planning that
cannot be reduced to a missing field or a better lookup. First identify the
specific gap and the evidence that deterministic procedures cannot handle it.

A future model-assisted subgraph should have explicit input/output contracts,
permitted actions, budgets, state effects, independent guards, and verification.
Its reasoning and tool authority must be disclosed and measured. It cannot be
an unrestricted agent hidden behind a nominally graph-controlled edge.

Neither case study added such a runtime planner. The remaining multi-goal
retail/airline failures are limits of the measured method, not evidence that
this extension has already solved them.
