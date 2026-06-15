## Why

OpenSpec fully regenerates the AI tool configuration layer — skills in `.claude/skills/`, commands in `.claude/commands/opsx/`, and the equivalents for every other tool — on each `openspec update`. These files are owned by OpenSpec and overwritten wholesale, so any project-specific guidance a user adds to them is lost on the next update.

Today the only escape hatches are to layer a separate tool (e.g. rulesync) on top, or to hand-edit generated files that get blown away. Neither is a clean, first-class way to say "this is how AI should behave in *this* project." Users need a customization layer that survives updates with predictable precedence — and crucially, one that does not require merging hand-edits with package defaults.

## What Changes

The source of truth lives in version-controlled `openspec/`; generated artifacts under tool directories stay fully managed and disposable. Customization happens by **full swap, not merge**: when a project provides an override for a workflow, that override *replaces* the default body — there is no line-level merge to maintain.

### 1. Add a project override directory

- Introduce `openspec/overrides/` where users place one file per workflow, named by its canonical workflow identifier (e.g. `overrides/apply.md`, `overrides/propose.md`).
- A file is matched to its workflow by explicit identifier from the canonical workflow list, never by pattern matching against tool directories.
- An override whose name matches no known workflow produces a clear warning rather than a silent no-op.

### 2. Full-swap composition (no merge)

- When `overrides/<workflow>.md` is present, its content becomes the instruction **body** for that workflow's skill and command across every configured tool, replacing the package default body.
- OpenSpec continues to own the **frontmatter** (`name`, `description`, `generatedBy`, tags) and all per-tool formatting, so overridden artifacts stay valid and tool-detectable and `generatedBy` stays current.
- An overridden workflow no longer inherits upstream changes to its default body. To keep this visible, `openspec update` reports which workflows are currently overridden.

### 3. Add a project constitution (the global layer)

- Introduce a single optional `openspec/constitution.md` — a project "supreme law" prepended to all generated AI tool context **and** to the workflow instruction context, ahead of `config.yaml` `context`.
- Because it applies everywhere, the constitution serves as the global layer; there is no separate per-everything override file.
- When absent, generation behaves exactly as today. Precedence is **constitution → config context → defaults**.

### 4. Add a scaffold helper

- Add `openspec override <workflow>` to copy the current rendered default body for a workflow into `openspec/overrides/<workflow>.md`, so users edit from the real default instead of authoring a full body from scratch.
- The command refuses to clobber an existing override unless forced, and reports the file it wrote.

### Out of scope

- No changes to the spec workflow itself (propose/apply/archive) or to spec/proposal/design/task artifacts.
- No line-level merge or templating engine — overrides fully replace a workflow body; the constitution is prepended as a distinct layer.
- No new external dependencies.

## Capabilities

### New Capabilities

- `tool-config-overrides`: Per-workflow override files under `openspec/overrides/` that fully replace a workflow's generated body, matched by explicit identifier, surviving `openspec update`.
- `project-constitution`: A single `openspec/constitution.md` supreme-law file prepended to all generated AI tool context and workflow instruction context.
- `cli-override`: An `openspec override <workflow>` command that scaffolds an override from the current rendered default.

### Modified Capabilities

- `cli-update`: Compose regenerated bodies as override-or-default (full swap), keep frontmatter managed, report overridden workflows, and keep cleanup/drift scoped to canonical lists so override sources are never removed.
- `cli-init`: Scaffold `openspec/overrides/` and an optional `openspec/constitution.md`, and apply the override/constitution layer when generating initial artifacts.
- `command-generation`: Select each generated body from the matching override when present, else the package default, while OpenSpec retains ownership of frontmatter and per-tool formatting.
- `context-injection`: Prepend constitution content ahead of `config.yaml` context in the workflow instruction context, preserving the source text exactly.

## Impact

- `src/core/update.ts` — override-or-default body selection in the regenerate loop; overridden-workflow report; keep removal/drift on canonical lists.
- `src/core/init.ts` — scaffold override directory and constitution; apply the layer on initial generation.
- `src/core/command-generation/` and `src/core/shared/skill-generation.ts` — full-swap body selection with managed frontmatter.
- `src/core/artifact-graph/instruction-loader.ts` / `src/core/project-config.ts` — read and prepend constitution content.
- `src/commands/` and `src/cli/` — new `openspec override <workflow>` command.
- `docs/opsx.md`, `docs/cli.md` — document the override directory, constitution, precedence, and the new command.
- `test/core/update.test.ts`, `test/core/init.test.ts`, generation tests, new override-command tests — cover full-swap selection, survival across update, missing-workflow warnings, overridden-workflow reporting, and Windows path handling.
