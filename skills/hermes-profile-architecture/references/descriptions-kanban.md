# Profile descriptions & kanban routing

`hermes profile describe <name>` manages the `description` field in `profile.yaml`.

Source: `profile_describer.py` (full implementation), `profiles.py:read_profile_meta()` / `write_profile_meta()`.

## `--text <description>`
Write a literal description. Overwrites any existing value.

## `--auto`
Auto-generate description via the auxiliary LLM (`auxiliary.profile_describer` in config.yaml). Reads:
- Profile name
- Model/provider from config.yaml
- Up to 60 skill names (evenly sampled across alphabetically sorted list, not just the head)

Writes description with `description_auto: true` — the dashboard surfaces a "review" badge. Existing user-authored descriptions are preserved unless `--overwrite` is passed.

### Auto-generate prompt design

System prompt instructs the LLM to:
- Lead with the profile's strongest capability
- Stay concrete (avoids "an AI agent that helps users")
- 1-2 sentences, <= 280 characters
- Never invent capabilities the skills don't suggest
- Output JSON: `{"description": "..."}`

Source: `profile_describer.py:_SYSTEM_PROMPT` (lines 22-41), `_USER_TEMPLATE`.

## `--all`
With `--auto`, sweep every profile missing a description.

## `--overwrite`
With `--auto`, replace even user-authored descriptions. Without it, only fills in missing or previously-auto descriptions.

## Kanban integration

The kanban decomposer (`kanban_specify.py`, `kanban_decompose.py`) reads profile descriptions to route tasks to the best-fit profile. Profiles with empty descriptions fall back to the profile name alone. This enables orchestrator routing based on role rather than name.
