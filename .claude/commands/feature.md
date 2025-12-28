# Feature Planning

- Create the plan in the `specs/*.md` file. Name it appropriately based
  on the  `Feature`. The filename should start with the today's date, example 2025-11-13-<feature>.md

- Follow the  Instructions to create the plan use the `Relevant Files` to focus on the right files.

- DO NOT IMPLEMENT THE PLAN

## PURPOSE

Produce a **rigid, auditable, implementation-ready specification** for migrating or completing a feature in a new
system, using an old system as the authoritative reference.

## INPUT
- `<feature>` — the feature to be migrated or completed.

## HARD ASSUMPTIONS
- The current system is a rewrite or successor of an older system.
- The old system is the **ground truth** for intended behavior.
- This task produces a **specification only**, not code.

## NON‑NEGOTIABLE RULES
- DO NOT implement code.
- DO NOT redesign or optimize behavior.
- DO NOT guess undocumented behavior; require it to be verified.
- DO NOT leave placeholders.
- DO NOT omit validation requirements.
- ALL claims must be explicit and reviewable.

## REQUIRED PROCESS

### 1. Verify Feature State in Current System
Explicitly determine whether the feature:
- does not exist,
- is partially implemented,
- behaves differently,
- or is already complete.

State the conclusion clearly and justify it.

### 2. Extract Old‑System Ground Truth
Describe the feature as implemented in the old system:
- behavior and semantics
- inputs and outputs
- configuration and flags
- side effects
- error handling
- edge cases

Treat this as binding.

### 3. Map to the New System
Describe how the old behavior maps onto the new system:
- architecture
- APIs
- data models
- constraints

Any deviation MUST be explicitly justified.

### 4. Define Acceptance Criteria
Specify concrete, testable criteria that define “done”, including:
- functional parity
- allowed divergences
- validation expectations

# Plan Format

```md
# Feature: <feature name>

## Chore Description
<describe the feature in detail>

## Relevant Files
Use these files to resolve the chore:

<find and list the files that are relevant to the chore describe why they are relevant in bullet points. If there are new files that need to be created to accomplish the chore, list them in an h3 'New Files' section.
DO NOT REFERENCE LINE NUMBERS.
>

## Step by Step Tasks
IMPORTANT: Execute every step in order, top to bottom.

<list step by step tasks as h3 headers plus bullet points. use as many h3 headers as needed to accomplish the chore. Order matters, start with the foundational shared changes required to fix the chore then move on to the specific changes required to fix the chore. Your last step should be running the `Validation Commands` to validate the chore is complete with zero regressions.>

## Validation Commands
Execute every command to validate the chore is complete with zero regressions.

<list commands you'll use to validate with 100% confidence the chore is complete with zero regressions. every command must execute without errors so be specific about what you want to run to validate the chore is complete with zero regressions. Don't validate with curl commands.>

## Document changes

<if necessary, list steps to document the changes proposed>

## Git log

<summarize the changes in a way that can be used as the log for a git commit>

## Notes
<optionally list any additional notes or context that are relevant to the chore that will be helpful to the developer>
```

## Feature

$ARGUMENTS
