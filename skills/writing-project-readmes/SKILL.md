---
name: writing-project-readmes
description: Writes concise, focused project README files following best practices. Use when creating or updating README.md files, documenting projects, or simplifying overly detailed documentation. Covers description, purpose, quick start, essential commands, and avoiding unnecessary details. Focuses on getting users started quickly.
---

# Writing Project READMEs

READMEs should be **concise and action-oriented**, not comprehensive documentation. The goal is to help users understand what the project does and get started quickly.

## README Structure

### 1. Title and One-Line Description

```markdown
# Project Name

One sentence describing what this does.
```

**Good:** "Run Claude Code in a secure Docker container with filesystem and network isolation."
**Bad:** "This is a comprehensive solution for managing Claude Code instances with advanced security features..."

### 2. What Is This? (Optional, for non-obvious projects)

3-5 bullet points explaining:
- What problem it solves
- Key features (not all features, just the important ones)
- Who should use it

**Example:**
```markdown
## What is this?

A containerized Claude Code setup that:
- **Limits filesystem access** to the current project directory only
- **Restricts network access** to approved domains via iptables firewall
- **Isolates credentials** from the host system
```

### 3. Quick Start (Required)

**Clear commands with inline comments:**

```markdown
## Quick Start

\`\`\`bash
make install    # Setup the project
make uninstall  # Remove the project
\`\`\`

## Run

\`\`\`bash
cd /path/to/project
command-name
\`\`\`
```

**Rules:**
- Commands with inline comments (no sub-headings for each command)
- Separate "Run" section for usage
- Each command should be copy-pasteable
- Keep to 2-3 essential commands only
- Put detailed setup in a separate doc if needed

### 4. Essential Information Only

Include **only** what users need to know:
- Configuration options (if simple)
- Common use cases
- Critical limitations

**Don't include:**
- Implementation details
- Complete API documentation
- Detailed architecture (unless it's the project's purpose)
- Contribution guidelines (use CONTRIBUTING.md)
- Every single feature
- Version history (use CHANGELOG.md)

### 5. Troubleshooting (Optional)

Only the 2-3 most common issues:

```markdown
## Troubleshooting

**"Error X":**
\`\`\`bash
solution-command
\`\`\`
```

## Common Mistakes to Avoid

### ❌ Too Much Detail

```markdown
## Security Model

### What Claude CAN Access
- Current workspace: Full read/write access...
- Credentials: Read-only access with X permissions...
- Session state: Persistent storage using Docker volumes with...

### What Claude CANNOT Access
- Host filesystem outside mounted workspace
- Other project directories
- System files (/etc, /var, etc.)
...
```

**Why it's bad:** Users don't need to know all security details upfront. Link to SECURITY.md if needed.

### ❌ Too Many Commands

```markdown
## Commands

make help        # Show help
make build       # Build
make install     # Install
make uninstall   # Uninstall
make clean       # Clean
make test        # Test
make lint        # Lint
...
```

**Why it's bad:** Overwhelming. Show only essential commands.

**Better:**
```markdown
## Quick Start

\`\`\`bash
make install    # Setup the project
make uninstall  # Remove the project
\`\`\`
```

### ❌ Architecture Diagrams for Simple Projects

Only include architecture diagrams if:
- The project is a system/platform
- The diagram helps users understand how to use it
- It's truly simple (3-5 boxes max)

### ❌ Repeating Package Manager Commands

```markdown
## Installation

**npm:**
\`\`\`bash
npm install package
\`\`\`

**yarn:**
\`\`\`bash
yarn add package
\`\`\`

**pnpm:**
\`\`\`bash
pnpm add package
\`\`\`
```

**Why it's bad:** Users know how their package manager works. Just show one.

## README Length Guidelines

- **Libraries/Tools:** 50-150 lines max
- **Applications:** 100-250 lines max
- **Platforms/Systems:** 200-400 lines max

If your README exceeds these, **move content to separate docs**:
- `docs/ARCHITECTURE.md` - System design
- `docs/API.md` - API reference
- `CONTRIBUTING.md` - Contribution guidelines
- `SECURITY.md` - Security details
- `docs/ADVANCED.md` - Advanced usage

## Good README Template

```markdown
# Project Name

One-line description.

## Quick Start

\`\`\`bash
make install    # Setup the project
make uninstall  # Remove the project
\`\`\`

## Run

\`\`\`bash
cd /path/to/project
command-name
\`\`\`

## Configuration (if needed)

Key config options only.

## Troubleshooting

Most common issue only.

## License

License info.
```

## Examples of Good READMEs

**Good:** Simple library README
```markdown
# slugify

Convert strings to URL-friendly slugs.

## Install

\`\`\`bash
npm install @sindresorhus/slugify
\`\`\`

## Usage

\`\`\`javascript
import slugify from '@sindresorhus/slugify';

slugify('Hello World!');
//=> 'hello-world'
\`\`\`

## Options

See [API docs](docs/api.md) for all options.
```

**Good:** Tool README
```markdown
# claude-isolated

Run Claude Code in an isolated Docker container.

## Quick Start

\`\`\`bash
make install    # Setup the project
make uninstall  # Remove the project
\`\`\`

## Run

\`\`\`bash
cd /your/project
claude-isolated
\`\`\`

## Requirements

- Docker with CAP_NET_ADMIN
- Claude credentials (~/.claude/.credentials.json)
```

## When Writing READMEs

1. **Start with the title and one-line description**
2. **Add Quick Start section** with 2-3 steps maximum
3. **Add only essential configuration/options**
4. **Add 1-2 troubleshooting items** if needed
5. **Stop there** - resist adding more

## Measuring README Quality

A good README should:
- [ ] Can be read in under 2 minutes
- [ ] User can start using the project in under 5 minutes
- [ ] No sections are optional for basic usage
- [ ] Architecture diagrams (if any) are simple and helpful
- [ ] Doesn't repeat information from package.json/setup.py
- [ ] Links to detailed docs instead of including them inline

## Progressive Disclosure

Start simple, provide paths to complexity:

```markdown
## Configuration

Basic usage works with defaults. See [docs/config.md](docs/config.md) for advanced options.
```

Not:

```markdown
## Configuration

### Basic Options
...
### Advanced Options
...
### Expert Options
...
```
