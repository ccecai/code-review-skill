---
name: code-review
description: >-
  Reviews code changes for correctness, coupling, compatibility, robustness,
  concurrency, resource use, and merge readiness, and reports findings in Chinese.
  Use automatically when the user requests an actual review with phrases such as
  "代码审查", "审查代码", "code review", or "review current changes".
  Do not activate merely because these phrases appear in quoted text or a request
  to create or edit a review prompt or skill.
---

# Code Review

## Invocation and boundaries

Apply this workflow automatically to an actual code review request. The user
does not need to repeat the checklist or explicitly name this skill. Honor any
scope or focus supplied with the request. This skill's instructions are in
English; write the review results in plain Chinese, unless the user explicitly
requests another output language.

Determine whether the changes are suitable for merging into the intended target
sprint or branch. Review only: do not fix code unless the user requests fixes.
Do not commit, stash, reset, rebase, switch branches, or alter the user's changes
as part of a review. Inspect another revision without disturbing the working tree.
Follow applicable AGENTS.md instructions and project-specific review rules.

## Establish scope and context

1. If no scope is given, inspect staged changes, unstaged changes, and relevant
   untracked files. If the user instead specifies a branch, PR, commit, or file,
   review that scope rather than silently substituting the working tree.
2. For a merge review, identify the target branch and exact base commit from the
   request and available repository or PR metadata. Do not guess which branch
   means "latest sprint". Use the appropriate merge base to identify introduced
   changes, then inspect relevant target-branch changes for integration conflicts.
   State when local target-branch information has not been verified as current.
3. If an essential base or requirement remains ambiguous, continue independent
   inspection and request only the missing information. Do not claim that target
   integration was checked when the target is unknown.
4. Read beyond the diff: understand the affected behavior, callers, callees,
   shared state, configuration, hardware interactions, and relevant tests.
   Trace related paths to check whether a fix reaches the shared root cause.
5. If the requested scope has no changes, say so. Do not invent findings or
   silently expand to unrelated code.

## Review checklist

Apply the relevant checks below. Hardware-specific checks apply when the change
affects hardware; do not force unrelated checks onto other projects.

### 1. Behavior and root cause

Compare the implementation with requirements, the original bug trigger, and
expected results. Check normal, failure, and boundary cases. Determine whether
the root cause is fixed across affected callers, rather than masking one symptom.
Identify regressions introduced by the change. Clearly label pre-existing issues
that become relevant; do not present unrelated old defects as new regressions.

### 2. Responsibilities, layers, and coupling

Check whether the code belongs in its module and layer. Look for unrelated
features tied together, bypassed interfaces, direct access to another module's
internal state or hardware, circular dependencies, and multiple owners of the
same state or hardware output. Check whether a shared fix would be smaller and
safer than separate fixes in callers.

### 3. Necessary changes and reuse

Within correctness, safety, and maintainability requirements, check whether the
diff is limited to what the task needs. Look for existing functions, interfaces,
standard-library features, native facilities, and established project patterns
before proposing new code. Flag duplicated behavior, speculative abstractions,
unnecessary dependencies, and unrelated refactoring or formatting when they have
a concrete cost. Never trade required validation, error handling, data protection,
security, accessibility, or hardware calibration for a smaller diff.

### 4. Readability and maintenance

Check whether names and structure explain the main flow to someone unfamiliar
with the change. Comments should explain necessary reasons, limits, timing, and
units. Look for misleading or outdated comments, unnecessarily complex conditions,
duplicated logic, and unexplained constants. Distinguish concrete maintenance
problems from personal style preferences.

### 5. Compatibility and integration

Check affected callers, build configurations, hardware variants, and existing
features. Inspect changes to protocols, interfaces, data structures, defaults,
and persistent data formats. Can upgraded code read existing data safely? If
downgrade support is required, does it still work? Consider interactions with
related changes already in the target sprint, including logical conflicts that
would not cause a textual merge conflict.

### 6. States and execution order

Trace startup, initialization, operation, failure, recovery, shutdown, and restart.
Check repeated calls, duplicate or reordered events, and interrupted operations.
Look for stale state, incomplete cleanup, timers left running, and old callbacks
affecting a later operation.

### 7. Failure handling and recovery

Check failed calls, timeouts, unresponsive devices, full queues, and insufficient
memory. Are resources released and necessary state restored on every exit path?
Does hardware remain safe during failure and after recovery? Are retries bounded?
Look for ignored errors and later steps that execute after an earlier step failed.
For persistent updates, consider interruption or power loss and partial writes.

### 8. Data correctness and input protection

Check null pointers, array bounds, lengths, integer overflow, signedness,
conversions, units, precision, and timer wraparound. Validate external inputs and
check whether invalid, stale, or partially updated data can be treated as valid.
Where relevant, inspect authorization, sensitive-data handling, and unsafe input
use. Retain necessary limits and calibration for real hardware behavior.

### 9. Concurrency and conflicting operations

Trace shared data and calls across tasks, interrupts, timers, and callbacks.
Check races, inconsistent snapshots, lifetime errors, double release, deadlocks,
lock ordering, and priority inversion. Check whether synchronization covers the
whole shared operation; volatile alone does not establish synchronization.
Look for blocking operations in contexts where they are prohibited, excessive
critical sections, competing peripheral access, and overwritten hardware settings.

### 10. Memory, timing, power, and hardware costs

Assess RAM, Flash, heap, task-stack growth, leaks, allocation churn, and stack
overflow risks. Check CPU use, polling, logging, busy waits, lock waits, interrupt
latency, disabled-interrupt time, and blocking calls. Could these delay responses
or protection actions? Could the change increase wakeups, prevent sleep, monopolize
a bus, or cause excessive Flash writes?

Minimize resource use while meeting functional, safety, timing, and reliability
requirements. Explain tradeoffs instead of pursuing the lowest number regardless
of consequences. Support significant resource changes with available build maps,
measurements, or a defensible estimate. Label estimates and unmeasured risks;
never invent RAM, Flash, CPU, power, or timing figures.

### 11. Validation and evidence

Check coverage of the original reproduction, corrected behavior, affected-feature
regression, boundaries, and important failures. Non-trivial new logic should have
a small useful runnable check, proportionate to the risk. Do not demand tests
that merely mirror implementation or add no useful coverage.

Run appropriate existing safe builds, tests, and project-required checks when
feasible. Do not flash devices, change live hardware outputs, or run destructive
tests without authorization. Separate static inspection from build, test, and
actual-device evidence. Record failures and unavailable checks accurately.
Never describe an assumption as a passing test.

## Chinese review report

Lead with actionable findings, ordered by severity:

- P0: Critical safety/security problems or widespread failures requiring immediate
  action.
- P1: Problems that must be fixed before merging.
- P2: Ordinary defects that should be fixed.
- P3: Low-impact defects.

For each finding, provide a short title with severity, an exact file path and
verified line number, the relevant function when useful, the trigger, the concrete
consequence, supporting evidence or call relationships, and the smallest practical
fix. Anchor each finding to the changed code where possible; cite related code
when needed to explain the impact. Keep line references narrow and accurate.

Separate confirmed defects from risks requiring validation. Explain what evidence
is missing and why the risk matters. Do not assert a defect based on an imagined
caller or unsupported scenario. Avoid duplicate findings and do not pad the report.
Put optional improvements separately from defects, without assigning defect
severity to personal preferences.

Finish with the reviewed scope and base commit when applicable, validation run,
validation not run, and whether any findings block merging. If no clear defects
were found, say so while retaining the validation limits; do not promise the code
is defect-free or fully verified.

Use short, plain Chinese explanations. Preserve exact identifiers, paths, units,
values, and error messages needed to understand or reproduce a finding. Use a
compact list of findings rather than forcing all checklist items into the report.
