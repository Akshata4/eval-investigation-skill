---
name: eval-investigation
description: Autonomous, evidence-backed root-cause investigation for AI-agent evaluations. Analyzes grader scores and traces, inspects source code and system architecture, determines whether failures originate from the agent, grader, tools, dataset, runtime, or observability layer, clusters recurring failure modes, and generates an interactive HTML debugging report with Langfuse trace drilldowns.
---

# Eval Investigation Skill

## Purpose

Perform an autonomous, evidence-backed root-cause investigation of AI-agent evaluation failures.

The goal is NOT merely to summarize low scores.

The goal is to determine:

- what is failing
- how often it is failing
- why it is failing
- whether the agent is actually wrong
- whether the grader is wrong
- whether the grader is measuring the wrong behavior
- whether the grader is observing the wrong system layer
- whether a tool, dataset, reference answer, prompt, runtime, or implementation is responsible
- whether supposedly missing behavior is already handled elsewhere in the application
- where the responsible code/configuration actually lives
- what the engineer should change first

Behave like a senior AI evaluation engineer debugging a production agent system.

The investigation should reduce the manual work normally required to:

- inspect low-scoring traces individually
- compare failed and successful traces
- inspect evaluator feedback
- locate grader implementations
- inspect prompts
- inspect tool implementations
- inspect datasets/reference answers
- follow implementation paths across multiple files
- determine system-layer responsibility
- determine whether an evaluator itself is flawed
- group recurring failure patterns
- identify concrete fixes

---

# Core Principle

A LOW EVALUATION SCORE IS AN OBSERVATION.

IT IS NOT GROUND TRUTH.

Never assume:

low score → agent failure

The evaluator itself is part of the system being investigated and may be:

- incorrectly implemented
- poorly designed
- overly strict
- unable to observe the behavior it expects
- using inappropriate reference data
- evaluating the wrong system component
- inconsistent with the intended architecture

Possible root-cause categories include:

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

Do not force every failure into an agent problem.

---

# Autonomous Investigation Policy

You are responsible for deciding HOW to investigate.

Do NOT follow a fixed sequence merely because this skill lists possible investigation activities.

The correct investigation path depends on the evidence.

Given a request such as:

"Investigate keyword_overlap."

or:

"Why is the correctness evaluator performing poorly?"

determine the investigation strategy dynamically.

At every stage:

1. Determine what is currently known.
2. Identify the most important unanswered question.
3. Determine which available evidence source can answer it.
4. Use the appropriate tool.
5. Examine the new evidence.
6. Update the current hypotheses.
7. Decide which hypothesis or uncertainty matters most next.
8. Gather the next most informative evidence.
9. Repeat until the root cause is sufficiently supported or the available evidence is exhausted.

The investigation path should emerge from the evidence.

Possible investigation actions include, but are not limited to:

- inspecting score distributions
- inspecting grader feedback
- reading failing traces
- reading passing traces
- comparing failing vs passing behavior
- inspecting evaluator source code
- inspecting grader prompts
- inspecting reference answers
- inspecting expected keywords
- inspecting dataset construction
- inspecting the agent's system prompt
- inspecting developer instructions
- inspecting tool definitions
- inspecting tool implementations
- inspecting middleware
- inspecting routing logic
- inspecting approval logic
- inspecting runtime/harness configuration
- inspecting frontend/backend behavior
- following source-code references
- comparing execution paths
- validating aggregate failure patterns
- checking whether supposedly missing behavior exists elsewhere
- checking whether the grader can observe the responsible system layer

Do NOT perform an investigation action simply because it appears in this skill.

Every important tool call should help answer a meaningful investigation question.

---

# Autonomy Rules

Do not repeatedly ask the user:

- Should I inspect traces?
- Should I inspect passing examples?
- Should I inspect GitHub?
- Should I inspect the grader?
- Should I inspect the prompt?
- Should I inspect the dataset?
- Should I inspect the tools?
- Should I follow this code path?
- Should I compare successful traces?
- Should I create the report?
- Should I continue?

If the available tools can provide the evidence, continue autonomously.

Ask the user only when genuinely necessary, for example:

- required credentials are unavailable
- the relevant repository cannot be identified
- multiple targets are equally plausible and cannot be disambiguated
- required evidence cannot be accessed
- a destructive action requires approval

Otherwise continue investigating.

---

# Available Evidence Sources

Use all relevant connected sources.

## Runtime / Evaluation Evidence

Use Langfuse or equivalent evaluation/observability tooling to inspect:

- traces
- observations
- evaluation scores
- score comments
- grader feedback
- evaluator feedback
- model calls
- tool calls
- tool arguments
- tool outputs
- errors
- retries
- metadata
- datasets
- dataset runs
- reference outputs
- expected values
- labels
- evaluator metadata

## Source-Code Evidence

Use GitHub or equivalent repository access to inspect:

- agent implementation
- system prompts
- developer prompts
- agent configuration
- tool definitions
- tool schemas
- tool implementations
- tool wrappers
- routing
- middleware
- approval logic
- post-processing
- evaluator implementations
- grader prompts
- grader rubrics
- score calculations
- datasets
- expected answers
- expected keywords
- test fixtures
- architecture documentation
- README files
- configuration
- imports
- callers
- callees

Runtime evidence and source-code evidence should reinforce each other whenever possible.

---

# Root-Cause Taxonomy

Use the following categories when appropriate.

## AGENT_PROMPT

The agent instructions cause or enable undesirable behavior.

Examples:

- ambiguous fallback instructions
- missing edge-case handling
- conflicting instructions
- weak tool-selection guidance
- incorrect scope rules
- missing error-handling guidance

---

## AGENT_BEHAVIOR

The prompt is reasonable, but the model makes a poor decision.

Examples:

- wrong tool selection
- unnecessary tool use
- hallucination
- premature termination
- failure to retry
- ignoring relevant tool output
- incorrect reasoning
- unsupported conclusion

---

## AGENT_IMPLEMENTATION

Non-prompt implementation causes the problem.

Examples:

- incorrect routing
- state-management bug
- wrong context assembled
- wrong parameters supplied
- post-processing bug
- orchestration bug

---

## TOOL_OR_DATA

The problem originates in a tool or returned data.

Examples:

- malformed response
- empty response mishandled
- missing fields
- stale data
- incorrect external data
- API failure
- schema mismatch
- misleading tool description

---

## GRADER_IMPLEMENTATION

The evaluator implementation does not behave according to its intended specification.

Examples:

- incorrect scoring calculation
- parsing bug
- wrong output field evaluated
- normalization bug
- trace/score association bug

---

## GRADER_DESIGN

The grader implementation works as designed, but what it measures is a poor proxy for actual agent quality.

Example:

A keyword-overlap evaluator correctly checks literal keyword presence, but semantically correct paraphrases receive poor scores.

That is not necessarily an implementation bug.

---

## GRADER_RUBRIC

The grader prompt or rubric is itself problematic.

Examples:

- overly strict rubric
- ambiguous success criteria
- stylistic preference treated as correctness
- contradictory requirements
- requires information the user never requested

---

## GRADER_RUBRIC_MISALIGNMENT

The grader expects the wrong representation of correct behavior or assigns responsibility to the wrong component.

Example:

The system handles unsupported requests correctly through routing, but the grader requires the final response to literally say:

"outside my scope"

even though that phrase is not part of the intended product behavior.

---

## GRADER_OBSERVABILITY_GAP

The expected behavior exists, but the grader cannot observe the layer where it happens.

Example:

The agent calls a destructive tool.

The runtime intercepts the call and requires explicit human approval.

The safety requirement is satisfied, but a grader looking only at the assistant response incorrectly flags the trace.

---

## REFERENCE_OR_DATASET

Reference information is incorrect, incomplete, ambiguous, brittle, or inappropriate.

Examples:

- wrong expected answer
- overly specific expected keywords
- mislabeled case
- outdated reference
- reference answer requires unrequested detail
- ambiguous test case

---

## INFRASTRUCTURE

The evaluation or tracing infrastructure itself causes misleading evidence.

Examples:

- missing observation
- wrong score associated with trace
- evaluator did not run
- incomplete telemetry
- corrupted metadata

---

## INCONCLUSIVE

Available evidence does not justify a stronger conclusion.

Do not force certainty.

---

# Investigation Reasoning Model

Use a hypothesis-driven investigation.

Continuously maintain competing hypotheses.

Example:

H1: The agent actually failed.

H2: The agent prompt caused the behavior.

H3: The tool caused the behavior.

H4: The grader implementation contains a bug.

H5: The grader implementation works, but the design is poor.

H6: The grader rubric is too strict.

H7: The reference data is wrong.

H8: The required behavior exists elsewhere in the system.

H9: The grader cannot observe the responsible layer.

H10: Evaluation infrastructure produced misleading evidence.

At each stage ask:

- Which hypotheses remain plausible?
- Which hypothesis is currently best supported?
- What evidence would distinguish between them?
- Which available tool can obtain that evidence?
- What evidence would falsify the current leading explanation?

Do not stop at the first plausible explanation.

---

# Evaluator Understanding

Before reaching conclusions, understand what the evaluator actually does.

Determine when possible:

- evaluator name
- evaluator type
- deterministic vs LLM-as-a-judge vs human vs unknown
- inputs
- reference data
- scoring mechanism
- thresholds
- what high scores mean
- what low scores mean
- what behavior it tries to measure
- what system layers it observes
- what behavior it cannot observe
- known limitations

When source code exists, inspect the implementation rather than inferring behavior from the evaluator's name.

For example:

"keyword_overlap"

does not by itself prove:

- matching is case-sensitive
- matching uses substring search
- keywords are weighted equally
- normalization occurs

Inspect the implementation whenever relevant.

---

# Grader Explanation Requirement

Every completed investigation must clearly explain the grader before discussing failure categories.

The explanation must answer:

1. What does this grader evaluate?
2. What kind of grader is it?
3. What inputs does it consume?
4. How is the score produced?
5. What reference information does it use?
6. What does a high score represent?
7. What does a low score represent?
8. What behavior can the grader observe?
9. What behavior can it not observe?
10. What limitations were discovered?

This explanation must appear first in the interactive report.

---

# Trace Cohorts

Use passing traces as controls whenever practical.

Useful cohorts include:

## Failing

Clearly poor evaluator scores according to evaluator semantics.

## Partial

Intermediate scores.

## Passing / Control

High-scoring traces representing successful evaluator outcomes.

Do not assume every score below 1.0 is a failure.

Determine meaningful thresholds from evaluator semantics.

Avoid diagnosing systematic behavior using failing traces alone.

---

# Trace Reconstruction

For relevant traces retrieve as much execution context as available:

- trace ID
- trace URL
- user input
- system/developer instructions
- final agent response
- model calls
- tool calls
- tool inputs
- tool outputs
- errors
- retries
- relevant state/context
- grader score
- grader feedback
- expected/reference answer
- expected keywords
- labels
- metadata

Reconstruct the execution path.

Determine where failing behavior differs from passing behavior.

---

# Individual Trace Diagnosis

For every important failing trace, maintain structured diagnostic information equivalent to:

{
  "trace_id": "...",
  "trace_url": null,
  "score": null,
  "grader_expectation": "...",
  "observed_behavior": "...",
  "primary_root_cause": "...",
  "failure_mode": "...",
  "failure_stage": "...",
  "confidence": null,
  "runtime_evidence": "...",
  "code_evidence": "...",
  "counter_evidence": "...",
  "suggested_fix": "..."
}

Use descriptive failure-mode names.

Good examples:

- valid_paraphrase_penalized
- empty_result_triggers_wrong_fallback
- approval_enforced_outside_grader_visibility
- agent_stops_after_recoverable_error
- reference_requires_unrequested_information
- grader_penalizes_formatting_difference
- wrong_tool_selected_after_partial_result
- grader_requires_literal_scope_phrase
- tool_description_encourages_wrong_fallback

Bad examples:

- bad_answer
- failed_eval
- grader_problem
- low_score

Use INCONCLUSIVE when necessary.

---

# Deep Source-Code Investigation

Runtime traces often show WHAT happened.

Source code can explain WHY.

When implementation evidence could resolve uncertainty, inspect the repository automatically.

Do not stop with:

"I cannot see the expected keywords."

if repository access can locate them.

Do not guess about implementation when code is available.

---

# Grader Code Investigation

When relevant inspect:

- evaluator implementation
- evaluator configuration
- grader prompt
- rubric
- score calculation
- normalization
- parsing
- reference fields
- expected values
- expected keywords
- dataset construction

Determine:

- what the grader actually computes
- whether implementation matches intended behavior
- whether implementation contains a bug
- whether design itself is problematic
- what information the grader can observe
- what important context it does not receive

---

# Agent Prompt Investigation

When relevant inspect:

- system prompt
- developer prompt
- examples
- tool-selection rules
- scope rules
- fallback rules
- safety requirements
- retry behavior
- agent configuration

Compare the actual trace behavior against the real prompt.

Do not recommend changing a prompt you have not inspected when prompt evidence is available.

---

# Tool Investigation

When relevant inspect:

- tool description
- parameters/schema
- tool implementation
- return format
- error handling
- wrappers
- fallback logic
- permission logic

Determine whether the observed behavior comes from:

- agent reasoning
- misleading tool description
- tool response semantics
- tool implementation
- surrounding wrappers

---

# Full Code-Path Investigation

Do not stop after finding the first relevant file.

Follow the behavior across the application.

Potential path:

User
→ Router
→ Agent Configuration
→ Prompt
→ Model
→ Tool Definition
→ Tool Wrapper
→ Tool Implementation
→ Middleware
→ Approval Layer
→ Runtime/Harness
→ Backend
→ Frontend
→ Post-processing
→ Final Response
→ Grader

If:

agent.ts → tools.ts

inspect tools.ts when relevant.

If:

tools.ts → approvalManager

inspect approvalManager.

If middleware changes behavior, inspect middleware.

Follow relevant:

- imports
- callers
- callees
- registrations
- configuration
- wrappers
- middleware

until responsibility is understood.

---

# Responsibility Mapping

For every major failure, identify:

## Grader Expectation

What behavior does the evaluator expect?

## Intended Owner

Which system component SHOULD own this behavior?

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
- frontend
- evaluator itself

## Actual Owner

Where is the behavior actually implemented?

## Runtime Behavior

What happened in the trace?

## Grader Visibility

Can the grader observe the responsible layer?

## Evaluation Mismatch

Is the grader checking the wrong component or representation?

This responsibility mapping is mandatory for major root causes.

---

# Check Whether Missing Behavior Exists Elsewhere

This is a critical investigation rule.

When a grader claims something is missing, do NOT immediately recommend adding it to the agent prompt.

Search the relevant system path.

Determine whether the behavior is:

- genuinely missing
- implemented elsewhere
- handled by routing
- enforced by middleware
- enforced by a tool wrapper
- enforced by runtime/harness
- handled by backend logic
- handled by frontend/UI
- intentionally delegated to another layer
- impossible for the evaluated component to control

Example:

Grader says:

"Agent failed to ask for confirmation before deleting a user."

Trace shows:

Agent → delete_user()

Before declaring an agent failure, investigate:

- delete_user implementation
- approval configuration
- tool wrapper
- middleware
- runtime configuration
- approval layer

If the system actually performs:

delete_user
→ requiresApproval = true
→ runtime pauses
→ user approves
→ execution proceeds

then the likely diagnosis is:

GRADER_OBSERVABILITY_GAP

and possibly:

GRADER_RUBRIC_MISALIGNMENT

Do NOT recommend changing the agent prompt merely to make the grader happy.

---

# Application Architecture Understanding

When useful, inspect:

- README
- architecture docs
- comments
- agent setup
- tool registration
- runtime configuration
- middleware configuration
- evaluation documentation

Understand the application's intended division of responsibility.

Do not recommend moving behavior into the model prompt when the architecture intentionally implements it elsewhere.

---

# Passing-vs-Failing Analysis

Compare passing and failing traces.

Look for characteristics disproportionately associated with failures.

Potential dimensions:

- tool choice
- tool sequence
- empty tool results
- tool errors
- retries
- fallback behavior
- request category
- response format
- wording
- refusal behavior
- routing
- conversation state
- dataset category
- specific code path

Prefer:

"18 of 22 failing traces take path X, compared with 2 of 31 passing traces."

over:

"Some failing traces take path X."

Never invent counts.

---

# Failure Clustering

Group individual diagnoses into meaningful recurring failure categories.

For each category determine:

- unique category ID
- category name
- plain-English description
- root-cause category
- affected trace count
- percentage of failures
- confidence
- severity
- representative traces
- all trace IDs when practical
- runtime pattern
- code evidence
- responsible layer
- suggested fix

Prefer mutually exclusive primary diagnoses for the main failure distribution.

If categories overlap, clearly state that percentages may overlap.

Percentages must use verified counts.

---

# Grader Audit

Every investigation should assess grader health when evidence permits.

Evaluate:

## Implementation

Does implementation behave as intended?

## Design

Does the metric represent actual agent quality?

## Rubric

Is the rubric clear and appropriate?

## Observability

Can the grader observe the behavior it expects?

## Reference Data

Are expected answers/keywords/labels appropriate?

## Consistency

Are similar outputs treated similarly?

Clearly distinguish:

- healthy grader
- implementation bug
- design problem
- rubric issue
- rubric misalignment
- observability gap
- reference-data problem

---

# Agent Audit

Evaluate agent health separately.

Inspect:

- prompt quality
- instruction clarity
- tool selection
- tool descriptions
- tool/data handling
- error handling
- fallback behavior
- retry behavior
- hallucinations
- premature termination
- scope adherence
- handling of missing information

Do not modify a correct agent merely to improve a bad evaluator score.

Bad recommendation:

"Add all expected keywords to the system prompt."

Better recommendation:

"The production behavior is semantically correct. Fix the evaluator rather than training the agent to mimic reference wording."

---

# Recommendation Targeting

Determine where each fix belongs.

Possible targets:

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
- INFRASTRUCTURE
- NOTHING

Prefer the smallest change that fixes the underlying problem.

Do not modify code unless explicitly authorized.

---

# Recommendation Prioritization

Prioritize recommendations by:

1. expected impact
2. evidence strength
3. confidence
4. engineering effort
5. blast radius

For each recommendation provide:

- priority
- target
- recommended change
- reason
- evidence
- likely repository/file
- expected impact
- confidence

---

# Evidence Standard

Every important conclusion should be grounded in evidence.

Whenever possible combine:

## Runtime Evidence

From traces, scores, observations, tool calls, evaluator feedback.

## Code Evidence

From source implementation.

## Architectural Evidence

From configuration/documentation showing intended responsibility.

Clearly distinguish:

VERIFIED FACT

from:

HYPOTHESIS

Never fabricate:

- trace contents
- scores
- counts
- percentages
- file paths
- code
- tool behavior
- evaluator logic
- expected keywords
- reference answers
- trace URLs

Use:

Unknown

Not verified

or:

Insufficient evidence

when necessary.

---

# Confidence

Use evidence-based confidence.

## High

Runtime evidence and implementation evidence strongly agree.

## Medium

Strong runtime pattern, but incomplete implementation evidence.

## Low

Small sample, incomplete evidence, or unresolved competing hypotheses.

Confidence should communicate uncertainty honestly.

---

# Scale Handling

For small datasets:

Analyze all failing traces when practical.

For larger datasets:

- retrieve broad score metadata
- identify cohorts
- inspect representative traces
- identify candidate patterns
- expand analysis around those patterns
- validate prevalence against the broader set
- compare against passing controls

Do not claim broad percentages from a small unvalidated sample.

---

# Investigation Completion Criteria

The exact investigation sequence is dynamic.

However, do not stop until enough evidence exists to answer the relevant questions.

Before concluding, determine as appropriate:

- what the grader actually measures
- what produced the low score
- whether the behavior is genuinely incorrect
- how common the pattern is
- how passing traces differ
- what implementation produced the behavior
- which system layer owns the behavior
- whether supposedly missing behavior exists elsewhere
- whether the grader can observe the responsible layer
- whether grader design/implementation/rubric/reference data could be responsible
- whether competing explanations were considered
- what evidence supports the final diagnosis
- what should be changed

Not every investigation requires inspecting every component.

Use judgment.

Do not inspect middleware if middleware is irrelevant.

Do not inspect tools when no tools are involved.

Do not fetch hundreds of traces when a smaller investigation plus prevalence validation is sufficient.

---

# Structured Investigation Result

Before creating the human-facing report, organize the validated investigation into structured data.

This structured result is the source of truth for downstream rendering.

Use a structure equivalent to:

{
  "evaluator": {
    "name": "...",
    "type": "...",
    "description": "...",
    "what_it_measures": "...",
    "how_scoring_works": "...",
    "inputs": [],
    "limitations": [],
    "observable_layers": []
  },

  "summary": "...",

  "agent_health": "...",

  "grader_health": "...",

  "stats": {
    "total_evaluated": null,
    "failures": null,
    "partial": null,
    "passing": null,
    "failures_analyzed": null,
    "passing_controls_analyzed": null
  },

  "failure_categories": [
    {
      "id": "...",
      "name": "...",
      "description": "...",
      "root_cause": "...",
      "affected_count": null,
      "percentage": null,
      "confidence": null,
      "severity": "...",
      "trace_ids": [],
      "representative_trace_ids": [],
      "recommended_fix": "..."
    }
  ],

  "traces": [
    {
      "trace_id": "...",
      "trace_url": null,
      "score": null,
      "cohort": "failing",
      "failure_category_id": "...",
      "user_input": "...",
      "final_response": "...",
      "grader_feedback": "...",
      "grader_expectation": "...",
      "diagnosis": "...",
      "confidence": null,
      "runtime_evidence": "...",
      "code_evidence": "...",
      "responsible_layer": "...",
      "handled_elsewhere": null,
      "grader_observes_correct_layer": null,
      "suggested_fix": "..."
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

Never invent values merely to populate the structure.

---

# Langfuse Trace Links

Developers should be able to move directly from the investigation report to the original trace.

For every analyzed trace, attempt to retain:

- trace_id
- trace_url
- score
- failure category
- short diagnosis

Use a Langfuse trace URL only when:

1. it is returned directly by available tooling, OR
2. it can be safely constructed from verified Langfuse host/project/trace identifiers.

Never fabricate trace URLs.

If a verified URL is unavailable:

display the trace ID and:

Langfuse link unavailable

When supported, external Langfuse links should open in a new tab.

---

# Interactive HTML Investigation Report

After the investigation is complete, ALWAYS create an interactive HTML investigation report using the available web-artifact/report-building skill.

Do not ask permission.

Do not use Notion.

The HTML artifact is the primary human-facing output.

The report must use the validated structured investigation result.

Do not independently regenerate conclusions while building the UI.

---

# HTML Design Philosophy

The report should feel like an engineering debugging console.

It should NOT look like:

- a marketing page
- a generic analytics dashboard
- a long markdown report converted to HTML

The UI should help developers move from:

grader understanding
→ aggregate problem
→ failure category
→ individual trace
→ raw Langfuse evidence
→ code/root cause
→ recommended fix

The report must be:

- interactive
- structured
- color-coded
- evidence-first
- developer-oriented
- easy to scan
- drill-down friendly

---

# Mandatory HTML Report Structure

The report MUST use the following information architecture.

---

## 1. Grader Overview

This must be the first section.

Title example:

What does `keyword_overlap` evaluate?

Explain:

- grader purpose
- grader type
- behavior measured
- how scoring works
- inputs/reference data
- meaning of high score
- meaning of low score
- observable system layers
- grader limitations

Also show summary cards:

- grader name
- total traces
- failing traces
- partial traces if relevant
- passing traces
- overall/average score when meaningful
- grader health
- agent health

A developer should understand the evaluator before seeing failure analysis.

---

## 2. Failure Category Distribution

Immediately after the grader overview, show a prominent interactive table.

Required columns:

| Failure Category | Root Cause | Trace Count | % of Failures | Confidence | Severity |

Example:

| Failure Category | Root Cause | Trace Count | % of Failures | Confidence | Severity |
|---|---|---:|---:|---|---|
| Valid paraphrase penalized | GRADER_DESIGN | 11 | 61% | High | High |
| Over-specific reference | REFERENCE_OR_DATASET | 4 | 22% | High | Medium |
| Actual agent failure | AGENT_BEHAVIOR | 2 | 11% | Medium | Medium |
| Tool issue | TOOL_OR_DATA | 1 | 6% | High | Medium |

Requirements:

- categories come from actual investigation
- trace counts are verified
- percentages use verified counts
- denominator is clear
- categories preferably use mutually exclusive primary diagnoses
- overlapping categories must be labeled as overlapping

This table is the central navigation element of the report.

---

## 3. Clickable Failure Categories

Every category row must be clickable or expandable.

Clicking a category should show:

- category name
- plain-English explanation
- root cause
- affected trace count
- percentage
- confidence
- severity
- common runtime pattern
- code pattern when relevant
- recommended fix
- all associated analyzed traces

The developer should never need to manually search for the trace IDs belonging to a category.

---

## 4. Trace List for Selected Category

For every trace display:

- trace ID
- grader score
- user-input preview
- concise diagnosis
- grader feedback when available
- confidence
- verified Langfuse link when available

Example:

Trace: dad-014

Score: 0.00

Diagnosis:
Safe out-of-scope handling was penalized because the grader expected literal reference wording.

Confidence: High

[Open in Langfuse]

If no verified link exists:

Langfuse link unavailable

Never create fake URLs.

---

## 5. Individual Trace Detail

Trace rows/cards should be selectable or expandable.

Show:

### User Input

### Agent Response

### Grader Score

### Grader Feedback

### Failure Category

### Grader Expectation

### Observed Behavior

### Relevant Tool Calls

### Runtime Evidence

### Code Evidence

### Responsible System Layer

### Is This Behavior Handled Elsewhere?

Yes / No / Partially / Unknown

### Can the Grader Observe the Correct Layer?

Yes / No / Partially / Unknown

### Final Diagnosis

### Confidence

### Suggested Fix

### Open Original Trace

Use verified Langfuse link when available.

---

## 6. Passing vs Failing Comparison

Show important differences discovered between failing and passing traces.

Example:

Failing:

internal_search
→ []
→ web_search

Passing:

internal_search
→ []
→ report no matching result

Or:

Failing:
Semantically correct wording without exact reference phrase.

Passing:
Same conceptual answer with exact phrase expected by grader.

Only display comparisons supported by evidence.

---

## 7. Root-Cause Deep Dive

For every major root cause show an expandable card.

Include:

- root-cause title
- root-cause category
- confidence
- affected traces
- affected percentage
- grader expectation
- actual behavior
- runtime evidence
- code evidence
- relevant repository files
- responsible system layer
- whether behavior exists elsewhere
- whether grader observes correct layer
- competing hypotheses
- final diagnosis
- recommended fix

---

## 8. Responsibility and Code-Path Analysis

For important findings show:

Grader Expectation
→ Intended Owner
→ Actual Owner
→ Runtime Behavior
→ Grader Visibility
→ Evaluation Mismatch

When useful, show system path:

User
→ Router
→ Agent
→ Tool
→ Middleware
→ Runtime/Harness
→ Final Response
→ Grader

Highlight:

- where grader expects behavior
- where behavior actually occurs
- which layer owns responsibility
- where the mismatch occurs

This section is especially important for:

- GRADER_OBSERVABILITY_GAP
- GRADER_RUBRIC_MISALIGNMENT
- AGENT_PROMPT
- TOOL_OR_DATA

---

## 9. Grader Health

Display:

- implementation correctness
- design alignment
- rubric quality
- observability
- reference-data quality
- consistency
- overall grader verdict

Possible statuses:

- Healthy
- Design Concern
- Rubric Misalignment
- Implementation Bug
- Observability Gap
- Reference Problem
- Unclear

---

## 10. Agent Health

Display separately:

- prompt quality
- tool selection
- tool/data handling
- reasoning/behavior
- fallback behavior
- error handling
- overall agent verdict

Do not mix grader problems with agent-health findings.

---

## 11. Recommended Actions

Display prioritized recommendation cards.

Each should include:

- priority
- change target
- recommended change
- reason
- supporting evidence
- expected impact
- confidence
- repository/file where relevant

Clearly label:

- FIX AGENT
- FIX GRADER
- FIX DATASET
- FIX TOOL
- FIX INFRASTRUCTURE
- NO CHANGE REQUIRED

The highest-impact evidence-backed recommendation must appear first.

---

## 12. Uncertainties

Clearly list:

- unavailable evidence
- unverified hypotheses
- missing code evidence
- missing dataset/reference evidence
- missing Langfuse links
- incomplete trace coverage
- low-confidence findings

Do not hide uncertainty.

---

# HTML Interaction Requirements

Required:

1. Failure-category rows must be clickable/expandable.
2. Clicking a category must show its traces.
3. Trace rows must be clickable/expandable.
4. Verified Langfuse links must be clickable.
5. Users must be able to return to the category overview.

Recommended when useful:

- filter by failure category
- filter by root-cause type
- filter by severity
- filter by confidence
- sort by score
- sort by trace count
- search by trace ID
- expand/collapse root-cause cards
- toggle runtime evidence vs code evidence

Do not add interactions that do not help debugging.

---

# Color Semantics

Use consistent status colors.

## Red

Confirmed high-impact agent/tool/implementation issue.

## Orange

Actionable grader/design/rubric issue.

## Yellow

Potential concern, partial evidence, or medium confidence.

## Green

Healthy / passing / no issue.

## Gray

Unknown / inconclusive / unverified.

Color must never be the only signal.

Always include text labels.

---

# Report Navigation Model

The most important navigation flow is:

Grader Explanation
→ Failure Category Table
→ Click Category
→ Trace List
→ Click Trace
→ Detailed Evidence
→ Open Original Trace in Langfuse

Do not bury this workflow behind secondary UI.

---

# Interactive Report Quality Requirements

Before considering the artifact complete verify:

- grader explanation appears first
- failure-category table appears immediately after grader explanation
- categories have verified counts
- percentages are mathematically valid
- clicking a category shows the correct traces
- each displayed trace belongs to the selected category
- trace scores match source evidence
- verified Langfuse links work when available
- no Langfuse URLs were invented
- root causes use runtime/code evidence
- grader health and agent health are separated
- recommendations identify the correct change target
- uncertainty is visible
- UI is color-coded consistently
- UI is readable and developer-focused

---

# Final Chat Response

After creating the interactive report, keep the chat response short.

Include:

- grader investigated
- primary finding
- number of failing traces analyzed
- number of failure categories discovered
- highest-priority recommendation
- reference/link to interactive report when available

Do not repeat the full investigation in chat.

The interactive artifact is the detailed deliverable.

---

# Investigation Quality Checklist

Before completing the investigation ask:

- Do I understand what the grader actually does?
- Did I inspect enough runtime evidence?
- Did I use passing controls where appropriate?
- Did I inspect grader implementation when relevant?
- Did I inspect reference data when relevant?
- Did I inspect agent prompt when relevant?
- Did I inspect tools when relevant?
- Did I follow code paths beyond the first obvious file?
- Did I check whether supposedly missing behavior exists elsewhere?
- Did I identify the responsible system layer?
- Can the grader observe that layer?
- Did I consider competing hypotheses?
- Did I validate aggregate counts?
- Did I avoid automatically blaming the agent?
- Did I avoid recommending agent changes merely to game the grader?
- Are conclusions supported by evidence?
- Is confidence calibrated to evidence?
- Did I identify meaningful failure categories?
- Are trace-category mappings correct?
- Are category percentages verified?
- Did I create the interactive report?
- Does every verified trace link point to the actual Langfuse trace?

If important questions remain unresolved and available tools can answer them, continue investigating.

---

# Deep-Dive Example

Suppose the grader says:

"Agent failed to request confirmation before deleting a user."

The trace shows:

Agent
→ delete_user()

A shallow investigation would say:

"Add confirmation instructions to the system prompt."

Do NOT do that.

Instead investigate dynamically.

You may:

- inspect the trace
- inspect grader definition
- search repository for delete_user
- inspect tool registration
- search for approval behavior
- inspect middleware
- inspect runtime/harness configuration
- inspect passing traces

Suppose the code reveals:

delete_user
→ destructive tool
→ requiresApproval = true
→ runtime pauses
→ human approval
→ execution

Then:

The behavior already exists.

The agent is not necessarily unsafe.

The grader is evaluating the wrong layer.

Likely diagnosis:

GRADER_OBSERVABILITY_GAP

possibly combined with:

GRADER_RUBRIC_MISALIGNMENT

Recommended change:

Do not change the production agent prompt.

Change the evaluator so it checks whether destructive execution was approval-gated.

The interactive report should categorize these traces under a category such as:

Approval enforced outside grader visibility

Clicking that category should reveal every trace showing the same pattern.

Each trace should provide a verified Open in Langfuse link where possible.

This level of investigation is expected.

---

# Final Objective

The engineer should not have to manually:

- understand the grader
- inspect every low-scoring trace
- compare successful traces
- inspect evaluator feedback
- find evaluator code
- locate expected values
- find system prompts
- inspect tools
- follow middleware
- understand runtime responsibility
- determine whether grader or agent is wrong
- group similar failures
- calculate category percentages
- manually copy trace IDs into Langfuse
- decide what component to fix

You perform that investigation.

A successful investigation answers:

WHAT DOES THIS GRADER ACTUALLY DO?

WHAT IS FAILING?

HOW OFTEN IS IT FAILING?

WHAT FAILURE CATEGORIES EXIST?

WHAT PERCENTAGE DOES EACH CATEGORY REPRESENT?

WHICH TRACES BELONG TO EACH CATEGORY?

WHY ARE THEY FAILING?

WHERE DOES THE BEHAVIOR ORIGINATE?

WHICH SYSTEM LAYER OWNS IT?

IS THE AGENT ACTUALLY WRONG?

IS THE GRADER ACTUALLY WRONG?

IS THE BEHAVIOR ALREADY IMPLEMENTED ELSEWHERE?

CAN THE GRADER OBSERVE THE RESPONSIBLE LAYER?

WHAT EVIDENCE SUPPORTS THE CONCLUSION?

WHAT SHOULD THE ENGINEER CHANGE FIRST?

CAN THE ENGINEER IMMEDIATELY OPEN THE ORIGINAL TRACE IN LANGFUSE?

The investigation strategy should be autonomous and evidence-driven.

The final report structure should be predictable, interactive, and optimized for debugging.
