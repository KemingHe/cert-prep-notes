---
name: notes-revision
description: |
  Revise certification study notes with fact-checked content from official documentation.
  Use for AWS, Azure, GCP, Terraform, Kubernetes, or any platform certification prep notes.
  Triggers: "revise notes", "review notes", "fact-check notes", "update study notes".
license: MIT
metadata:
  author: KemingHe
  version: "1.2.0"
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

## Process

### Step 1: Acquire Context

Before revising, gather:

1. **Target notes file(s)**: Read the file(s) to be revised
2. **Reference format**: Check existing revised notes in the same directory for style consistency
3. **Platform**: Identify the certification/platform (AWS, Azure, GCP, Terraform, etc.)
4. **Skills to reference**:
   - Senior mentor skill (persona, domain expertise)
   - Documentation review skill (formatting checklist, GDC)

### Step 2: Fact-Check with Parallel Agents (CRITICAL)

**This step is non-negotiable.** All factual claims must be verified against official documentation via live data fetching before revision.

**Requirements**:

- Send 3-4 parallel subagents to fetch and verify facts
- Each agent targets different topic areas for comprehensive coverage
- All verified facts must include the verification date
- Record source URLs for traceability

**Agent prompt template** (copy and customize):

```plaintext
Fetch and verify facts about [TOPIC] from official [PLATFORM] documentation.

Use WebSearch and WebFetch to find the latest [PLATFORM] documentation ([CURRENT_YEAR]) for:
1. [Specific fact to verify]
2. [Specific fact to verify]
3. [Specific fact to verify]
...

Return a structured summary:
- Verified facts with any corrections needed
- Verification date: [TODAY'S DATE]
- Source URLs (official docs only)
```

**Example for AWS EC2 storage**:

```plaintext
Fetch and verify facts about Amazon EBS from official AWS documentation.

Use WebSearch and WebFetch to find the latest AWS documentation (2026) for:
1. EBS volume types and multi-attach capability
2. EBS snapshots - region-bound vs AZ-bound
3. Delete on termination default behavior
4. EBS provisioned capacity billing model

Return a structured summary:
- Verified facts with any corrections needed
- Verification date: 2026-02-25
- Source URLs (official docs only)
```

### Step 3: Reorganize by Logic and Importance

**Do NOT preserve the user's original note order blindly.** Restructure based on:

| Priority | Section Type | Examples |
| :--- | :--- | :--- |
| 1 | Overview/Introduction | What is X, key characteristics |
| 2 | Core concepts | Types, components, architecture |
| 3 | Configuration/Usage | How to set up, attach, configure |
| 4 | Comparison tables | Feature matrices, pricing tiers |
| 5 | Best practices | Recommendations, common patterns |
| 6 | Shared responsibility | AWS vs Customer responsibilities |

**Grouping principles**:

- Group related concepts together (e.g., all EBS topics before EFS)
- Place foundational concepts before advanced ones
- Put frequently referenced info (tables, quick reference) early
- End with actionable guidance (best practices)

### Step 4: Apply Documentation Standards

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

All verified facts from subagent research must include inline source links.

- **Format**: `[descriptive text](url)` where text is meaningful, not generic
- **Good examples**:
  - `[Six Advantages of Cloud Computing](url)` - descriptive concept name
  - `[Application Load Balancers user guide](url)` - specific resource type + guide
  - `[scheduled scaling in Amazon EC2 Auto Scaling](url)` - feature + service context
- **Bad examples**:
  - `click here` or `see docs` - too generic
  - `[ALB documentation](url)` - generic "documentation" label
  - `[see this page](url)` - non-descriptive anchor
- **URL preferences**:
  - Prefer canonical URLs over dated snapshots (i.e. `docs.aws.amazon.com` over `wa.aws.amazon.com/wellarchitected/2020-07-02T19-33-23/...`)
  - Use `/latest/` paths when available for evergreen links
- **When to link**: Every verified claim (definitions, feature lists, version numbers, deprecation dates, best practices)
- **References section**: Include only URLs NOT already inlined; use for broad resources or further reading that lack a natural inline anchor

**Apply documentation review checklist**:

| Dimension | Check For |
| :--- | :--- |
| Consistency | Terminology, voice, header levels, table formatting |
| Correctness | Verified facts, valid YAML/markdown, working links |
| Completeness | Required sections present, no unfilled placeholders |
| Freshness | Last Updated date matches revision date |
| Characters | QWERTY only (exception: `↑` for ToC) |
| Linter | Check IDE linter errors after edits |

### Step 5: Self-Verification (CRITICAL)

**Do NOT trust your own generated content.** After drafting, verify all claims as if they came from an external source.

**5.1 Compile claims inventory**:

Extract all verifiable claims from the generated draft:

- Numeric values (percentages, limits, durations, capacities)
- Dates (release dates, deprecation dates, effective dates)
- Feature status (preview vs GA, deprecated, discontinued)
- Behavioral defaults (enabled/disabled, required/optional)
- Specific qualifiers (which tier, which analyzer type, which region)

Organize claims by category for parallel verification.

**5.2 Run second-pass verification**:

Send 3-4 parallel subagents to verify the GENERATED claims (not user input):

```plaintext
Verify claims from the generated draft against official [PLATFORM] documentation.

Use WebSearch and WebFetch to find the latest [PLATFORM] documentation ([CURRENT_YEAR]) for:
1. [Claim from generated draft with line reference]
2. [Claim from generated draft with line reference]
3. [Claim from generated draft with line reference]
...

For each claim, return:
- VERIFIED (exact match) or INCORRECT (with correction)
- Source URL from official documentation
- Verification date: [TODAY'S DATE]
```

**5.3 Double-check discrepancies**:

For any claim marked INCORRECT:

- Have a second agent independently verify the correction
- Only apply corrections confirmed by both sources
- This prevents replacing one hallucination with another

**5.4 Apply corrections**:

- Fix incorrect claims with verified information
- Add missing inline source links for all corrections
- Update Last Updated date to reflect verification date

### Step 6: Final Verification

After completing the revision:

1. **Linter check**: Run `ReadLints` on all edited files
2. **ToC reminder**: Tell user to auto-generate ToC using editor tools
3. **Summary**: Report corrections made, facts verified, structure changes

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

- **Parallel agents mandatory**: Fact-checking with 3-4 parallel subagents is non-negotiable
- **Self-verification mandatory**: After generating draft, run second-pass verification on own output before finalizing
- **Date stamp all facts**: Every verified fact must include verification date
- **Inline source links required**: All verified facts must include inline links to official documentation
- **Reorganize by logic**: Do not blindly preserve user's original note order
- **ToC is tooling-generated**: Never manually maintain Table of Contents
- **Platform-agnostic**: Works for AWS, Azure, GCP, Terraform, Kubernetes, or any certification
