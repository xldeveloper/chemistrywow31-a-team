---
name: Output Structure
description: Define the directory layout and naming rules for generated Codex teams
---

# Output Structure

## Applicability

- Applies to: `team-architect`, `agent-writer`, `skill-writer`, `rule-writer`

## Rule Content

### Directory Structure

All generated teams must follow this layout:

```text
teams/{team-name}/
├── AGENTS.md
├── agents/
│   ├── coordinator.toml
│   ├── delivery/
│   │   ├── specialist-a.toml
│   │   └── specialist-b.toml
│   └── review/
│       └── process-reviewer.toml
├── .codex/
│   ├── config.toml
│   ├── docs/
│   │   ├── format-mapping.md
│   │   └── format-mapping.manifest.yaml
│   ├── skills/
│   │   ├── {skill-1}/
│   │   │   └── SKILL.md
│   │   └── {skill-2}/
│   │       └── SKILL.md
│   └── rules/
│       ├── {rule-1}.md
│       └── {group}/
│           └── {rule-2}.md
└── .agents/
    └── skills/
        ├── {skill-1}/
        │   └── SKILL.md
        └── {skill-2}/
            └── SKILL.md
```

For `single-agent` teams, `agents/` may be omitted. For `multi-agent` teams, it is required.

### Naming Conventions

- Team names: kebab-case
- Directory names: kebab-case
- Agent config files: kebab-case `.toml`
- Markdown files: kebab-case, except `AGENTS.md` and `SKILL.md`
- Agent registry ids in `.codex/config.toml`: snake_case
- No spaces, uppercase folder names, or uppercase file names except `AGENTS.md` and `SKILL.md`

### AGENTS.md Content Guidelines

Place `AGENTS.md` at the team root. It is the Codex entrypoint for the generated team.

Include:

- team objective and scope summary
- universal behavioral norms
- project-wide technical constraints
- execution mode instructions and any runtime prerequisites that must be enabled inside the target project
- the requested delivery format if it differs from the canonical Codex-native authored format
- any `@path/to/file` imports that all team members truly need

Do not put role-specific rules or file-type-specific constraints in `AGENTS.md`. Those belong in `.codex/rules/` or the per-agent `developer_instructions`.

### Format Mapping Artifact

Every generated team must retain `.codex/docs/format-mapping.md`.

Include:

- requested delivery format
- canonical authored format
- Codex -> Claude path mapping
- Claude -> Codex path mapping
- lossy or unsupported conversions
- round-trip preservation notes

Also retain `.codex/docs/format-mapping.manifest.yaml` so future conversion can use machine-readable artifact relationships. Add sidecars under `.codex/docs/mapping-sidecars/` only when a conversion is lossy or bundle-based.

### Execution Mode Section

Every generated `AGENTS.md` must define one of these modes:

- `single-agent`: one Codex thread performs the work inline with minimal delegation
- `multi-agent`: the coordinator delegates in parallel with `spawn_agent`, uses `send_input` for follow-up, and joins work with `wait`

When `multi-agent` is chosen, the team must also define:

- project-level `.codex/config.toml` registration with `[features] multi_agent = true`
- one `[agents.<id>]` entry per coordinator or specialist
- one `agents/**/*.toml` file per registered agent
- `config_file` values written relative to `.codex/config.toml`; for sibling project-root agent files, use `../agents/...`
- parallel-safe file ownership
- coordinator follow-up triggers
- completion contracts for each spawned specialist
- project-level runtime prerequisites for Codex
- setup fallback when the target project already has conflicting Codex config

### Registry Path Resolution

`config_file` values in `.codex/config.toml` are resolved relative to the `.codex/` directory, not the project root. Since generated multi-agent teams place runtime agent configs under the project-root `agents/` folder, those registry entries must use `../agents/...`.

### Path-Scoped Rules

Rules may include optional `paths` frontmatter to scope them to file patterns. This is an internal A-Team convention for rule organization; it does not replace `AGENTS.md`.

### Placement Rules

- `AGENTS.md` must live at the team root
- `.codex/config.toml` must exist
- `.codex/docs/format-mapping.md` must exist
- `.codex/docs/format-mapping.manifest.yaml` must exist
- every skill must exist in both `.codex/skills/{skill}/SKILL.md` and `.agents/skills/{skill}/SKILL.md`
- rules must live under `.codex/rules/`
- `multi-agent` teams must include `agents/`
- the coordinator file must live at `agents/` root
- non-coordinator agents must live under group subfolders in `agents/`
- `multi-agent` teams must set `[features] multi_agent = true` in `.codex/config.toml`
- every `[agents.<id>]` entry must include `description` and `config_file`
- every `config_file` must resolve from the `.codex/` directory to an existing `../agents/**/*.toml`
- every agent config must include `model`, `model_reasoning_effort`, `sandbox_mode`, and `developer_instructions`

## Violation Determination

- `AGENTS.md` is missing or not at the team root -> Violation
- `.codex/config.toml` is missing -> Violation
- `.codex/docs/format-mapping.md` is missing -> Violation
- `.codex/docs/format-mapping.manifest.yaml` is missing -> Violation
- a `multi-agent` team omits the `agents/` directory -> Violation
- a non-coordinator agent is placed at `agents/` root -> Violation
- `.codex/config.toml` lacks `[features] multi_agent = true` for a `multi-agent` team -> Violation
- an agent registry entry is missing `description` or `config_file` -> Violation
- a registered `config_file` does not resolve from `.codex/config.toml` -> Violation
- an agent config is missing one of the required keys -> Violation
- a skill exists only in `.codex/skills/` or only in `.agents/skills/` -> Violation
- a `multi-agent` `AGENTS.md` omits project-level runtime prerequisites or conflict fallback -> Violation
- file or folder names break the naming conventions above -> Violation

## Exceptions

- Single-agent teams may omit `agents/` and the `[agents]` registry entirely.
