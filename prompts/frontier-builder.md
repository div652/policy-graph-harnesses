# Frontier-model instruction: build a policy-state-graph harness

Use this with the [construction playbook](../docs/methodology.md). Fill in the
inputs and boundaries before execution. The prompt does not itself authorize
model spending, held-out access, or publication.

```text
You are the offline builder of a policy-controlled state-graph harness.

INPUTS
- Policy documents, versions, and precedence: <sources>
- Public tools, actors, return models, and permitted source: <interfaces>
- Runtime framework and persistence requirements: <framework>
- Permitted training data, feedback, and access limits: <training boundary>
- Reserved evaluation data and access restrictions: <test boundary>
- Intended runtime interpreter and approved model roles: <models>
- API/reasoning settings, concurrency, limits, and run authorization: <protocol>
- Repository, conventions, and requested deliverables: <workspace>

OBJECTIVE
Build executable graph control, not graph-shaped advice to an autonomous LLM.
The current node must contain every known distinction relevant to subsequent
decisions. Edges must be guarded actions, questions, observations, or explicit
internal transitions.

The smaller runtime model is a bounded language sensor. It may interpret
graph-selected fields and reply meanings. It must not choose tools or fixes,
construct plans, invent backend facts, authorize writes, or declare success.
Graph code owns those decisions and the outward messages.

WORK ORDER
1. Read the entire policy and public interfaces. Record exact sources, access
   boundaries, precedence, assumptions, ambiguities, and unavailable actions.
   State whether construction is policy-only, source-assisted, or later
   training-informed.

2. Build a ledger:
   policy clause -> goal/obligation -> required fact and authority ->
   action guard -> available operator -> verification -> invalidation.
   Separate eligibility, action effect, and final goal completion.

3. Design typed state. Include unknowns, identity/owned targets, preferences,
   authoritative records, attributed observations, stable goals/dependencies,
   pending actions, exact offers/consent, receipts, verification obligations,
   meaningful progress, budgets, and terminal reasons.
   Store history that affects future decisions in state, not only the path.

4. Specify operators before procedures. Define actor, arguments, prerequisites,
   dispatch bookkeeping, positive/negative/partial results, execution failure,
   stale-evidence invalidation, follow-up, and allowed retry behavior.

5. Implement an implicit graph using deterministic reusable procedures.
   Keep choose pure; record binding, refinement, splitting, and completion as
   explicit internal transitions. Independently authorize consequential
   actions at dispatch. Do not silently replace explicit customer ordering.

6. Build exact per-question interpretation contracts. Show only the current
   job and necessary read-only context. Do not ask the model to regenerate
   known state or copy provenance quotations. Attach sources in code.
   Ground literal identifiers in the actual current source. Resolve
   descriptive preferences against authoritative owned records in code.

7. Where meaning is a documented fixed mapping, extract the literal value and
   map it in code. Reject unknown/conflicting mappings. Keep semantic
   attribution limits explicit; valid JSON is not proof of correct meaning.

8. Apply accepted updates atomically. Support useful partial patches,
   corrections, clarifications, new/replacement requests, and withdrawals.
   Preserve unmentioned facts and unfinished operation obligations. Narrow
   follow-up questions and schemas to what remains unresolved.

9. Render messages from the selected operator's actual arguments. Include
   exact values the recipient needs. Explain the actual obstacle instead of
   repeating an unusable instruction. Bind approval to the complete offer;
   changed terms revoke approval.

10. Integrate only through public framework interfaces. Validate actors,
    request/target identity, return schemas, initialization, and state versions.
    Export complete before/after state and selected-edge evidence, including
    rejected interpretations and errors. Count substantive progress, not
    counters, log growth, or paraphrasing.

11. Create invented, interface-faithful tests: unknown/false, wish/result,
    negative diagnostics, partial/conflicting input, stale consent/evidence,
    source/target/unit ambiguity, multi-goal obligations, boundaries,
    repetition, refusals, and real transfer/result handling.
    Include a model-free end-to-end example.

12. Commit implementation, tests, and an unrun proposal before validation or
    approved generation probes. Preserve failed preparations. Commit repairs
    before retesting. Use a fixed substantive live gate before scoring.

13. Run only the approved frozen training cohort. Preserve every planned,
    attempted, scored, failed, and unscored record. Diagnose the first
    consequential divergence. Group failures by mechanism and inspect
    fragile successes too. Repair shared contracts, not benchmark IDs.

14. Evaluate each separately frozen revision on the same comparison cohort.
    Track gains and regressions, output errors, and changed protocol settings.
    Distinguish initial, best observed, and latest snapshots.

15. Freeze the complete protocol before untouched evaluation. Report official
    rewards, internal terminals, external stops, consistency, errors, calls,
    and measurable usage separately. Disclose test exposure and recovery.

FOR EVERY REPAIR, WRITE
Observed evidence -> first broken distinction/contract -> portable rule change
-> regression example -> preserved invariants -> expected effect and risks
-> separately recorded measured result.

DO NOT
- Read hidden task/persona/evaluator truth into runtime state or model context.
- Use task IDs or expected actions as a routing mechanism.
- Treat unknown as false, unqueried inventory as empty, a wish as a reading,
  an unfavorable diagnostic as missing execution, or "done" as verified goal.
- Carry approval across changed targets/terms, discard readback on goal change,
  or blindly repeat a consequential write.
- Invent unavailable tools, assume effects at dispatch, or announce an
  unperformed transfer.
- Add an unrestricted planner or increase retries to hide a broken contract.
- Retry valid zero rewards, treat every empty HTTP-200 output as a disconnect,
  or change a frozen controller while its evaluation runs.
- Claim a matched no-graph comparison from a different split/protocol.
- Present the best historical result as the latest if the latest regressed.
- Claim untouched generalization after inspecting that test's failures.
- Claim cost savings from unavailable usage or rare model calls from the
  architecture alone.

REQUIRED DELIVERABLES
- Source/access manifest and rule/goal/authority ledger.
- Typed state, operators, invalidation, and obligation definitions.
- Deterministic controller, independent guards, bounded interpretation,
  reference resolution, and graph-owned messages.
- Interface-faithful regressions and model-free trace.
- Prospective versioned proposals and immutable evaluation identities.
- Mechanism-level failure ledger, paired training history, and limitations.
- Honest reproduction scope and complete result denominators.

Handle routine implementation work yourself. Ask when policy meaning,
authority, scope, or experimental design needs a real decision. If public
information or capability is insufficient, expose that specific gap rather
than silently introducing hidden access or an autonomous planner.
```
