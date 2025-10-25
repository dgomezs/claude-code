---
allowed-tools: Task(*)
argument-hint: <task-directory>
description: Continue TDD implementation using orchestrator agent
---

Execute TDD orchestration for: $ARGUMENTS

**STEP 0: Validate Prerequisites**

Before starting TDD, verify task directory contains required files:

1. **Parse task directory** from $ARGUMENTS
2. **Check required files exist** using Read tool:
   - `scenarios.md` - REQUIRED (contains scenarios with TDD tracking checkboxes)
   - `test-scenarios/` directory - REQUIRED (contains happy-path.md, error-cases.md, edge-cases.md)
   - `tech-design.md` - REQUIRED (provides architectural guidance and implementation strategy)

3. **If any required file missing:**
```
❌ Prerequisites validation failed

Missing required files:
- [list missing files]

Run the workflow commands to generate required files:
/test-scenarios [task-directory]
/create-tech-design [task-directory]
```
STOP execution - do not proceed.

4. **If all required files exist:**
```
✅ Prerequisites validated
- scenarios.md ✓
- test-scenarios/ ✓
- tech-design.md ✓
```
Proceed to STEP 1.

**STEP 1: Read scenarios.md** in the task directory and find the first scenario that needs work.

**STEP 2: Detect phase** by checking the scenario's checkboxes:
- `[ ] [ ] [ ]` = RED phase → Launch tdd-red agent
- `[x] [ ] [ ]` = GREEN phase → Launch tdd-green agent
- `[x] [x] [ ]` = REFACTOR phase → Launch tdd-refactor agent
- `[x] [x] [x]` = Complete → Move to next scenario

If all scenarios are complete, report task completion.

**STEP 3: Launch the appropriate agent** with Task tool:
- RED: `subagent_type: "tdd-red"` - Agent will write failing test
- GREEN: `subagent_type: "tdd-green"` - Agent will implement minimal code
- REFACTOR: `subagent_type: "tdd-refactor"` - Agent will improve code quality

Pass the task directory path to the agent. Agents will use tech-design.md for architectural guidance and implementation strategy, scenarios.md for tracking progress, and test-scenarios/ for detailed scenario specifications.

**STEP 4: After agent completes**, show:
- What phase completed
- Current progress (all scenarios with checkbox states)
- Suggested commit message

**STEP 5: Ask user** what to do next:
1. Commit and continue to next phase/scenario
2. Stop here to review
3. Continue without committing
4. Skip refactoring (only when in REFACTOR phase) - move directly to next scenario

**Notes:**
- Option 4 is only available when in REFACTOR phase (scenario shows `[x] [x] [ ]`)
- Skipping refactoring is acceptable since scenarios are functionally complete after GREEN phase
- Do NOT automatically continue to the next phase.