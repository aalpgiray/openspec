## ADDED Requirements

### Requirement: Project override directory

The system SHALL recognize an `openspec/overrides/` directory as the home for project-specific guidance that augments generated skills and commands.

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

### Requirement: Override-to-artifact matching by explicit name

The system SHALL match each override file to the generated artifact it extends by explicit skill or command identifier drawn from the canonical generated lists, not by pattern matching against tool directories.

#### Scenario: Override matches a generated artifact

- **WHEN** an override file names a skill or command identifier that exists in the canonical generated lists
- **THEN** its content SHALL be applied to that artifact during generation

#### Scenario: Override has no matching base artifact

- **WHEN** an override file names an identifier that is not in the canonical generated lists
- **THEN** the system SHALL emit a clear warning naming the unmatched override
- **AND** SHALL continue generating the remaining artifacts

### Requirement: Overrides survive update

The system SHALL preserve project override sources across `openspec update` runs.

#### Scenario: Override persists after update

- **GIVEN** a project with override files in `openspec/overrides/`
- **WHEN** the user runs `openspec update`
- **THEN** the override files in `openspec/overrides/` SHALL remain unchanged
- **AND** their content SHALL appear in the regenerated skills and commands
