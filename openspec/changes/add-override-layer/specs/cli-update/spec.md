## ADDED Requirements

### Requirement: Update composes overrides and constitution

The update command SHALL regenerate skills and commands by layering package defaults first and then project overrides, and SHALL prepend the project constitution to the generated AI tool context.

#### Scenario: Update layers overrides on defaults

- **WHEN** a user runs `openspec update`
- **AND** `openspec/overrides/` contains override files matching generated artifacts
- **THEN** each regenerated artifact SHALL contain the package default content followed by the matching override content

#### Scenario: Update prepends constitution

- **WHEN** a user runs `openspec update`
- **AND** `openspec/constitution.md` exists
- **THEN** the regenerated AI tool context SHALL begin with the constitution content

#### Scenario: Update without overrides or constitution

- **WHEN** a user runs `openspec update`
- **AND** neither `openspec/overrides/` nor `openspec/constitution.md` is present
- **THEN** the regenerated artifacts SHALL be identical to today's output

### Requirement: Cleanup never targets override sources

The update command SHALL scope cleanup and drift detection to the canonical generated skill and command lists, leaving project override sources untouched.

#### Scenario: Override sources excluded from removal

- **WHEN** the update command removes or detects drift in generated artifacts
- **THEN** the removal SHALL be limited to entries in the canonical generated lists
- **AND** `openspec/overrides/` and `openspec/constitution.md` SHALL NOT be deleted or modified
