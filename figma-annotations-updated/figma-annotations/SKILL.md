---
name: figma-annotations
description: Compare current Figma frames or instances with saved historical states using Southleft Figma Console MCP, then add numbered developer handoff annotations with Old and New content. Use for documenting design or offer updates, including inherited library changes, without requiring detached before frames.
---

# Figma Annotations

Document verified differences on the designer's handoff instances while preserving their component connections. The handoff page collects changes for developers and shows page context for QA; it is often not where the edits originated.

## Inputs and prerequisites

Use the destination Figma links and a baseline date, saved version, or previous approved handoff supplied by the user. Source component and library links are optional. Request only missing information needed to establish a reliable comparison.

Requires Southleft **Figma Console MCP**, an active Desktop Bridge for the destination file, and REST access to file content and version history. Discover the connected tool schemas; server prefixes may vary. The native Figma MCP alone does not supply the historical reads this workflow needs through the tools verified for this skill. If historical tools are unavailable, explain the missing capability rather than comparing current instances and calling them historical.

## Establish the baseline

- Verify the file and destination nodes. Trace main components, selected variants, nested instances, and external library references as needed. Names alone do not establish component identity.
- Treat user-linked frames as annotation destinations. A change can originate in a nested library component, propagate into a consuming file's component, and appear on a new handoff instance.
- Use `figma_get_file_versions` to identify saved versions. Include autosaves when needed; default listings may omit them. Version IDs belong to a specific file.
- Confirm the intended before-state before writing. If no baseline is specified, offer relevant dates and ask. Do not assume that a latest publication, a “Ready for dev” label, and the previous approved state mean the same thing.
- If the user says “last published,” identify the relevant publication and its scope. A file-wide “Components published” checkpoint does not prove that this particular component changed. If that publication already contains today's content, report the match and clarify whether an earlier pre-update state is intended. Never silently substitute an older date to produce annotations.
- Record the chosen file key, version ID, timestamp, and source node mapping outside the Figma design. Display timestamps in the user's timezone.

## Retrieve and compare

1. Read the actual current destination instance, including its overrides and variant. Use `figma_get_file_at_version` to retrieve the consuming file's state before the relevant library updates were accepted. An older instance still on today's canvas is not a frozen baseline.
2. If the destination did not yet exist, retrieve its corresponding historical source component or containing frame. Verify correspondence through component identity, structure, and instance paths. Resolve historical component references using that version's data rather than today's IDs.
3. Consult library history when needed to explain origin, but determine applicability from what the consuming file and current instance actually contain. Unaccepted library changes and overridden changes should not become page annotations.
4. Compare content, offers, prices, benefits, legal copy, layer additions/removals, visibility, order, layout, spacing, dimensions, styling, variants, and documented interactions. Follow nested dependencies when necessary.
5. Use the built-in `figma_diff_versions` as supporting evidence only: its coverage can exclude raw visual/layout properties, instance changes, variable values, and style content. Inspect the historical layer data directly for the relevant properties. Read the returned coverage warnings.
6. Normalize REST and Plugin API representations before comparing: absent defaults, RGB versus RGBA, gradient representations, image references, and small numeric differences can create false changes. Resolve variable values and bindings at the appropriate state before claiming a token changed.
7. Establish annotation eligibility from the current handoff view before comparing changes. A node with a visible LAYER_BLUR effect, or any descendant of a blurred ancestor, is outside the annotation scope. Skip its entire subtree: do not annotate its content, offers, prices, legal text, layout, or styling even when historical data differs. Never attach those excluded changes to an unblurred ancestor to bypass this rule. This workflow uses blur to mark content that is not part of the handoff update; include blurred regions only when the user explicitly requests them. Also skip hidden subtrees. Separate direct edits from automatic reflow and ignore canvas repositioning and annotation edits.
8. Use current and version-specific screenshots when available to verify visual changes. Current bridge screenshots do not prove historical appearance. Do not present a reconstruction combining different dates as an authentic historical screen.

Only claim the categories and time interval actually verified. Observed differences across an interval do not by themselves establish the exact publication or acceptance event. Describe what changed without inventing why. If nothing meaningful differs, report that and add no change annotations.

## Desktop-first annotation placement

- Write annotations for changes shared across breakpoints on Desktop only. Do not duplicate those annotations on Tablet or Mobile.
- Review corresponding Tablet and Mobile views within the requested scope. Add annotations there only when a verified change has a breakpoint-specific variation in content, layout, styling, or behavior that the Desktop annotation does not cover. Describe the variation on the affected breakpoint's relevant element.
- Keep annotation writes within the supplied destination scope. If Desktop is unavailable or outside that scope, clarify the intended destination rather than placing shared annotations on Tablet or Mobile by default.

## Annotation format

Write native Figma annotations using `labelMarkdown`, with this exact structure. Designer attribution and the date added belong in the annotation body; this requirement applies to annotations only, not Figma comments:

```markdown
**[number] · [short title summarizing the change]**

**Old:** [previous content or property value]

**New:** [updated content or property value]

[Optional note only when needed.]

**Designer:** [designer name] · **Added:** [YYYY-MM-DD]
```

- Include the designer's name and the date the annotation was first added in every generated annotation. Use a designer identity explicitly supplied by the user or reliably established in the task context; ask if unknown. Do not infer the designer from a file owner, version author, or local username. For new annotations, use the actual creation date in the user's timezone, formatted `YYYY-MM-DD`; this is separate from the historical comparison baseline.
- When updating an existing annotation, preserve its original designer attribution and Added date. If either is missing, establish it from reliable records or ask the user; never invent attribution or replace an unknown creation date with today's date.
- Use short, specific titles and sequential numbering in reading order across the requested scope. When updating existing notes, preserve their numbering unless the user requests renumbering; continue the existing sequence for new notes.
- Quote changed copy exactly, preserving punctuation, prices, and legal language. Include enough context to avoid implying that an unchanged condition was removed.
- For layout/styling changes, state precise before/after values when verified; otherwise use a concrete description and flag the limitation.
- For additions, Old is “Not present.” For removals, New is “Removed.” Attach removal notes to the nearest relevant surviving element.
- Group closely related changes. Add a note only for useful placement, behavior, or uncertainty context. Do not repeat the baseline date in every annotation; report baseline metadata in chat unless needed to disambiguate a note.

Example:

```markdown
**1 · Equipment fee wording**

**Old:** No equipment fees

**New:** No equipment or set-up fees

**Designer:** Alex Morgan · **Added:** 2026-11-12
```

## Apply and verify

- A request to annotate authorizes writing within the supplied destination scope. A request only to inspect or draft does not authorize insertion. Resolve material baseline uncertainty before writing.
- Inspect existing annotations and update matching generated annotations instead of duplicating them. Preserve unrelated notes and their categories/pinned properties. Every annotation added or updated by this skill must use the **Development** category. Resolve the destination file’s categories explicitly by `id`, `label`, and `color`; reuse a case-insensitive label match for “development.” Do not hardcode a category ID across files. If absent, create a Development category through the supported API before writing; if category assignment is unavailable, report the blocker instead of creating uncategorized notes. Use `figma_get_annotations`, `figma_set_annotations`, or an equivalent targeted Plugin API operation. When rewriting an annotation, supply only one of `label` or `labelMarkdown`; read-back objects may contain both, so do not spread them unchanged into a write.
- Attach each note directly to the relevant destination element, not the source master. Keep all mutations inside the requested nodes and their descendants. Do not detach instances, edit product copy, change source components, accept library updates, or create extra pages as part of this workflow.
- Read back annotations to verify Markdown, numbering, text, designer attribution, date added, target containment, and Development category assignment. Recheck effective visibility and blur along each target’s ancestor chain; no generated annotation may remain in an excluded subtree. When correcting scope mistakes, remove only the matching generated notes and renumber the remaining generated sequence where necessary. Confirm that source masters were not edited. When native annotation markers do not appear in image exports, verify them through annotation data rather than treating the export as evidence of visible markers.
- Verify that shared change annotations appear only on Desktop and that every generated Tablet or Mobile annotation documents a verified breakpoint-specific variation. Check for duplicate shared notes across breakpoints; when correcting duplicates, remove only matching generated notes and preserve unrelated annotations.
- For visible canvas edits explicitly requested by the user, follow the connected tool's visual validation workflow. After a partial failure, inspect what succeeded and retry only the missing work; stop and explain a repeated unresolved failure.
- Finish with a link to the annotated destination, the count added/updated, the baseline date/version used, and material limitations. Tell the user native annotations can be inspected in Dev Mode. Do not claim full visual or interaction verification if only content was checked.
