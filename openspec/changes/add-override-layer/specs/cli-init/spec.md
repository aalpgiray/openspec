## ADDED Requirements

### Requirement: Init scaffolds the override layer

The init command SHALL scaffold the override layer entry points and apply them when generating initial artifacts.

#### Scenario: Init creates override directory

- **WHEN** a user runs `openspec init`
- **THEN** an `openspec/overrides/` directory SHALL exist after initialization
- **AND** it SHALL contain guidance explaining how overrides layer on top of generated skills and commands

#### Scenario: Init applies the layer on first generation

- **WHEN** initialization generates skills and commands
- **THEN** generation SHALL apply the override and constitution layer using the same precedence as `openspec update`

#### Scenario: Cross-platform scaffolding paths

- **WHEN** init creates the override directory or constitution file
- **THEN** it SHALL construct paths using path join utilities
- **AND** SHALL NOT assume a forward-slash separator
