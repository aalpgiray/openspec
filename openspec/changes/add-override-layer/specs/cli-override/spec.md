## ADDED Requirements

### Requirement: Scaffold an override from the current default

The system SHALL provide an `openspec override <workflow>` command that writes the current rendered default body for a workflow into `openspec/overrides/<workflow>.md`.

#### Scenario: Scaffold a new override

- **WHEN** a user runs `openspec override apply`
- **AND** `openspec/overrides/apply.md` does not yet exist
- **THEN** the system SHALL write the current rendered default body for the `apply` workflow to `openspec/overrides/apply.md`
- **AND** SHALL report the path it wrote

#### Scenario: Cross-platform override path

- **WHEN** the command writes the override file
- **THEN** it SHALL construct the path using path join utilities
- **AND** SHALL NOT assume a forward-slash separator

### Requirement: Validate the requested workflow

The system SHALL reject a scaffold request for an unknown workflow.

#### Scenario: Unknown workflow

- **WHEN** a user runs `openspec override <name>` and `<name>` is not a canonical workflow identifier
- **THEN** the system SHALL display an error naming the unknown workflow
- **AND** SHALL list the valid workflow identifiers
- **AND** SHALL exit without writing a file

### Requirement: Protect an existing override

The system SHALL avoid silently overwriting an existing override file.

#### Scenario: Override already exists without force

- **WHEN** a user runs `openspec override <workflow>` and `openspec/overrides/<workflow>.md` already exists
- **THEN** the system SHALL refuse to overwrite it
- **AND** SHALL indicate that a force option is required to replace it

#### Scenario: Override already exists with force

- **WHEN** a user runs `openspec override <workflow>` with the force option and the file already exists
- **THEN** the system SHALL overwrite the file with the current rendered default body
- **AND** SHALL report the path it wrote
