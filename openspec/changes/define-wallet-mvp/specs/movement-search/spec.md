## ADDED Requirements

### Requirement: PostgreSQL financial authority and Meilisearch search projection
The system SHALL persist commands, financial movements, payment results and
idempotency records in PostgreSQL. It SHALL use Meilisearch as an eventually
consistent, rebuildable read model for movement and concept searches extending
HU-02 CA3, without using the index to authorize payments or determine balances.

#### Scenario: A user searches movements by concept
- **WHEN** an authenticated user submits a concept query using the Q-17-approved search contract
- **THEN** the backend searches Meilisearch within that user's permitted scope and returns only currently authorized movements

#### Scenario: An authoritative payment result is requested
- **WHEN** the payer consults a payment reference or retries an idempotency key
- **THEN** the result is resolved from PostgreSQL rather than inferred from search hits or index absence

### Requirement: Durable asynchronous indexing
The system SHALL record projection work in a PostgreSQL outbox atomically with
the source change and SHALL index asynchronously without coupling financial
commit success to Meilisearch availability. A projection checkpoint SHALL only
advance after confirmed completion of the asynchronous indexing task.

#### Scenario: Meilisearch is unavailable during payment completion
- **WHEN** a payment commits while Meilisearch is unavailable
- **THEN** its financial result remains committed, indexing work remains recoverable and search does not report the index error as a failed payment

#### Scenario: An event is duplicated or arrives out of order
- **WHEN** projection work for an existing movement is retried or reordered
- **THEN** no duplicate searchable movement is created and older data cannot overwrite a newer projection

### Requirement: Search privacy and ownership enforcement
The backend SHALL enforce session and ownership restrictions on each search
and returned movement. Browsers SHALL NOT access Meilisearch directly or receive
its credentials. Indexed fields SHALL be limited to the approved search contract;
tokens, KYC documents and idempotency keys MUST NOT be indexed.

#### Scenario: A user attempts to override the ownership filter
- **WHEN** a user supplies another owner identifier or a crafted search filter
- **THEN** the backend rejects the override or replaces it with server-derived scope and returns no unauthorized result, snippet, facet or count

#### Scenario: The index contains stale authorization data
- **WHEN** an indexed movement no longer belongs to the requesting user's authorized scope
- **THEN** current PostgreSQL authorization prevents its disclosure even before index correction completes

### Requirement: Rebuildable and observable read model
The system SHALL support reindexing from PostgreSQL, detect indexing failure or
lag and distinguish search unavailability from an empty result. Q-12/Q-17 SHALL
define freshness, search fields, concept semantics, filters and pagination before
construction; these rules MUST NOT be invented from the original HU text.

#### Scenario: The search index is lost
- **WHEN** operators rebuild Meilisearch from authoritative data
- **THEN** authorized search results are recovered without modifying ledger entries or replaying financial commands

#### Scenario: Search freshness cannot be met
- **WHEN** indexing lag exceeds the approved freshness policy
- **THEN** the API and UI expose the defined degraded state instead of implying a missing movement means a missing payment