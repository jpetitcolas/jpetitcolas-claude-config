# Jonathan Petitcolas' Claude Configuration

This is a **Claude Code plugin** repository that serves as a shared configuration for jpetitcolas' projects. It provides reusable commands, agents, and skills that can be installed across multiple projects via Claude's plugin system.

**Important**: This is primarily a configuration repository. Commands, agents, and skills are pure markdown/JSON. The `hooks/` directory contains TypeScript code with dependencies managed via pnpm.

## Architecture

### Plugin System
- Plugin metadata defined in `.claude-plugin/plugin.json`
- Marketplace registration in `.claude-plugin/marketplace.json`
- All customizations are file-based with no build or compilation step

### Directory Structure
- **commands/** - Slash commands (`.md` files with YAML frontmatter)
- **agents/** - Specialized subagents (`.md` files with YAML frontmatter)
- **skills/** - Model-invoked capabilities (directories containing `SKILL.md`)
- **hooks/** - Event handlers that auto-register via `hooks.json` (TypeScript + shell scripts)

## Prerequisites

- **Node.js** (v18 or higher) - Required for hook execution
- **pnpm** (v10 or higher) - Package manager for hook dependencies

## Testing Changes

Since this is a plugin system:

1. **Install hook dependencies** (first time only):
   ```bash
   cd /home/jpetitcolas/dev/jpetitcolas-claude-config/hooks
   pnpm install
   ```

2. **Install plugin locally during development**:
   ```bash
   claude plugins install /home/jpetitcolas/dev/jpetitcolas-claude-config
   ```

3. **Verify installation**:
   ```bash
   claude plugins list
   ```

4. **Test commands/agents/skills/hooks** in a Claude Code session

5. **Reload after changes**: Disable and re-enable the plugin:
   ```bash
   claude plugins disable jpetitcolas-claude-config
   claude plugins enable jpetitcolas-claude-config
   ```

## Key Principles

- **No code compilation**: This is a pure content/configuration repository
- **Reusability**: Commands and agents should be project-agnostic where possible
- **Markdown-first**: All extensibility happens through markdown files with frontmatter
- **Version control**: Update version in `.claude-plugin/plugin.json` for releases
- **Documentation**: Keep README.md in sync with actual capabilities
