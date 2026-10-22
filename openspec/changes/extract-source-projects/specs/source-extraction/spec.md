# Spec Delta

## Purpose

Discovers what exists in a source work-management system: verifying a connection,
listing the spaces available to an operator, walking a chosen space's folder tree
to find every project within it, and preserving each discovery as a durable run
record that later migration stages can read.

## ADDED Requirements

### Requirement: Source connection verification

The system SHALL verify the configured source credentials and base URL before
beginning any extraction, and SHALL refuse to begin when verification fails.

#### Scenario: Valid credentials accepted

- **GIVEN** the operator has configured a reachable source base URL and a valid token
- **WHEN** a connection check is performed
- **THEN** the check succeeds and extraction is permitted to begin

#### Scenario: Rejected credentials

- **GIVEN** the configured token is expired or invalid
- **WHEN** a connection check is performed
- **THEN** the check fails, reports that the source rejected the credentials, and no extraction begins

#### Scenario: Source unreachable

- **GIVEN** the configured base URL does not resolve or refuses the connection
- **WHEN** a connection check is performed
- **THEN** the check fails, reports the source as unreachable, and no extraction begins

### Requirement: Space listing

The system SHALL report the spaces that the configured credentials are permitted
to see, so the operator can choose one to extract.

#### Scenario: Spaces returned

- **GIVEN** the credentials can see Space A and Space B
- **WHEN** the operator requests the list of spaces
- **THEN** both spaces are returned, each with a stable identifier and a display name

#### Scenario: No spaces visible

- **GIVEN** the credentials can see no spaces at all
- **WHEN** the operator requests the list of spaces
- **THEN** an empty list is returned and this is reported as a successful result, not an error

#### Scenario: Space missing a display name

- **GIVEN** the source returns a space with an identifier but no name
- **WHEN** the operator requests the list of spaces
- **THEN** the request fails and identifies the offending space, rather than substituting a placeholder name

### Requirement: Recursive project discovery

The system SHALL discover every project contained in the selected space, at any
depth of folder nesting, including projects nested inside other projects.

#### Scenario: Projects found at multiple depths

- **GIVEN** Space A contains a project at its root, and a folder containing two further projects
- **WHEN** the space is extracted
- **THEN** all three projects appear in the resulting inventory

#### Scenario: Space containing no projects

- **GIVEN** Space A contains only folders, with no projects at any depth
- **WHEN** the space is extracted
- **THEN** the run completes successfully with an inventory of zero projects

#### Scenario: Deeply nested project

- **GIVEN** Space A contains a project nested ten folders deep
- **WHEN** the space is extracted
- **THEN** that project appears in the inventory and no depth limit truncates the walk

### Requirement: Project hierarchy recorded

Each discovered project SHALL record the ordered sequence of folder names leading
from the space root to that project, so later stages can reproduce the hierarchy.

#### Scenario: Nested project records its ancestry

- **GIVEN** a project sits inside folder "Team X", which sits inside folder "Delivery"
- **WHEN** the space is extracted
- **THEN** that project records the ordered ancestry "Delivery" then "Team X"

#### Scenario: Top-level project records empty ancestry

- **GIVEN** a project sits directly in the space root
- **WHEN** the space is extracted
- **THEN** that project records an empty ancestry

### Requirement: Repeat-visit and cycle safety

The system SHALL include each discovered project exactly once in a run's
inventory, and SHALL terminate even when the source's folder graph contains a
cycle.

#### Scenario: Project reachable by two paths

- **GIVEN** one project is reachable through two different folders in the same space
- **WHEN** the space is extracted
- **THEN** the project appears exactly once in the inventory

#### Scenario: Folder cycle

- **GIVEN** folder "A" reports folder "B" as a child and folder "B" reports folder "A" as a child
- **WHEN** the space is extracted
- **THEN** the walk terminates and the run completes rather than looping indefinitely

### Requirement: Complete paginated retrieval

The system SHALL retrieve every page of results the source offers for a given
listing, and SHALL NOT treat a partial page sequence as a complete result.

#### Scenario: Results span several pages

- **GIVEN** the source returns the space's contents across three pages
- **WHEN** the space is extracted
- **THEN** records from all three pages appear in the inventory

#### Scenario: Results fit in one page

- **GIVEN** the source returns all of the space's contents in a single page with no continuation
- **WHEN** the space is extracted
- **THEN** the inventory contains those records and no further page is requested

#### Scenario: Pagination fails midway

- **GIVEN** the source returns the first page, then fails when the second page is requested
- **WHEN** the space is extracted
- **THEN** the run fails and the partial results are not reported as a complete inventory

### Requirement: Deterministic inventory ordering

Given identical source content and identical configuration, the system SHALL
produce an inventory whose contents and ordering are identical on every run.

#### Scenario: Repeated extraction matches

- **GIVEN** Space A is unchanged between two extractions using the same configuration
- **WHEN** the space is extracted twice
- **THEN** the two inventories list the same projects in the same order

#### Scenario: Source returns records in a different order

- **GIVEN** the source returns the same projects in a different sequence on a later call
- **WHEN** the space is extracted again
- **THEN** the resulting inventory ordering is unchanged from the earlier run

### Requirement: Malformed project records fail the run

The system SHALL fail a run when a project record is missing information required
to identify it, rather than guessing a value or silently skipping the record.

#### Scenario: Project missing an identifier

- **GIVEN** the source returns a project record with no identifier
- **WHEN** the space is extracted
- **THEN** the run fails and reports where the unusable record was encountered

#### Scenario: Project missing a name

- **GIVEN** the source returns a project record with an identifier but no name
- **WHEN** the space is extracted
- **THEN** the run fails and identifies the offending project by its identifier

#### Scenario: Two projects share an identifier

- **GIVEN** the source returns two project records with the same identifier
- **WHEN** the space is extracted
- **THEN** the run fails and reports the duplicated identifier

### Requirement: Durable run records

The system SHALL persist each extraction as a run record that remains readable
after the system restarts, and SHALL NOT overwrite or alter a previous run.

#### Scenario: Re-extracting a space creates a new run

- **GIVEN** Space A has already been extracted once
- **WHEN** Space A is extracted again
- **THEN** a second run record is created and the first remains readable and unchanged

#### Scenario: Run survives restart

- **GIVEN** a run completed before the system was restarted
- **WHEN** that run is requested after the restart
- **THEN** its inventory is returned exactly as it was recorded

#### Scenario: Run record unreadable

- **GIVEN** a stored run record has been corrupted or truncated
- **WHEN** that run is requested
- **THEN** the request fails and identifies the run as unreadable, rather than returning a partial inventory

### Requirement: Failed runs are distinguishable from complete runs

A run that did not finish SHALL be recorded as failed, carry the reason it failed,
and SHALL NOT be readable as a completed inventory.

#### Scenario: Extraction interrupted by source failure

- **GIVEN** the source stops responding partway through a walk
- **WHEN** the run ends
- **THEN** the run is recorded as failed with the reason, and its partial contents are not served as an inventory

#### Scenario: Completed run reports success

- **GIVEN** a walk finished without error
- **WHEN** the run ends
- **THEN** the run is recorded as complete and its inventory is readable
