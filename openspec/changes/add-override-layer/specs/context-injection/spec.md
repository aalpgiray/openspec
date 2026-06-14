## ADDED Requirements

### Requirement: Prepend constitution ahead of config context

The system SHALL prepend project constitution content ahead of the `openspec/config.yaml` context when composing AI tool context, preserving each source's text exactly.

#### Scenario: Constitution and config context both present

- **WHEN** both a constitution and a config `context` field exist
- **THEN** the composed context SHALL contain the constitution content first
- **AND** the config context content second
- **AND** both SHALL be preserved exactly as written

#### Scenario: Only config context present

- **WHEN** a config `context` field exists and no constitution is present
- **THEN** context injection SHALL behave exactly as it does today

#### Scenario: Only constitution present

- **WHEN** a constitution is present and the config omits the `context` field
- **THEN** the composed context SHALL contain the constitution content
- **AND** no empty config-context section SHALL be emitted
