# jpetitcolas-claude-config

Shared Claude Code configuration plugin for jpetitcolas projects.

## Installation

### From Marketplace (Recommended)

Add the marketplace, then install the plugin:

```bash
claude plugin marketplace add git@github.com:jpetitcolas/jpetitcolas-claude-config.git
claude plugins install jpetitcolas-claude-config
```

### From Local Path

For development or testing:

```bash
claude plugins install /path/to/jpetitcolas-claude-config
```

## What's Included

This plugin provides the structure for:

- **Slash Commands**: Add custom commands in the `commands/` directory
- **Subagents**: Add specialized agents in `agents/` for specific tasks
- **Skills**: Add model-invoked capabilities in `skills/`
- **Hooks**: Optionally add event handlers by creating a `hooks/` directory

## Usage

After installation, your custom commands, agents, hooks, and skills will be available across all projects where this plugin is installed.

To verify installation:

```bash
claude plugins list
```

To enable/disable the plugin:

```bash
claude plugins disable jpetitcolas-claude-config
claude plugins enable jpetitcolas-claude-config
```
