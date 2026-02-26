# README - notes-revision

> **Last Updated**: 2026-02-25 by Keming He

Revise certification study notes with fact-checked content from official documentation - ensuring accuracy for AWS, Azure, GCP, Terraform, Kubernetes, or any platform certification prep.

## What This Skill Does

- Fact-checks notes against official documentation using parallel verification agents
- Reorganizes content by logical importance - not original note order
- Applies consistent documentation standards and formatting
- Adds verification dates and source URLs for traceability

## Why It Exists

Certification study notes often contain outdated or incorrect information. This skill ensures accuracy by verifying every factual claim against current official documentation before revision. The result is reliable reference material for exam preparation.

## Quick Start

Tell your AI agent:

```plaintext
Revise notes for [topic] - ref @.custom-skills/notes-revision
```

**Example**:

```plaintext
Revise notes for AWS EC2 storage - ref @.custom-skills/notes-revision
```

## Files

| File | Purpose |
| :--- | :--- |
| `SKILL.md` | Skill definition following agentskills.io specification |
| `assets/notes-template.md` | Template for revised study notes output |

## Quick Links

- [Parent Directory](../README.md) - .custom-skills staging area

## Related Skills

- [documentation-review](../../.agents/skills/documentation-review/README.md) - Review docs for consistency and correctness
- [senior-mentor](../../.agents/skills/senior-mentor/README.md) - Guided learning through Socratic questioning

## References

- [agentskills.io Specification](https://agentskills.io) - Standard for Agent Skills
