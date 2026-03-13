---
name: Coordinator Mandate
description: Require one non-executing coordinator in every generated multi-agent team
---

# Coordinator Mandate

## Applicability

- Applies to: all generated `multi-agent` teams and all A-Team writers

## Rule Content

### Coordinator Must Exist

Every generated `multi-agent` team must include one coordinator agent registration in `teams/{team-name}/.codex/config.toml` and one matching config file under `teams/{team-name}/agents/`.

Because `.codex/config.toml` lives under `.codex/`, the coordinator registration must point to the project-root agent file with `../agents/coordinator.toml`.

Recommended shape:

```toml
[agents.coordinator]
description = "Coordinate delegation, follow-ups, and final synthesis."
config_file = "../agents/coordinator.toml"
```

### Coordinator Does Not Execute

The coordinator may plan, assign, track, follow up, resolve dependencies, and enforce quality gates. The coordinator must not absorb delivery work that belongs to specialist agents. Its `developer_instructions` must stay coordination-only.

### Flat Architecture Only

Use one coordinator with direct specialists. Do not add sub-coordinators. In Codex, extra relay layers around `spawn_agent` and `send_input` create unnecessary information loss and slower joins.

## Violation Determination

- No coordinator registration exists in `.codex/config.toml` for a `multi-agent` team -> Violation
- The coordinator `config_file` path does not resolve from `.codex/config.toml` to `../agents/coordinator.toml` -> Violation
- The coordinator `developer_instructions` contain production work instead of coordination work -> Violation
- A second coordinator exists only to relay tasks between the main coordinator and workers -> Violation

## Exceptions

Single-agent teams do not require a coordinator registration.
