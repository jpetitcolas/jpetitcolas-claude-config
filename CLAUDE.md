# Jonathan Petitcolas' Claude Configuration

This is a **Claude Code plugin** repository that serves as a shared configuration for jpetitcolas' projects. It provides reusable commands, agents, and skills that can be installed across multiple projects via Claude's plugin system.

**Important**: This is NOT a code project with build steps or dependencies. It's a collection of markdown files and JSON configuration that extend Claude Code's capabilities.

## Architecture

### Plugin System
- Plugin metadata defined in `.claude-plugin/plugin.json`
- Marketplace registration in `.claude-plugin/marketplace.json`
- All customizations are file-based with no build or compilation step

### Directory Structure
- **commands/** - Slash commands (`.md` files with YAML frontmatter)
- **agents/** - Specialized subagents (`.md` files with YAML frontmatter)
- **skills/** - Model-invoked capabilities (directories containing `SKILL.md`)
- **hooks/** - (Optional) Event handlers with `hooks.json` and shell scripts

## Testing Changes

Since this is a plugin system:

1. **Install locally during development**:
   ```bash
   claude plugins install /home/jpetitcolas/dev/jpetitcolas-claude-config
   ```

2. **Verify installation**:
   ```bash
   claude plugins list
   ```

3. **Test commands/agents/skills** in a Claude Code session

4. **Reload after changes**: Disable and re-enable the plugin:
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
