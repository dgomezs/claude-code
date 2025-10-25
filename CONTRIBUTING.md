# Contributing to TDD SpecFlow

Thank you for your interest in contributing to TDD SpecFlow! This document provides guidelines for contributing to the plugin.

## How to Contribute

### Reporting Issues

If you find a bug or have a feature request:

1. Check if the issue already exists in the [issue tracker](https://gitlab.com/dgomezs2/ai/claude-code/-/issues)
2. If not, create a new issue with:
   - Clear description of the problem or feature
   - Steps to reproduce (for bugs)
   - Expected vs actual behavior
   - Claude Code version and environment details

### Submitting Changes

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Make your changes following the guidelines below
4. Test your changes thoroughly
5. Commit with clear messages (`git commit -m 'Add amazing feature'`)
6. Push to your fork (`git push origin feature/amazing-feature`)
7. Open a Pull Request

## Development Guidelines

### Plugin Structure

The plugin follows this structure:

```
tdd-specflow/
├── .claude/
│   ├── agents/         # Specialized agents
│   └── commands/       # Slash commands
├── .claude-plugin/
│   └── marketplace.json # Marketplace manifest
├── plugin.json         # Plugin manifest
└── README.md          # Documentation
```

### Agent Guidelines

When creating or modifying agents in `.claude/agents/`:

1. **Frontmatter**: Include proper YAML frontmatter:
   ```yaml
   ---
   name: agent-name
   description: Clear description of what the agent does
   model: sonnet
   color: blue
   ---
   ```

2. **Documentation**: Start with clear explanation of:
   - Agent's purpose and responsibilities
   - When to use the agent
   - Expected inputs and outputs
   - Example usage scenarios

3. **Workflow**: Define clear step-by-step processes

4. **Error Handling**: Document how the agent handles errors

5. **Examples**: Include realistic examples in the description field

### Command Guidelines

When creating or modifying commands in `.claude/commands/`:

1. **Frontmatter**: Include proper YAML frontmatter:
   ```yaml
   ---
   allowed-tools: Task(*), Read(*), Glob(*)
   argument-hint: <required-arg> [optional-arg]
   description: Short description of the command
   ---
   ```

2. **Clear Instructions**: Provide step-by-step instructions

3. **Prerequisites**: Document required files or conditions

4. **Output**: Clearly specify what files or artifacts are created

5. **Examples**: Include usage examples in the description

### Manifest Updates

When adding new agents or commands:

1. **Update plugin.json**: Add entry to `agents` or `commands` array
2. **Update README.md**: Document the new feature
3. **Update marketplace.json**: If changing plugin metadata
4. **Validate JSON**: Ensure all JSON files are valid

Test validation:
```bash
python3 -m json.tool < plugin.json > /dev/null
python3 -m json.tool < .claude-plugin/marketplace.json > /dev/null
```

### Testing

Before submitting a PR:

1. **Install locally**: Test the plugin installation
   ```bash
   /plugin install /path/to/your/fork/claude-code
   ```

2. **Test commands**: Verify all slash commands work
   ```bash
   /create-spec "Test feature"
   /test-scenarios test-dir/
   /create-tech-design test-dir/
   /start-tdd test-dir/
   ```

3. **Test agents**: Verify agents can be invoked and complete their tasks

4. **Validate JSON**: Ensure all JSON files are syntactically correct

5. **Check documentation**: Ensure README.md is accurate

### Code Quality

- **Clear and Concise**: Write clear, understandable instructions
- **Consistent Style**: Follow existing patterns in the codebase
- **No Hardcoded Paths**: Use relative paths, never absolute
- **Error Messages**: Provide helpful error messages
- **Examples**: Include examples in documentation

### Commit Messages

Use clear, descriptive commit messages:

- `feat: add new agent for database migrations`
- `fix: correct path handling in requirements-analyzer`
- `docs: update README with new workflow examples`
- `refactor: simplify tdd-red agent logic`
- `test: add validation for marketplace.json`

## Plugin Versioning

We follow [Semantic Versioning](https://semver.org/):

- **MAJOR**: Breaking changes to commands, agents, or output format
- **MINOR**: New features that are backwards compatible
- **PATCH**: Bug fixes and minor improvements

When submitting PRs, indicate if the change requires a version bump.

## Documentation

All changes should include documentation updates:

1. **README.md**: Update if adding/changing features
2. **CONTRIBUTING.md**: Update if changing contribution process
3. **Agent/Command files**: Keep inline documentation current
4. **Examples**: Add/update examples as needed

## Questions?

If you have questions about contributing:

1. Check existing [issues](https://gitlab.com/dgomezs2/ai/claude-code/-/issues)
2. Open a [merge request](https://gitlab.com/dgomezs2/ai/claude-code/-/merge_requests) for discussion
3. Reference [Claude Code documentation](https://docs.claude.com/en/docs/claude-code)

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

Thank you for contributing to TDD SpecFlow!
