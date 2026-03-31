---
name: notes-revision
description: |
  Revise certification study notes with fact-checked content from official documentation.
  Use for AWS, Azure, GCP, Terraform, Kubernetes, or any platform certification prep notes.
  Triggers: "revise notes", "review notes", "fact-check notes", "update study notes".
license: MIT
metadata:
  author: KemingHe
  version: "2.0.0"
---

# Notes Revision

Revise certification study notes by fact-checking against official documentation, reorganizing by logical importance, and applying documentation standards.

**Temporary persona**: Expert mentor in the target platform (AWS, Azure, GCP, etc.) with 15+ years experience. Technical editor focused on accuracy and clarity.

## When to Use This Skill

- Revising rough certification study notes into polished reference material
- Fact-checking existing notes against current official documentation
- Restructuring notes for better logical flow and exam preparation
- Applying consistent formatting standards across note files

## Asset Resolution

This skill references the following assets:

- `./assets/notes-template.md` - template for revised notes structure and required sections

## Orchestrator Role

The orchestrator (main agent context) coordinates subagents without introducing bias.

**Responsibilities**:

- Compile subagent results into structured summaries
- Identify discrepancies between subagent reports
- Defer judgment until multiple sources confirm
- Track claims with conflicting reports across phases
- Maintain edit context history with user
- Generate draft only after Initial Verification Phase completes

**Prohibited**:

- Auto-trusting any single subagent output
- Applying corrections without multi-source confirmation
- Adding interpretive bias when compiling results

**Conflict Resolution**: When official sources conflict, prioritize by: (1) most recent date, (2) more specific documentation (API reference over marketing), (3) primary service documentation over cross-references. Document justification for user transparency.

## Process Overview

The revision process uses named phases instead of numbered rounds for clarity:

| Phase | Purpose | Agents |
| :--- | :--- | :--- |
| Preparation | Token warning, claims inventory | Orchestrator only |
| Initial Verification | Fact-check claims against official docs | 3-4 parallel subagents by domain |
| Draft Generation | Generate revised document with verified facts | Orchestrator only |
| Self-Verification | Verify generated content with fresh context | 3-4 fresh-context subagents by claim type |
| Resolution | Resolve discrepancies via tie-breaking | Independent subagents as needed |
| Validation | Link checks, linter, final review | Orchestrator + subagents |

## Preparation Phase

### Token Cost Warning

Before proceeding, warn user:

```plaintext
This revision process uses multiple parallel subagents across several phases to ensure accuracy. It will incur significant token cost in exchange for high-quality, thoroughly verified output. Proceed?
```

Wait for explicit user confirmation before continuing.

### Acquire Context

Gather before revising:

1. **Target notes file(s)**: Read the file(s) to be revised
2. **Reference format**: Check existing revised notes in the same directory for style consistency
3. **Platform**: Identify the certification/platform (AWS, Azure, GCP, Terraform, etc.)
4. **Skills to reference**: Senior mentor skill (persona), documentation review skill (formatting checklist)

### Claims Inventory

Before launching subagents, compile all verifiable claims from source notes:

| Category | Examples |
| :--- | :--- |
| Numeric | Percentages, limits, durations, capacities |
| Feature status | Serverless, managed, HA, deprecated |
| Integrations | Service X works with Service Y |
| Performance | Latency, throughput, cost comparisons |

Organize into groups for parallel agent distribution. Each group should target 10-20 claims to balance parallelism with manageability.

## Initial Verification Phase

**This phase is non-negotiable.** All factual claims must be verified against official documentation via live data fetching.

### Launch Parallel Subagents

Send 3-4 parallel subagents, each targeting different topic areas (by domain/service):

**Agent prompt template**:

```plaintext
Fetch and verify facts about [TOPIC] from official [PLATFORM] documentation.

Use WebSearch and WebFetch to find the latest [PLATFORM] documentation ([CURRENT_YEAR]) for:
1. [Specific claim with source line reference]
2. [Specific claim with source line reference]
...

Return a structured report:
| Claim # | Status | Original | Correction (if any) | Source URL |
| :--- | :--- | :--- | :--- | :--- |
| 1 | VERIFIED/INCORRECT/UNVERIFIED | ... | ... | https://... |

Verification date: [TODAY'S DATE]
```

### Orchestrator Compilation

After subagents complete:

1. Compile all results into unified summary
2. Identify claims with INCORRECT or UNVERIFIED status
3. Note any conflicting reports between subagents
4. Do NOT apply corrections yet - only compile

## Draft Generation Phase

### Reorganize by Logic and Importance

**Do NOT preserve the user's original note order blindly.** Restructure based on:

| Priority | Section Type | Examples |
| :--- | :--- | :--- |
| 1 | Overview/Introduction | What is X, key characteristics |
| 2 | Core concepts | Types, components, architecture |
| 3 | Configuration/Usage | How to set up, attach, configure |
| 4 | Comparison tables | Feature matrices, pricing tiers |
| 5 | Best practices | Recommendations, common patterns |
| 6 | Shared responsibility | AWS vs Customer responsibilities |

### Apply Documentation Standards

**Required sections for each revised file**:

1. Title with `# Study Notes - [Topic]` format
2. `> **Last Updated**: [DATE] by [AUTHOR]` metadata
3. `## Table of Contents` - remind user to auto-generate
4. Content sections with `##` headers
5. `> [↑ Back to Table of Contents](#table-of-contents)` after each level 2 section
6. `---` separator between back link and next section (except at EOF)
7. `## Shared Responsibility Model` section (for cloud services)
8. `## Best Practices` section (second-to-last section)
9. `## References` section (final section)

**Reference linking policy**:

- **Format**: `[descriptive text](url)` where text is meaningful, not generic
- **Good**: `[Six Advantages of Cloud Computing](url)`, `[scheduled scaling in Amazon EC2 Auto Scaling](url)`
- **Bad**: `click here`, `see docs`, `[documentation](url)`
- **URL preferences**: Prefer canonical URLs over dated snapshots; use `/latest/` paths
- **When to link**: Every verified claim (definitions, feature lists, version numbers, deprecation dates)
- **References section**: Include only URLs NOT already inlined

## Self-Verification Phase

**Do NOT trust your own generated content.** After drafting, verify all claims as if they came from an external source.

### Fresh-Context Requirement

Self-verification agents MUST have zero context from Initial Verification Phase. Launch new Task subagents - do not resume or continue previous agents. This prevents confirmation bias.

### Categorization Strategy

| Phase | Categorize By | Rationale |
| :--- | :--- | :--- |
| Initial Verification | Domain/service | Expertise grouping |
| Self-Verification | Claim type | Different error patterns |

**Claim type categories for Self-Verification**:

- Numeric claims (percentages, limits, durations)
- Feature status claims (serverless, managed, HA)
- Integration claims (what works with what)
- Performance claims (latency, throughput, cost)

### Launch Fresh-Context Subagents

Send 3-4 parallel subagents to verify the GENERATED claims (not user input):

```plaintext
Verify claims from the generated draft against official [PLATFORM] documentation.

Use WebSearch and WebFetch to find the latest [PLATFORM] documentation ([CURRENT_YEAR]) for:
1. [Claim from generated draft with line reference]
2. [Claim from generated draft with line reference]
...

For each claim, return:
- VERIFIED (exact match) or INCORRECT (with correction) or UNVERIFIED (could not confirm)
- Source URL from official documentation
- Verification date: [TODAY'S DATE]
```

### Orchestrator Identifies Discrepancies

Compare Self-Verification results against Initial Verification and draft:

- Claims marked INCORRECT in Self-Verification proceed to Resolution Phase
- Claims with conflicting status between phases proceed to Resolution Phase
- Claims consistently VERIFIED across phases are confirmed

## Resolution Phase

For any claim marked INCORRECT or with conflicting reports:

1. Launch independent subagent with fresh context to verify the correction
2. Only apply corrections confirmed by both phases
3. If still conflicting, apply conflict resolution rules (see Orchestrator Role)
4. This prevents replacing one hallucination with another

## Validation Phase

### Link Accessibility

Verify all inline URLs return HTTP 200:

- Use subagents to batch-check links
- Report broken links for correction

### Link Semantic Validation

Verify link target matches claim context:

- RDS claim should link to RDS documentation, not Aurora
- Service-specific claims should link to that service's docs
- Flag mismatched links for correction

### Linter Check

1. Run `ReadLints` on all edited files
2. Fix introduced errors before finalizing

### Final Report

Present to user:

- Total claims verified
- Corrections applied with justification
- Structure changes made
- Any UNVERIFIED claims requiring manual review
- Reminder to auto-generate ToC using editor tools

## Subagent Failure Handling

| Failure Type | Action |
| :--- | :--- |
| Timeout or garbage output | Restart subagent with fresh context; orchestrator may modify prompt |
| Cannot find documentation | Mark claim as UNVERIFIED; report to user for manual review |
| Conflicting reports (same phase) | Escalate to Resolution Phase for tie-breaking |
| Unrecoverable error | Halt and report to user with full context |

## General Doc Constraints

Apply to all generated output. If a discovered template deviates from any rule (e.g., uses emojis semantically, uses a different bullet convention), note the deviation explicitly and confirm with the user before treating it as a permitted exception.

- **Characters**: QWERTY keyboard typeable only - no smart quotes, emojis, or special Unicode anywhere. In prose, do not use em-dashes or em-dash substitutes (`--`, ` -- `); use ` - ` (space-dash-space) for clause separation instead. Exception: `↑` for ToC navigation.
- **Inline formatting**: Use `_underscore_` for italics, not `*single-star*`. Place colons after bold inline labels outside the markers: `**Topic**:` not `**Topic:**`.
- **Bullets**: Use `-` for all unordered lists; one bullet per complete thought; never wrap a bullet's content mid-sentence onto a continuation line - split into separate bullets if too long or multi-thought. Nested sub-bullets for component grouping are permitted. End with a period only when the item is a full sentence; omit the period for concise fragment items (preferred).
- **Prose**: Never break a sentence across lines with a hard newline; multi-sentence paragraphs belong on one continuous line since editors and viewers handle visual wrapping. Exception: commit message bodies use one sentence per line for `git log` readability.
- **Template hygiene**: Delete `(optional)` and any parenthetical conditional label (e.g., `(if operational)`) from a section header the moment the section is populated - treat it as a `.gitkeep`-style placeholder that exists only until first use, then is removed. Omit the entire section (header and body) when unused. Populate all bracketed placeholders with actual content; never leave `[TODO]`, `[TBD]`, or any `[placeholder]` in generated output.
- **Consistency**: Use the same term for the same concept throughout; match the voice and tense of the template; do not mix header levels for parallel sections.
- **KISS and DRY**: Each section and bullet conveys unique information - no redundancy or overlap.

> General Doc Constraints v1.1.0 - KemingHe/common-devx

## Skill Constraints

- **Token warning mandatory**: Always warn user about cost before proceeding
- **Parallel agents mandatory**: Fact-checking with 3-4 parallel subagents is non-negotiable
- **Fresh context mandatory**: Self-verification must use new subagents with no prior context
- **Link validation mandatory**: Verify both accessibility and semantic correctness
- **Orchestrator neutrality**: Compile without bias; defer judgment until multi-source confirmation
- **Self-verification mandatory**: After generating draft, run second-pass verification on own output
- **Date stamp all facts**: Every verified fact must include verification date
- **Inline source links required**: All verified facts must include inline links to official documentation
- **Reorganize by logic**: Do not blindly preserve user's original note order
- **ToC is tooling-generated**: Never manually maintain Table of Contents
- **Failure escalation**: Unrecoverable errors halt with user report
- **Platform-agnostic**: Works for AWS, Azure, GCP, Terraform, Kubernetes, or any certification
