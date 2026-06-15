## ADDED Requirements

### Requirement: Update composes override-or-default bodies and prepends constitution

The update command SHALL regenerate each skill and command using the matching override body when present and the package default otherwise, and SHALL prepend the project constitution to the generated context.

#### Scenario: Update uses override bodies

- **WHEN** a user runs `openspec update`
- **AND** `openspec/overrides/` contains files matching known workflows
- **THEN** each matching workflow's regenerated skill and command SHALL use the override content as its body
- **AND** OpenSpec-managed frontmatter SHALL still be generated for those artifacts

#### Scenario: Update prepends constitution

- **WHEN** a user runs `openspec update`
- **AND** `openspec/constitution.md` exists
- **THEN** the regenerated context SHALL begin with the constitution content

#### Scenario: Update without overrides or constitution

- **WHEN** a user runs `openspec update`
- **AND** neither `openspec/overrides/` nor `openspec/constitution.md` is present
- **THEN** the regenerated artifacts SHALL be identical to today's output

### Requirement: Update reports overridden workflows

The update command SHALL report which workflows are overridden so users know those workflows did not receive package default updates.

#### Scenario: Overridden workflows surfaced in output

- **WHEN** `openspec update` regenerates artifacts and one or more workflows are overridden
- **THEN** the command output SHALL name the overridden workflows
- **AND** SHALL indicate their package defaults were not applied

### Requirement: Cleanup never targets override sources

The update command SHALL scope cleanup and drift detection to the canonical generated skill and command lists, leaving project override sources untouched.

#### Scenario: Override sources excluded from removal

- **WHEN** the update command removes or detects drift in generated artifacts
- **THEN** the removal SHALL be limited to entries in the canonical generated lists
- **AND** `openspec/overrides/` and `openspec/constitution.md` SHALL NOT be deleted or modified
