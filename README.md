# hermes-dev-skills

A library of agent skills for developing with the
[Hermes Agent](https://github.com/earendil-works/hermes-agent) in mind.
Memory providers, curator logic, deterministic auto-triggers, profile
distributions, REST API integration — everything an agent needs to build on
top of Hermes.

Each skill is self-contained and individually installable; the repo is a
collection.

## Skills

| Skill | Description |
| --- | --- |
| [`hermes-profile-architecture`](skills/hermes-profile-architecture/SKILL.md) | Comprehensive reference for every aspect of Hermes Agent profiles — the multi-profile isolation model, HERMES_HOME resolution, CLI subcommands, config.yaml and SOUL.md, wrapper aliases, sticky active profile, credential shadowing, distributions, and gateway multiplexing. |

<!-- Append new skills as rows above this line: | [`skill-name`](skills/skill-name/SKILL.md) | One-sentence description. | -->

## Installation

Install with the [skills CLI](https://github.com/vercel-labs/skills):

```bash
# interactive picker across all skills in this repo
npx skills add r3b1s/hermes-dev-skills

# list available skills without installing
npx skills add r3b1s/hermes-dev-skills --list

# install a single skill
npx skills add r3b1s/hermes-dev-skills --skill hermes-profile-architecture

# one-shot use without installing
npx skills use r3b1s/hermes-dev-skills@hermes-profile-architecture | claude
```

## Structure

Each skill lives in its own directory under `skills/` and must be fully
self-contained (the CLI installs only the selected skill's directory):

```
skills/
  <skill-name>/
    SKILL.md        # frontmatter (name, description) + core workflow
    metadata.json   # version, abstract, references
    resources/      # templates, catalogs, checklists the SKILL.md links to
```

Conventions for new skills:

- The frontmatter `name` matches the directory name — it is what
  `--skill <name>` selects on.
- SKILL.md may link to its own `resources/`, never to sibling skills or the
  repo root.
- Add a row to the skills table above.
