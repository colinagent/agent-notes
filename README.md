# Agent Notes

A portable skill that teaches an AI agent how to write and maintain **Agent Notes** — durable decision records kept inside the repository where the decision was made.

Notes are written in Chinese, live at `<repo>/.agents/notes/`, and follow the proposed / implemented / rejected / archived lifecycle. There is no central notes repository, no i18n sidecar, and no verification tooling: the skill is the specification.

## Install

Copy or symlink the skill into a project (or into the global skills directory):

```bash
# global, available to every project
ln -s "$(pwd)" ~/.agents/skills/agent-notes

# or per project
mkdir -p .agents/skills
ln -s /path/to/agent-notes .agents/skills/agent-notes
```

If your agent reads `AGENTS.md`, add one line to the project's `AGENTS.md`:

```markdown
Agent Notes live in `.agents/notes/`; follow `.agents/skills/agent-notes/SKILL.md`.
```

## What the skill defines

- **Layout** — `.agents/notes/{proposed,implemented,rejected}/{class}/YYYY-MM-DD-slug.md`, archived under `.agents/notes/archived/{class}/`.
- **Classes** — `feature`, `bug-fix`, `simplification`, `architecture`, `process`, `testing`.
- **Format** — fixed header, `## 问题` opener, mandatory `## 备选方案`, lifecycle-specific sections.
- **Lifecycle** — supersession checks, moves between folders, consolidation, archiving as frozen snapshots, deletion with a stated reason and repaired inbound links.

## Contents

- [`SKILL.md`](SKILL.md) — the skill.
- [`examples/`](examples/) — a sample note showing the expected shape.
