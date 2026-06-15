## ADDED Requirements

### Requirement: Body selection from override or default

Generation SHALL select each generated skill and command body from the matching project override when present, and otherwise from the package default.

#### Scenario: Override drives the body

- **WHEN** a workflow has a matching override
- **THEN** the generated body SHALL be the override content
- **AND** the package default body SHALL NOT be emitted for that workflow

#### Scenario: Default drives the body

- **WHEN** a workflow has no matching override
- **THEN** the generated body SHALL be the package default content

### Requirement: Frontmatter and formatting remain managed

Generation SHALL retain ownership of artifact frontmatter and per-tool formatting even when a workflow's body is overridden.

#### Scenario: Overridden artifact keeps managed frontmatter

- **WHEN** a workflow body is overridden
- **THEN** the generated skill and command SHALL still carry OpenSpec-managed frontmatter (including `name`, `description`, `generatedBy`, and tags where applicable)
- **AND** SHALL apply the same per-tool formatting and instruction transforms as a non-overridden artifact

#### Scenario: Constitution prepended ahead of the body

- **WHEN** a constitution is present
- **THEN** its content SHALL be prepended to the generated body, ahead of the override-or-default content
