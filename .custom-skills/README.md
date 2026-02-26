# README - .custom-skills

> **Last Updated**: 2026-02-25 by Keming He

Staging area for custom Agent Skills pending integration into [common-devx](https://github.com/KemingHe/common-devx). Skills here follow the [agentskills.io](https://agentskills.io) specification but are stored locally until the skill addition workflow is supported.

## Context

- `.agents/skills/` is synced from common-devx and is currently treated as read-only
- This directory holds project-specific skills in development
- Once common-devx supports skill contribution, skills here will migrate upstream

## Directory Structure

```plaintext
.custom-skills/
├── notes-revision/   # Revise certification study notes with fact-checking
└── README.md         # This file
```

## Quick Start

Reference a custom skill in your AI prompt:

```plaintext
ref @.custom-skills/notes-revision in your prompt
```

## Quick Links

- [notes-revision](./notes-revision/README.md) - Certification study notes revision skill

## References

- [agentskills.io Specification](https://agentskills.io) - Standard for Agent Skills
- [common-devx Skills](./../.agents/skills/README.md) - Production skills synced from common-devx
