---
allowed-tools: Task(*), Read(*), Glob(*), Write(*), Edit(*), Bash(mkdir:*)
argument-hint: <task-directory> [operation-prompt]
description: Create or manage test scenarios from requirements.md using the qa-engineer agent. Supports creating, adding, modifying, or discovering scenarios.
---

# Test Scenarios Command

Execute test scenario creation or management for: $ARGUMENTS

## Overview

This command creates and manages test scenarios from a requirements.md file containing high-level acceptance criteria.

It orchestrates scenario generation by:
1. Determining what operation to perform (create all, add one, modify one, discover gaps)
2. Reading requirements.md and preparing context
3. Calling qa-engineer agent to generate scenario content
4. Handling all file operations (writing, numbering, organizing, linking)

The qa-engineer agent ONLY generates scenario content. This command handles everything else.

## Step 1: Parse Arguments and Determine Operation

### Operation Detection

Analyze `$ARGUMENTS` to determine the operation:

**EXPAND Mode** - Create comprehensive scenarios from requirements.md:
- Single argument: directory path containing `requirements.md`
- Example: `/test-scenarios apps/feature/task-001/`

**ADD Mode** - Add single scenario to existing set:
- First argument: directory path
- Second argument contains "add"
- Example: `/test-scenarios apps/feature/task-001/ "add scenario for null input to AC-1"`

**MODIFY Mode** - Edit existing scenario:
- First argument: directory path
- Second argument contains "modify" or "edit"
- Example: `/test-scenarios apps/feature/task-001/ "modify scenario 1.2 to test empty string"`

**DISCOVER Mode** - Find gaps in existing scenarios:
- First argument: directory path
- Second argument contains "discover" or "gaps"
- Optional: Tag additional context files (e.g., @research.md, @tech-design.md) for deeper analysis
- Example: `/test-scenarios apps/feature/task-001/ "discover gaps"`
- Example with context: `/test-scenarios apps/feature/task-001/ "discover gaps" @research.md @tech-design.md`

**DELETE Mode** - Remove a scenario:
- First argument: directory path
- Second argument contains "delete" or "remove"
- Example: `/test-scenarios apps/feature/task-001/ "delete scenario 1.3"`

## Step 2: Gather Context

All modes use `requirements.md` as the base input containing high-level acceptance criteria.

Read relevant files from the task directory:

### For EXPAND Mode:
```
Required:
- <directory>/requirements.md (high-level acceptance criteria)
```

### For ADD/MODIFY/DISCOVER/DELETE Modes:
```
Required:
- <directory>/requirements.md (high-level acceptance criteria - for context)
- <directory>/scenarios.md (existing scenarios with implementation tracking)
- <directory>/test-scenarios/*.md (existing scenario details)

Optional (if tagged in user prompt):
- Any additional context files (research.md, tech-design.md, code files, etc.)
- These provide deeper context for scenario discovery and analysis
```

## Step 3: Prepare Agent Request

Based on operation mode, formulate a simple request for qa-engineer:

### EXPAND Mode Request:
```
Generate comprehensive test scenarios for these acceptance criteria:

[paste requirements.md content - high-level acceptance criteria]

Generate multiple scenarios per acceptance criterion, ordered by implementation priority.
```

### ADD Mode Request:
```
Generate ONE scenario for AC-[N] to test: [specific behavior]

Acceptance Criterion:
[AC-[N] from requirements.md]

Existing scenarios for AC-[N]:
[list existing scenarios with names only]

Check for duplicates against existing scenarios.
```

### MODIFY Mode Request:
```
Modify scenario [N.M]:

Current scenario:
[paste current scenario]

Current implementation progress: [checkboxes state]

Requested change: [what user specified]

Preserve structure and warn if existing tests/implementation may need updates.
```

### DISCOVER Mode Request:
```
Analyze existing scenarios for gaps and generate NEW scenarios to fill those gaps.

Acceptance Criteria:
[paste all ACs from requirements.md]

Existing Scenarios:
[paste organized list of scenarios by AC with their types]

[IF additional context files were tagged, include them here:]
Additional Context:

[For each tagged file, include a section:]
File: [filename]
[paste file content]

[Repeat for all tagged files]

Generate new scenarios to fill any gaps. If no gaps found, return: "No gaps found - coverage is complete"
```

## Step 4: Invoke qa-engineer Agent

Use Task tool to launch qa-engineer with the prepared request:

```
Task: qa-engineer
Description: Generate test scenario content

[Paste the request from Step 3]
```

The agent is the QA domain expert. It will:
- Ask clarification questions if needed (wait for answers)
- Apply its QA heuristics automatically
- Assign scenario types and priorities based on QA expertise
- Generate scenario content in Given-When-Then format
- Return structured JSON with scenarios, warnings, and optional context requests

**Expected JSON Response:**
```json
{
  "scenarios": [...],
  "warnings": {
    "duplicates": [...],
    "gaps": [...],
    "implementation_impact": [...]
  },
  "context_requests": [...]  // Optional, for DISCOVER mode
}
```

**Handle Context Requests (DISCOVER mode only):**
If agent returns `context_requests` array:
1. Read each requested file
2. Append to the original request with section: "Additional Context:\n[file content]"
3. Re-invoke agent with updated request
4. Repeat until agent returns scenarios or "No gaps found"

## Step 5: Process Agent Output

After receiving JSON output from qa-engineer, perform file operations based on mode:

### For EXPAND Mode:

**Agent returns:** JSON object with scenarios array

**Your tasks:**

1. **Parse JSON response:**
   ```javascript
   const response = JSON.parse(agent_output);
   const scenarios = response.scenarios;
   const warnings = response.warnings;
   ```

2. **Display warnings** if any:
   ```
   ⚠️  Warnings from QA Engineer:
   - Duplicates: [list warnings.duplicates]
   - Gaps: [list warnings.gaps]
   - Implementation Impact: [list warnings.implementation_impact]
   ```

3. **Group scenarios by type** using the `type` field:
   - Filter scenarios where `type === "happy-path"` → happy-path.md
   - Filter scenarios where `type === "error-case"` → error-cases.md
   - Filter scenarios where `type === "edge-case"` → edge-cases.md

4. **Create directory structure**:
   ```bash
   mkdir -p <directory>/test-scenarios
   ```

5. **Write scenario files**:

   For each scenario type file, write header and scenarios:

   **happy-path.md**:
   ```markdown
   # Happy Path Scenarios

   Valid inputs and successful outcomes that represent typical user workflows.

   [For each scenario where type === "happy-path", paste scenario.content]
   ```

   **error-cases.md**:
   ```markdown
   # Error Case Scenarios

   Invalid inputs and failure conditions.

   [For each scenario where type === "error-case", paste scenario.content]
   ```

   **edge-cases.md**:
   ```markdown
   # Edge Case Scenarios

   Boundary values, limits, and unusual but valid conditions.

   [For each scenario where type === "edge-case", paste scenario.content]
   ```

6. **Assign scenario numbers using agent's priority**:
   - Group scenarios by `acceptance_criterion` field (e.g., "AC-1")
   - Sort scenarios within each AC by `priority` field (ascending: 1, 2, 3...)
   - Assign numbers: `AC-N → Scenario N.1, N.2, N.3...`
   - Example: AC-1 scenarios sorted by priority → 1.1, 1.2, 1.3

   **Important**: The agent determines priority based on QA expertise:
   - Priority 1-N: Happy-path scenarios (ordered by business criticality)
   - Priority N+1+: Error-case scenarios
   - Lowest priority: Edge-case scenarios

   The command simply respects the agent's priority ordering.

7. **Generate scenarios.md** with implementation tracking:
   ```markdown
   # Test Scenarios

   This document lists all test scenarios derived from acceptance criteria in requirements.md.
   Each scenario has implementation tracking checkboxes for implementation progress.

   ## AC-1: [Title from requirements.md]

   **Source**: [requirements.md#ac-1](requirements.md#ac-1)

   ### Scenario 1.1: [Name from agent]
   - **Type**: happy-path
   - **Details**: [test-scenarios/happy-path.md#scenario-11](test-scenarios/happy-path.md#scenario-11)
   - **Implementation Progress**: [ ] Test Written [ ] Implementation [ ] Refactoring

   ### Scenario 1.2: [Name from agent]
   - **Type**: error-case
   - **Details**: [test-scenarios/error-cases.md#scenario-12](test-scenarios/error-cases.md#scenario-12)
   - **Implementation Progress**: [ ] Test Written [ ] Implementation [ ] Refactoring

   [Continue for all scenarios]

   ## AC-2: [Next AC Title]
   ...
   ```

8. **Update scenario files with anchors**:
   - Add heading anchors for linking in each type file
   - Format each scenario heading: `## Scenario N.M: [Name]`
   - Use the scenario number assigned in step 6

### For ADD Mode:

**Agent returns:** JSON object with single scenario, warnings about duplicates

**Parse JSON:**
```javascript
const response = JSON.parse(agent_output);
const scenario = response.scenarios[0];  // Single scenario
const warnings = response.warnings;
```

**Your tasks:**

1. **Check for duplicate warnings:**
   ```
   if (warnings.duplicates.length > 0) {
     Display: ⚠️  Possible duplicates: [list warnings.duplicates]
     Ask user: "Proceed anyway? (yes/no)"
     If no, abort
   }
   ```

2. **Determine next scenario number:**
   - Read existing scenarios.md to find all scenarios for the AC specified in `scenario.acceptance_criterion`
   - Find highest number (e.g., if 1.1, 1.2 exist → next is 1.3)

3. **Append to appropriate file** using `scenario.type`:
   ```bash
   # Determine file: test-scenarios/{scenario.type}.md
   # Example: if scenario.type === "happy-path" → test-scenarios/happy-path.md
   # Append:

   ## Scenario [N.M]: [scenario.name]

   [Paste scenario.content]
   ```

4. **Update scenarios.md**:
   ```bash
   # Find AC-N section (from scenario.acceptance_criterion)
   # Append after existing scenarios:

   ### Scenario N.M: [scenario.name]
   - **Type**: [scenario.type]
   - **Details**: [test-scenarios/{scenario.type}.md#scenario-nm](test-scenarios/{scenario.type}.md#scenario-nm)
   - **Implementation Progress**: [ ] Test Written [ ] Implementation [ ] Refactoring
   ```

5. **Display confirmation**:
   ```
   ✅ Added Scenario N.M: [scenario.name]
   - Type: [scenario.type]
   - AC: [scenario.acceptance_criterion]
   - File: test-scenarios/{scenario.type}.md
   - Ready for implementation
   ```

### For MODIFY Mode:

**Agent returns:** JSON object with updated scenario and warnings

**Parse JSON:**
```javascript
const response = JSON.parse(agent_output);
const scenario = response.scenarios[0];  // Updated scenario
const warnings = response.warnings;
```

**Your tasks:**

1. **Update scenario file** using `scenario.type`:
   ```bash
   # Find scenario in <directory>/test-scenarios/{scenario.type}.md
   # Locate heading: ## Scenario N.M: [Old Name]
   # Replace entire section with:
   ## Scenario N.M: [scenario.name]

   [scenario.content]
   ```

2. **Update scenarios.md** (if name or type changed):
   ```bash
   # Find scenario entry: ### Scenario N.M
   # Update:
   ### Scenario N.M: [scenario.name]
   - **Type**: [scenario.type]
   - **Details**: [test-scenarios/{scenario.type}.md#scenario-nm]
   # PRESERVE existing checkbox states - DO NOT reset them
   ```

3. **Display warnings**:
   ```
   ✅ Modified Scenario N.M: [scenario.name]

   ⚠️ Warnings from QA Engineer:
   - Implementation Impact: [list warnings.implementation_impact]

   If tests/implementation exist, review them for needed updates.
   ```

### For DISCOVER Mode:

**Agent returns:** JSON object with new scenarios OR context requests OR "No gaps found"

The agent handles gap analysis and Q&A internally, then generates scenarios to fill gaps.

**Parse JSON:**
```javascript
const response = JSON.parse(agent_output);
```

**Your tasks:**

1. **If agent returns context_requests:**
   ```
   if (response.context_requests && response.context_requests.length > 0) {
     For each requested file in context_requests:
       - Read the file
       - Append to original request: "Additional Context:\n\nFile: {filename}\n{content}\n"
     Re-invoke qa-engineer agent with updated request
     Continue to step 2 when agent returns scenarios or "No gaps"
   }
   ```

2. **If agent returns new scenarios:**
   ```
   if (response.scenarios && response.scenarios.length > 0) {
     Display warnings if any:
       ⚠️  Identified Gaps: [list response.warnings.gaps]

     For each scenario in response.scenarios:
       - Get AC from scenario.acceptance_criterion
       - Determine next scenario number for that AC:
         - Read scenarios.md to find existing scenarios for that AC
         - Find highest number (e.g., if AC-2 has 2.1, 2.2 → next is 2.3)
       - Append to test-scenarios/{scenario.type}.md:
         ## Scenario N.M: [scenario.name]

         [scenario.content]

         ---
       - Update scenarios.md under AC section:
         ### Scenario N.M: [scenario.name]
         - **Type**: [scenario.type]
         - **Details**: [test-scenarios/{scenario.type}.md#scenario-nm]
         - **Implementation Progress**: [ ] Test Written [ ] Implementation [ ] Refactoring

     Display summary:
       ✅ Discovered and added {count} new scenarios:
       - Scenario X.Y: [scenario.name] ({scenario.type})
       - Scenario X.Z: [scenario.name] ({scenario.type})
       ...

       Updated files:
       - scenarios.md
       - test-scenarios/{types}.md
   }
   ```

3. **If agent returns "No gaps found":**
   ```
   if (response.message === "No gaps found - coverage is complete") {
     Display:
       ✅ No gaps found - scenario coverage is complete

       All acceptance criteria have comprehensive test scenarios.
   }
   ```

### For DELETE Mode:

**No agent needed** - this is a file operation only

**Your tasks:**

1. **Parse scenario number** from user request (e.g., "1.3")

2. **Remove from scenarios.md**:
   ```bash
   # Find and remove the scenario entry:
   ### Scenario N.M: [Name]
   - **Type**: [type]
   - **Details**: [link]
   - **Implementation Progress**: [checkboxes]
   ```

3. **Remove from scenario detail file**:
   ```bash
   # Find and remove from <directory>/test-scenarios/{type}.md
   ## Scenario N.M: [Name]
   [entire scenario content]
   ---
   ```

4. **Display confirmation**:
   ```
   ✅ Deleted Scenario N.M: [Name]
   - Removed from scenarios.md
   - Removed from test-scenarios/{type}.md

   ⚠️ Note: If this scenario had implementation (tests/code), you may need to remove those manually.
   ```

## Expected File Structure

After EXPAND mode, the directory structure will be:

```
<task-directory>/
├── requirements.md            (INPUT - read-only, contains acceptance criteria)
├── scenarios.md               (OUTPUT - all scenarios with Implementation tracking)
└── test-scenarios/
    ├── happy-path.md         (OUTPUT - success cases)
    ├── error-cases.md        (OUTPUT - failure cases)
    └── edge-cases.md         (OUTPUT - boundary cases)
```

## Usage Examples

### Example 1: Create comprehensive scenarios from requirements.md
```bash
/test-scenarios apps/snyk-cmd/docs/features/bulk-ignore/tasks/task-001/
```

Creates all scenarios from requirements.md in that directory.

### Example 2: Add single scenario
```bash
/test-scenarios apps/feature/task-001/ "add scenario for null organization name to AC-3"
```

Adds one scenario to existing set for AC-3.

### Example 3: Modify existing scenario
```bash
/test-scenarios apps/feature/task-001/ "modify scenario 2.1 to test empty string instead of null"
```

Updates scenario 2.1 with new behavior.

### Example 4: Discover and add missing scenarios
```bash
/test-scenarios apps/feature/task-001/ "discover gaps in existing scenarios"
```

Analyzes scenarios for gaps and automatically generates and adds missing test scenarios.

### Example 4a: Discover gaps with additional context
```bash
/test-scenarios apps/feature/task-001/ "discover gaps" @research.md @tech-design.md
```

Uses additional context files to discover edge cases, technical constraints, and integration scenarios that may not be obvious from requirements alone. The qa-engineer agent may ask clarifying questions, then generates and adds the missing scenarios automatically.

### Example 5: Delete scenario
```bash
/test-scenarios apps/feature/task-001/ "delete scenario 1.3"
```

Removes scenario 1.3 from scenarios.md and test-scenarios files.

## Key Principles

This command is the **orchestrator**:
- ✅ Determines what to do (mode selection)
- ✅ Prepares context for agent
- ✅ Calls agent with simple request
- ✅ Parses structured JSON output
- ✅ Handles all file operations
- ✅ Manages scenario numbering based on agent's priority
- ✅ Creates links and tracking
- ✅ Displays warnings to user
- ✅ Handles context requests from agent (DISCOVER mode)

The qa-engineer agent is the **QA expert**:
- ✅ Applies QA heuristics
- ✅ Classifies scenarios by type (happy-path/error-case/edge-case)
- ✅ Assigns priority based on QA expertise
- ✅ Links scenarios to acceptance criteria
- ✅ Generates Given-When-Then scenarios
- ✅ Uses business-friendly language
- ✅ Warns about duplicates and implementation impact
- ✅ Requests additional context if needed (DISCOVER mode)
- ✅ Returns structured JSON for easy parsing
- ❌ Does NOT handle files, numbering, or organization