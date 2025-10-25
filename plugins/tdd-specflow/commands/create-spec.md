---
allowed-tools: Task(*), Read(*), Glob(*)
argument-hint: <ticket-path | prompt> [research.md]
description: Create requirements.md with testable acceptance criteria from a ticket or prompt
---

Execute requirements analysis for: $ARGUMENTS

**Step 0: Parse Arguments and Determine Context**
1. Parse $ARGUMENTS to extract:
   - Primary argument: Either a ticket file path (e.g., ticket.md) OR a direct text prompt
   - Optional second argument: research.md path
2. Determine if first argument is a file path or prompt:
   - If it ends with .md or contains path separators, treat as file path
   - Otherwise, treat as direct prompt text
3. If file path: Get the directory path where the ticket file is located
4. If prompt: Use current working directory as output location
5. This directory will be used as the output location for requirements.md
6. Check if research.md was provided as second argument

**Step 1: Requirements Analysis**
Launch the requirements-analyzer agent:

Source: [ticket path from $ARGUMENTS OR direct prompt text]
Research file: [research.md path if provided as second argument]
Output directory: [Same directory as ticket file OR current working directory]
Output filename: requirements.md (not spec.md)
Task: Analyze requirements and create requirements.md with testable acceptance criteria. If research.md is provided, use it to inform requirements based on existing implementation patterns. If source is a direct prompt, use it as the requirements input.

**Expected Output:**
✅ requirements.md with acceptance criteria in the target directory

**Next Steps:**
After requirements.md is created, you can:
1. Run `/test-scenarios <directory>` to generate detailed test scenarios
2. Or continue with the full workflow using `/create-design <directory>`