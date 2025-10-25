---
allowed-tools: Task(*), Read(*), Glob(*), Grep(*), TodoWrite(*)
argument-hint: <question or topic>
description: Research codebase to understand existing implementation
---

# Research Codebase

You are tasked with conducting comprehensive research across the codebase to understand existing implementation and create documentation that can feed into the TDD workflow.

## CRITICAL: YOUR ONLY JOB IS TO DOCUMENT AND EXPLAIN THE CODEBASE AS IT EXISTS TODAY
- DO NOT suggest improvements or changes
- DO NOT critique the implementation or identify problems
- DO NOT recommend refactoring, optimization, or architectural changes
- ONLY describe what exists, where it exists, how it works, and how components interact
- You are creating technical documentation of the existing system

## Workflow

### Step 1: Understand the Research Question

When invoked with: `/create-research "How does user authentication work?"`

Parse the question and identify:
- What components/features to investigate
- What architectural layers/modules are involved
- What patterns to look for

### Step 2: Create Research Plan

Use TodoWrite to track your research tasks:
- Find relevant files and components
- Analyze implementation details
- Discover patterns and conventions
- Document findings

### Step 3: Launch Parallel Research Agents with Rich Context

Spawn multiple specialized agents concurrently using Task tool.

**IMPORTANT**: Each agent must receive:
1. The original research question
2. The task/ticket context (if available)
3. Specific deliverable requirements
4. Cross-referencing instructions

**Agent 1 - codebase-locator:**

Provide detailed instructions:
```
CONTEXT:
- Research question: [original question]
- Ticket/requirement: [if available, include ticket path or summary]
- Related components: [any components mentioned in question]

TASK:
Find all files related to:
1. [Main component/feature]
2. Dependencies and imports/requires
3. Core business logic and data structures
4. External integrations and data access
5. Test files for all above

DELIVERABLES:
1. Complete file paths with descriptions
2. File roles (entry point/implementation/utility/test)
3. Import/dependency graph (which files depend on which)
4. Files grouped by modules/layers (adapt to discovered structure)
5. Test coverage mapping (which test tests which implementation)

OUTPUT FORMAT:
## Component Locations

### [Module/Layer Name] - [Component Type]
- `path/to/file.ext` - [description] [ROLE: entry point/implementation/utility/test]
  - Dependencies: [list imports/requires]
  - Tests: [test file name]

### Dependencies
- `file-a.ext` imports/requires from `file-b.ext` at line X
- [map all import/dependency relationships]

### Test Coverage
- `test_file.ext` or `file.test.ext` - tests for [component]
```

**Agent 2 - codebase-analyzer:**

Provide detailed instructions:
```
CONTEXT:
- Research question: [original question]
- Ticket/requirement: [if available]
- Core files to analyze: [specify key files or reference Agent 1 results if sequential]

TASK:
Analyze implementation details:
1. Complete execution flow from entry to exit
2. Data transformations at each step
3. Validation and error handling points
4. Dependencies on other components
5. Edge cases and special handling

DELIVERABLES:
1. Function/method signatures with file:line references
2. Step-by-step execution flow
3. Data transformation pipeline
4. Error handling and validation logic
5. Dependencies referenced by file path

OUTPUT FORMAT:
## Implementation Analysis

### Execution Flow
1. Entry point: `file.ext:line` - [what happens, parameters received]
2. Validation: `file.ext:line` - [what's validated, how, error cases]
3. Processing: `file.ext:line` - [transformations, business logic]
4. Exit/return: `file.ext:line` - [what's returned, side effects]

### Key Functions/Methods
- `functionName()` at `file.ext:line`
  - Parameters: [types and descriptions]
  - Returns: [type and description]
  - Dependencies: [list files/components used with lines]
  - Error cases: [what throws/returns, when, with examples]
  - Edge cases: [special handling]

### Data Transformations
- Input: [type/structure] → Output: [type/structure]
- Transformation at `file.ext:line`: [description]
```

**Agent 3 - codebase-pattern-finder:**

Provide detailed instructions:
```
CONTEXT:
- Research question: [original question]
- Components analyzed: [reference from Agent 1/2]
- Ticket/requirement: [if available]

TASK:
Find examples of patterns in the codebase:
1. Naming conventions (files, classes, methods)
2. Repeated architectural patterns
3. Common validation approaches
4. Error handling strategies
5. Layer separation rules

DELIVERABLES:
1. Pattern name and description
2. At least 2-3 concrete examples with file:line references
3. Consistency analysis (always/mostly/sometimes used)
4. Code snippets showing pattern in action
5. Exceptions or variations noted

OUTPUT FORMAT:
## Patterns Discovered

### Pattern: [Name]
- **Description**: [how it works]
- **Found in**: `file1.ext:line`, `file2.ext:line`, `file3.ext:line`
- **Consistency**: [always/mostly/sometimes used]
- **Example**:
  ```
  [code snippet showing pattern]
  ```
- **Variations**: [note any deviations from standard pattern]

### Pattern: [Name]
[repeat for each pattern found]
```

**EXECUTION**: Run these agents in PARALLEL (single message with multiple Task tool calls).

### Step 4: Synthesize Agent Findings (CRITICAL QUALITY STEP)

After ALL agents complete, perform comprehensive synthesis:

**1. Cross-Reference Findings**
- Map Agent 1's file locations to Agent 2's implementation details
- Link Agent 3's patterns to specific files from Agent 1
- Identify gaps: Did Agent 2 analyze all files from Agent 1?
- Verify: Do patterns from Agent 3 appear in files from Agent 1?

**2. Validate Completeness**
- Does Agent 2's execution flow reference all core components from Agent 1?
- Do Agent 3's patterns explain Agent 2's implementation choices?
- Are there files in Agent 1 not covered by Agent 2?
- Are test files from Agent 1 explained (what they test)?

**3. Resolve Conflicts**
- If agents report different information, investigate yourself
- Read files directly to confirm ambiguous findings
- Use Grep to verify pattern claims from Agent 3
- Check import statements to validate Agent 1's dependency graph

**4. Fill Gaps**
- If Agent 2 missed error handling, search for it yourself using Grep
- If Agent 3 found only 1 example of a pattern, search for more
- If Agent 1 missed test files, glob for common test patterns (`*test*`, `*spec*`, `test_*`)
- If execution flow is incomplete, read the source files directly

**5. Enrich with Context**
- Read the original ticket/requirement again
- Check if research fully answers the original question
- Add any missing details from your own investigation
- Note any architectural insights discovered during synthesis

**6. Document Synthesis Process**
- Track which findings came from agents vs your investigation
- Note any conflicts resolved
- Record gaps that were filled
- Document manual verification performed

### Step 5: Quality Verification Checklist

Before writing research.md, verify:

- [ ] All files from Agent 1 are explained in Agent 2's analysis
- [ ] Agent 2's execution flow has file:line references for every step
- [ ] Agent 3's patterns have at least 2-3 concrete examples each
- [ ] Error handling is documented (not just happy path)
- [ ] Test files are identified and their purpose explained
- [ ] Architectural layers are clearly separated in documentation
- [ ] Cross-references between agents are resolved
- [ ] Original research question is fully answered
- [ ] Import dependencies are verified (not just assumed)
- [ ] Edge cases and special handling are documented

**If any item is unchecked**, investigate yourself using:
- Read tool for specific files
- Grep for patterns across codebase
- Glob for missing files
- Manual code tracing

### Step 6: Generate Research Document

Create `research.md` in the task directory (or root if no task context) with explicit synthesis markers:

```markdown
# Research: [Question/Topic]

**Date**: [Current date]
**Question**: [Original research question]

## Summary
[Synthesized high-level overview from all agents + your analysis]

## Detailed Findings

### Component Structure
[From Agent 1 + your verification]

**Files Discovered** (Agent 1):
- `path/to/file.ext` - [description] [ROLE]
  - Dependencies: [imports/requires]
  - Tests: [test files]

**Verified Dependencies** (Your investigation):
- Cross-referenced import/require statements at `file.ext:line`
- Confirmed dependency chain: A → B → C
- [any corrections or additions]

### Implementation Details
[From Agent 2 + your enrichment]

**Core Flow** (Agent 2):
1. Entry: `file.ext:45` - [description with parameters]
2. Validation: `file.ext:67` - [what's validated, how]
3. Processing: `file.ext:89` - [transformations]
4. Exit: `file.ext:102` - [what's returned]

**Additional Details** (Your research):
- Error case at `file.ext:110` - throws/returns error when [condition]
- Edge case handling at `file.ext:125` - [special behavior]
- Missing from agent analysis: [any gaps you filled]

### Patterns Discovered
[From Agent 3 + your validation]

**Pattern: [Name]** (Agent 3):
- **Description**: [how it works]
- **Found in**: `file1.ext:line`, `file2.ext:line`
- **Example**:
  ```
  [code snippet]
  ```

**Pattern Validation** (Your verification):
- Confirmed with grep: [X] occurrences across [module/directory]
- Additional examples found: `file3.ext:line`, `file4.ext:line`
- Exceptions noted: `legacy-file.ext` uses different pattern because [reason]

### Cross-Agent Synthesis
[Your analysis connecting all findings]

Agent 1 identified [X] core files. Agent 2 analyzed [Y] of them. The remaining files are:
- `helper.ext` - Utility used by all components at: `file-a.ext:12`, `file-b.ext:34`
- Purpose: [description from your reading]

Agent 3 found pattern X in [locations]. This explains Agent 2's implementation choice at `file.ext:45` because [architectural reasoning].

The execution flow from Agent 2 aligns with the file structure from Agent 1, confirming [architectural principle].

### Test Coverage
- Unit tests: `test_file.ext` or `file.test.ext` - tests [what] from `file.ext`
- Integration tests: `integration_test.ext` - tests [what flow]
- Coverage gaps: [any areas without tests]

## Architecture Insights
[Synthesized patterns with evidence]

- **Pattern observed**: [description]
  - Evidence: [file references from agents + your verification]
- **Module separation**: [how it's maintained]
  - [Module type] never imports from [other module type] (verified in [X] files)
- **Error handling strategy**: [approach used]
  - Examples: [file:line references]

## File References
[Complete, verified list grouped by module/layer/directory]

### [Module/Layer Name 1]
- `path/to/file.ext` - [description]

### [Module/Layer Name 2]
- `path/to/file.ext` - [description]

### [Module/Layer Name 3]
- `path/to/file.ext` - [description]

(Adapt structure to discovered codebase organization)

## Research Quality Notes
[Transparency about synthesis process]

**Agent Coverage**:
- Agent 1 (codebase-locator): Found [X] files across [Y] modules/layers
- Agent 2 (codebase-analyzer): Analyzed [X] core files, [Y] execution flows
- Agent 3 (codebase-pattern-finder): Found [X] patterns with [Y] total examples

**Manual Enrichment**:
- Added error handling details not covered by Agent 2
- Verified Agent 3's pattern claims with [X] additional examples via Grep
- Cross-referenced ticket requirements - all [X] requirements addressed
- Filled [X] gaps identified during synthesis

**Conflicts Resolved**:
- Agent [X] reported [Y], but file reading confirmed [Z]
- [any other discrepancies and resolutions]

## Next Steps
This research can be used to:
1. Inform requirements and specifications for modifications
2. Understand impact of proposed changes
3. Identify test scenarios that need coverage
4. Guide refactoring and architectural decisions
```

### Step 7: Report Completion

After creating research.md, provide this summary:

```
✅ Research Complete: [Topic]

Research document created at: ./research.md

Key findings:
- [Major finding 1 with file reference]
- [Major finding 2 with file reference]
- [Major finding 3 with file reference]

Research quality:
- Agent 1 found [X] files across [Y] modules/layers
- Agent 2 analyzed [X] execution flows
- Agent 3 identified [X] patterns
- Manual enrichment: [X] gaps filled, [Y] verifications performed

This research can now be used to:
- Inform specification creation (if using TDD workflow)
- Guide architectural decisions
- Understand impact of proposed changes
```

## Integration with Development Workflow

This research can be used in various workflows:
```
/create-research → research.md → [use as input for specifications, design docs, or implementation]
```

## Important Notes

**Research Quality Strategy**:
- All agents work in parallel for efficiency (speed benefit)
- Rich context provided to each agent (quality benefit)
- Mandatory synthesis phase (quality assurance)
- Quality checklist before documentation (completeness verification)
- Transparency about agent vs manual findings (traceability)

**Research Purpose**:
- Research is purely documentary - no suggestions or improvements
- Focus on understanding what exists, not what should exist
- Output is structured to inform specifications, designs, and implementations
- Document actual implementation, not ideal implementation
- All findings must include specific file:line references