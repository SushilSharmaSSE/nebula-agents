# Action: Plan Review

## User Intent

Independently review completed `plan.md` output and answer:

```text
Is this plan ready to build?
```

This action is a post-plan readiness audit. It does not replace the approval,
tracker-sync, ontology-sync, or validation gates inside `plan.md`; it gives a
fresh reviewer a focused way to challenge the completed planning package before
`feature.md` or `build.md` starts.

## Agent Flow

```
(Product Manager + Architect + Code Reviewer)
  v [Parallel Readiness Review]
[SELF-REVIEW GATE: reviewers verify evidence-backed findings]
  v
[READINESS GATE: Ready / Conditionally Ready / Not Ready]
  v
Plan Review Complete
```

**Flow Type:** Parallel read-only review with a readiness gate

---

## Reviewer Independence Contract

- Prefer running this action in a fresh session or different coding tool than
  the one that produced the plan.
- Reviewers must inspect source artifacts directly; do not approve from a
  summary, chat transcript, or generated checklist alone.
- The action is read-only except for writing the review report under the
  non-feature evidence run folder.
- Findings route back to `plan.md` or a targeted owning-role rework prompt.
  Reviewers do not silently repair planning artifacts while reviewing.

## Output Location

Write review outputs to the base/manual run evidence path:

```text
{PRODUCT_ROOT}/planning-mds/operations/evidence/runs/{RUN_ID}/
```

Required output:

- `plan-review-report.md`

Recommended supporting outputs when useful:

- `commands.log`
- `artifact-trace.md`
- `gate-decisions.md`

This action must not create or modify a feature evidence package. Feature
evidence packages are owned by `feature.md` / `build.md`.

## Context Files

Load in this order when the work is feature-scoped:

1. `agents/ROUTER.md`
2. `agents/agent-map.yaml`
3. `agents/docs/AGENT-USE.md`
4. `agents/actions/plan-review.md`
5. `agents/actions/plan.md`
6. `agents/actions/feature.md`
7. `{PRODUCT_ROOT}/planning-mds/BLUEPRINT.md`
8. `{PRODUCT_ROOT}/planning-mds/features/F{NNNN}-{slug}/**`
9. `{PRODUCT_ROOT}/planning-mds/knowledge-graph/solution-ontology.yaml`
10. `{PRODUCT_ROOT}/planning-mds/knowledge-graph/canonical-nodes.yaml`
11. `{PRODUCT_ROOT}/planning-mds/knowledge-graph/feature-mappings.yaml`
12. `{PRODUCT_ROOT}/planning-mds/knowledge-graph/code-index.yaml`
13. `{PRODUCT_ROOT}/planning-mds/knowledge-graph/coverage-report.yaml`

## On-Demand Paths

- `{PRODUCT_ROOT}/planning-mds/api/<openapi-spec>.yaml`
- `{PRODUCT_ROOT}/planning-mds/schemas/**`
- `{PRODUCT_ROOT}/planning-mds/security/authorization-matrix.md`
- `{PRODUCT_ROOT}/planning-mds/security/policies/policy.csv`
- `{PRODUCT_ROOT}/planning-mds/architecture/**`
- `{PRODUCT_ROOT}/planning-mds/domain/**`
- `agents/<role>/references/**` only after a matching `agents/ROUTER.md` row

## Primary Question

The review must answer whether a competent implementation agent could begin
`feature.md` Step 0 without inventing product rules, architecture decisions,
API contracts, workflow states, authorization rules, or acceptance criteria.

## Forbidden

- Editing plan artifacts, KG artifacts, trackers, stories, contracts, schemas,
  or architecture files during the review.
- Approving based only on the plan action's prior approval tokens.
- Treating lookup/KG mappings as authoritative over raw feature, ADR, schema,
  API, or policy artifacts.
- Requiring `feature-assembly-plan.md` as a plan deliverable. That file belongs
  to `feature.md` Step 0.
- Widening scope outside the requested feature, feature set, or declared plan
  target.
- Downgrading missing build-critical detail to a low-severity documentation
  note.

## Stop Conditions

- The target plan scope cannot be identified.
- Required raw artifacts are missing and cannot be located from trackers or KG.
- `python3 {PRODUCT_ROOT}/scripts/kg/lookup.py <feature-id>` returns only
  ambiguous or low-confidence matches for a declared in-scope feature that the
  review must assess.
- Reviewers cannot cite concrete files/sections for their readiness decision.

---

## Execution Steps

### Step 1: Parallel Readiness Review

Execute these review roles in parallel.

#### 1a. Product Manager Readiness Review

1. Activate Product Manager agent by reading `agents/product-manager/SKILL.md`.
2. Review requirement artifacts:
   - feature `PRD.md`, `README.md`, `STATUS.md`, `GETTING-STARTED.md`
   - colocated story files
   - `REGISTRY.md`, `ROADMAP.md`, `STORY-INDEX.md` when present
   - relevant `BLUEPRINT.md` sections
3. Check product readiness:
   - [ ] Every feature has clear user value and explicit non-goals.
   - [ ] Every story has specific, testable acceptance criteria.
   - [ ] Mutation stories name entry points, editable/read-only states,
         persistence evidence, roles, lifecycle constraints, validation
         failures, and audit/timeline expectations.
   - [ ] "Display or capture", "view or edit", and "manage" language cannot be
         satisfied by read-only rendering when a write path is intended.
   - [ ] UI-bearing scope has screen responsibilities and ASCII layouts, or a
         written "No UI" justification.
   - [ ] Personas, workflow goals, and priorities are consistent.
   - [ ] No TODOs, placeholders, or vague words remain in build-critical areas.
   - [ ] No invented business rules appear without traceable user need or
         explicit assumption.
4. Record findings with severity and file/section references.

#### 1b. Architect Readiness Review

1. Activate Architect agent by reading `agents/architect/SKILL.md`.
2. Review architecture artifacts:
   - `BLUEPRINT.md` architecture sections
   - `planning-mds/architecture/**`
   - relevant API contracts and schemas
   - authorization matrix / policy artifacts
   - KG artifacts and feature mappings
3. Check build readiness:
   - [ ] Architecture can satisfy every in-scope story without new decisions.
   - [ ] API contracts or explicit no-API justification exist for story-driven
         backend behavior.
   - [ ] Data model, workflow states, and lifecycle transitions support the
         requirements.
   - [ ] Authorization model names resources, actions, roles, and constraints
         needed by the stories.
   - [ ] NFRs are measurable enough for implementation and testing.
   - [ ] ADRs exist for consequential decisions.
   - [ ] KG feature mappings and canonical bindings reflect the raw artifacts.
   - [ ] `feature-assembly-plan.md` can be produced from the available plan
         artifacts without guessing.
4. Record findings with severity and file/section references.

#### 1c. Code Reviewer Buildability Challenge

1. Activate Code Reviewer agent by reading `agents/code-reviewer/SKILL.md`.
2. Review the plan as an implementation handoff, not as code.
3. Check implementation readiness:
   - [ ] Feature is small enough to build as a vertical slice.
   - [ ] Backend, frontend, AI, QE, and DevOps responsibilities are inferable
         from the plan without role conflict.
   - [ ] Acceptance criteria can be mapped to unit, integration, component,
         E2E, security, and deployability checks.
   - [ ] Planned changes have enough file/API/schema direction to avoid broad
         code search or speculative implementation.
   - [ ] Known dependencies, sequencing constraints, and cross-feature impacts
         are explicit.
   - [ ] Risky or high-blast-radius nodes are identified for additional review.
4. Record findings with severity and file/section references.

### Step 2: Required Validation Commands

Run applicable commands and cite the result in `plan-review-report.md`:

1. `python3 agents/product-manager/scripts/validate-stories.py {FEATURE_PATH}`
2. `python3 agents/product-manager/scripts/validate-trackers.py`
3. `python3 {PRODUCT_ROOT}/scripts/kg/validate.py`
4. `python3 {PRODUCT_ROOT}/scripts/kg/validate.py --check-drift`
5. `python3 agents/scripts/validate_templates.py`

If a command is not applicable, record why. If a command cannot run, record the
failure, likely cause, and whether it affects the readiness decision.

### Step 3: SELF-REVIEW GATE

Each reviewer checks their section before the readiness decision:

- [ ] Findings cite exact files/sections.
- [ ] Severity matches the impact on build readiness.
- [ ] No generic best-practice findings without plan-specific evidence.
- [ ] No hidden fixes were made during review.
- [ ] Any skipped command or artifact has an explicit justification.

### Step 4: READINESS GATE

Compute readiness from combined findings:

```text
IF any critical finding:
  STATUS: NOT READY
  NEXT: repair via plan.md or owning-role rework, then rerun plan-review.md

ELSE IF any high finding:
  STATUS: CONDITIONALLY READY
  NEXT: fix high findings before feature.md, or capture explicit user risk
        acceptance with owner and target date

ELSE:
  STATUS: READY
  NEXT: start feature.md Step 0
```

Machine-readable gate state:

```json
{
  "gate": "plan_review",
  "question": "is_this_plan_ready_to_build",
  "status": "ready | conditionally_ready | not_ready",
  "findings": {
    "critical": 0,
    "high": 0,
    "medium": 0,
    "low": 0
  },
  "can_start_feature_action": true,
  "requires_risk_acceptance": false,
  "available_actions": ["start_feature", "fix_findings", "accept_risk", "cancel"]
}
```

### Step 5: Produce Plan Review Report

Use this structure:

```markdown
# Plan Review Report

Scope: <feature / feature set / planning target>
Run ID: <RUN_ID>
Date: YYYY-MM-DD
Review Question: Is this plan ready to build?

## Decision
- Status: READY / CONDITIONALLY READY / NOT READY
- Rationale: <short evidence-backed rationale>
- Next Action: <start feature.md / repair plan / accept risk>

## Findings By Severity

### Critical
- [critical] <finding> - Location: <file:section>; Impact: <why build is blocked>; Owner: <role>; Recommendation: <fix>

### High
- [high] <finding> - Location: <file:section>; Impact: <risk>; Owner: <role>; Recommendation: <fix or risk acceptance>

### Medium
- [medium] <finding> - Location: <file:section>; Recommendation: <fix or defer>

### Low
- [low] <finding> - Location: <file:section>; Recommendation: <optional improvement>

## Product Readiness
- Requirements quality:
- Story testability:
- Mutation contracts:
- UI/screen readiness:
- Tracker state:

## Architecture Readiness
- API/schema readiness:
- Data/workflow readiness:
- Authorization readiness:
- ADR and NFR readiness:
- KG/ontology alignment:

## Buildability Challenge
- Vertical slice size:
- Role handoffs:
- Testability:
- Dependency and sequencing clarity:
- Risk hotspots:

## Validation Evidence
- <command>: PASS/FAIL/SKIPPED - <notes>

## Artifact Trace
- <artifact path>: <why reviewed>
```

---

## Completion Criteria

- [ ] `plan-review-report.md` exists under the base run evidence path.
- [ ] Readiness decision answers "Is this plan ready to build?"
- [ ] Findings are severity-ranked and cite concrete files/sections.
- [ ] Required validation commands ran or have explicit skip/failure notes.
- [ ] No plan or product artifacts were edited by the review action.

## Prerequisites

- [ ] `plan.md` has completed for the target scope.
- [ ] Phase A and Phase B approval decisions are recorded.
- [ ] Tracker sync and ontology sync have completed or their failures are in
      scope for this review.
- [ ] User has identified the feature, feature set, or planning target to
      review.

## Related Actions

- **Before:** [plan action](./plan.md) - Produce requirements and architecture.
- **Alternative:** [validate action](./validate.md) - Broad artifact alignment.
- **After Passing:** [feature action](./feature.md) - Build a vertical slice.
- **After Findings:** Return to [plan action](./plan.md) or direct owning-role
  rework for targeted repairs.
