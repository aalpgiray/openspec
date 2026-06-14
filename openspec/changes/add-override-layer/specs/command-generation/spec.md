## ADDED Requirements

### Requirement: Layered generation with a managed override region

Generation SHALL compose each skill and command from package defaults followed by matching project override content, placing the override content inside a clearly demarcated OpenSpec-managed region.

#### Scenario: Override content is appended within a managed region

- **WHEN** a generated artifact has matching override content
- **THEN** the default content SHALL appear first
- **AND** the override content SHALL be appended inside an explicitly demarcated, OpenSpec-managed region

#### Scenario: Generated artifact remains fully managed

- **WHEN** generation runs again
- **THEN** the generated artifact in the tool directory SHALL be fully regenerated from defaults plus the current override source
- **AND** no prior hand-edits to the generated file SHALL be required to preserve override content

#### Scenario: No override content

- **WHEN** a generated artifact has no matching override content
- **THEN** the artifact SHALL contain only the package default content
- **AND** no empty managed region SHALL be emitted
