## Context

OpenSpec generates the AI tool configuration layer (skills, commands) from package-owned workflow templates and fully overwrites those files on every `openspec update`. The pipeline is constant-driven: canonical lists of workflow/skill/command identifiers determine what is written and cleaned up, and cleanup operates by explicit list lookup. Project customization today is limited to `openspec/config.yaml` (`context`, `rules`), consumed at instruction-load time, and cannot shape generated skill/command bodies in a way that survives updates.

This change adds a customization layer for that generated tool configuration. It deliberately does not touch the spec workflow (propose/apply/archive).

## Goals / Non-Goals

**Goals**
- Let users replace the body of a generated skill/command per workflow, surviving `openspec update`.
- Provide a single supreme-law file prepended everywhere AI context is composed.
- Keep generated tool-directory files fully managed and re-generable — never a merge surface.
- One documented precedence order; no surprises.

**Non-Goals**
- No line-level merge or token-level templating of default bodies.
- No changes to spec/proposal/design/task artifacts or the apply/archive flow.
- No new external dependencies.

## Decisions

**1. Full swap, not merge.**
When `openspec/overrides/<workflow>.md` exists, it *replaces* the workflow's default body wholesale; otherwise the default is used. There is no line-level merge. Alternatives considered: append-augment (overrides added after defaults) and section-replacement (target and replace parts of the default). Both were rejected because they make every generated artifact a merge surface — the user's content drifts against changing defaults and must be reconciled. Full swap keeps the model trivial: you either own a workflow's body or you don't. The accepted trade-off is that an overridden workflow stops inheriting upstream default improvements; decision 3 makes that visible.

**2. Override the body, keep the frontmatter managed.**
An override supplies the instruction body only. OpenSpec continues to generate frontmatter (`name`, `description`, `generatedBy`, tags) and run per-tool formatting/instruction transforms. Alternative considered: let the override replace the entire file. Rejected because it would force users to hand-maintain per-tool frontmatter and escaping, breaking the cross-tool promise and detaching `generatedBy` from the installed version.

**3. Make shadowing visible.**
Because overridden workflows freeze against default changes, `openspec update` reports which workflows are overridden and that their package defaults were not applied. This kills the silent-staleness footgun without adding complexity.

**4. Match by canonical workflow identifier.**
Overrides bind to workflows via the canonical workflow list (one file per workflow, named by ID), consistent with the project rule "if we generate it, we track it by name in a constant." A non-matching file warns rather than failing silently. Pattern matching against tool directories was rejected per the same rule and to avoid coupling to per-tool path layouts.

**5. Constitution is the global layer; it spans both surfaces.**
`openspec/constitution.md` is prepended to both the generated skill/command bodies and the runtime instruction-loader context (ahead of `config.yaml` `context`). Precedence is constitution → config context → defaults. Because it already applies everywhere, it serves the "applies to every artifact" role, so no separate global override file (`_all.md`) is introduced — that would be a second prepend channel doing the same job. The constitution is prepended (composition), never merged into a body, so it coexists cleanly with full-swap overrides.

**6. Provide a scaffold helper.**
`openspec override <workflow>` writes the current rendered default body into `openspec/overrides/<workflow>.md` so users edit from the real default instead of authoring a full body from scratch. It validates the workflow name and refuses to clobber an existing file without a force option. This directly offsets the main cost of full swap (you have to supply a complete body).

## Risks / Trade-offs

- **Overridden workflows miss default improvements** → `openspec update` reports overridden workflows (decision 3); the scaffold helper makes re-syncing from a fresh default straightforward.
- **Constitution and config context overlap** → A single documented precedence order (constitution → config context → defaults) and docs in `docs/opsx.md` resolve which wins.
- **Windows path handling for the new directory/file/command** → All path construction uses path-join utilities; tests cover Windows path resolution per project convention.

## Migration Plan

Additive and backward-compatible. Projects with no `openspec/overrides/` and no `openspec/constitution.md` see identical output to today. `openspec init` scaffolds the override directory with explanatory guidance; existing projects gain it on their next `openspec init`/`openspec update` without affecting current generated files.

## Open Questions

- Should `openspec override <workflow>` default to scaffolding the body only (matching what overrides replace), or offer an option to include the rendered frontmatter for reference?
- Should `openspec update` (or `openspec list`) gain a compact "overridden workflows" summary beyond the per-run report, e.g. for CI visibility?
