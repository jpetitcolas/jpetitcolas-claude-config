---
name: skill-developer
description: Create and manage Claude Code skills following Anthropic best practices. Use when creating new skills, understanding skill structure, writing SKILL.md files, or implementing progressive disclosure. Covers YAML frontmatter, the 500-line rule, rich descriptions for auto-activation, and reference file patterns.
---

# Skill Developer Guide

## Purpose

Guide for creating skills in Claude Code using the native auto-activation system, following Anthropic's best practices including the 500-line rule and progressive disclosure pattern.

## When to Use This Skill

Automatically activates when you mention:
- Creating or adding skills
- Understanding how skill activation works
- Claude Code skill best practices
- Progressive disclosure
- YAML frontmatter
- 500-line rule

---

## How Skills Work (Native System)

Claude Code skills are **model-invoked** — Claude autonomously decides when to use them based on your request and the skill's description.

### Auto-Discovery

Skills are automatically discovered from:
- Personal Skills (`~/.config/claude-code/skills/`)
- Project Skills (`.claude/skills/`)
- Plugin Skills (installed plugins' `skills/` directories)

### Trigger Mechanism

The **only native trigger** is the `description` field in YAML frontmatter. Claude uses semantic matching to determine relevance.

**Key insight:** The richer your description (with concrete trigger terms), the better Claude matches it to user requests.

---

## Quick Start: Creating a New Skill

### Step 1: Create Skill File

**Location:** `skills/{skill-name}/SKILL.md`

**Template:**
```markdown
---
name: my-new-skill
description: Brief description including keywords that trigger this skill. Use when [specific scenarios]. Covers [topics, patterns, use cases].
---

# My New Skill

## Purpose
What this skill helps with

## When to Use
Specific scenarios and conditions

## Key Information
The actual guidance, documentation, patterns, examples
```

### Step 2: Write a Rich Description

The description is your **only trigger mechanism**. Make it count:

```yaml
# ❌ Too vague - won't trigger reliably
description: Help with testing

# ✅ Rich with triggers - activates appropriately
description: Testing best practices for JavaScript/TypeScript with Vitest. Use when writing tests, creating test files, fixing failing tests, mocking time or functions. Covers vi.useFakeTimers, vi.stubEnv, it.each, hard-coded assertions, behavior-focused testing.
```

**Guidelines:**
- Max 1024 characters
- Include action phrases: "Use when...", "Covers..."
- List specific keywords users might mention
- Name tools, libraries, patterns explicitly

### Step 3: Follow Best Practices

- **Name**: Lowercase, hyphens, max 64 characters
- **Content**: Under 500 lines — use reference files for details
- **Examples**: Include real code examples
- **Structure**: Clear headings, lists, code blocks

---

## The 500-Line Rule

Skills should be under 500 lines. For comprehensive topics:

1. Keep `SKILL.md` as an overview/quick reference
2. Create reference files for detailed content
3. Link to reference files from the main skill

**Example structure:**
```
skills/my-skill/
├── SKILL.md              # Overview (< 500 lines)
├── DETAILED_GUIDE.md     # Deep dive
├── PATTERNS.md           # Pattern library
└── TROUBLESHOOTING.md    # Common issues
```

---

## Progressive Disclosure Pattern

**Principle:** Start concise, provide paths to depth.

```markdown
## Quick Reference

[Essential information here - enough for common cases]

---

## Reference Files

For detailed information:
- [DETAILED_GUIDE.md](DETAILED_GUIDE.md) - Complete walkthrough
- [PATTERNS.md](PATTERNS.md) - Pattern library
```

**Guidelines:**
- Add table of contents to reference files > 100 lines
- Keep nesting one level deep (don't chain references)
- Each file should be self-contained

---

## Testing Your Skill

1. **Install/reload the plugin** containing the skill
2. **Test with natural prompts** that should trigger it
3. **Verify Claude uses the Skill tool** when appropriate
4. **Refine the description** based on activation patterns

If Claude isn't using your skill when expected, enhance the description with more specific trigger keywords.

---

## Checklist

When creating a new skill:

- [ ] Skill file created in `skills/{name}/SKILL.md`
- [ ] YAML frontmatter with `name` and `description`
- [ ] Description includes trigger keywords and "Use when..." phrases
- [ ] Content under 500 lines
- [ ] Reference files for detailed content (if needed)
- [ ] Real code examples included
- [ ] Tested with 3+ real prompts

---

## YAML Frontmatter Reference

```yaml
---
name: skill-name           # Required: lowercase, hyphens, max 64 chars
description: Rich text...  # Required: max 1024 chars, trigger keywords
allowed-tools:             # Optional: restrict which tools can be used
  - Bash
  - Write
  - Read
---
```

**Fields:**
- `name`: Identifier (lowercase, hyphens only)
- `description`: Trigger text for semantic matching
- `allowed-tools`: Optional security restriction

---

## Examples of Good Descriptions

**Testing skill:**
```yaml
description: Testing best practices for JavaScript/TypeScript with Vitest. Use when writing tests, creating test files, fixing failing tests, mocking time or functions. Covers vi.useFakeTimers, vi.stubEnv, it.each, hard-coded assertions, behavior-focused testing.
```

**Commit message skill:**
```yaml
description: Write human-focused, semantic git commit messages. Use when creating commits, writing commit messages, committing code changes, or preparing to push. Covers commit format, motivation sections, impact descriptions.
```

**API documentation skill:**
```yaml
description: Generate OpenAPI/Swagger documentation for REST APIs. Use when documenting endpoints, creating API specs, or setting up Swagger UI. Covers path definitions, request/response schemas, authentication.
```

---

**Line Count**: < 200 (well under 500-line rule)
