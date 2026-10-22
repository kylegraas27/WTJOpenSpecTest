# Spec Delta

## Purpose

The browser view through which a human drives the migration: choosing what to work
on, starting work, watching it run, and reviewing what came back. It is where the
operator makes the decisions the pipeline refuses to guess.

## ADDED Requirements

### Requirement: Operator selects a space before extracting

The console SHALL present the available spaces and SHALL require the operator to
choose one before an extraction can be started.

#### Scenario: Space chosen

- **GIVEN** two spaces are available
- **WHEN** the operator opens the console
- **THEN** both spaces are listed and the extraction control becomes usable only once one is selected

#### Scenario: No spaces available

- **GIVEN** no spaces are available to the configured credentials
- **WHEN** the operator opens the console
- **THEN** the console states that no spaces were found and does not offer to start an extraction

#### Scenario: Spaces cannot be loaded

- **GIVEN** the source rejects the configured credentials
- **WHEN** the operator opens the console
- **THEN** the console shows why the spaces could not be loaded, rather than an empty list

### Requirement: Operator starts an extraction and sees it running

The console SHALL start an extraction for the selected space on the operator's
explicit action, and SHALL show that the run is in progress until it finishes.

#### Scenario: Extraction started

- **GIVEN** the operator has selected Space A
- **WHEN** the operator starts the extraction
- **THEN** the console shows the run as in progress and does not appear idle

#### Scenario: Extraction not started automatically

- **GIVEN** the operator has selected Space A but taken no further action
- **WHEN** the console is displayed
- **THEN** no extraction has been started

### Requirement: Operator reviews the extracted projects

On completion, the console SHALL display the projects the run found, including
each project's position in the source folder hierarchy.

#### Scenario: Projects displayed with hierarchy

- **GIVEN** a run found a project nested inside folder "Team X"
- **WHEN** the operator views the finished run
- **THEN** the project is listed and its containing folder path is visible

#### Scenario: Run found no projects

- **GIVEN** a run completed and found no projects
- **WHEN** the operator views the finished run
- **THEN** the console states that the space contained no projects, distinct from showing an error

#### Scenario: Large inventory remains reviewable

- **GIVEN** a run found several hundred projects
- **WHEN** the operator views the finished run
- **THEN** every project is reachable in the view without the page becoming unusable

### Requirement: Operator is told why a run failed

When a run fails, the console SHALL show that it failed and the reason reported
for the failure.

#### Scenario: Failure reason displayed

- **GIVEN** a run failed because the source became unreachable
- **WHEN** the operator views the run
- **THEN** the console shows the run as failed and states that the source became unreachable

#### Scenario: Failed run shows no inventory

- **GIVEN** a run failed partway through the walk
- **WHEN** the operator views the run
- **THEN** no project inventory is presented for that run
