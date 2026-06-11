# Grid Plan DSL

Use this skill when a user wants to structure architectural drawing information, maintain a `building-card.yaml`, render a terminal-readable plan, or prepare evidence-backed inputs for building code review.

## Goal

Maintain a structured building memory from conversational drawing review.

The building memory should be useful to:

- the user, through readable summaries and ASCII feedback
- the agent, through stable grid/cell/edge references
- downstream tools, through deterministic YAML
- code review workflows, through explicit evidence and unresolved issues

## Core Principle

Do not treat OCR or vision as the primary source of truth.

The preferred loop is:

```text
human reads drawing
-> human explains to agent
-> agent updates building-card.yaml
-> agent renders ASCII feedback
-> human corrects
-> agent tracks evidence against drawings
```

## Building Memory

Prefer a `building-card.yaml` with these top-level sections:

- `project`
- `site`
- `grid`
- `outline`
- `rooms`
- `openings`
- `equipment`
- `evidence`
- `unresolved_issues`
- `decisions`

Use `null` for unknown scalar values. Use `unresolved_issues` when missing information affects review readiness.

## Grid Rules

- X axis labels are numbers: `1, 2, 3, ...`
- Y axis labels are kana: `い, ろ, は, ...`
- Cell IDs use the lower-left grid label, such as `い-1`.
- `い-1` means the cell between `い-ろ` and `1-2`.
- Horizontal edges use `い通り 1-2間`.
- Vertical edges use `2通り い-ろ間`.

## Update Rules

When the user provides new drawing information:

1. Identify whether it affects grid, outline, rooms, openings, equipment, site, evidence, unresolved issues, or decisions.
2. Update the building memory consistently.
3. Preserve existing confirmed facts unless the user explicitly corrects them.
4. Record assumptions separately from facts.
5. Report any conflicts instead of silently resolving them.

## ASCII Feedback

After grid, room, or opening updates, provide a compact ASCII plan when useful.

The diagram should:

- include X and Y labels
- include a legend
- show room abbreviations inside cells
- show openings on edges when practical
- be readable in a plain terminal

If the model contains conflicts, list them separately after the diagram.

## Evidence Tracking

Use these statuses:

- `unchecked`: entered but not verified
- `matched`: verified against a source
- `conflict`: source exists but disagrees
- `missing`: required source was not found

Evidence records should include:

- `target`: YAML path or logical target
- `source`: drawing sheet, OCR snippet, schedule, or note
- `status`
- `note`

## Review Readiness

When asked if the project is ready for review, separate the answer into:

- ready facts
- missing information
- conflicts
- legal checks that can run now
- legal checks blocked by missing facts

Prefer "insufficient information" over weak OK/NG claims.

## Learning Loop

When a correction pattern repeats, propose a reusable checklist or skill note.

Do not store private project facts as general knowledge. Generalize only workflow patterns, naming conventions, or validated review steps.

Examples of reusable learning:

- typical intake questions for a wood detached house
- common missing facts for lighting/ventilation checks
- office-specific room abbreviations, if approved by the user
- municipality-specific document checklist, if sourced and approved

## Output Style

Be direct and practical. Keep the current building memory, diagram, and unresolved issues easy to scan.
