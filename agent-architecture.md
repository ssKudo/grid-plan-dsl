# Architecture Agent: Hermes Integration Concept

This note describes how Conversational Grid Plan DSL can become a domain layer for a Hermes-style self-improving architectural agent.

## Positioning

Hermes Agent provides a general-purpose loop:

```text
conversation and work history
-> persistent memory
-> skill creation and improvement
-> tool use
-> scheduled automation
-> cross-session recall
```

This project adds an architectural layer:

```text
drawing reading and permit preparation
-> building memory
-> architectural review skills
-> evidence checks against drawings
-> unresolved issue tracking
-> reusable project workflows
```

The first implementation target is not a fork of Hermes. It is a skill bundle and memory schema that can run inside Hermes while keeping the architectural ideas portable.

## Core Claim

Grid Plan DSL is a structured building memory for self-improving architectural agents.

The agent should not only answer questions about a drawing. It should gradually maintain a project model that can be checked, corrected, compared with evidence, and reused across sessions.

## Agent Responsibilities

### 1. Build the Building Memory

The agent converts user explanations into `building-card.yaml`.

Examples:

- grid axes and module
- outline path
- room cells
- openings and doors
- equipment locations
- site and project attributes
- pending unknowns

### 2. Render Feedback

The agent produces an ASCII plan from the current building memory so the user can quickly catch mistakes.

This keeps the loop cheap:

```text
user explanation -> YAML -> ASCII -> user correction -> YAML
```

### 3. Track Evidence

Every important value should eventually have evidence.

Examples:

- `rooms.ldk.area_m2` comes from a room area table on A-201
- `openings.w1.template` comes from the opening schedule
- `site.road_width_m` comes from a site survey drawing

The agent should distinguish between:

- `unchecked`: entered but not verified
- `matched`: verified against a source
- `conflict`: source exists but disagrees
- `missing`: required but not found

### 4. Surface Missing Information

Instead of trying to finish everything silently, the agent should keep an explicit list of unresolved issues.

Examples:

- 用途地域が未入力
- LDK の採光補正係数に必要な隣地境界距離がない
- 建具表の W-1 と平面図の窓位置が一致していない

### 5. Improve Skills From Repetition

Repeated project patterns should become skills.

Examples:

- 木造住宅の1階平面図から通り芯グリッドを起こす
- 採光チェックに必要な居室・開口・方位情報を集める
- 建具表と平面図の開口記号を照合する
- 自治体ごとの添付資料チェックリストを作る

The skill should improve only through explicit artifacts: corrected YAML, checklists, evidence logs, and user-approved decisions.

## Hermes Mapping

| Hermes concept | Architectural equivalent |
| --- | --- |
| Memory | `building-card.yaml`, project facts, user preferences |
| Skills | drawing reading, code checks, evidence checks |
| Sessions | project-specific review history |
| Tools | PDF OCR, image vision, file search, YAML validation |
| Cron | periodic missing-info and consistency checks |
| Plugins | jurisdiction rules, office templates, document sources |
| Subagents | parallel review of drawings, code clauses, schedules |

## Non-Goals For The First Version

- Replacing CAD or BIM
- Fully automatic drawing recognition
- Producing final permit documents without human review
- Forking Hermes before the skill interface is proven
- Building a terminal main menu before the agent workflow is clear

## Future Interface Direction

A project-specific terminal menu can be added later.

Possible menu items:

```text
1. Start project intake
2. Edit grid and rooms
3. Render ASCII plan
4. Check missing permit information
5. Run evidence check
6. Export building-card.yaml
7. Review unresolved issues
```

For now, the repository should define the memory model and Hermes skill behavior first. A menu is useful only after the commands behind it are stable.
