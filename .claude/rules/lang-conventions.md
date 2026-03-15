---
name: lang-conventions
description: Language conventions for code, comments, commits, and communication in this project.
---

## Code & technical artifacts → English

Everything written in code must be in **English**:

- Variable names, function names, class names, constants
- Database column names, table names, migration names
- Route names and URIs (`/api/users`, not `/api/utilisateurs`)
- Git commit messages (subject line and body)
- Code comments and docblocks
- API response keys (`{ "error_message": "..." }`, not `{ "message_erreur": "..." }`)
- Test descriptions (`it('returns 404 when user not found')`)

## Communication → French

Exchanges with the user (explanations, questions, summaries) are in **French**.

## Blade / Vue templates → English for UI text

User-facing strings displayed in the interface are in **English** for now.
