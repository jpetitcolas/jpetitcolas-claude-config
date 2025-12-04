---
name: developing-skills
description: Creates and manages Claude Code skills following Anthropic best practices. Use when creating new skills, understanding skill structure, writing SKILL.md files, or implementing progressive disclosure. Covers YAML frontmatter, the 500-line rule, rich descriptions for auto-activation, and reference file patterns.
---

# Developing Skills

**Location:** `skills/{skill-name}/SKILL.md`

## Skill Template

```yaml
---
name: processing-something  # Required: gerund form, lowercase, hyphens, max 64 chars
description: Rich text...   # Required: third person, max 1024 chars, trigger keywords
---

# Skill Title

[Core guidance, patterns, examples - what Claude needs to know to help effectively]
```

## Examples of Good Descriptions

* **writing-tests:** Provides testing best practices for JavaScript/TypeScript with Vitest. Use when writing tests, creating test files, fixing failing tests, mocking time or functions. Covers vi.useFakeTimers, vi.stubEnv, it.each, hard-coded assertions, behavior-focused testing.
* **writing-semantic-commits:** Writes human-focused, semantic git commit messages. Use when creating commits, writing commit messages, committing code changes, or preparing to push. Covers commit format, motivation sections, impact descriptions.
* **generating-api-docs:** Generates OpenAPI/Swagger documentation for REST APIs. Use when documenting endpoints, creating API specs, or setting up Swagger UI. Covers path definitions, request/response schemas, authentication.

## Checklist for Effective Skills

### Core Quality

* [ ] Description is specific and includes key terms
* [ ] Description includes both what the Skill does and when to use it
* [ ] SKILL.md body is under 500 lines
* [ ] Additional details are in separate files (if needed)
* [ ] No time-sensitive information (or in "old patterns" section)
* [ ] Consistent terminology throughout
* [ ] Examples are concrete, not abstract
* [ ] File references are one level deep
* [ ] Progressive disclosure used appropriately: start concise, provide paths to depth.
* [ ] Workflows have clear steps

### Testing

* [ ] At least three evaluations created
* [ ] Tested with Haiku, Sonnet, and Opus
* [ ] Tested with real usage scenarios
* [ ] Team feedback incorporated (if applicable)

## Testing A Skill

1. **Install/reload the plugin** containing the skill
2. **Test with natural prompts** that should trigger it
3. **Verify Claude uses the Skill tool** when appropriate
4. **Refine the description** based on activation patterns

If Claude isn't using your skill when expected, enhance the description with more specific trigger keywords.