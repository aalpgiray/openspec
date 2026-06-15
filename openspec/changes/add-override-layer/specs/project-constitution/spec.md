## ADDED Requirements

### Requirement: Project constitution file

The system SHALL recognize an optional `openspec/constitution.md` file as a single project-wide "supreme law" prepended to both the generated AI tool context and the workflow instruction context.

#### Scenario: Constitution present in generated artifacts

- **WHEN** `openspec/constitution.md` exists and is non-empty
- **THEN** its content SHALL be prepended to the generated body of every skill and command

#### Scenario: Constitution present in workflow instruction context

- **WHEN** `openspec/constitution.md` exists and is non-empty
- **THEN** its content SHALL be prepended to the workflow instruction context for every artifact

#### Scenario: Constitution absent

- **WHEN** `openspec/constitution.md` does not exist
- **THEN** generation and instruction loading SHALL behave exactly as they do today
- **AND** no error SHALL be produced

#### Scenario: Cross-platform constitution path resolution

- **WHEN** the system locates the constitution file
- **THEN** it SHALL construct the path using path join utilities
- **AND** SHALL NOT assume a forward-slash separator

### Requirement: Constitution precedence

The system SHALL apply a single, documented precedence when composing context: constitution first, then project config context, then defaults.

#### Scenario: Constitution precedes config context

- **GIVEN** both `openspec/constitution.md` and a `context` field in `openspec/config.yaml`
- **WHEN** context is composed
- **THEN** the constitution content SHALL appear before the config context content

### Requirement: Constitution content preserved exactly

The system SHALL inject constitution content without modification, escaping, or interpretation.

#### Scenario: Constitution contains markup and special characters

- **WHEN** the constitution includes Markdown, URLs, or characters such as `<`, `>`, `&`
- **THEN** the content SHALL be preserved exactly as written
