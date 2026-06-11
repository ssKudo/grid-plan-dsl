# Hermes Skill Bundle Sketch

This document sketches a future Hermes skill bundle for architectural review workflows.

The initial goal is to run as a Hermes skill bundle, not as a Hermes fork.

## Bundle Name

`grid-plan-dsl`

## Purpose

Help a Hermes agent maintain a structured building memory from conversational drawing review.

The bundle should teach the agent to:

- convert natural language drawing descriptions into `building-card.yaml`
- preserve uncertainty instead of guessing
- render ASCII feedback from the current grid model
- track evidence against drawings and schedules
- report unresolved permit-review information
- turn repeated corrections into better project checklists
- switch behavior by architectural role while keeping the same building memory

## Role Layers

The root building memory should be shared. The agent behavior should change by role.

```text
shared core
  building-card.yaml
  evidence
  unresolved issues
  decisions

role layers
  designer
  permit reviewer
  estimator
  site supervisor
```

The same window, room, or wall has different meaning depending on who is using the agent.

| Role | Primary concerns |
| --- | --- |
| Designer | room layout, code assumptions, design intent, missing planning decisions |
| Permit reviewer | legal classification, required drawings, evidence, unresolved code facts |
| Estimator | quantities, specifications, exclusions, unit-price assumptions |
| Site supervisor | constructability, interference, ordering, inspections, field changes |

Role layers should not fork the building memory. They should read the same project repository and add role-specific notes, evidence, issues, and decisions.

## GitHub Project Memory

The intended project workflow uses GitHub as the shared memory between people and agents.

```text
designer creates initial building-card.yaml
-> permit reviewer checks code and evidence
-> estimator clones the same repository for quantity takeoff
-> site supervisor clones the same repository for construction checks
```

Later roles should not be forced to re-enter all details. By the time the estimator or site supervisor starts, most basic geometry and room/opening data should already exist. Their work should focus on confirming, enriching, and correcting the shared memory from their role-specific point of view.

See [github-workflow.md](./github-workflow.md) for the repository-level workflow.

## Skill 1: Building Card Intake

### Trigger

Use when the user describes a building, grid, room layout, openings, equipment, site condition, or drawing sheet.

### Inputs

- natural language instructions
- existing `building-card.yaml`
- drawing sheet names or references

### Outputs

- updated `building-card.yaml`
- explicit assumptions
- unresolved issues

### Rules

- Keep unknown values as `null` or unresolved issues.
- Do not invent dimensions or legal conditions.
- Prefer grid cells and edge references over vague room descriptions.
- Ask for human confirmation when a fact changes legal judgment.

## Skill 2: ASCII Plan Feedback

### Trigger

Use after grid, room, or opening data changes.

### Inputs

- `grid`
- `rooms`
- `openings`

### Outputs

- compact ASCII plan
- legend
- warnings for overlapping rooms or orphan openings

### Rules

- Make the diagram readable in a terminal.
- Use stable room abbreviations.
- Show openings on edges when possible.
- Report modeling conflicts separately from the drawing.

## Skill 3: Evidence Check

### Trigger

Use when PDF drawings, OCR text, schedules, or source notes are available.

### Inputs

- `building-card.yaml`
- OCR text or drawing references
- evidence targets

### Outputs

- evidence status updates
- source references
- conflicts and missing evidence

### Rules

- Treat OCR as verification support, not the source of truth.
- Distinguish `matched`, `conflict`, `missing`, and `unchecked`.
- Never overwrite user-confirmed building memory only because OCR disagrees.

## Skill 4: Permit Readiness Review

### Trigger

Use when the user asks whether the project is ready for code review or building confirmation.

### Inputs

- `building-card.yaml`
- jurisdiction or municipality notes
- target checks

### Outputs

- missing information list
- likely blocking issues
- next data collection steps

### Rules

- Separate missing facts from legal interpretation.
- Keep code judgments traceable to inputs.
- Prefer "insufficient information" over weak OK/NG claims.

## Skill 5: Workflow Learning

### Trigger

Use after repeated corrections, completed checks, or project closeout.

### Inputs

- correction history
- unresolved issue patterns
- approved decisions
- final project artifacts

### Outputs

- improved checklist
- proposed skill note
- reusable project template

### Rules

- Only learn from explicit artifacts or user-approved decisions.
- Keep project-specific facts separate from reusable process knowledge.
- Do not encode private client facts into general skills.

## Skill 6: Role-Aware Review

### Trigger

Use when the user identifies their role, such as designer, permit reviewer, estimator, or site supervisor.

### Inputs

- active role
- `building-card.yaml`
- existing evidence, decisions, and unresolved issues
- current project phase

### Outputs

- role-specific summary
- role-specific unresolved issues
- suggested next actions
- files or sections that should be updated

### Rules

- Keep the shared building memory as the source of truth.
- Do not duplicate facts into role-specific files unless the role adds a different interpretation or decision.
- Ask fewer low-level intake questions in later phases if the shared memory already contains the needed data.
- For site supervisor workflows, prefer generated checklists and short field updates over detailed manual modeling.

## Repository Layout Proposal

```text
hermes-skills/
  grid-plan-dsl/
    SKILL.md
    examples/
      building-card.yaml
      correction-session.md
    templates/
      building-card.yaml
      unresolved-issues.md
```

## Future Menu Layer

The future terminal main menu should call these skills and tools, not replace them.

The menu can be a thin launcher around stable operations:

- start intake
- update building card
- render ASCII plan
- run evidence check
- review unresolved issues
- export package
