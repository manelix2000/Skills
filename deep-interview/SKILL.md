---
name: deep-interview
description: Socratic requirements interview for turning Jira tickets into execution-ready specs
argument-hint: "[--quick|--standard|--deep] <jira ticket or vague request>"
version: 2.0-portable
type: conversational-skill

input_schema:
  ticket_id: string
  title: string
  description: string
  comments: [string]
  reporter: string
  labels: [string]
  attachments_summary: [string]

---

<Purpose>
Deep Interview is an intent-first Socratic clarification skill. It turns Jira tickets, vague requests, or ideas into execution-ready specifications by asking one high-leverage question per round, reducing ambiguity, and forcing explicit boundaries before implementation begins.
</Purpose>

<Use_When>
- The request is broad, ambiguous, or missing acceptance criteria
- The input is a Jira ticket, issue, request, or task description that does not clearly define what should be built
- The host can provide structured Jira fields such as title, description, comments, labels, and reporter
- The user wants to avoid incorrect implementation caused by underspecified requirements
- The user says things like "interview me", "clarify this", "don't assume", or "help me define what needs to be done"
- A requirements artifact is needed before planning or implementation
</Use_When>

<Do_Not_Use_When>
- The request already names concrete targets and has clear acceptance criteria
- The user explicitly asks to skip clarification and execute immediately
- The user only wants lightweight brainstorming
- A full approved spec already exists and work should start directly
</Do_Not_Use_When>

<Why_This_Exists>
Execution quality is usually limited more by intent ambiguity than by implementation detail. A single expansion pass often misses why the user wants the change, where scope should stop, what must stay out of scope, and which decisions can be made without asking again. This skill applies Socratic pressure plus quantitative ambiguity scoring so execution starts from an explicit, testable, intent-aligned spec.
</Why_This_Exists>

<Depth_Profiles>
- Quick (`--quick`): fast clarification pass; target threshold `<= 0.30`; max rounds 5
- Standard (`--standard`, default): full requirement interview; target threshold `<= 0.20`; max rounds 12
- Deep (`--deep`): high-rigor clarification; target threshold `<= 0.15`; max rounds 20

If no flag is provided, use Standard.
</Depth_Profiles>

<Execution_Policy>
- Ask exactly ONE question per round
- Ask about intent and boundaries before implementation detail
- Target the weakest clarity dimension each round, respecting the stage priority rules
- Treat every answer as a claim to pressure-test before moving on
- Prefer one layer deeper on the same thread over premature topic rotation
- Before crystallizing, complete at least one explicit pressure pass that revisits an earlier answer with a deeper evidence, assumption, or tradeoff follow-up
- Reduce user effort: ask only the highest-leverage unresolved question
- Re-score ambiguity after each answer and show progress transparently
- Do not crystallize while `Non-goals` or `Decision Boundaries` remain unresolved
- Do not hand off to implementation while ambiguity remains above threshold unless the user explicitly chooses to proceed with warning
- Treat early exit as a safety valve, not the default success path
</Execution_Policy>

<State_Model>
The host environment must persist interview state between rounds. The persistence mechanism is implementation-specific and may be memory, JSON, database, or session storage.

Minimum state shape:

```json
{
  "active": true,
  "skill": "deep-interview",
  "interview_id": "<uuid>",
  "profile": "quick|standard|deep",
  "initial_input": "<ticket or request>",
  "rounds": [],
  "scores": {
    "intent": 0.0,
    "outcome": 0.0,
    "scope": 0.0,
    "constraints": 0.0,
    "success": 0.0,
    "context": 0.0
  },
  "non_goals_explicit": false,
  "decision_boundaries_explicit": false,
  "pressure_pass_complete": false,
  "current_ambiguity": 1.0,
  "threshold": 0.20,
  "max_rounds": 12,
  "current_stage": "intent-first",
  "current_focus": "intent",
  "context_snapshot": {
    "task_statement": "",
    "desired_outcome": "",
    "stated_solution": "",
    "probable_intent_hypothesis": "",
    "known_facts": [],
    "constraints": [],
    "unknowns": [],
    "decision_boundary_unknowns": []
  },
  "final_spec": null
}
```
</State_Model>

<Steps>

## Phase 0: Preflight Context Intake

1. Parse the input and derive a short task label.
2. If structured Jira fields are available, normalize title, description, comments, labels, reporter, and attachment summaries into a single intake model.
3. Create a minimum context snapshot with:
   - Task statement
   - Desired outcome
   - Stated solution, if any
   - Probable intent hypothesis
   - Known facts/evidence
   - Constraints
   - Unknowns/open questions
   - Decision-boundary unknowns
4. Persist the snapshot in the host environment.
5. Initialize interview state from the chosen depth profile.

## Phase 1: Initialize

1. Parse profile (`--quick|--standard|--deep`).
2. Set threshold and max rounds:
   - Quick → threshold 0.30, max rounds 5
   - Standard → threshold 0.20, max rounds 12
   - Deep → threshold 0.15, max rounds 20
3. Initialize state with:
   - current ambiguity = 1.0
   - rounds = empty
   - readiness gates unresolved
4. Announce kickoff with profile, threshold, and current ambiguity.

## Phase 2: Socratic Interview Loop

Repeat until:
- ambiguity `<= threshold`
- readiness gates are satisfied
- pressure pass is complete
- the user exits with warning
- or max rounds is reached

### 2a) Generate the next question

Use:
- original input
- prior Q&A rounds
- current scores
- unresolved readiness gates

Stage priority:
- Stage 1 — Intent-first:
  - Intent
  - Outcome
  - Scope
  - Non-goals
  - Decision Boundaries
- Stage 2 — Feasibility:
  - Constraints
  - Success Criteria
- Stage 3 — Context grounding:
  - Existing process/system context, only if relevant to the task

Follow-up pressure ladder after each answer:
1. Ask for a concrete example, counterexample, or evidence signal
2. Probe the hidden assumption, dependency, or belief
3. Force a boundary or tradeoff: what should explicitly not be done, deferred, or rejected?
4. If the answer describes symptoms only, reframe toward root cause before moving on

Prefer staying on the same thread when it still has the highest leverage.

Detailed dimensions:
- Intent Clarity — why this is wanted
- Outcome Clarity — what end state is desired
- Scope Clarity — how far the change should go
- Constraint Clarity — what limits must hold
- Success Criteria Clarity — how completion will be judged
- Context Clarity — relevant operational or system context, if needed

### 2b) Ask the question

Output format for each round:

Round {n} | Target: {weakest_dimension} | Ambiguity: {score}%

{question}

### 2c) Score ambiguity

Score each dimension in `[0.0, 1.0]` with:
- score
- brief justification
- remaining gap

Default ambiguity formula:

`ambiguity = 1 - (intent × 0.30 + outcome × 0.25 + scope × 0.20 + constraints × 0.15 + success × 0.10)`

Optional contextual variant when context is materially relevant:

`ambiguity = 1 - (intent × 0.25 + outcome × 0.20 + scope × 0.20 + constraints × 0.15 + success × 0.10 + context × 0.10)`

Readiness gates:
- `Non-goals` must be explicit
- `Decision Boundaries` must be explicit
- A pressure pass must be complete

Even if weighted ambiguity is below threshold, continue interviewing until all readiness gates are satisfied.

### 2d) Report progress

After each answer, show:
- weighted breakdown table
- readiness-gate status
- next focus dimension

### 2e) Persist state

Append the round:
- question
- answer
- target dimension
- updated scores
- updated ambiguity
- readiness gate status

### 2f) Round controls

- Do not offer early exit before:
  - at least one explicit assumption probe
  - and one persistent follow-up on a previous answer
- Round 4+: allow explicit proceed-with-warning
- Soft warning at profile midpoint
- Hard stop at max rounds

## Phase 3: Challenge Modes

Use each mode once when applicable:

- Contrarian:
  Challenge a core assumption when an answer rests on an untested belief
- Simplifier:
  Probe the minimum viable scope when scope expands faster than outcome clarity
- Ontologist:
  Ask for an essence-level reframing when the user keeps describing symptoms instead of the real problem

Track which modes have been used to avoid repetition.

## Phase 4: Crystallize the Spec

When threshold is met, or the user exits with warning, or max rounds is reached, generate a final specification.

The final spec must include:

- Metadata
  - profile
  - rounds used
  - final ambiguity
  - threshold
- Clarity breakdown table
- Intent
- Desired Outcome
- In Scope
- Out of Scope / Non-goals
- Decision Boundaries
- Constraints
- Testable Acceptance Criteria
- Exposed Assumptions and Resolutions
- Pressure-pass Findings
- Open Risks / Residual Ambiguity
- Transcript Summary

Recommended output formats:
- Markdown
- JSON

## Phase 5: Return Execution Readiness

Return one of:

- `ready_for_implementation`
- `needs_more_clarification`
- `proceed_with_warning`

If not ready, indicate the highest-leverage unresolved gap.
</Steps>

<Output_Contracts>

## Turn Output
For every interview round, return:

```json
{
  "type": "question",
  "round": 1,
  "target_dimension": "intent",
  "ambiguity": 0.84,
  "question": "What problem do you actually want this change to solve for the user or the team?",
  "score_breakdown": {
    "intent": 0.2,
    "outcome": 0.3,
    "scope": 0.1,
    "constraints": 0.0,
    "success": 0.0
  },
  "readiness": {
    "non_goals_explicit": false,
    "decision_boundaries_explicit": false,
    "pressure_pass_complete": false
  },
  "next_focus": "intent"
}
```

## Final Output
At crystallization time, return:

```json
{
  "type": "final_spec",
  "status": "ready_for_implementation|needs_more_clarification|proceed_with_warning",
  "final_ambiguity": 0.18,
  "threshold": 0.20,
  "rounds_used": 7,
  "readiness": {
    "non_goals_explicit": true,
    "decision_boundaries_explicit": true,
    "pressure_pass_complete": true
  },
  "spec_markdown": "<execution-ready markdown spec>",
  "spec_json": {
    "intent": "",
    "desired_outcome": "",
    "in_scope": [],
    "out_of_scope": [],
    "decision_boundaries": [],
    "constraints": [],
    "acceptance_criteria": [],
    "assumptions_and_resolutions": [],
    "pressure_pass_findings": [],
    "open_risks": []
  }
}
```
</Output_Contracts>

<Stop_Conditions>
- User says stop, cancel, abort → persist state and stop
- Ambiguity stalls for 3 rounds within ±0.05 → force Ontologist mode once
- Max rounds reached → crystallize with explicit residual-risk warning
- All dimensions >= 0.9 and readiness gates are satisfied → allow early crystallization
</Stop_Conditions>


<Jira_Input_Mapping>
When the host provides Jira-native fields, map them into the interview context as follows:

- `title` → initial task statement
- `description` → primary problem statement and candidate scope signals
- `comments` → supporting evidence, edge cases, and conflicting interpretations
- `reporter` → source of the request and possible intent signal
- `labels` → taxonomy, domain hints, and routing clues
- `attachments_summary` → supplemental evidence or specification fragments

The skill should treat Jira content as a starting hypothesis, not as a complete source of truth. Missing intent, non-goals, decision boundaries, and acceptance criteria must still be clarified through interview rounds.
</Jira_Input_Mapping>

<Host_Requirements>
The host environment must provide:
- a way to receive user replies
- a way to persist state between turns
- a way to render structured progress after each round
- a way to emit final markdown and/or JSON artifacts

The host environment does NOT need:
- repository exploration
- OMX command handoffs
- filesystem conventions
- runtime-specific tool names
</Host_Requirements>

<Final_Checklist>
- [ ] One question per round
- [ ] Intent-first ordering before implementation detail
- [ ] Ambiguity score shown each round
- [ ] Weakest-dimension targeting used within current stage
- [ ] At least one explicit assumption probe happened
- [ ] At least one pressure pass revisited a prior answer
- [ ] `Non-goals` explicitly resolved
- [ ] `Decision Boundaries` explicitly resolved
- [ ] Final spec generated
- [ ] No direct implementation performed inside this skill
</Final_Checklist>
