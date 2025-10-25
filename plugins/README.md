# Plugins Directory

This directory contains additional plugins that are part of the TDD SpecFlow marketplace.

## Current Plugins

Currently, no additional plugins are hosted in this directory. The main `tdd-specflow` plugin is located in the repository root.

## Adding a New Plugin

To add a new plugin to this marketplace:

### 1. Create Plugin Directory

```bash
mkdir -p plugins/your-plugin-name
cd plugins/your-plugin-name
```

### 2. Create Plugin Structure

```
plugins/your-plugin-name/
├── plugin.json              # Plugin manifest (required)
├── README.md               # Plugin documentation
├── LICENSE                 # Plugin license
└── .claude/
    ├── agents/             # Custom agents (optional)
    │   └── agent-name.md
    └── commands/           # Slash commands (optional)
        └── command-name.md
```

### 3. Create plugin.json

```json
{
  "name": "your-plugin-name",
  "version": "1.0.0",
  "description": "Your plugin description",
  "author": "your-name",
  "license": "MIT",
  "homepage": "https://gitlab.com/dgomezs2/ai/claude-code/-/tree/main/plugins/your-plugin-name",
  "keywords": ["keyword1", "keyword2"],
  "commands": [
    {
      "name": "/your-command",
      "description": "Command description",
      "path": ".claude/commands/your-command.md"
    }
  ],
  "agents": [
    {
      "name": "your-agent",
      "description": "Agent description",
      "path": ".claude/agents/your-agent.md"
    }
  ]
}
```

### 4. Update Marketplace Manifest

Add your plugin to `.claude-plugin/marketplace.json`:

```json
{
  "plugins": [
    {
      "name": "tdd-specflow",
      "source": ".",
      ...
    },
    {
      "name": "your-plugin-name",
      "source": "./plugins/your-plugin-name",
      "description": "Your plugin description",
      "version": "1.0.0",
      "author": "your-name",
      "license": "MIT",
      "category": "appropriate-category",
      "keywords": ["keyword1", "keyword2"],
      "tags": ["tag1", "tag2"]
    }
  ]
}
```

### 5. Document Your Plugin

Create a comprehensive README.md in your plugin directory explaining:
- What the plugin does
- Installation instructions
- Available commands and agents
- Usage examples
- Configuration options

### 6. Test Your Plugin

```bash
# Validate JSON
python3 -m json.tool < plugins/your-plugin-name/plugin.json > /dev/null

# Test installation
/plugin install /path/to/claude-code/plugins/your-plugin-name

# Test commands
/your-command
```

### 7. Use Your Customized Plugin

Install your customized plugin locally in your fork:
```bash
/plugin install /path/to/your/fork/plugins/your-plugin-name
```

**Note:** This repository does not accept merge requests. Maintain plugins in your own fork.

## Plugin Categories

Use standard categories:
- `development-workflow`
- `testing`
- `code-quality`
- `documentation`
- `productivity`
- `integration`
- `debugging`
- `deployment`

## Guidelines

- Follow the same quality standards as the main tdd-specflow plugin
- Include comprehensive documentation
- Test thoroughly before submitting
- Use semantic versioning
- Include a LICENSE file
- Add meaningful keywords for discoverability

## Examples

For reference, see the main tdd-specflow plugin structure in the repository root.

## Documentation

For information about the marketplace structure:
- [MARKETPLACE.md](../MARKETPLACE.md) - Detailed marketplace documentation
- [CONTRIBUTING.md](../CONTRIBUTING.md) - Development guidelines for forking
- [Claude Code Docs](https://docs.claude.com/en/docs/claude-code) - Official Claude Code documentation

**Note:** This repository does not accept issues or merge requests. Fork and customize as needed.
