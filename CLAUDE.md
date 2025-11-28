# Jonathan Petitcolas' Claude Configuration

This is a **Claude Code plugin** repository that serves as a shared configuration for jpetitcolas' projects. It provides reusable skills that can be installed across multiple projects via Claude's plugin system.

**Important**: This is primarily a configuration repository. Skills are pure markdown files with YAML frontmatter — no build or compilation step required.

## Architecture

### Plugin System
- Plugin metadata defined in `.claude-plugin/plugin.json`
- Marketplace registration in `.claude-plugin/marketplace.json`
- All customizations are file-based

### Directory Structure
- **skills/** - Model-invoked capabilities (directories containing `SKILL.md`)

## Testing Changes

Since this is a plugin system:

1. **Install plugin locally during development**:
   ```bash
   claude plugins install /home/jpetitcolas/dev/jpetitcolas-claude-config
   ```

2. **Verify installation**:
   ```bash
   claude plugins list
   ```

3. **Test skills** in a Claude Code session

4. **Reload after changes**: Disable and re-enable the plugin:
   ```bash
   claude plugins disable jpetitcolas-claude-config
   claude plugins enable jpetitcolas-claude-config
   ```

## Key Principles

- **No code compilation**: This is a pure content/configuration repository
- **Reusability**: Skills should be project-agnostic where possible
- **Markdown-first**: All extensibility happens through markdown files with frontmatter
- **Version control**: Update version in `.claude-plugin/plugin.json` for releases
- **Documentation**: Keep README.md in sync with actual capabilities

## Skills

Skills are auto-invoked by Claude based on their description. Write rich descriptions with trigger keywords for reliable activation.
