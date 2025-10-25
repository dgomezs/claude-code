# TDD SpecFlow - Claude Code Plugin & Marketplace

A comprehensive Test-Driven Development workflow plugin for Claude Code that combines TDD with Specification by Example (BDD). Streamline your development process with intelligent agents that guide you through requirements analysis, technical design, and the complete RED-GREEN-REFACTOR cycle.

**Note**: This repository also serves as a **plugin marketplace** infrastructure, currently hosting the tdd-specflow plugin with support for adding more plugins in the future. See [MARKETPLACE.md](MARKETPLACE.md) for details on the marketplace structure.

## Overview

TDD SpecFlow provides a complete, automated workflow for building features using Test-Driven Development and Behavior-Driven Development principles. The plugin includes specialized agents for each phase of development and slash commands that orchestrate the entire process.

### Key Features

- **Complete TDD Workflow**: Automated RED-GREEN-REFACTOR cycle with progress tracking
- **Specification by Example**: Transform acceptance criteria into concrete test scenarios with Given-When-Then format
- **Requirements Analysis**: Structured specification generation from JIRA tickets or user stories
- **Technical Design**: Automated technical design documents following architectural patterns
- **9 Specialized Agents**: Each agent is an expert in their specific domain
- **5 Slash Commands**: Orchestrate the entire development workflow
- **Progress Tracking**: Visual checkboxes track progress through each TDD phase

## Installation

Install the plugin using Claude Code's plugin command:

```bash
/plugin install https://gitlab.com/dgomezs2/ai/claude-code
```

Or if using a local clone:

```bash
/plugin install /path/to/claude-code
```

## Quick Start

### 1. Create Requirements Specification

Start with a JIRA ticket, user story, or direct prompt:

```bash
/create-spec ticket.md
```

or with a direct prompt:

```bash
/create-spec "Add user authentication with JWT tokens"
```

This creates `requirements.md` with testable acceptance criteria in Given-When-Then format.

### 2. Generate Test Scenarios

Transform acceptance criteria into detailed test scenarios:

```bash
/test-scenarios <task-directory>
```

This creates:
- `scenarios.md` - High-level scenario overview with TDD tracking checkboxes
- `test-scenarios/happy-path.md` - Successful flow scenarios
- `test-scenarios/error-cases.md` - Error handling scenarios
- `test-scenarios/edge-cases.md` - Boundary and edge cases

### 3. Create Technical Design

Generate an implementation plan following your project's architecture:

```bash
/create-tech-design <task-directory>
```

This creates `tech-design.md` with:
- Component architecture
- Implementation strategy
- Testing guidelines
- Integration points

### 4. Run TDD Implementation

Execute the complete RED-GREEN-REFACTOR cycle:

```bash
/start-tdd <task-directory>
```

The orchestrator will:
1. Detect which phase you're in (RED, GREEN, or REFACTOR)
2. Launch the appropriate specialized agent
3. Track progress with checkboxes in `scenarios.md`
4. Suggest commits after each phase
5. Continue until all scenarios are complete

## Workflow Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    TDD SpecFlow Workflow                    │
└─────────────────────────────────────────────────────────────┘

1. Requirements Analysis
   ↓
   /create-spec → requirements.md

2. Test Scenarios
   ↓
   /test-scenarios → scenarios.md + test-scenarios/

3. Technical Design
   ↓
   /create-tech-design → tech-design.md

4. TDD Implementation Loop
   ↓
   /start-tdd → RED-GREEN-REFACTOR cycle

   For each scenario:
   ┌─────────────────────────────────────┐
   │  RED: Write failing test            │
   │    ↓                                │
   │  GREEN: Make it pass (minimal code) │
   │    ↓                                │
   │  REFACTOR: Improve quality          │
   └─────────────────────────────────────┘
```

## Slash Commands

### `/create-spec`
**Purpose**: Analyze requirements and create structured specification

**Usage**:
```bash
/create-spec <ticket-path | prompt> [research.md]
```

**Examples**:
```bash
/create-spec docs/tickets/AUTH-123.md
/create-spec "Add password reset functionality" research.md
/create-spec "Implement user profile editing"
```

**Output**: `requirements.md` with testable acceptance criteria

---

### `/test-scenarios`
**Purpose**: Generate detailed test scenarios from requirements

**Usage**:
```bash
/test-scenarios <task-directory> [operation-prompt]
```

**Examples**:
```bash
/test-scenarios docs/tasks/task-001/
/test-scenarios docs/tasks/task-001/ "Add scenario for concurrent access"
```

**Output**:
- `scenarios.md` - Overview with TDD tracking
- `test-scenarios/happy-path.md`
- `test-scenarios/error-cases.md`
- `test-scenarios/edge-cases.md`

---

### `/create-tech-design`
**Purpose**: Create technical design from requirements

**Usage**:
```bash
/create-tech-design <directory>
```

**Example**:
```bash
/create-tech-design docs/tasks/task-001/
```

**Output**: `tech-design.md` with architecture and implementation strategy

---

### `/create-research`
**Purpose**: Research codebase to understand existing implementations

**Usage**:
```bash
/create-research <question or topic>
```

**Example**:
```bash
/create-research "How is user authentication currently implemented?"
```

**Output**: `research.md` with findings and patterns

---

### `/start-tdd`
**Purpose**: Execute TDD workflow with intelligent orchestration

**Usage**:
```bash
/start-tdd <task-directory>
```

**Example**:
```bash
/start-tdd docs/tasks/task-001/
```

**Behavior**:
- Detects current phase from `scenarios.md` checkboxes
- Launches appropriate agent (tdd-red, tdd-green, or tdd-refactor)
- Updates progress automatically
- Suggests commits after each phase
- Continues to next scenario when current is complete

## Specialized Agents

### Requirements Analysis

#### `requirements-analyzer`
Transforms JIRA tickets, user stories, or prompts into structured specifications with testable acceptance criteria.

**When to use**: Start of any feature development
**Output**: `requirements.md` with Given-When-Then acceptance criteria

---

#### `qa-engineer`
Generates detailed test scenarios using Specification by Example methodology.

**When to use**: After requirements analysis
**Output**: Concrete test scenarios in `test-scenarios/` directory

---

### Architecture & Research

#### `software-architect`
Creates technical designs that translate requirements into implementation plans following project patterns.

**When to use**: After test scenarios are defined
**Output**: `tech-design.md` with component architecture and strategy

---

#### `codebase-locator`
Finds WHERE files and components live in the codebase.

**When to use**: Understanding project structure
**Output**: File locations and component organization

---

#### `codebase-pattern-finder`
Discovers patterns, conventions, and repeated structures.

**When to use**: Learning architectural patterns
**Output**: Pattern documentation and examples

---

#### `codebase-analyzer`
Analyzes HOW specific code works with implementation details.

**When to use**: Understanding existing implementations
**Output**: Technical analysis with file:line references

---

### TDD Phase Agents

#### `tdd-red` (RED Phase)
Writes ONE failing test for a specific scenario.

**When to use**: Starting a new scenario or explicitly requested
**Process**:
1. Reads scenario from `test-scenarios/`
2. Follows test location from `tech-design.md`
3. Writes descriptive test with one assertion
4. Verifies test fails
5. Marks `[x] Test Written` in `scenarios.md`

---

#### `tdd-green` (GREEN Phase)
Implements minimal code to make the failing test pass.

**When to use**: After RED phase (test written and failing)
**Process**:
1. Reads failing test
2. Follows `tech-design.md` architecture
3. Implements minimal production code
4. Verifies test passes
5. Marks `[x] Implementation` in `scenarios.md`

---

#### `tdd-refactor` (REFACTOR Phase)
Improves code quality without changing behavior.

**When to use**: After GREEN phase (tests passing)
**Process**:
1. Identifies code smells and duplication
2. Applies refactoring patterns
3. Ensures all tests still pass
4. Marks `[x] Refactoring` in `scenarios.md`

## Progress Tracking

Each scenario in `scenarios.md` uses checkboxes to track TDD phases:

```markdown
### ⏳ Scenario: User login with valid credentials
**Given** a registered user with valid credentials
**When** the user attempts to log in
**Then** access token is returned and user is authenticated

**Progress:**
- [ ] **Test Written** - RED phase
- [ ] **Implementation** - GREEN phase
- [ ] **Refactoring** - REFACTOR phase
```

**Status Icons**:
- ⏳ Not started: `[ ] [ ] [ ]`
- 🔄 In progress: `[x] [ ] [ ]` or `[x] [x] [ ]`
- ✅ Functionally complete: `[x] [x] [ ]`
- ✨ Polished: `[x] [x] [x]`

## Example Workflow

Here's a complete example implementing a password reset feature:

```bash
# Step 1: Create requirements from ticket
/create-spec docs/tickets/AUTH-456-password-reset.md

# Step 2: Generate test scenarios
/test-scenarios docs/tasks/password-reset/

# Step 3: Create technical design
/create-tech-design docs/tasks/password-reset/

# Step 4: Start TDD implementation
/start-tdd docs/tasks/password-reset/
```

The orchestrator will guide you through each scenario:

1. **RED**: Writes failing test for "Valid reset token generates new password"
2. **GREEN**: Implements password reset logic
3. **REFACTOR**: Extracts token validation to separate function
4. Suggests commit: "feat: implement password reset with valid token"
5. Continues to next scenario: "Invalid token returns error"

## Best Practices

### 1. Always Start with Requirements
Use `/create-spec` to establish clear acceptance criteria before writing code.

### 2. Generate Complete Test Scenarios
Run `/test-scenarios` to cover happy paths, error cases, and edge cases upfront.

### 3. Follow the Architecture
Use `/create-tech-design` to align implementation with project patterns from `CLAUDE.md`.

### 4. One Scenario at a Time
Let `/start-tdd` guide you through each scenario methodically.

### 5. Commit After Each Phase
The orchestrator suggests commits after RED, GREEN, and REFACTOR phases.

### 6. Refactoring is Optional
You can skip REFACTOR phase if code quality is acceptable (scenario is functionally complete after GREEN).

### 7. Use Research for Context
Run `/create-research` when you need to understand existing implementations before starting.

## File Structure

A typical task directory structure:

```
docs/tasks/task-001-user-auth/
├── requirements.md              # Acceptance criteria
├── tech-design.md              # Technical design
├── scenarios.md                # TDD progress tracking
└── test-scenarios/
    ├── happy-path.md          # Success scenarios
    ├── error-cases.md         # Error handling
    └── edge-cases.md          # Boundaries and edges
```

## Advanced Usage

### Custom Research
Use research findings to inform requirements:

```bash
/create-research "How is email validation currently implemented?"
/create-spec "Add phone number validation" research.md
```

### Modify Scenarios
Add or update scenarios after initial generation:

```bash
/test-scenarios docs/tasks/task-001/ "Add scenario for rate limiting"
```

### Skip Refactoring
When in REFACTOR phase, you can choose to skip and move to the next scenario if code quality is acceptable.

## Troubleshooting

### Prerequisites Missing
If `/start-tdd` reports missing files:
```bash
# Generate missing files in order:
/test-scenarios <task-directory>
/create-tech-design <task-directory>
```

### Test Passes Unexpectedly (RED Phase)
The `tdd-red` agent will diagnose why and suggest actions:
- Strengthen the test
- Skip scenario (if already implemented)
- Ask for guidance

### Unclear Requirements
The `requirements-analyzer` will ask clarifying questions about business logic, not implementation details.

## Marketplace

This repository is structured as a **Claude Code plugin marketplace** that can host multiple plugins. Currently, it contains the tdd-specflow plugin with infrastructure ready for expansion.

### Current Plugins
- **tdd-specflow** (v0.0.1) - Main TDD workflow plugin (this document)

### Adding More Plugins
See [MARKETPLACE.md](MARKETPLACE.md) for:
- How to add plugins to this marketplace
- Marketplace structure and guidelines
- Plugin installation instructions
- Contributing plugins to the marketplace

### Marketplace Features
- Supports multiple plugins in subdirectories
- Can reference external GitHub repositories
- Centralized plugin discovery and distribution
- Structured plugin metadata and versioning

## Usage and Forking

This repository is provided as-is for your use and learning. You are free to:
- Use the plugin in your projects
- Fork and modify it for your own needs
- Adapt the code and patterns to your workflow

**Note:** This repository is not actively maintained and does not accept issues or merge requests. Feel free to fork and customize as needed.

See [CONTRIBUTING.md](CONTRIBUTING.md) for development guidelines if you want to fork and customize.

## License

MIT License - see LICENSE file for details

## Documentation

- **Repository**: https://gitlab.com/dgomezs2/ai/claude-code
- **Claude Code Docs**: https://docs.claude.com/en/docs/claude-code

**Note:** This repository does not accept issues or merge requests. For general Claude Code questions, refer to the official Claude Code documentation.

## Credits

Created by dgomezs for the Claude Code community.

Built with Claude Code's powerful plugin system to bring enterprise-grade TDD workflows to AI-assisted development.

### Acknowledgments

The `/create-research` command is an adaptation of the innovative codebase research approach developed by [HumanLayer](https://github.com/humanlayer/humanlayer). We are grateful for their work in advancing AI-powered code analysis and documentation techniques.
