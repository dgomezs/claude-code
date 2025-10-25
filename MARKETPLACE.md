# TDD SpecFlow Marketplace

This repository serves as a **Claude Code plugin marketplace** that can host multiple plugins. Currently, it contains the `tdd-specflow` plugin, with the infrastructure ready to add more plugins in the future.

## Current Plugins

### tdd-specflow (Main Plugin)
A comprehensive TDD workflow plugin that combines Test-Driven Development with Specification by Example.

**Location**: Root directory
**Version**: 0.0.1
**Installation**: `/plugin install https://gitlab.com/dgomezs2/ai/claude-code`

See [README.md](README.md) for complete documentation.

## Marketplace Structure

```
tdd-specflow/
├── .claude-plugin/
│   └── marketplace.json      # Marketplace manifest
├── plugin.json               # Main plugin (tdd-specflow)
├── .claude/                  # Main plugin components
│   ├── agents/
│   └── commands/
└── plugins/                  # Directory for additional plugins
    └── (future plugins here)
```

## Adding More Plugins to This Marketplace

### Option 1: Add Plugin in Subdirectory

1. **Create plugin directory**:
   ```bash
   mkdir -p plugins/your-plugin-name
   cd plugins/your-plugin-name
   ```

2. **Create plugin structure**:
   ```
   plugins/your-plugin-name/
   ├── plugin.json
   └── .claude/
       ├── agents/
       │   └── your-agent.md
       └── commands/
           └── your-command.md
   ```

3. **Update marketplace.json**:
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
         "description": "Description of your plugin",
         "version": "1.0.0",
         "author": "your-name",
         "license": "MIT",
         "keywords": ["keyword1", "keyword2"]
       }
     ]
   }
   ```

4. **Create plugin.json** in `plugins/your-plugin-name/`:
   ```json
   {
     "name": "your-plugin-name",
     "version": "1.0.0",
     "description": "Your plugin description",
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

### Option 2: Reference External Plugin Repository

Add a plugin from another GitHub repository:

```json
{
  "plugins": [
    {
      "name": "tdd-specflow",
      "source": ".",
      ...
    },
    {
      "name": "external-plugin",
      "source": {
        "type": "github",
        "repo": "username/repo-name"
      },
      "description": "External plugin description",
      "version": "1.0.0",
      "author": "external-author"
    }
  ]
}
```

### Option 3: Reference Plugin from Git URL

```json
{
  "plugins": [
    {
      "name": "tdd-specflow",
      "source": ".",
      ...
    },
    {
      "name": "git-plugin",
      "source": {
        "type": "url",
        "url": "https://github.com/org/plugin.git"
      },
      "description": "Plugin from git URL"
    }
  ]
}
```

## Plugin Guidelines

When adding plugins to this marketplace:

1. **Follow Claude Code Plugin Standards**
   - Include valid `plugin.json` manifest
   - Document all agents and commands
   - Provide examples and usage instructions

2. **Maintain Quality**
   - Test plugins before adding to marketplace
   - Include comprehensive documentation
   - Follow semantic versioning

3. **Licensing**
   - Ensure compatible licenses (MIT recommended)
   - Include LICENSE file in plugin directory
   - Credit original authors

4. **Categories**
   Use standard categories:
   - `development-workflow`
   - `testing`
   - `code-quality`
   - `documentation`
   - `productivity`
   - `integration`

5. **Keywords and Tags**
   - Use descriptive keywords for searchability
   - Add relevant tags for categorization
   - Include technology/language-specific tags

## Installing Plugins from This Marketplace

### Install Specific Plugin

```bash
# Install tdd-specflow (main plugin)
/plugin install https://gitlab.com/dgomezs2/ai/claude-code

# Install plugin from subdirectory (when available)
/plugin install https://gitlab.com/dgomezs2/ai/claude-code/plugins/plugin-name
```

### Install via Marketplace Reference

Users can reference this marketplace in their Claude Code settings to browse all available plugins.

## Testing Plugin Additions

Before committing new plugins:

1. **Validate JSON**:
   ```bash
   python3 -m json.tool < .claude-plugin/marketplace.json > /dev/null
   python3 -m json.tool < plugins/your-plugin/plugin.json > /dev/null
   ```

2. **Test Installation**:
   ```bash
   /plugin install /path/to/claude-code/plugins/your-plugin
   ```

3. **Verify Commands and Agents**:
   ```bash
   /your-command
   # Test that agents can be invoked
   ```

## Marketplace Metadata

The marketplace includes metadata in `.claude-plugin/marketplace.json`:

- **name**: Unique marketplace identifier
- **owner**: Maintainer information and repository URL
- **metadata.description**: Brief marketplace overview
- **metadata.version**: Marketplace version (follows semver)
- **metadata.pluginRoot**: Base path for relative plugin sources (default: "plugins")

## Contributing Plugins

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on contributing plugins to this marketplace.

To propose a new plugin:
1. Fork this repository
2. Add your plugin following the structure above
3. Update `.claude-plugin/marketplace.json`
4. Submit a pull request with plugin documentation

## Support

- **Issues**: https://gitlab.com/dgomezs2/ai/claude-code/-/issues
- **Discussions**: https://gitlab.com/dgomezs2/ai/claude-code/-/merge_requests
- **Claude Code Docs**: https://docs.claude.com/en/docs/claude-code

## License

Marketplace infrastructure: MIT License

Individual plugins may have their own licenses - see each plugin's LICENSE file.
