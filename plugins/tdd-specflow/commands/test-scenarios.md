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

## Step 5: Common File Operations

All modes (except DELETE) use these common operations after receiving agent JSON output.

### 5.1: Parse Agent JSON Response

```javascript
const response = JSON.parse(agent_output);
const scenarios = response.scenarios || [];
const warnings = response.warnings || {};
const contextRequests = response.context_requests || [];
```

### 5.2: Display Warnings

```
if (warnings.duplicates?.length > 0) {
  Display: ⚠️  Duplicate warnings: [list warnings.duplicates]
}
if (warnings.gaps?.length > 0) {
  Display: ℹ️  Identified gaps: [list warnings.gaps]
}
if (warnings.implementation_impact?.length > 0) {
  Display: ⚠️  Implementation impact: [list warnings.implementation_impact]
  Display: "Review existing tests/code for needed updates"
}
```

### 5.3: Write Scenario Content to Type File

For any scenario that needs to be written or updated:

```bash
# Determine target file from scenario.type:
target_file = test-scenarios/{scenario.type}.md
# Where scenario.type is one of: "happy-path", "error-case", "edge-case"

# Write scenario.content exactly as received from agent:
[Paste scenario.content]

# Agent's content includes:
# - ## Scenario N.M: [Name] heading
# - Proper blank lines (MD022/MD032 compliance)
# - Trailing --- separator
```

### 5.4: Update scenarios.md Tracking

For any scenario that needs tracking entry:

```markdown
### Scenario N.M: [scenario.name]
- **Type**: [scenario.type]
- **Details**: [test-scenarios/{scenario.type}.md#scenario-nm](test-scenarios/{scenario.type}.md#scenario-nm)
- **Implementation Progress**: [ ] Test Written [ ] Implementation [ ] Refactoring
```

**Important**: When modifying existing entry, PRESERVE checkbox states.

### 5.5: Determine Next Scenario Number

When adding new scenarios, find the next available number for an AC:

```bash
# Read scenarios.md
# Find all scenarios for the target AC (e.g., "AC-2")
# Find highest number (e.g., if 2.1, 2.2, 2.3 exist → next is 2.4)
# Return next_number
```

## Step 6: Mode-Specific Workflows

After receiving JSON output from qa-engineer, perform mode-specific operations:

### For EXPAND Mode:

Creates all scenario files from scratch.

**Your tasks:**

1. **Use Step 5.1** to parse JSON response

2. **Use Step 5.2** to display warnings

3. **Group scenarios by type**:
   - Filter `scenarios` where `type === "happy-path"` → happy-path.md
   - Filter `scenarios` where `type === "error-case"` → error-cases.md
   - Filter `scenarios` where `type === "edge-case"` → edge-cases.md

4. **Create directory**:
   ```bash
   mkdir -p <directory>/test-scenarios
   ```

5. **Assign scenario numbers**:
   - Group scenarios by `acceptance_criterion` field (e.g., "AC-1")
   - Sort within each AC by `priority` field (1, 2, 3...)
   - Assign numbers: AC-1 → 1.1, 1.2, 1.3...; AC-2 → 2.1, 2.2...
   - Agent determines priority (happy-path first, then error-case, then edge-case)

6. **Write type files** with headers:

   **happy-path.md**:
   ```markdown
   # Happy Path Scenarios

   Valid inputs and successful outcomes that represent typical user workflows.

   ---

   [For each happy-path scenario, use Step 5.3 to write scenario.content]
   ```

   **error-cases.md**:
   ```markdown
   # Error Case Scenarios

   Invalid inputs and failure conditions.

   ---

   [For each error-case scenario, use Step 5.3 to write scenario.content]
   ```

   **edge-cases.md**:
   ```markdown
   # Edge Case Scenarios

   Boundary values, limits, and unusual but valid conditions.

   ---

   [For each edge-case scenario, use Step 5.3 to write scenario.content]
   ```

7. **Create scenarios.md** with tracking:
   ```markdown
   # Test Scenarios

   [For each AC in requirements.md:]
   ## AC-N: [Title from requirements.md]

   **Source**: [requirements.md#ac-n](requirements.md#ac-n)

   [For each scenario for this AC, use Step 5.4 to create tracking entry]
   ```

### For ADD Mode:

Adds one new scenario to existing set.

**Your tasks:**

1. **Use Step 5.1** to parse JSON (get `scenarios[0]` as the single scenario)

2. **Use Step 5.2** to display warnings

3. **Check for duplicates** - if `warnings.duplicates` is not empty:
   ```
   Ask user: "Proceed anyway? (yes/no)"
   If no, abort
   ```

4. **Use Step 5.5** to determine next scenario number for the AC

5. **Use Step 5.3** to append scenario.content to appropriate type file

6. **Use Step 5.4** to add tracking entry to scenarios.md under the AC section

7. **Display confirmation**:
   ```
   ✅ Added Scenario N.M: [scenario.name]
   - Type: [scenario.type]
   - AC: [scenario.acceptance_criterion]
   ```

### For MODIFY Mode:

Updates an existing scenario.

**Your tasks:**

1. **Use Step 5.1** to parse JSON (get `scenarios[0]` as the updated scenario)

2. **Use Step 5.2** to display warnings

3. **Update type file**:
   - Find scenario in `test-scenarios/{scenario.type}.md`
   - Locate section starting with `## Scenario N.M:`
   - Replace entire section (from heading to `---`) with **Step 5.3** content

4. **Update scenarios.md** (if name or type changed):
   - Find scenario entry `### Scenario N.M`
   - Update name and type using **Step 5.4** format
   - **PRESERVE existing checkbox states**

5. **Display confirmation**:
   ```
   ✅ Modified Scenario N.M: [scenario.name]

   If tests/implementation exist, review them for needed updates.
   ```

### For DISCOVER Mode:

Analyzes existing scenarios for gaps and adds missing scenarios.

**Your tasks:**

1. **Use Step 5.1** to parse JSON

2. **Handle context requests** (if any):
   ```
   if (response.context_requests?.length > 0) {
     For each requested file:
       - Read the file
       - Append to original request: "Additional Context:\n\nFile: {filename}\n{content}\n"
     Re-invoke qa-engineer agent with updated request
     Return to step 1 when agent responds
   }
   ```

3. **If scenarios found** (`response.scenarios.length > 0`):

   a. **Use Step 5.2** to display warnings (especially gaps identified)

   b. **For each new scenario**:
      - **Use Step 5.5** to determine next scenario number for the AC
      - **Use Step 5.3** to append to appropriate type file
      - **Use Step 5.4** to add tracking entry to scenarios.md

   c. **Display summary**:
      ```
      ✅ Discovered and added {count} new scenarios:
      - Scenario X.Y: [scenario.name] ({scenario.type})
      [list all new scenarios]
      ```

4. **If no gaps found** (`response.message === "No gaps found - coverage is complete"`):
   ```
   Display:
     ✅ No gaps found - scenario coverage is complete
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
