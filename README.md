# jpetitcolas-claude-config

Shared Claude Code configuration plugin for jpetitcolas projects. Provides reusable skills that auto-activate based on context.

## Installation

### From Marketplace

1. Start Claude:
   ```bash
   claude
   ```

2. Add the marketplace:
   - Type `/plugins`
   - Select "Add marketplace"
   - Enter: `https://github.com/jpetitcolas/jpetitcolas-claude-config.git`

3. Install the plugin:
   - Type `/plugins`
   - Select "Browse and install plugins"
   - Choose `jpetitcolas-claude-config`

### From Local Path (Development)

```bash
claude plugins install /path/to/jpetitcolas-claude-config
```

## What's Included

### Agents

Agents are specialized Claude instances that handle complex tasks. Claude automatically invokes them when appropriate, or you can request them explicitly:

| Agent | Purpose | Model |
|-------|---------|-------|
| **test-verifier** | Automatically verifies tests pass after code changes with deep failure analysis | Opus 4.5 |
| **format-checker** | Autonomous formatting & linting agent that actively fixes code quality issues | Haiku 4.5 |

**How test-verifier works:**
- Auto-triggers when code changes in `apps/` or `packages/` directories
- Progressive testing: single file → package → dependencies
- Deep root cause analysis for test failures
- Always invokes `writing-tests` skill for best practices validation
- Fully generic across all Turbo+pnpm monorepo projects

**How format-checker works:**
- Runs automated fixes with `pnpm format:fix` (Biome, Prettier, ESLint, etc.)
- Analyzes remaining linting errors
- **Actively fixes issues** using Read + Edit tools (replaces `any` types, fixes bracket notation, etc.)
- Iterates up to 3 times until all issues resolved
- Reports what it fixed or why it needs manual intervention
- Generic across any project with `format:fix` and `format:check` scripts

### Skills (Auto-Activated)

Skills are automatically invoked by Claude based on your prompt context:

| Skill | Purpose |
|-------|---------|
| **semantic-commits** | Human-focused commit message guidelines (no feat/fix prefixes) |
| **writing-tests** | Vitest best practices: time mocking, assertions, behavior-focused testing |
| **skill-developer** | Guide for creating Claude Code skills with 500-line rule |

### How Skills Work

Claude automatically uses skills when your prompt matches their description. No manual invocation needed.

**Examples:**
- "Let's commit these changes" → triggers `semantic-commits`
- "Write tests for this function" → triggers `writing-tests`
- "How do I create a new skill?" → triggers `skill-developer`

## Plugin Management

```bash
# Verify installation
claude plugins list

# Enable/disable
claude plugins disable jpetitcolas-claude-config
claude plugins enable jpetitcolas-claude-config
```

## Project-Level Configuration

If a project has `.claude/settings.json`, enable this plugin in `.claude/settings.local.json`:

```json
{
  "enabledPlugins": {
    "jpetitcolas-claude-config@jpetitcolas-claude-config": true
  }
}
```
