## ADDED Requirements

### Requirement: Project override directory

The system SHALL recognize an `openspec/overrides/` directory holding one file per workflow, named by its canonical workflow identifier, as the source of project-specific bodies for generated skills and commands.

#### Scenario: Override directory is optional

- **WHEN** `openspec/overrides/` does not exist
- **THEN** generation SHALL behave exactly as it does today
- **AND** no error or warning SHALL be produced

#### Scenario: Override directory is version-controlled source

- **WHEN** `openspec update` runs
- **THEN** the contents of `openspec/overrides/` SHALL be treated as input to generation
- **AND** the directory SHALL never be a target for cleanup or deletion

#### Scenario: Cross-platform override path resolution

- **WHEN** the system locates the override directory or its files
- **THEN** it SHALL construct paths using path join utilities
- **AND** SHALL NOT assume a forward-slash separator

### Requirement: Override matching by canonical workflow identifier

The system SHALL match each override file to a workflow by explicit canonical workflow identifier, not by pattern matching against tool directories.

#### Scenario: Override matches a known workflow

- **WHEN** an override file is named for a workflow identifier in the canonical workflow list (for example `apply.md`)
- **THEN** its content SHALL be used as that workflow's body during generation

#### Scenario: Override matches no known workflow

- **WHEN** an override file's name does not correspond to any canonical workflow identifier
- **THEN** the system SHALL emit a clear warning naming the unmatched file
- **AND** SHALL continue generating the remaining artifacts

### Requirement: Full-swap replacement, no merge

The system SHALL use a present override as the full body for its workflow, replacing the package default body without merging.

#### Scenario: Override present

- **WHEN** `openspec/overrides/<workflow>.md` exists
- **THEN** the generated skill and command for that workflow SHALL use the override content as the body across every configured tool
- **AND** the package default body for that workflow SHALL NOT be included

#### Scenario: Override absent

- **WHEN** no override exists for a workflow
- **THEN** the generated skill and command SHALL use the package default body

### Requirement: Overrides survive update

The system SHALL preserve project override sources across `openspec update` runs.

#### Scenario: Override persists after update

- **GIVEN** a project with override files in `openspec/overrides/`
- **WHEN** the user runs `openspec update`
- **THEN** the override files in `openspec/overrides/` SHALL remain unchanged
- **AND** their content SHALL appear as the body of the regenerated skills and commands
