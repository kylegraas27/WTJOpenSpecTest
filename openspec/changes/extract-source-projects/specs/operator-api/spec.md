# Spec Delta

## Purpose

The programmatic surface the operator console drives: it exposes the pipeline's
long-running work as requests a browser can make, reporting progress and results
without the caller needing to know how the underlying stages run.

## ADDED Requirements

### Requirement: Spaces are retrievable

The interface SHALL provide the list of spaces available to the configured source
credentials, so the operator can choose one.

#### Scenario: Spaces available

- **GIVEN** the configured credentials can see two spaces
- **WHEN** the caller requests the available spaces
- **THEN** both are returned with their identifiers and display names

#### Scenario: Source rejects the credentials

- **GIVEN** the configured source token is invalid
- **WHEN** the caller requests the available spaces
- **THEN** the response reports that the source rejected the credentials, and is distinguishable from the tool having no spaces to show

#### Scenario: No spaces available

- **GIVEN** the credentials can see no spaces
- **WHEN** the caller requests the available spaces
- **THEN** an empty list is returned as a successful response

### Requirement: Extraction is startable and returns a handle

The interface SHALL accept a request to extract a named space and SHALL
immediately return an identifier for that run, without waiting for the walk to
finish.

#### Scenario: Extraction accepted

- **GIVEN** the operator selects Space A
- **WHEN** an extraction of Space A is requested
- **THEN** the response returns a run identifier that can be used to follow the run

#### Scenario: Unknown space requested

- **GIVEN** the operator requests a space identifier the source does not recognise
- **WHEN** an extraction is requested
- **THEN** no run is created and the response reports the space as unknown

#### Scenario: Extraction requested before configuration

- **GIVEN** no source base URL or token has been configured
- **WHEN** an extraction is requested
- **THEN** no run is created and the response reports which configuration is missing

### Requirement: Run progress is observable

The interface SHALL report whether a given run is in progress, complete, or
failed, while the run is still executing and after it finishes.

#### Scenario: Run in progress

- **GIVEN** a run is still walking the folder tree
- **WHEN** its status is requested
- **THEN** the response reports the run as in progress

#### Scenario: Run finished

- **GIVEN** a run has completed successfully
- **WHEN** its status is requested
- **THEN** the response reports the run as complete

#### Scenario: Unknown run requested

- **GIVEN** a run identifier that was never issued
- **WHEN** its status is requested
- **THEN** the response reports the run as not found

### Requirement: Completed run results are retrievable

The interface SHALL return the project inventory of a completed run, including
each project's recorded ancestry.

#### Scenario: Inventory returned

- **GIVEN** a run completed and found three projects
- **WHEN** the run's results are requested
- **THEN** all three projects are returned with their identifiers, names, and ancestry

#### Scenario: Results requested while still running

- **GIVEN** a run is still in progress
- **WHEN** its results are requested
- **THEN** no partial inventory is returned and the response reports the run as not yet complete

#### Scenario: Empty inventory returned

- **GIVEN** a run completed and found no projects
- **WHEN** the run's results are requested
- **THEN** an empty inventory is returned as a successful response, distinguishable from a failed run

### Requirement: Failure reasons reach the caller

When a run fails, the interface SHALL report the reason in a form the console can
display to the operator, rather than an unexplained failure.

#### Scenario: Source failure surfaced

- **GIVEN** a run failed because the source stopped responding
- **WHEN** the run's status is requested
- **THEN** the response reports the run as failed and conveys that the source became unreachable

#### Scenario: Malformed data failure surfaced

- **GIVEN** a run failed because a project record was missing an identifier
- **WHEN** the run's status is requested
- **THEN** the response reports the run as failed and conveys that unusable source data was encountered

### Requirement: Credentials are never disclosed

No response from the interface SHALL contain the configured source token, and no
log entry SHALL record it.

#### Scenario: Token absent from error responses

- **GIVEN** a request fails because the source rejected the configured token
- **WHEN** the error response is produced
- **THEN** the response describes the rejection without reproducing the token value

#### Scenario: Token absent from run records

- **GIVEN** a run has completed
- **WHEN** its stored record and its responses are inspected
- **THEN** neither contains the configured token
