---
name: eval-investigation
description: Perform an autonomous, evidence-backed root-cause investigation of AI-agent evaluation failures. Determines what is failing, how often, whether it's a real agent failure, which system layer owns the behavior, and whether the grader/dataset/tool/prompt/agent is actually responsible — instead of assuming a low score means the agent is wrong.
---

# Eval Investigation Skill

## Purpose

Perform an autonomous, evidence-backed root-cause investigation of AI-agent evaluation failures.

The goal is not merely to explain low scores.

Determine:

- what is failing
- how often it fails
- whether it is a real agent failure
- where the behavior originates
- which system layer owns the behavior
- whether the behavior is already implemented elsewhere
- whether the grader is evaluating the correct behavior/layer
- whether the grader, dataset, tool, prompt, or agent is actually responsible
- what should be changed

Behave like a senior AI evaluation engineer debugging a production agent system.

---

# Core Principle

A low evaluation score is an observation, NOT ground truth.

Never assume:

low score → agent failure

Possible root causes include:

- AGENT_PROMPT
- AGENT_BEHAVIOR
- AGENT_IMPLEMENTATION
- TOOL_OR_DATA
- GRADER_IMPLEMENTATION
- GRADER_DESIGN
- GRADER_RUBRIC
- GRADER_RUBRIC_MISALIGNMENT
- GRADER_OBSERVABILITY_GAP
- REFERENCE_OR_DATASET
- INFRASTRUCTURE
- INCONCLUSIVE

The evaluator itself is part of the system being investigated.

---

# Autonomy

When given a request such as:

"Investigate keyword_overlap"

perform the complete investigation autonomously.

Do not ask the user whether you should:

- inspect traces
- inspect passing examples
- inspect GitHub
- inspect grader implementation
- inspect agent prompts
- inspect tools
- inspect datasets
- follow code references
- continue investigating

If available tools can provide evidence, use them.

Only ask the user when required information genuinely cannot be
obtained using available tools.

---

# Investigation Workflow

## Step 1 — Understand the Evaluator

Use the evaluation/observability system to retrieve:

- evaluator name
- score type
- available scores
- score distribution
- score comments/reasons
- number of evaluated traces
- evaluator metadata
- dataset/run metadata when available

Determine whether the evaluator appears to be:

- deterministic/code-based
- LLM-as-a-judge
- human-generated
- unknown

Do not infer implementation solely from the evaluator's name.

---

## Step 2 — Understand the Grader's Claim

For low-scoring cases determine:

- what criterion failed
- what feedback was provided
- what expected/reference information exists
- what behavior the grader expected

Express this explicitly:

GRADER EXPECTATION:
<behavior being evaluated>

Do not yet assume the expectation itself is correct.

---

## Step 3 — Build Evaluation Cohorts

Separate traces into:

### FAILING
Clearly low-scoring traces.

### PARTIAL
Intermediate or ambiguous scores.

### PASSING / CONTROL
High-scoring traces.

Passing traces are important.

Never diagnose systematic failures using failing traces alone when
passing controls are available.

---

## Step 4 — Reconstruct Executions

For relevant traces retrieve as much as available:

- trace ID
- user input
- system/developer instructions
- final response
- model calls
- tool calls
- tool arguments
- tool outputs
- errors
- retries
- relevant state/context
- grader score
- grader feedback
- expected/reference answer
- expected keywords/labels
- metadata

Reconstruct the execution sequence.

Determine where failing behavior begins to diverge from passing
behavior.

---

## Step 5 — Diagnose Individual Failures

For every failing trace analyzed, construct a diagnosis:

{
  "trace_id": "...",
  "score": 0.0,
  "grader_expectation": "...",
  "observed_behavior": "...",
  "primary_root_cause": "...",
  "failure_mode": "...",
  "failure_stage": "...",
  "confidence": 0.0,
  "runtime_evidence": "...",
  "code_evidence": "...",
  "counter_evidence": "...",
  "suggested_fix": "..."
}

Use descriptive failure modes.

Good:

- valid_paraphrase_penalized
- empty_result_triggers_wrong_fallback
- approval_enforced_outside_grader_visibility
- agent_stops_after_recoverable_error
- reference_requires_unrequested_information
- grader_penalizes_formatting_difference
- wrong_tool_selected_after_partial_result

Avoid vague labels such as:

- bad_answer
- grader_problem
- failed_eval

---

# Source-Code Investigation

Trace evidence alone may not explain WHY behavior occurred.

When implementation evidence could resolve uncertainty, inspect the
connected source repository automatically.

---

## Step 6 — Inspect Grader Implementation

Search using:

- evaluator name
- score name
- evaluator comments
- expected/reference field names
- rubric text

Locate:

- grader implementation
- scoring logic
- evaluator prompt/rubric
- normalization/parsing
- expected/reference data
- evaluator configuration

Determine:

1. What does the grader actually measure?
2. How is the score calculated?
3. Is implementation consistent with its specification?
4. What information does it observe?
5. What information can it NOT observe?

Never claim implementation behavior without inspecting supporting code
when repository access is available.

---

## Step 7 — Inspect Dataset / Reference Data

Locate when relevant:

- expectedKeywords
- expectedOutput
- idealAnswer
- referenceAnswer
- labels
- fixtures
- dataset construction

Determine whether the evaluation target itself is:

- correct
- ambiguous
- overly specific
- outdated
- incomplete
- stylistic rather than semantic

Do not guess hidden reference values if they can be found in source
code.

---

## Step 8 — Inspect Agent Prompt

Locate:

- system prompt
- developer instructions
- agent configuration
- few-shot examples
- scope instructions
- fallback instructions
- tool-selection guidance

Compare agent behavior against actual instructions.

Do not recommend prompt changes before verifying what the prompt
currently says.

---

## Step 9 — Inspect Relevant Tools

For tools involved in important failures inspect:

- tool description
- schema
- implementation
- return format
- error handling
- fallback logic
- wrappers

Determine whether apparent agent failures actually originate from tool
behavior or misleading tool descriptions.

---

# Deep Code-Path Investigation

## Step 10 — Follow the Full Behavior Path

Do NOT stop after finding the first relevant file.

Follow references across the system.

Potential path:

User
→ Router
→ Agent configuration
→ Prompt
→ Model
→ Tool definition
→ Tool wrapper
→ Tool implementation
→ Middleware
→ Approval layer
→ Runtime/harness
→ Post-processing
→ UI/backend
→ Final response
→ Evaluator

If:

agent.ts → tools.ts

inspect tools.ts.

If:

tools.ts → approvalManager

inspect approvalManager.

If middleware transforms results, inspect it.

Follow imports, callers, callees, configuration and registration logic
until responsibility for the behavior is understood.

---

# Responsibility Mapping

## Step 11 — Determine Who Actually Owns the Behavior

For every major grader criterion determine:

### Grader Expectation
What exactly does the evaluator expect?

### Intended Owner
Which component SHOULD own that behavior?

Possible owners:

- agent prompt
- agent reasoning
- agent implementation
- tool
- tool wrapper
- middleware
- router
- approval gate
- runtime/harness
- backend
- frontend/UI
- evaluator

### Actual Owner
Where is the behavior actually implemented?

### Runtime Evidence
What actually happened?

### Grader Visibility
Can the grader observe the layer where the behavior occurs?

### Evaluation Mismatch
Is the evaluator checking the wrong layer or representation?

This responsibility mapping is mandatory for major findings.

---

# Check Whether the "Missing" Behavior Exists Elsewhere

## Step 12 — Search Before Declaring Something Missing

When a grader claims behavior is absent, do NOT immediately recommend
adding it to the agent prompt.

Search the relevant code path.

Determine whether the behavior is:

- genuinely missing
- implemented elsewhere
- enforced by middleware
- enforced by tool wrappers
- enforced by runtime/harness
- handled by routing
- handled by UI
- intentionally delegated to another component
- impossible for the evaluated component to control

Example:

Grader says:

"Agent failed to request approval before destructive action."

Trace shows:

agent → delete_user()

Do not immediately classify AGENT_PROMPT.

Search:

- delete_user
- approval
- confirmation
- requires_approval
- middleware
- tool configuration
- runtime configuration

If execution is already intercepted by a human approval gate, classify
appropriately as:

GRADER_OBSERVABILITY_GAP

and/or:

GRADER_RUBRIC_MISALIGNMENT

The correct fix may be the grader, not the agent.

---

# Understand Intended Architecture

## Step 13 — Determine How the Application Is Supposed to Work

When necessary inspect:

- README
- architecture documentation
- comments
- configuration
- tool registration
- agent initialization
- middleware
- evaluation documentation

Understand the intended division of responsibility before recommending
changes.

Do not move behavior into the agent prompt merely because the grader
cannot see where the architecture currently implements it.

---

# Hypothesis-Driven Investigation

## Step 14 — Form Competing Hypotheses

Do not stop at the first plausible explanation.

For important failures create multiple possible explanations.

Example:

Agent receives score 0 despite apparently correct response.

Possible hypotheses:

H1: Agent omitted important information.

H2: Reference data is incorrect.

H3: Grader implementation contains a bug.

H4: Grader implementation is correct but design is brittle.

H5: Grader cannot observe where required behavior occurs.

H6: Score is associated with incorrect trace.

H7: Agent prompt caused behavior.

H8: Tool output caused behavior.

Use runtime and implementation evidence to support or eliminate
hypotheses.

---

# Passing-vs-Failing Analysis

## Step 15 — Find Systematic Differences

Compare failing traces against passing controls.

Look for patterns involving:

- tool choice
- tool sequence
- empty results
- errors
- retries
- fallback paths
- request category
- conversation state
- response formatting
- wording
- refusal behavior
- routing
- dataset category

Prefer evidence such as:

"18/22 failing traces take path X, compared with 2/31 passing traces."

over:

"Some failures take path X."

Never invent counts.

---

# Failure Clustering

## Step 16 — Group Recurring Failures

Cluster individual diagnoses into meaningful failure modes.

For each cluster determine:

- failure mode
- root-cause category
- affected trace count
- percentage of failures
- representative trace IDs
- confidence
- runtime evidence
- code evidence
- responsible system layer
- recommended fix

Do not claim prevalence from a small sample unless validated against the
broader dataset.

---

# Grader Audit

## Step 17 — Evaluate Grader Health

Explicitly assess:

### Implementation
Does it behave according to specification?

### Design
Does what it measures correspond to actual agent quality?

### Rubric
Is it clear, appropriate and sufficiently flexible?

### Observability
Can it observe the behavior it expects?

### Reference Data
Are expected answers/keywords/labels appropriate?

### Consistency
Are semantically similar responses treated consistently?

Distinguish carefully between:

- GRADER_IMPLEMENTATION
- GRADER_DESIGN
- GRADER_RUBRIC
- GRADER_RUBRIC_MISALIGNMENT
- GRADER_OBSERVABILITY_GAP
- REFERENCE_OR_DATASET

---

# Agent Audit

## Step 18 — Evaluate Actual Agent Health

Separately inspect:

- prompt quality
- instruction clarity
- tool selection
- tool descriptions
- fallback behavior
- error handling
- retry behavior
- hallucinations
- premature termination
- scope adherence
- handling of missing/empty data

Do not modify an otherwise-correct agent merely to game a poorly
designed evaluator.

Bad recommendation:

"Add the evaluator's expected keywords to the system prompt."

Better recommendation:

"The response is semantically correct. Fix the evaluator rather than
optimizing production behavior for lexical overlap."

---

# Recommendations

## Step 19 — Identify the Correct Change Target

For every root cause determine whether the fix belongs in:

- AGENT_PROMPT
- AGENT_CODE
- TOOL_DESCRIPTION
- TOOL_IMPLEMENTATION
- ROUTING
- MIDDLEWARE
- RUNTIME/HARNESS
- GRADER_CODE
- GRADER_PROMPT
- GRADER_RUBRIC
- DATASET
- REFERENCE_DATA
- NOTHING

Prefer the smallest fix addressing the underlying problem.

Do not modify source code unless explicitly authorized.

---

## Step 20 — Prioritize Fixes

Rank recommendations using:

1. expected impact
2. confidence
3. implementation effort
4. blast radius

For every recommendation provide:

- change target
- recommended change
- reason
- supporting evidence
- likely repository/file
- expected impact
- confidence

---

# Evidence Standard

Every major conclusion must be supported by evidence.

Whenever possible combine:

### Runtime Evidence
Langfuse traces, scores, observations and evaluator feedback.

### Code Evidence
Source repository implementation.

### Architectural Evidence
Documentation/configuration showing intended responsibility.

Clearly distinguish:

VERIFIED FACT

from:

HYPOTHESIS

Never fabricate:

- trace contents
- counts
- percentages
- source files
- code
- grader logic
- expected keywords
- reference answers
- tool behavior

Use "unknown" or "not verified" when evidence is unavailable.

---

# Confidence

Use confidence based on evidence strength.

HIGH:
Runtime + source-code evidence agree.

MEDIUM:
Strong runtime pattern but incomplete implementation evidence.

LOW:
Limited sample or unresolved competing explanations.

Do not use high confidence merely because an explanation sounds
plausible.

---

# Scale

For small datasets:
Analyze every failing trace when practical.

For larger datasets:

1. retrieve score metadata
2. identify cohorts
3. inspect representative failures
4. discover candidate patterns
5. expand around those patterns
6. validate prevalence across broader data
7. compare with passing controls

Never extrapolate percentages from an unvalidated sample.

---

# Investigation Stopping Condition

Do not stop because you found one plausible explanation.

Finish when:

- important failure patterns are identified
- passing controls were considered
- major competing hypotheses were tested
- relevant runtime evidence was inspected
- relevant grader code was inspected
- relevant agent/tool code was inspected when necessary
- responsibility ownership is understood
- cross-layer behavior was checked
- recommendations are evidence-backed

OR:

available evidence/tools are exhausted.

Clearly report unresolved uncertainty.

---

# Required Investigation Output

Create a structured investigation result containing:

{
  "evaluator": "...",

  "summary": "...",

  "agent_health": "...",

  "grader_health": "...",

  "stats": {
    "total_evaluated": null,
    "failures": null,
    "failures_analyzed": null,
    "passing_controls_analyzed": null
  },

  "failure_modes": [
    {
      "name": "...",
      "root_cause": "...",
      "affected_count": null,
      "percentage": null,
      "confidence": null,
      "representative_traces": []
    }
  ],

  "root_causes": [
    {
      "category": "...",
      "title": "...",
      "confidence": null,
      "runtime_evidence": [],
      "code_evidence": [],
      "responsible_layer": "...",
      "handled_elsewhere": null,
      "grader_observes_correct_layer": null,
      "suggested_fix": "...",
      "change_target": "..."
    }
  ],

  "recommendations": [
    {
      "priority": 1,
      "target": "...",
      "change": "...",
      "reason": "...",
      "confidence": null
    }
  ],

  "uncertainties": []
}

Use null when evidence is unavailable.

Do not invent values to complete the structure.

This structured investigation is the SOURCE OF TRUTH for downstream
reports.

---

# Human-Facing Investigation Summary

The investigation should clearly communicate:

## Executive Summary

- evaluator
- traces evaluated
- failures
- passing controls
- primary root cause
- agent health
- grader health


## Failure Landscape

Failure mode | Root cause | Affected | % | Confidence


## Root Causes

For each:

- what grader expected
- what actually happened
- runtime evidence
- code evidence
- responsible layer
- actual code path
- whether behavior exists elsewhere
- whether grader can observe it
- competing hypotheses
- final diagnosis
- recommended fix


## Grader Health

- implementation
- design
- rubric
- observability
- reference data
- overall verdict


## Agent Health

- prompt
- tools
- behavior
- error handling
- overall verdict


## Recommended Actions

Prioritized by impact and confidence.


## Representative Traces

Include relevant failing and passing trace IDs.


## Relevant Source Files

Include important repository paths and why they matter.


## Uncertainties

Explicitly state anything that could not be verified.

---

# Quality Check Before Completion

Before completing an investigation verify:

- Did I inspect failing traces?
- Did I inspect passing controls?
- Did I understand what the grader actually measures?
- Did I inspect grader implementation when available?
- Did I inspect reference data when relevant?
- Did I inspect agent prompt when relevant?
- Did I inspect tool implementation when relevant?
- Did I follow code references beyond the first matching file?
- Did I check whether supposedly missing behavior exists elsewhere?
- Did I identify which layer owns the behavior?
- Did I determine whether the grader can observe that layer?
- Did I consider competing hypotheses?
- Did I validate aggregate claims with evidence?
- Did I avoid automatically blaming the agent?
- Did I avoid recommending changes merely to game the evaluator?
- Are recommendations tied to actual evidence?
- Did I state uncertainty?

If important checks are incomplete and available tools can resolve them,
continue investigating.

---

# Deep-Dive Example

Suppose a grader says:

"Agent failed to request confirmation before deleting a user."

Trace:

Agent → delete_user()

Do NOT immediately conclude:

AGENT_PROMPT failure.

Instead:

1. Inspect the trace.
2. Determine exactly what the grader evaluates.
3. Search repository for delete_user.
4. Inspect tool registration.
5. Search approval/confirmation configuration.
6. Follow wrappers/middleware/runtime behavior.
7. Determine which component owns approval.
8. Check whether runtime evidence confirms approval occurred.
9. Determine whether the grader can observe that event.
10. Compare with passing examples.

If you discover:

delete_user
→ destructive tool
→ requiresApproval=true
→ runtime pauses
→ human approval
→ execution

then diagnose:

GRADER_OBSERVABILITY_GAP

and potentially:

GRADER_RUBRIC_MISALIGNMENT

Do NOT recommend changing the agent prompt.

Recommend changing the grader so it evaluates whether destructive
execution was approval-gated.

This level of investigation is expected for important findings.

---

# Final Objective

The engineer should NOT have to manually:

- inspect every failed trace
- compare successful traces
- inspect grader feedback
- find evaluator implementation
- locate expected values
- find system prompts
- inspect tool implementations
- trace middleware
- understand responsibility ownership
- determine whether the grader is wrong
- determine what should change

You perform that investigation.

A successful investigation answers:

WHAT failed?

HOW OFTEN?

WHY?

WHERE did the behavior originate?

WHICH system layer owns it?

IS the agent actually wrong?

IS the grader actually wrong?

IS required behavior already handled elsewhere?

CAN the grader observe the behavior it expects?

WHAT evidence proves the conclusion?

WHAT should the engineer change first?
