## Context

OpenSpec generates the AI tool configuration layer (skills, commands) from package-owned templates and fully overwrites those files on every `openspec update`. The generation pipeline is constant-driven: canonical lists of skill and command identifiers determine what is written and what is cleaned up, and cleanup operates by explicit list lookup. Project-specific customization today is limited to `openspec/config.yaml` (`context`, `rules`), which is consumed at instruction-load time and does not let users shape the generated skill/command bodies in a way that survives updates.

This change adds a customization layer for that generated tool configuration. It deliberately does not touch the spec workflow (propose/apply/archive).

## Goals / Non-Goals

**Goals**
- Let users add project-specific guidance to generated skills/commands that survives `openspec update`.
- Provide a single supreme-law file always prepended to generated AI context.
- Keep generated tool-directory files fully managed and re-generable — no manual merge.
- One documented precedence order; no surprises.

**Non-Goals**
- No arbitrary file-merge engine or token-level template rewriting.
- No changes to spec/proposal/design/task artifacts or the apply/archive flow.
- No new external dependencies.

## Decisions

**1. Overrides are inputs, not edited outputs.**
Override sources live in version-controlled `openspec/overrides/`; the generated files under tool directories stay fully managed and disposable. They survive updates because they are never overwritten — not because we parse and preserve hand-edits. Alternative considered: managed-marker blocks inside the generated tool files (like the existing `OPENSPEC:START/END` legacy markers) so user edits survive in place. Rejected as the primary store because it makes every generated file a merge surface and invites drift; markers are used only to demarcate the woven-in region, not as the source of truth.

**2. Match overrides by explicit identifier, not pattern.**
Overrides bind to artifacts via the canonical generated skill/command identifier lists, consistent with the project rule "if we generate it, we track it by name in a constant." A non-matching override warns rather than failing silently. Alternative considered: glob/pattern matching against tool directories — rejected per the same rule and to avoid coupling to per-tool path layouts.

**3. Constitution extends context injection rather than competing with it.**
`openspec/constitution.md` reuses the existing context-injection path and is prepended ahead of `config.yaml` `context`. Precedence is constitution → config context → defaults. This keeps one mental model for "project context" instead of introducing a parallel channel. Alternative considered: a new `constitution` field in `config.yaml` — rejected because a dedicated top-level file is easier to find, diff, and treat as supreme law.

**4. Cleanup/drift stay scoped to canonical lists.**
Removal and drift detection continue to iterate the canonical generated lists only, so `openspec/overrides/` and `openspec/constitution.md` are structurally outside the deletion surface.

## Risks / Trade-offs

- **Override content drifts from default behavior it augments** → Overrides append guidance and sit in a labeled region; they do not silently replace defaults, so a stale override is visible rather than masked.
- **Two context channels (constitution + config context) confuse users** → A single documented precedence order and docs in `docs/opsx.md` resolve which wins.
- **Windows path handling for the new directory/file** → All path construction uses path-join utilities; tests cover Windows path resolution per project convention.

## Migration Plan

Additive and backward-compatible. Projects with no `openspec/overrides/` and no `openspec/constitution.md` see identical output to today. `openspec init` scaffolds the override directory with explanatory guidance; existing projects gain the directory on their next `openspec update` or `openspec init` without affecting current generated files.

## Open Questions

- Should override files be organized one-per-artifact (named by identifier) or grouped, and should there be a shared/global override applied to every artifact in addition to per-artifact ones?
- Should the constitution also be surfaced to the spec-workflow instructions, or strictly scoped to the generated tool configuration layer as proposed here?
