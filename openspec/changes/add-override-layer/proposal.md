## Why

OpenSpec fully regenerates the AI tool configuration layer — skills in `.claude/skills/`, commands in `.claude/commands/opsx/`, and the equivalents for every other tool — on each `openspec update`. These files are owned by OpenSpec and overwritten wholesale, so any project-specific guidance a user adds to them is lost on the next update.

Today the only escape hatches are to layer a separate tool (e.g. rulesync) on top, or to hand-edit generated files that get blown away. Neither is a clean, first-class way to say "this is how AI should behave in *this* project." Users need a customization layer that survives updates and has predictable precedence over the defaults.

## What Changes

The fix follows OpenSpec's existing model: the source of truth lives in version-controlled `openspec/`, and generated artifacts under tool directories stay fully managed and disposable. Overrides are *inputs* to generation, not hand-edits to *outputs* — so they survive updates by never being overwritten.

### 1. Add a project override directory

- Introduce `openspec/overrides/` where users place project-specific guidance that augments generated skills and commands.
- Overrides are matched to the artifact they extend by explicit name (skill/command IDs from the canonical generated lists), never by pattern matching against tool directories.
- An override with no matching base artifact produces a clear warning rather than a silent no-op.

### 2. Layer overrides on top of package defaults during generation

- `openspec init` and `openspec update` compose each generated skill/command from **package defaults first, then project overrides** — a single, documented precedence order.
- Override content is appended to the generated artifact's body inside a clearly demarcated, OpenSpec-managed region so the result stays predictable and re-generable. The user's override *source* in `openspec/overrides/` is the durable copy; the woven-in tool-directory file remains fully managed.
- Cleanup and drift detection continue to operate on the canonical generated lists only; `openspec/overrides/` is never a deletion target.

### 3. Add a project constitution

- Introduce a single `openspec/constitution.md` — a project "supreme law" file whose content is prepended to all generated AI tool context, ahead of `config.yaml` `context`.
- The constitution is optional. When absent, generation behaves exactly as today.
- This extends the existing context-injection mechanism with a dedicated, first-class file rather than a new competing channel: precedence is **constitution → config context → defaults**.

### Out of scope

- No changes to the spec workflow (propose/apply/archive) or to spec/proposal/design/task artifacts.
- No arbitrary file-merge or templating engine — overrides append guidance, they do not rewrite default behavior token-by-token.
- No new external dependencies.

## Capabilities

### New Capabilities

- `tool-config-overrides`: Project override files under `openspec/overrides/` that layer on top of generated skills and commands with defined precedence and survive `openspec update`.
- `project-constitution`: A single `openspec/constitution.md` supreme-law file prepended to all generated AI tool context.

### Modified Capabilities

- `cli-update`: Compose regenerated skills/commands from defaults plus project overrides, and prepend the constitution, instead of plain replacement; keep cleanup/drift scoped to canonical generated lists so override sources are never removed.
- `cli-init`: Scaffold `openspec/overrides/` and an optional `openspec/constitution.md`, and apply the override/constitution layer when generating initial artifacts.
- `command-generation`: Generate command/skill content by layering package defaults then project overrides, with the override region explicitly demarcated.
- `context-injection`: Prepend constitution content ahead of `config.yaml` context in the generated AI context, preserving the source text exactly.

## Impact

- `src/core/update.ts` — weave overrides + constitution into the regenerate loop; keep removal/drift on canonical lists.
- `src/core/init.ts` — scaffold override directory and constitution; apply the layer on initial generation.
- `src/core/command-generation/` and `src/core/shared/skill-generation.ts` — compose default + override content with a managed override region.
- `src/core/artifact-graph/instruction-loader.ts` / `src/core/project-config.ts` — read and prepend constitution content.
- `docs/opsx.md`, `docs/cli.md` — document the override directory, constitution, and precedence rules.
- `test/core/update.test.ts`, `test/core/init.test.ts`, generation tests — cover precedence, survival across update, missing-base warnings, and Windows path handling for the new directory/file.
