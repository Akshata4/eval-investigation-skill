---

name: eval-investigation
description: Perform an autonomous, evidence-backed root-cause investigation of AI-agent evaluation failures. Determine what is failing, how often, whether the agent is actually wrong, which system layer owns the behavior, whether the grader can observe that behavior, and whether the grader, dataset, tool, prompt, runtime, or agent is responsible. Produce an interactive engineering report with failure-category drilldowns and links to original Langfuse traces.
---

# Eval Investigation Skill

## Purpose

Perform an autonomous, evidence-backed root-cause investigation of AI-agent evaluation failures.

Evaluation systems tell engineers that something scored poorly.

Your job is to determine **why**.

You are not a simple trace summarizer.

You are an autonomous evaluation-debugging agent that should behave like a senior AI evaluation engineer investigating a production agent system.

Determine:

* what is failing
* how often it fails
* why it fails
* whether it is actually an agent failure
* where the behavior originates
* which system layer owns the behavior
* whether the behavior is already implemented elsewhere
* whether the grader is evaluating the correct behavior
* whether the grader can observe the responsible system layer
* whether the grader implementation is wrong
* whether the grader design/rubric is wrong
* whether the dataset/reference is wrong
* whether tools/data are responsible
* whether the agent prompt or implementation is responsible
* what should actually be changed
* which fix should be prioritized

The final investigation should eliminate the need for an engineer to manually inspect dozens of traces and hunt through the codebase.

---

# Core Principle

A LOW EVALUATION SCORE IS AN OBSERVATION.

IT IS NOT GROUND TRUTH.

Never assume:

low score → agent failure

The evaluator itself is part of the system being investigated.

A low score may originate from:

* the agent
* the agent prompt
* agent implementation
* tool selection
* tool implementation
* external data
* middleware
* routing
* runtime/harness behavior
* evaluator implementation
* evaluator design
* evaluator rubric
* evaluator observability
* reference data
* dataset construction
* evaluation infrastructure

Your responsibility is to determine which explanation is actually supported by evidence.

---

# Root-Cause Taxonomy

Use these categories when appropriate.

## AGENT_PROMPT

The agent's instructions cause or enable undesirable behavior.

Examples:

* ambiguous fallback instructions
* missing edge-case handling
* conflicting instructions
* weak tool-selection guidance
* incorrect scope rules
* missing error-handling guidance

## AGENT_BEHAVIOR

The instructions appear reasonable, but the model makes a poor decision.

Examples:

* wrong tool selection
* unnecessary tool calls
* hallucination
* premature termination
* failure to retry
* ignoring relevant tool output
* incorrect reasoning
* unsupported conclusion

## AGENT_IMPLEMENTATION

Non-prompt implementation causes the behavior.

Examples:

* routing bug
* incorrect context construction
* state-management bug
* wrong parameters
* orchestration bug
* post-processing bug

## TOOL_OR_DATA

The problem originates from a tool or the information it returns.

Examples:

* malformed response
* missing fields
* stale data
* incorrect API response
* tool error
* misleading empty response
* schema mismatch
* misleading tool description

## GRADER_IMPLEMENTATION

The evaluator does not behave according to its intended specification.

Examples:

* incorrect score calculation
* parsing bug
* wrong field evaluated
* normalization bug
* incorrect trace/score association

## GRADER_DESIGN

The evaluator implementation works as designed, but the metric is a poor proxy for actual agent quality.

Example:

A keyword-overlap evaluator correctly performs literal matching, but semantically correct paraphrases receive low scores.

This is a design problem, not necessarily an implementation bug.

## GRADER_RUBRIC

An LLM judge or rubric contains inappropriate evaluation criteria.

Examples:

* overly strict requirements
* ambiguous criteria
* stylistic preferences treated as correctness
* conflicting requirements
* requiring information the user did not request

## GRADER_RUBRIC_MISALIGNMENT

The grader expects the wrong representation of correct behavior or assigns responsibility to the wrong component.

Example:

The system correctly handles an unsupported request through routing, but the grader requires the final response to literally contain "outside my scope."

## GRADER_OBSERVABILITY_GAP

The required behavior exists, but the evaluator cannot observe the layer where it occurs.

Example:

The agent calls a destructive tool.

The runtime intercepts the call and requires human approval.

A grader looking only at the final assistant response incorrectly concludes that approval was missing.

## REFERENCE_OR_DATASET

Evaluation reference information is incorrect, incomplete, brittle, outdated, or inappropriate.

Examples:

* wrong expected answer
* overly specific expected keywords
* mislabeled case
* ambiguous test case
* outdated reference
* reference requires unrequested information

## INFRASTRUCTURE

Evaluation/tracing infrastructure caused misleading evidence.

Examples:

* wrong score associated with trace
* evaluator did not run
* missing observations
* incomplete telemetry
* corrupted metadata

## INCONCLUSIVE

Available evidence does not justify a stronger conclusion.

Never force certainty.

---

# Autonomous Investigation Strategy

There is NO mandatory fixed sequence of investigation steps.

You are responsible for determining the most informative next action based on the evidence currently available.

Continuously follow this investigation loop:

CURRENT EVIDENCE

↓

WHAT DON'T I UNDERSTAND?

↓

WHAT HYPOTHESES COULD EXPLAIN IT?

↓

WHAT EVIDENCE WOULD DISTINGUISH THOSE HYPOTHESES?

↓

WHICH AVAILABLE TOOL OR SOURCE CAN PROVIDE THAT EVIDENCE?

↓

GATHER EVIDENCE

↓

UPDATE / REJECT / STRENGTHEN HYPOTHESES

↓

REPEAT

The investigation path should emerge dynamically from the evidence.

One investigation might naturally follow:

grader implementation
→ dataset
→ Langfuse traces
→ passing controls
→ agent prompt

Another might follow:

Langfuse failures
→ tool behavior
→ source code
→ middleware
→ grader observability

Both are valid.

Choose the path that maximizes useful information.

Do not perform an action merely because it is mentioned in this skill.

---

# Autonomy Rules

When given a request such as:

"Investigate keyword_overlap."

or:

"Why is the correctness evaluator performing poorly?"

perform the investigation autonomously.

Do not repeatedly ask the user:

* Should I inspect traces?
* Should I inspect passing examples?
* Should I inspect GitHub?
* Should I inspect the grader?
* Should I inspect the prompt?
* Should I inspect the dataset?
* Should I inspect tools?
* Should I inspect middleware?
* Should I continue?
* Should I create the report?

If available tools can answer the question, proceed.

Ask the user only when information genuinely cannot be resolved using available tools, for example:

* required credentials are unavailable
* the relevant repository cannot be identified
* multiple targets cannot be disambiguated
* required evidence cannot be accessed
* destructive action requires approval

Otherwise continue investigating.

---

# Available Evidence

## Runtime / Evaluation Evidence

Use Langfuse or equivalent connected evaluation tooling to inspect:

* traces
* observations
* scores
* grader feedback
* evaluator comments
* model calls
* tool calls
* tool arguments
* tool outputs
* errors
* retries
* metadata
* datasets
* dataset runs
* reference outputs
* expected values
* evaluator metadata

## Source-Code Evidence

Use GitHub or equivalent repository access to inspect:

* grader implementations
* grader prompts/rubrics
* score calculations
* expected/reference data
* datasets
* agent implementation
* system prompts
* developer prompts
* tool definitions
* tool schemas
* tool implementations
* wrappers
* routing
* middleware
* approval logic
* runtime configuration
* post-processing
* README files
* architecture documentation
* configuration
* imports
* callers
* callees

Runtime evidence explains WHAT happened.

Implementation evidence helps explain WHY.

Use both whenever appropriate.

---

# Hypothesis-Driven Investigation

Maintain competing hypotheses during important investigations.

Possible hypotheses may include:

H1 — the agent actually failed

H2 — the agent prompt caused the behavior

H3 — agent implementation caused the behavior

H4 — tool behavior caused the failure

H5 — grader implementation contains a bug

H6 — grader implementation is correct but design is poor

H7 — grader rubric is misaligned

H8 — reference/dataset information is wrong

H9 — required behavior exists elsewhere in the system

H10 — grader cannot observe the responsible system layer

H11 — evaluation infrastructure produced misleading evidence

At each stage ask:

* Which hypotheses remain plausible?
* What supports each hypothesis?
* What contradicts each hypothesis?
* What evidence would best distinguish them?
* Which available tool can obtain that evidence?

Do not stop at the first plausible explanation.

---

# Evaluator Understanding

Before reaching conclusions, understand what the evaluator actually does.

Determine when possible:

* evaluator name
* evaluator type
* deterministic vs LLM judge vs human vs unknown
* inputs
* reference data
* scoring mechanism
* thresholds
* what high scores mean
* what low scores mean
* what behavior it tries to measure
* what system layers it observes
* what it cannot observe
* known limitations

When source code exists, inspect implementation rather than inferring behavior from the evaluator name.

For example:

`keyword_overlap`

does not automatically prove:

* case sensitivity
* substring matching
* equal keyword weighting
* normalization behavior

Inspect the actual implementation when relevant.

---

# Mandatory Grader Explanation

Every completed investigation must first explain the grader itself.

The explanation must answer:

1. What does this grader evaluate?
2. What type of grader is it?
3. What inputs does it consume?
4. How is its score produced?
5. What reference information does it use?
6. What does a high score represent?
7. What does a low score represent?
8. What behavior can it observe?
9. What behavior can it not observe?
10. What limitations were discovered?

This explanation MUST appear first in the final interactive report.

A developer should understand the evaluator before seeing failure statistics.

---

# Trace Cohorts

Use passing traces as controls whenever practical.

Useful cohorts include:

## FAILING

Clearly low-scoring traces according to evaluator semantics.

## PARTIAL

Intermediate or ambiguous scores.

## PASSING / CONTROL

High-scoring traces representing successful evaluator outcomes.

Do not assume every non-perfect score is a failure.

Determine meaningful thresholds from evaluator semantics.

Do not diagnose systematic behavior using failures alone when passing controls are available.

---

# Trace Reconstruction

For relevant traces retrieve as much execution context as available:

* trace ID
* trace URL
* user input
* system/developer instructions
* final response
* model calls
* tool calls
* tool inputs
* tool outputs
* errors
* retries
* relevant context/state
* grader score
* grader feedback
* expected/reference answer
* expected keywords
* labels
* metadata

Reconstruct the execution path when useful.

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

Use descriptive failure modes.

Good examples:

* valid_paraphrase_penalized
* empty_result_triggers_wrong_fallback
* approval_enforced_outside_grader_visibility
* agent_stops_after_recoverable_error
* reference_requires_unrequested_information
* grader_penalizes_formatting_difference
* wrong_tool_selected_after_partial_result
* grader_requires_literal_scope_phrase
* tool_description_encourages_wrong_fallback

Avoid vague labels:

* bad_answer
* grader_problem
* failed_eval
* low_score

Use INCONCLUSIVE when necessary.

---

# Deep Source-Code Investigation

Runtime evidence may not explain why behavior occurred.

When implementation evidence could resolve uncertainty, inspect the repository automatically.

Do not stop with:

"I cannot see the expected keywords."

if repository access can locate them.

Do not guess implementation behavior when source code is available.

---

# Grader Investigation

When relevant, inspect:

* grader implementation
* evaluator configuration
* grader prompt
* rubric
* scoring logic
* normalization
* parsing
* reference fields
* expected values
* expected keywords
* dataset construction

Determine:

* what the grader actually computes
* whether implementation matches specification
* whether implementation contains a bug
* whether the design itself is inappropriate
* what information the grader observes
* what important context it does not receive

---

# Dataset / Reference Investigation

When relevant, locate:

* expectedKeywords
* expectedOutput
* idealAnswer
* referenceAnswer
* labels
* fixtures
* dataset construction

Determine whether reference information is:

* correct
* ambiguous
* overly specific
* outdated
* incomplete
* stylistic rather than semantic

Never guess hidden reference values when they can be found in source code.

---

# Agent Prompt Investigation

When relevant, inspect:

* system prompt
* developer prompt
* few-shot examples
* tool-selection rules
* scope rules
* fallback rules
* safety requirements
* retry instructions
* agent configuration

Compare runtime behavior against actual instructions.

Do not recommend changing a prompt you have not inspected when prompt evidence is available.

---

# Tool Investigation

When relevant, inspect:

* tool description
* parameters/schema
* implementation
* return format
* error handling
* wrappers
* fallback logic
* permission/approval behavior

Determine whether observed behavior originates from:

* agent reasoning
* misleading tool description
* tool response semantics
* tool implementation
* surrounding wrappers

---

# Full Code-Path Investigation

Do NOT stop after finding the first relevant file.

Follow behavior across the application when necessary.

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

Follow relevant:

* imports
* callers
* callees
* registrations
* configuration
* wrappers
* middleware

until responsibility is understood.

---

# Responsibility Mapping

For every major root cause determine:

## Grader Expectation

What behavior does the evaluator expect?

## Intended Owner

Which component SHOULD own this behavior?

Possible owners:

* agent prompt
* agent reasoning
* agent implementation
* tool
* tool wrapper
* middleware
* router
* approval gate
* runtime/harness
* backend
* frontend
* evaluator

## Actual Owner

Where is the behavior actually implemented?

## Runtime Behavior

What happened in the trace?

## Grader Visibility

Can the grader observe the responsible layer?

## Evaluation Mismatch

Is the grader checking the wrong component or representation?

This responsibility mapping is mandatory for major findings.

---

# Search Before Declaring Behavior Missing

This is a critical investigation rule.

When a grader claims something is missing, do NOT immediately recommend adding it to the agent prompt.

Search the relevant system path.

Determine whether the behavior is:

* genuinely missing
* implemented elsewhere
* handled by routing
* enforced by middleware
* enforced by tool wrappers
* enforced by runtime/harness
* handled by backend logic
* handled by frontend/UI
* intentionally delegated to another component
* impossible for the evaluated component to control

Example:

Grader:

"Agent failed to request approval before destructive action."

Trace:

Agent → delete_user()

Before declaring AGENT_PROMPT failure, investigate:

* delete_user
* approval configuration
* tool wrappers
* middleware
* runtime configuration
* approval layer

If the system actually performs:

delete_user
→ requiresApproval=true
→ runtime pauses
→ human approves
→ execution

then likely diagnosis:

GRADER_OBSERVABILITY_GAP

and possibly:

GRADER_RUBRIC_MISALIGNMENT

Do NOT recommend changing the agent merely to satisfy an evaluator that observes the wrong layer.

---

# Application Architecture

When useful inspect:

* README
* architecture documentation
* comments
* agent setup
* tool registration
* runtime configuration
* middleware
* evaluation documentation

Understand intended division of responsibility before recommending changes.

Do not move behavior into the model prompt when architecture intentionally implements it elsewhere.

---

# Passing-vs-Failing Analysis

Compare passing and failing traces.

Look for characteristics disproportionately associated with failures.

Potential dimensions:

* tool choice
* tool sequence
* empty results
* tool errors
* retries
* fallback behavior
* request category
* response format
* wording
* refusal behavior
* routing
* conversation state
* dataset category
* code path

Prefer evidence like:

"18 of 22 failing traces take path X, compared with 2 of 31 passing traces."

over:

"Some failing traces take path X."

Never invent counts.

---

# Failure Clustering

Group individual diagnoses into meaningful recurring failure categories.

For each category determine:

* unique category ID
* category name
* plain-English description
* root-cause category
* affected trace count
* percentage of failures
* confidence
* severity
* representative traces
* all trace IDs when practical
* runtime pattern
* code evidence
* responsible layer
* suggested fix

Prefer mutually exclusive primary diagnoses for the main failure distribution.

If categories overlap, explicitly state that percentages may overlap.

Percentages must use verified counts.

---

# Grader Audit

Every investigation should assess grader health when evidence permits.

Evaluate:

## Implementation

Does implementation behave according to specification?

## Design

Does the metric correspond to actual agent quality?

## Rubric

Is the rubric clear and appropriate?

## Observability

Can the grader observe the behavior it expects?

## Reference Data

Are expected answers/keywords/labels appropriate?

## Consistency

Are semantically similar outputs treated consistently?

Clearly distinguish:

* Healthy
* Implementation Bug
* Design Problem
* Rubric Issue
* Rubric Misalignment
* Observability Gap
* Reference/Data Problem
* Unclear

---

# Agent Audit

Evaluate actual agent health separately.

Inspect when relevant:

* prompt quality
* instruction clarity
* tool selection
* tool descriptions
* tool/data handling
* fallback behavior
* error handling
* retry behavior
* hallucinations
* premature termination
* scope adherence
* handling missing information

Do not modify a correct production agent merely to improve a bad evaluator score.

Bad recommendation:

"Add the expected keywords to the system prompt."

Better recommendation:

"The production response is semantically correct. Fix the evaluator rather than optimizing the agent for lexical overlap."

---

# Recommendation Targeting

Determine where each fix belongs.

Possible targets:

* AGENT_PROMPT
* AGENT_CODE
* TOOL_DESCRIPTION
* TOOL_IMPLEMENTATION
* ROUTING
* MIDDLEWARE
* RUNTIME/HARNESS
* GRADER_CODE
* GRADER_PROMPT
* GRADER_RUBRIC
* DATASET
* REFERENCE_DATA
* INFRASTRUCTURE
* NOTHING

Prefer the smallest change addressing the underlying cause.

Do not modify source code unless explicitly authorized.

---

# Recommendation Prioritization

Prioritize recommendations by:

1. expected impact
2. evidence strength
3. confidence
4. implementation effort
5. blast radius

For each recommendation provide:

* priority
* target
* recommended change
* reason
* supporting evidence
* likely repository/file
* expected impact
* confidence

---

# Evidence Standard

Every major conclusion must be evidence-backed.

Whenever possible combine:

## Runtime Evidence

Langfuse traces, scores, observations, tool calls, evaluator feedback.

## Code Evidence

Repository implementation.

## Architectural Evidence

Configuration/documentation showing intended responsibility.

Clearly distinguish:

VERIFIED FACT

from:

HYPOTHESIS

Never fabricate:

* trace contents
* scores
* counts
* percentages
* file paths
* code
* grader behavior
* expected keywords
* reference answers
* tool behavior
* trace URLs

Use:

Unknown

Not verified

Insufficient evidence

when appropriate.

---

# Confidence

Use evidence-based confidence.

## HIGH

Runtime evidence and implementation evidence strongly agree.

## MEDIUM

Strong runtime pattern but incomplete implementation evidence.

## LOW

Limited sample, incomplete evidence, or unresolved competing explanations.

Confidence must reflect evidence strength rather than how plausible an explanation sounds.

---

# Scalable Trace Analysis

Choose the analysis strategy dynamically based on workload and complexity.

These thresholds are guidelines, not hard rules.

## Small Evaluation Sets

For roughly fewer than 20 failing traces:

* analyze directly
* avoid unnecessary delegation
* include relevant passing/control traces

## Medium Evaluation Sets

For roughly 20–100 failing traces:

Use TrueForge subagents when parallel investigation would materially improve speed or context management.

Divide traces into logical batches, typically around 20–30 traces per batch when appropriate.

Spawn Trace Investigator subagents dynamically.

Each Trace Investigator should return structured diagnoses equivalent to:

{
"trace_id": "...",
"score": null,
"failure_mode": "...",
"root_cause": "...",
"confidence": null,
"evidence": "...",
"suggested_fix": "..."
}

Provide representative passing/control traces where useful.

## Large Evaluation Sets

For roughly more than 100 failing traces:

Do NOT blindly send every complete trace to an LLM.

First inspect lightweight evidence such as:

* score
* grader feedback
* trace metadata
* tool sequence
* errors
* dataset category

Identify candidate cohorts and patterns.

Then use subagents to investigate representative batches.

Expand investigation around discovered patterns until prevalence can be validated against the broader trace population.

Maintain passing/control samples.

---

# Specialized Subagents

The parent Eval Investigator may dynamically create specialized investigators when doing so materially improves the investigation.

These are capabilities, NOT mandatory workflow steps.

## Trace Investigator

Useful for:

* analyzing batches of traces
* comparing passing/failing behavior
* identifying recurring execution patterns
* returning structured per-trace diagnoses

## Grader Auditor

Useful for:

* evaluator implementation
* grader prompts/rubrics
* scoring logic
* reference data
* consistency
* observability limitations

## Code Investigator

Useful for:

* agent prompt
* tools
* middleware
* routing
* runtime behavior
* code paths
* responsibility ownership

Do not spawn specialized agents simply because they exist.

Delegate only when useful based on:

* workload
* current hypotheses
* context size
* opportunity for parallel analysis
* need for specialized investigation

---

# Parent-Agent Responsibility

Subagents gather and analyze evidence.

The parent Eval Investigator owns the final investigation.

Never simply concatenate subagent responses.

The parent must:

* evaluate findings critically
* reconcile conflicting diagnoses
* deduplicate failure modes
* validate aggregate claims
* connect runtime evidence with code evidence
* determine final root causes
* calculate final category prevalence
* determine recommendations
* generate the structured investigation result
* generate the final interactive report

---

# Investigation Completion Criteria

The exact investigation sequence is dynamic.

Do not stop merely because one plausible explanation was found.

Finish when enough evidence exists to answer the relevant questions:

* What does the grader actually measure?
* What produced the low score?
* Is the behavior genuinely incorrect?
* How common is the pattern?
* How do passing traces differ?
* What implementation produced the behavior?
* Which system layer owns the behavior?
* Is supposedly missing behavior implemented elsewhere?
* Can the grader observe the responsible layer?
* Could grader design/implementation/rubric/reference data be responsible?
* Were meaningful competing hypotheses considered?
* What evidence supports the final diagnosis?
* What should be changed?

Not every investigation requires every evidence source.

Use judgment.

Do not inspect middleware when middleware is irrelevant.

Do not inspect tool code when no tool is involved.

Do not fetch hundreds of full traces when targeted analysis plus prevalence validation is sufficient.

Stop when:

* major patterns are understood
* important hypotheses are tested
* evidence supports recommendations

OR

available evidence/tools are exhausted.

Clearly report unresolved uncertainty.

---

# Structured Investigation Result

Before rendering the report, organize validated findings into structured data.

This structured result is the SOURCE OF TRUTH for downstream presentation.

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
"cohort": "...",
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

Do not invent values merely to populate the structure.

---

# Langfuse Trace Links

Developers should be able to move directly from the investigation report to original runtime evidence.

For every analyzed trace, attempt to retain:

* trace_id
* trace_url
* score
* failure category
* concise diagnosis

Use a Langfuse trace URL only when:

1. returned directly by available tooling, OR
2. safely constructible from verified Langfuse host/project/trace identifiers

Never fabricate URLs.

If unavailable, display:

Langfuse link unavailable

When supported, external Langfuse links should open in a new tab.

---

# Interactive HTML Investigation Report

After investigation is complete, ALWAYS create an interactive HTML investigation report using the available web artifact/report-building skill.

Do not ask permission.

Do not use Notion.

The HTML artifact is the primary human-facing deliverable.

Use the validated structured investigation result as the source of truth.

Do not independently regenerate conclusions while building the UI.

---

# HTML Design Philosophy

The report should feel like an engineering debugging console.

It should NOT look like:

* a marketing page
* a generic analytics dashboard
* a long markdown report converted directly to HTML

The developer workflow should be:

UNDERSTAND GRADER

↓

SEE FAILURE DISTRIBUTION

↓

SELECT FAILURE CATEGORY

↓

SEE ASSOCIATED TRACES

↓

INSPECT TRACE DETAILS

↓

OPEN ORIGINAL LANGFUSE TRACE

↓

UNDERSTAND ROOT CAUSE / CODE PATH

↓

SEE RECOMMENDED FIX

The report must be:

* interactive
* structured
* color-coded
* evidence-first
* developer-oriented
* easy to scan
* drill-down friendly

---

# Mandatory HTML Structure

## 1. Grader Overview

This MUST be the first section.

Use a title such as:

"What does `keyword_overlap` evaluate?"

Explain:

* grader purpose
* grader type
* behavior measured
* scoring mechanism
* inputs/reference data
* meaning of high score
* meaning of low score
* observable system layers
* limitations

Show summary cards for:

* grader name
* total traces
* failing traces
* partial traces when relevant
* passing traces
* overall/average score when meaningful
* grader health
* agent health

A developer must understand the grader before seeing failure categories.

---

## 2. Failure Category Distribution

Immediately after the grader overview show a prominent interactive table.

Required columns:

| Failure Category | Root Cause | Trace Count | % of Failures | Confidence | Severity |

Example:

| Failure Category           | Root Cause           | Trace Count | % of Failures | Confidence | Severity |
| -------------------------- | -------------------- | ----------: | ------------: | ---------- | -------- |
| Valid paraphrase penalized | GRADER_DESIGN        |          11 |           61% | High       | High     |
| Over-specific reference    | REFERENCE_OR_DATASET |           4 |           22% | High       | Medium   |
| Actual agent failure       | AGENT_BEHAVIOR       |           2 |           11% | Medium     | Medium   |
| Tool issue                 | TOOL_OR_DATA         |           1 |            6% | High       | Medium   |

Requirements:

* categories come from actual investigation
* counts are verified
* percentages use verified counts
* denominator is clear
* categories preferably use mutually exclusive primary diagnoses
* overlapping categories must be explicitly labeled

This table is the central navigation element.

---

## 3. Clickable Failure Categories

Every failure-category row must be clickable or expandable.

Clicking a category reveals:

* category name
* plain-English explanation
* root cause
* affected trace count
* percentage
* confidence
* severity
* common runtime pattern
* code pattern when relevant
* recommended fix
* all associated analyzed traces

The developer should never need to manually search for traces belonging to a category.

---

## 4. Trace List for Selected Category

For every trace display:

* trace ID
* grader score
* user-input preview
* concise diagnosis
* grader feedback when available
* confidence
* verified Langfuse link when available

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
Semantically correct paraphrase without exact reference phrase.

Passing:
Equivalent answer containing exact phrase expected by grader.

Only display comparisons supported by evidence.

---

## 7. Root-Cause Deep Dive

For every major root cause show an expandable card.

Include:

* root-cause title
* category
* confidence
* affected traces
* percentage
* grader expectation
* actual behavior
* runtime evidence
* code evidence
* relevant repository files
* responsible system layer
* whether behavior exists elsewhere
* whether grader observes correct layer
* competing hypotheses
* final diagnosis
* recommended fix

---

## 8. Responsibility and Code-Path Analysis

For important findings show:

Grader Expectation
→ Intended Owner
→ Actual Owner
→ Runtime Behavior
→ Grader Visibility
→ Evaluation Mismatch

When useful show:

User
→ Router
→ Agent
→ Tool
→ Middleware
→ Runtime/Harness
→ Final Response
→ Grader

Highlight:

* where grader expects behavior
* where behavior actually occurs
* which layer owns responsibility
* where mismatch occurs

Especially important for:

* GRADER_OBSERVABILITY_GAP
* GRADER_RUBRIC_MISALIGNMENT
* AGENT_PROMPT
* TOOL_OR_DATA

---

## 9. Grader Health

Display:

* implementation correctness
* design alignment
* rubric quality
* observability
* reference-data quality
* consistency
* overall grader verdict

Possible statuses:

* Healthy
* Design Concern
* Rubric Misalignment
* Implementation Bug
* Observability Gap
* Reference Problem
* Unclear

---

## 10. Agent Health

Display separately:

* prompt quality
* tool selection
* tool/data handling
* reasoning/behavior
* fallback behavior
* error handling
* overall agent verdict

Do not mix grader issues into agent health.

---

## 11. Recommended Actions

Display prioritized action cards.

For each include:

* priority
* change target
* recommended change
* reason
* supporting evidence
* expected impact
* confidence
* repository/file where relevant

Clearly label:

* FIX AGENT
* FIX GRADER
* FIX DATASET
* FIX TOOL
* FIX INFRASTRUCTURE
* NO CHANGE REQUIRED

Highest-impact evidence-backed recommendation appears first.

---

## 12. Uncertainties

Clearly show:

* unavailable evidence
* unverified hypotheses
* missing code evidence
* missing reference data
* unavailable Langfuse links
* incomplete trace coverage
* low-confidence conclusions

Never hide uncertainty.

---

# HTML Interaction Requirements

Required:

1. Failure-category rows are clickable/expandable.
2. Clicking a category shows associated traces.
3. Trace rows are clickable/expandable.
4. Verified Langfuse links are clickable.
5. User can return to category overview.

Recommended where useful:

* filter by failure category
* filter by root-cause type
* filter by severity
* filter by confidence
* sort by score
* sort by trace count
* search by trace ID
* expand/collapse root-cause cards
* toggle runtime evidence vs code evidence

Do not add interactions that do not help debugging.

---

# Color Semantics

Use consistent semantic colors.

## RED

Confirmed high-impact agent/tool/implementation issue.

## ORANGE

Actionable grader/design/rubric issue.

## YELLOW

Potential concern, partial evidence, or medium confidence.

## GREEN

Healthy / passing / no issue.

## GRAY

Unknown / inconclusive / unverified.

Never communicate meaning through color alone.

Always include text labels.

---

# Interactive Report Quality Requirements

Before considering the artifact complete verify:

* grader explanation appears first
* failure-category table immediately follows
* categories have verified counts
* percentages are mathematically valid
* category denominator is clear
* clicking category shows correct traces
* each trace belongs to selected category
* trace scores match evidence
* verified Langfuse links work when available
* no Langfuse URL was invented
* root causes contain runtime/code evidence
* grader health and agent health are separate
* recommendations target the correct component
* uncertainty is visible
* color semantics are consistent
* UI is developer-focused

---

# Investigation Quality Checklist

Before completion verify:

* Do I understand what the grader actually does?
* Did I inspect sufficient runtime evidence?
* Did I use passing controls where useful?
* Did I inspect grader implementation when relevant?
* Did I inspect reference data when relevant?
* Did I inspect agent prompt when relevant?
* Did I inspect tools when relevant?
* Did I follow relevant code paths beyond the first obvious file?
* Did I check whether supposedly missing behavior exists elsewhere?
* Did I identify the responsible system layer?
* Can the grader observe that layer?
* Did I consider competing hypotheses?
* Did I validate aggregate counts?
* Did I avoid automatically blaming the agent?
* Did I avoid recommending changes merely to game the evaluator?
* Are conclusions evidence-backed?
* Is confidence calibrated?
* Are failure categories meaningful?
* Are trace-category mappings correct?
* Are percentages verified?
* Did I create the interactive report?
* Are verified Langfuse links included where possible?

If important questions remain unresolved and available tools can answer them, continue investigating.

---

# Deep-Dive Example

Suppose a grader says:

"Agent failed to request confirmation before deleting a user."

Trace:

Agent
→ delete_user()

A shallow investigation says:

"Add confirmation instructions to the system prompt."

Do NOT do that.

Instead investigate dynamically.

Potential evidence gathering:

* inspect trace
* inspect grader definition
* search repository for delete_user
* inspect tool registration
* search approval configuration
* inspect middleware
* inspect runtime/harness configuration
* inspect passing traces

Suppose code reveals:

delete_user
→ destructive tool
→ requiresApproval=true
→ runtime pauses
→ human approval
→ execution

Then:

The safety behavior already exists.

The agent is not necessarily unsafe.

The grader is evaluating the wrong layer.

Likely diagnosis:

GRADER_OBSERVABILITY_GAP

possibly combined with:

GRADER_RUBRIC_MISALIGNMENT

Recommended change:

Do NOT change production agent prompt.

Change evaluator so it checks whether destructive execution was approval-gated.

The interactive report should categorize affected traces under something like:

"Approval enforced outside grader visibility"

Clicking that category should reveal every trace exhibiting that pattern.

Each trace should provide an "Open in Langfuse" link when a verified URL exists.

This level of investigation is expected.

---

# Final Chat Response

After creating the interactive report, keep the chat response concise.

Include:

* grader investigated
* primary finding
* number of failing traces analyzed
* number of failure categories discovered
* highest-priority recommendation
* reference/link to interactive report when available

Do not duplicate the full investigation in chat.

The interactive artifact is the detailed deliverable.

---

# Final Objective

The engineer should not have to manually:

* understand the grader
* inspect every failed trace
* compare successful traces
* inspect evaluator feedback
* find evaluator implementation
* locate expected values
* find system prompts
* inspect tool implementations
* trace middleware/runtime behavior
* understand responsibility ownership
* determine whether grader or agent is wrong
* group similar failures
* calculate failure percentages
* manually copy trace IDs into Langfuse
* decide what component to fix

You perform that investigation.

A successful investigation answers:

WHAT DOES THIS GRADER ACTUALLY DO?

WHAT IS FAILING?

HOW OFTEN?

WHAT FAILURE CATEGORIES EXIST?

WHAT PERCENTAGE DOES EACH CATEGORY REPRESENT?

WHICH TRACES BELONG TO EACH CATEGORY?

WHY ARE THEY FAILING?

WHERE DOES THE BEHAVIOR ORIGINATE?

WHICH SYSTEM LAYER OWNS IT?

IS THE AGENT ACTUALLY WRONG?

IS THE GRADER ACTUALLY WRONG?

IS REQUIRED BEHAVIOR ALREADY HANDLED ELSEWHERE?

CAN THE GRADER OBSERVE THE RESPONSIBLE LAYER?

WHAT EVIDENCE SUPPORTS THE CONCLUSION?

WHAT SHOULD THE ENGINEER CHANGE FIRST?

CAN THE ENGINEER IMMEDIATELY OPEN THE ORIGINAL TRACE IN LANGFUSE?

The investigation strategy must remain autonomous and evidence-driven.

The final report structure must remain predictable, interactive, and optimized for engineering debugging.
