# jpetitcolas-claude-config

Shared Claude Code configuration plugin for jpetitcolas projects.

## Prerequisites

- **Node.js** (v18 or higher) - Required for hook execution
- **pnpm** (v10 or higher) - Package manager for hook dependencies

## Installation

### From Marketplace (Recommended)

1. Navigate to your home directory and start Claude:
   ```bash
   cd ~
   claude
   ```

2. Use the `/plugins` command to add the marketplace:
   - Type `/plugins`
   - Select "Add marketplace"
   - Enter: `https://github.com/jpetitcolas/jpetitcolas-claude-config.git`

3. Install the plugin:
   - Type `/plugins`
   - Select "Browse and install plugins"
   - Select the `jpetitcolas-claude-config` marketplace
   - Choose the `jpetitcolas-claude-config` plugin to install it

### From Local Path

For development or testing:

```bash
# First, install hook dependencies
cd /path/to/jpetitcolas-claude-config/hooks
pnpm install

# Then install the plugin
claude plugins install /path/to/jpetitcolas-claude-config
```

## What's Included

### Auto-Registering Hooks
**No manual configuration required** - hooks activate automatically when plugin is enabled:

- **skill-activation-prompt** (UserPromptSubmit hook): Analyzes user prompts and proactively suggests relevant skills before Claude processes requests

### Skills

- **skill-developer**: Meta-skill for creating and managing Claude Code skills
- **semantic-commits**: Guidance for creating semantic commit messages

### Slash Commands
- **semantic-commit**: Creates semantic commit messages following conventional commit standards

### Future Additions
- **Subagents**: Specialized agents for specific tasks (coming soon)

## Usage

After installation, everything works automatically:

- **Hooks** auto-register and activate (no `.claude/settings.json` configuration needed)
- **Skills** are available via the `Skill` tool
- **Commands** are available as `/semantic-commit`

The skill-activation hook will proactively suggest relevant skills based on your prompts, helping you follow best practices automatically.

### Plugin Management

Verify installation:
```bash
claude plugins list
```

Enable/disable the plugin:
```bash
claude plugins disable jpetitcolas-claude-config
claude plugins enable jpetitcolas-claude-config
```

### How It Works

When you type a prompt like "Let's add a new API endpoint", the UserPromptSubmit hook automatically:
1. Analyzes your prompt for keywords and intent patterns
2. Matches against skill trigger rules
3. Suggests relevant skills (e.g., "backend-dev-guidelines")
4. Claude sees these suggestions and can invoke the skill before responding

## Project-Level Configuration

**Important:** Project-level settings override user-level settings. If you have a project with its own `.claude/settings.json`, you may need to enable the plugin for that specific project.

### Enable Plugin in a Specific Project

To enable this plugin in a project without committing the configuration, add it to your project's `.claude/settings.local.json`:

```json
{
  "enabledPlugins": {
    "jpetitcolas-claude-config@jpetitcolas-claude-config": true
  }
}
```

**Why this is needed:**
- User-level plugins (`~/.claude/settings.json`) are enabled globally
- But project-level settings (`.claude/settings.json`) can override this
- Using `.claude/settings.local.json` keeps the configuration local (not committed)

**Example:**
If you install this plugin globally but it's not triggering in a specific project, check if that project has `.claude/settings.json` that doesn't include this plugin in its `enabledPlugins` list.
