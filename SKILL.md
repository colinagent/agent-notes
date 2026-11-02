---
name: agent-notes
description: Use when recording, updating, superseding, archiving, reviewing, or deleting Agent Notes in the current repository. Writes durable Chinese decision records under .agents/notes/ with the fixed Problem / Decision / Alternatives considered / Consequences format, runs the supersession check every new note requires, and applies the proposed / implemented / rejected / archived lifecycle rules.
---

# Agent Notes

An Agent Note is an RFC written by an agent: a durable record of one decision and its *why*, the alternatives it beat, and what it cost. Notes belong to the repository where the decision was made, under that repository's `.agents/notes/` — never a shared or central notes repository.

Write note bodies in Chinese. Keep the machine tokens `# Agent Note: `, `Status:`, `Archived:` and the `Status` values in English. Filenames use an ASCII kebab-case slug. Code, paths, identifiers, and config keys keep their original spelling.

## Layout

`.agents/notes/{lifecycle}/{class}/YYYY-MM-DD-slug.md`

- **Lifecycle** is the status folder and changes as the note moves:
  - `proposed/` — reviewed but not built, or only partly built.
  - `implemented/` — shipped. Kept current with what actually shipped (paths, names, defaults, mechanisms).
  - `rejected/` — considered and declined. Kept only while it prevents a plausible, tempting mistake.
- **Class** is the kind of decision, from a closed set:

  | Class | Covers |
  |---|---|
  | `feature` | A new user- or model-facing capability. |
  | `bug-fix` | Corrects a defect or closes a gap a postmortem surfaced. |
  | `simplification` | Removes code, behavior, or surface area without adding a capability. |
  | `architecture` | A structural decision about the shipped source — how modules relate, what the runtime vocabulary is. |
  | `process` | Tooling, policy, or workflow around the code — gates, package manager, vendoring. |
  | `testing` | Test infrastructure and strategy. |

  `architecture` is the source you ship; `process` is the surrounding tooling. Do not invent an ad-hoc class: a new class is a deliberate change to this list.

- **Archived** lives at `.agents/notes/archived/{class}/YYYY-MM-DD-slug.md` — `implemented` is deliberately absent because only implemented notes enter the archive.

The date is when the topic was **first proposed**. Do not add a centralized `INDEX.md`; the path already encodes the metadata. Cross-link notes with relative markdown links (`[标题](../../implemented/architecture/2026-01-15-topic.md)`), never bare numbers or prose references.

If the repository already has `.agents/notes/AGENTS.md` or an established class set, follow it; otherwise create only the lifecycle/class folders you actually use.

## When to write one

Add or update a note in the same change when the decision's rationale — why it won, what was given up, what must not be reintroduced — is not recoverable from code, tests, or docs.

- A proposal for substantial future work starts in `proposed/`; a decision already made starts in `implemented/`.
- Mechanical edits, local UI or copy changes, and anything a future reader can infer from the diff are exempt.
- Updating the note that already owns the decision is enough; do not create a duplicate.
- A note is never edited into a *different* decision: supersede it with a new note and cross-link both.

## File format

The first three lines are exactly:

```markdown
# Agent Note: <标题>

Status: <status>
```

followed by a blank line. `Status` must match the folder: `proposed`, `implemented`, or `rejected — <一句话原因>`. The rejection reason is the only status with content; no dates or parentheticals otherwise (the filename holds the date, and an amendment belongs where the decision is stated).

The body opens with `## 问题` — the motivation, written to stand without the solution. Canonical section names:

- `proposed/`: `## 问题`, `## 提案`, optional bespoke sections, `## 备选方案`, `## 验收标准`, `## 风险`.
- `implemented/`: `## 问题`, `## 决策` (present tense, shipped reality), optional bespoke sections (schema, topology, wire contract), `## 备选方案`, `## 后果`.
- `rejected/`: the proposal frozen as written; only the header, `## 问题`, `## 提案`, and `## 备选方案` are mandatory.

`## 备选方案` is mandatory in every note: each genuine alternative and why it lost, one bold-led paragraph per alternative, or `### 为什么不选 <X>？` per contested one. Record alternatives that were actually considered — never invent them. A decision recorded without what it beat invites re-litigation.

Proposal-era headings — `## 提案`, `## 计划`, `## 迁移计划`, `## 验收标准` — must not appear in an `implemented/` note.

### Implemented template

```markdown
# Agent Note: <标题>

Status: implemented

## 问题

<为什么需要这个决定；不读方案也能读懂。>

## 决策

<已上线的现实，现在时。路径、名称、默认值、机制与代码一致。>

## 备选方案

**<备选项>**：<它是什么，为什么落选。>

## 后果

- <主动付出的代价与换来的收益。>
```

### Proposed template

```markdown
# Agent Note: <标题>

Status: proposed

## 问题

<动机。>

## 提案

<要做什么，迁移步骤，未决问题。>

## 备选方案

**<备选项>**：<它是什么，为什么落选。>

## 验收标准

<什么可观察状态算完成。>

## 风险

<可能出错的地方，以及主动放弃了什么。>
```

### Rejected note

Keep the proposal body frozen and put the verdict on the status line:

```markdown
Status: rejected — <为什么否决，一句话>
```

## Supersession check — every new note

Before drafting, apply the creation criteria, then search the active tree (`.agents/notes/{proposed,implemented,rejected}/`) for notes covering the same decision, mechanism, or rejected alternative. Search by topic and class, not only by title.

Classify every hit while writing the new note:

- **Full supersession** — the new note owns every unique proposition. Archive (or delete) the old implemented note in the same change, and repair inbound links.
- **Partial supersession** — keep both active and cross-link them; update the facts that remain current.
- **Obsolete proposal** — reject it with an honest reason instead of archiving.
- **Rejected note no longer preventing a tempting mistake** — delete it.

Do not defer a known match to a later audit.

## Moving between lifecycles

A move changes the folder, the `Status:` line, and the body shape in the same change:

- `proposed/` → `implemented/`: rewrite `## 提案` into a present-tense `## 决策`, fold `## 验收标准` and `## 风险` into `## 后果`, and drop plans in favor of what shipped.
- `proposed/` → `rejected/`: add the reason to the `Status:` line and freeze the file.
- `implemented/` → `archived/`: see [Archiving](#archiving).

## Keeping implemented notes current

Update paths, symbols, defaults, and mechanisms in place in the same change that alters them; do not append change history and do not rewrite the decision. When the decision itself reverses, write a new note and cross-link, and consolidate the old one only under the deletion rule below.

## Archiving

Archive an implemented note when the shipped decision is complete, its rationale is unlikely to guide future work, but the record still has historical value. Delete instead when it meets the [deletion](#deleting-a-note) criteria — archive is not a softer way to keep junk.

1. Move the file to `.agents/notes/archived/{class}/`, keeping the same filename.
2. Insert `Archived: YYYY-MM-DD` immediately below `Status: implemented`. Make no other body edits: no translating, reformatting, fact updates, or link repair inside the note.
3. Retarget inbound links to current authority, or to the archived path only when the history is intentionally cited; delete the rest.

Once archived, the file is a frozen historical snapshot: never edit, move, reformat, or delete it, and never treat it as authority for current behavior. Active prose may still link into it when citing history. Judge semantically; length and age are never archive criteria, and there is no quota.

## Deleting a note

Allowed when:

- an implemented note only records a small UI adjustment or a purely mechanical change (a local bug fix, performance change, new capability, or substantive behavior decision does not qualify merely because it is small);
- the note has been fully consolidated into its superseding owner, and the owner preserves every unique rationale, alternative, consequence, and verification detail;
- a rejected note's premise is obsolete, or it no longer prevents a plausible mistake;
- a proposal is no longer worth pursuing — reject it rather than deleting a live proposal.

Procedure: state the reason in the commit message and the final report, repair or remove inbound links, then delete the whole file. Never rely on git history as the only copy of rationale that still matters.

## Validate and report

Before finishing:

- header, status, folder, and section names agree with this format;
- `## 备选方案` is present and records real alternatives;
- relative links resolve, and no active prose points at a deleted note;
- archived files were only moved and given the `Archived:` line.

Report in the final response: notes added or updated, notes archived with their class, notes deleted with their reason, supersessions resolved, and any genuinely borderline case with its chosen outcome.
