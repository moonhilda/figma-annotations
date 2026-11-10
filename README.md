# Figma Annotations

A Figma skill that compares current designs with verified historical versions and adds numbered developer handoff annotations showing what changed.

The skill uses Southleft **Figma Console MCP** to retrieve historical file data, inspect current frames and instances, and write native Figma annotations. It preserves component connections and can trace changes inherited from nested components and external libraries.

## What it does

- Compares current handoff frames and instances with a specified historical baseline.
- Checks copy, offers, prices, benefits, legal text, layer additions and removals, visibility, order, layout, spacing, dimensions, styling, variants, and documented interactions.
- Follows nested component and library references when needed to establish what changed.
- Accounts for the destination instance’s current overrides and selected variants.
- Skips hidden content and blurred regions by default.
- Adds numbered annotations with **Old** and **New** values.
- Assigns generated annotations to the **Development** category.
- Updates matching generated annotations instead of creating duplicates.
- Verifies annotation content, numbering, placement, category, and scope.
- Reports the baseline used and any limits on the comparison.

Annotations document verified differences on the supplied handoff instances. The underlying edits may have originated elsewhere in the component or library structure.

## Requirements

- Codex with skills enabled.
- Southleft **Figma Console MCP** connected.
- An active Desktop Bridge for the destination Figma file.
- REST access to file content and version history.
- Edit access to the destination file.
- Access to relevant source components and libraries when historical tracing requires them.

The native Figma MCP alone does not provide the historical reads required by this workflow through the tools verified for this skill.

Each teammate needs the skill installed and their own authorized Figma connection. Installing the skill does not grant access to Figma files or libraries.

## Installation

Install the `figma-annotations` folder into your Codex skills directory:

```text
~/.codex/skills/figma-annotations/
```

Once installed, invoke it as `$figma-annotations` or ask Codex to use Figma Annotations with your destination links and baseline.

This repository contains one standalone skill. No coordinator or additional stage skills are required.

## Usage

Start a chat with the destination Figma links and a baseline date, saved version, or previous approved handoff:

```text
Use $figma-annotations to compare these handoff frames with the
approved version from October 20, 2026. Add developer annotations
for verified changes: [Figma links]
```

Include source component or library links when they would help trace inherited changes.

The skill will:

1. Verify the destination frames, instances, and component relationships.
2. Identify and confirm the historical baseline.
3. Retrieve the relevant historical file and component data.
4. Compare eligible content with the current destination instances.
5. Add or update native Development annotations.
6. Read back the annotations and verify their content and placement.
7. Report the destination link, annotation counts, baseline, and limitations.

Inspect the resulting native annotations in Figma Dev Mode.

### Inspect before writing

To review findings before inserting annotations, request a comparison or draft:

```text
Use $figma-annotations to compare these frames with version
[version ID]. Draft the proposed annotations in chat for review:
[Figma links]
```

A request to inspect or draft does not authorize annotation insertion.

## Choosing a baseline

The baseline defines the before-state for the comparison.

You can supply:

- A saved version.
- A date and enough context to identify the intended state.
- A previous approved handoff.

If the baseline is missing or ambiguous, the skill asks for clarification before writing.

“Last published,” “Ready for dev,” and “previously approved” can refer to different states. A file-wide publication checkpoint does not establish that a particular component changed.

If the requested baseline already matches the current content, the skill reports that match. It does not silently choose an older version to produce annotations.

An older instance still present on today’s canvas is not treated as a frozen historical baseline.

## Annotation format

Annotations use numbered titles followed by explicit Old and New values:

```markdown
**1 · Equipment fee wording**

**Old:** No equipment fees

**New:** No equipment or set-up fees
```

- Changed copy preserves exact punctuation, prices, and legal language.
- Additions use **Old: Not present.**
- Removals use **New: Removed.**
- Layout and styling changes include precise values when verified.
- Closely related changes may share an annotation.
- New annotations follow reading order across the requested scope.
- Existing annotation numbers are preserved unless renumbering is requested.

Removal notes attach to the nearest relevant surviving element. Other notes attach directly to the changed destination element.

## Scope and preservation

Component instances remain connected to their main components.

The skill writes annotations within the supplied destination nodes and their descendants. It preserves unrelated annotations and their properties.

It does not detach instances, edit product copy, modify source components, accept library updates, or create extra pages.

Hidden subtrees are excluded. A visible layer blur also excludes that node and its descendants unless you explicitly request blurred regions. Excluded changes are not moved onto an unblurred ancestor.

Canvas repositioning and annotation edits are ignored. Direct edits are distinguished from automatic layout reflow.

If no meaningful differences are verified, the skill reports that result and adds no change annotations.

## Verification and limitations

Historical layer data is the basis for the comparison. Built-in version diffs provide supporting evidence, but may omit instance changes, layout properties, variable values, or style content.

Current and historical screenshots are used when available. A current Desktop Bridge screenshot does not establish historical appearance.

The skill reports only the change categories and time interval it could verify. Differences between versions do not necessarily establish the exact publication or library-update acceptance event.

Missing historical access, unresolved component mappings, or unavailable annotation category assignment can prevent completion. These limitations are reported explicitly.

## Repository structure

```text
README.md
figma-annotations/
├── SKILL.md
└── agents/
    └── openai.yaml
```

- `SKILL.md` contains the comparison workflow, baseline rules, annotation format, scope, and verification requirements.
- `agents/openai.yaml` contains the Codex display name and default invocation prompt.

## Maintenance

- Treat this repository as the shared source of truth.
- Keep skill instructions and UI metadata consistent.
- Verify changes against representative Figma files and historical versions.
- Reinstall or update teammates’ local copies after repository changes.
- Start a fresh chat after updating so the revised instructions are loaded.

GitHub edits do not automatically update installed copies.
