---
name: semantic-commits
description: Write human-focused, semantic git commit messages. Use when creating commits, writing commit messages, committing code changes, or preparing to push. Covers commit format, motivation sections, impact descriptions, and avoiding conventional commit prefixes like feat/fix/chore.
---

# Semantic Commit Messages

When writing git commit messages, follow this human-focused, semantic style that emphasizes WHY and IMPACT over technical details.

## Structure

1. **Title**: Clear, action-oriented summary (no conventional commit prefixes)
2. **What Changed**: Brief explanation of the change
3. **Motivation**: Why this change was needed (user/business value)
4. **Impact**: What this means for users, developers, or the system

## Good Example

```
Add JWT-based session refresh mechanism

Automatic session refresh reduces logouts and improves UX consistency.
Access tokens expire after 15 minutes, while a 7-day refresh token renews
sessions transparently. Old `/token` endpoint removed.

Motivation:
Frequent user logouts were caused by short-lived tokens. This change keeps
security tight without hurting usability.

Impact:
Clients must switch to `/refresh`. Frontend updated accordingly.
```

## Bad Example (Avoid This)

```
feat(auth): add JWT-based session refresh mechanism

Introduce automatic session refresh using JWT tokens to reduce user logouts
and improve UX consistency. The access token now expires after 15 min, while
a refresh token (valid for 7 days) can renew it transparently.

Changes:
- Add `refreshToken` column to `users` table
- Update `/login` route to issue both tokens
- Add `/refresh` endpoint with proper validation
- Update frontend API client to auto-refresh on 401 responses

BREAKING CHANGE: old `/token` endpoint removed. Clients must switch to `/refresh`.
Co-authored-by: Alice Doe <alice@tint.ai>
```

## Key Principles

- **No conventional commit prefixes** (no `feat:`, `fix:`, `chore:`, etc.)
- **Focus on WHY, not just WHAT**: Explain the motivation and business value
- **Narrative style**: Write in prose, not bullet points
- **Human-readable**: Write for people who want to understand the change
- **Avoid implementation details**: Don't list every file or function changed
- **Context over details**: Explain impact and motivation over technical minutiae

## What to Include

- Clear, descriptive title without prefixes
- Brief summary of what changed
- Motivation section explaining why
- Impact section explaining consequences for users/developers/system
- Breaking changes highlighted in the Impact section (if applicable)

## What to Avoid

- Conventional commit format (`type(scope):`)
- Bullet-point lists of technical changes
- Exhaustive file/function change lists
- Overly technical implementation details
- Additional co-author tags (Claude Code's tag is acceptable)

## Tone

Write commits as if explaining the change to a teammate who wants to understand the decision and its implications, not just what files were modified.
