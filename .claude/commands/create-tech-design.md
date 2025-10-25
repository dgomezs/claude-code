---
allowed-tools: Task(*), Read(*), Glob(*)
argument-hint: <directory>
description: Create technical design (tech-design.md) from requirements.md
---

Execute technical design creation for the directory at: $ARGUMENTS

**Prerequisites:**
- requirements.md must exist in the directory (run `/create-spec` first if needed)

**Step 0: Parse Arguments and Validate**
1. Extract directory path from $ARGUMENTS
2. Verify that requirements.md exists in this directory (MANDATORY)
3. Check if research.md exists in the same directory (OPTIONAL - for context on existing patterns)
4. Check if scenarios.md exists in the same directory (OPTIONAL - for detailed test context)

**Step 1: Launch software-architect Agent**

Launch software-architect agent to create technical design:

- Read requirements.md from: [directory]/requirements.md (MANDATORY - contains acceptance criteria)
- Read research.md from: [directory]/research.md (OPTIONAL - if exists, use existing patterns and components)
- Read scenarios.md from: [directory]/scenarios.md (OPTIONAL - if exists, provides detailed test scenarios for design context)
- Output directory: [directory]
- Output filename: tech-design.md (not design.md)
- Task: Create tech-design.md with technical design covering ALL acceptance criteria in requirements.md. If research.md is provided, use existing patterns and components from the research. If scenarios.md is provided, use test scenarios to inform design decisions.

**Step 2: If Design Options Presented**

If the software-architect agent pauses to present multiple design alternatives:
1. Review the design options with the user
2. Present pros/cons and recommendation clearly
3. Wait for user to select an option
4. Once user selects, relaunch software-architect agent to complete tech-design.md with chosen option

**Expected Output:**
✅ tech-design.md with technical design in the directory

**Next Steps:**
After tech-design.md is created, you can begin implementation using your preferred development approach.