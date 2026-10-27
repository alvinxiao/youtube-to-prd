# Proposed implementation issues

- PRD: [PRD.md](./PRD.md)
- Implementation plan: [implementation-plan.md](./implementation-plan.md)
- Original video: https://youtu.be/_bieksxg6oY?si=pJ_3xd_0Z04IgdoG
- Export status: draft, Markdown only. No GitHub issues created or authorized.
- Organization: five flat issues, sequenced by dependency. All implementation scopes and checks are Proposed. Evidence describes the supplied transcript, not inspected kit source or tested software.

## ISSUE-001: Establish vault records and incremental source discovery

**Type:** Task
**Priority:** High
**Dependencies:** None
**PRD requirements:** FR-001, FR-002, NFR-004
**Source evidence:** 3:23–4:02; 10:15–10:58; 13:44–14:14; 41:05–41:22, Observed in supplied transcript.

### Outcome

A local owner can initialize a vault and identify new or modified sources without losing existing content or calling a model unnecessarily.

### Scope

Choose the runtime and document the intended authority boundary. Define versioned schemas for sources, proposals, decisions, and run records. Create Raw, Review, Wiki, and protected control areas, with index and log conventions. Track source origin, hashes, and processing state. Require explicit provider configuration for outbound source transmission.

### Acceptance criteria

- [ ] Repeated initialization preserves all pre-existing content.
- [ ] New and changed source fixtures are detected; unchanged fixtures produce no new proposals or model calls.
- [ ] Source body bytes remain unchanged during discovery.
- [ ] With no provider configured, no source content leaves the device and diagnostics expose no secrets.
- [ ] Record the chosen runtime, supported OS, schema version, and permission design in implementation documentation.

### Verification

Compare a fixture vault before and after repeated setup. Modify one source and inspect the exact changed-source list. Use a call-counting provider stub and a network-denied configuration to verify zero outbound requests on the no-change path.

### Implementation notes

No existing application module is assumed. Keep manifests separate from immutable source bodies. Resolve package licensing and compatibility before dependency selection.

## ISSUE-002: Generate grounded changes and explicit conflict proposals

**Type:** Feature
**Priority:** High
**Dependencies:** ISSUE-001
**PRD requirements:** FR-003, FR-004, NFR-006
**Source evidence:** 13:08–14:39; 14:56–18:29; 41:22–42:02, Observed in supplied transcript.

### Outcome

The semantic worker proposes linked wiki changes with source references and preserves conflicting positions for human review.

### Scope

Define a provider-independent structured change contract. Implement bounded source selection, proposed target content, provenance, conflict comparisons, and deterministic structural validation before persistence. Produce proposals without granting the worker publication authority.

### Acceptance criteria

- [ ] Two related source fixtures yield linked candidate changes with source references.
- [ ] An opposing-source fixture produces an attributed comparison without silently overwriting either position.
- [ ] Missing citations, unknown source IDs, and malformed provider output are rejected with an actionable error.
- [ ] Two provider stubs pass the same contract suite; the selected real model passes a documented qualification fixture.
- [ ] Proposal generation changes no wiki files.

### Verification

Inspect actual proposals and compare wiki file hashes before and after generation. Run related-source, contradiction, missing-reference, and malformed-output fixtures. Check that citations identify real supplied sources; this does not prove their claims true.

### Implementation notes

Do not promise complete contradiction detection. Keep deterministic checks separate from model judgment. Do not copy membership kit assets.

## ISSUE-003: Add persistent Markdown review and revision decisions

**Type:** Feature
**Priority:** High
**Dependencies:** ISSUE-002
**PRD requirements:** FR-005, FR-006, FR-010, NFR-005
**Source evidence:** 23:01–24:24; 25:18–26:13; 30:34–31:40; 36:01–37:04, Observed in supplied transcript.

### Outcome

A reviewer can inspect changes, give feedback, and decide on exact revisions without publishing them immediately.

### Scope

Persist proposal revisions and feedback. Support individual and batch approval, rejection, deferral, and revision requests. Provide a readable inbox with counts and create/update comparisons. Record owner decisions in a protected store shared by all views.

### Acceptance criteria

- [ ] Proposals show source, target, revision, status, rationale, and exact candidate content or diff.
- [ ] Review actions and feedback survive restart and work using keyboard commands and Markdown.
- [ ] Approval records the exact payload digest and revision, but does not change wiki content.
- [ ] A content edit creates a new revision that requires a new decision.
- [ ] Inbox counts match stored states; deferred proposals can return to review.
- [ ] A model-written approval field does not count as an owner decision.

### Verification

Exercise every decision transition, restart the process, and read persisted records. Compare wiki hashes after approve, reject, defer, and revise. Complete the workflow without a mouse.

### Implementation notes

Use one decision store. A separate decision studio and Obsidian Bases dashboard are deferred. Ordinary proposal editing must not grant approval authority.

## ISSUE-004: Enforce approved publication and update provenance

**Type:** Feature
**Priority:** High
**Dependencies:** ISSUE-003
**PRD requirements:** FR-007, FR-008, NFR-001, NFR-002
**Source evidence:** 27:02–28:43; 37:11–38:23; 38:40–39:37, Observed in supplied transcript. Permission isolation and source/target hash checks are Proposed extensions.

### Outcome

Only an authorized, current approval can publish exact content and update the wiki's derived records.

### Scope

Implement a single protected publisher with explicit apply. Validate decisions, revisions, payload digests, source hashes, target base hashes, schemas, paths, and links. Update pages, index, provenance, applied status, and coverage only as part of a successful operation. Establish separate worker capabilities before claiming enforcement.

### Acceptance criteria

- [ ] Missing approval, stale revision, changed source, changed target, and malformed change sets fail before wiki writes.
- [ ] The worker cannot write gold pages or valid owner decisions through any exposed interface.
- [ ] Traversal, symlink escape, and injected source instructions cannot authorize or redirect publication.
- [ ] Successful publication produces exactly the approved content and consistent links, index, applied states, and coverage.
- [ ] Rejected, deferred, pending, and failed proposals do not count as successfully covered sources.

### Verification

Run a matrix of valid and invalid apply attempts, comparing actual filesystem bytes and records. Attempt direct worker writes and forged decisions under the deployed permissions. Verify references against the combined approved change set and existing wiki.

### Implementation notes

The threat boundary must be concrete. A prompt instruction or container with broad writable mounts is insufficient. Structural verification must not be described as factual verification.

## ISSUE-005: Make publication recoverable and validate the full workflow

**Type:** Task
**Priority:** High
**Dependencies:** ISSUE-004
**PRD requirements:** FR-009, NFR-003; end-to-end regression for FR-001–FR-010 and NFR-001–NFR-006
**Source evidence:** 33:36–33:53; 38:16–38:35, Observed in supplied transcript. Journaling, concurrency handling, and interruption tests are Proposed extensions.

### Outcome

The owner can recover from mistakes and interrupted runs while preserving unrelated vault edits.

### Scope

Add scoped Git checkpoints, a publication journal, single-writer coordination, recovery, and retry behavior. Document migration on a vault copy, legacy-content status, provider/privacy setup, deployment permissions, rollout, and recovery.

### Acceptance criteria

- [ ] Each successful changed run records a checkpoint and a completed journal entry.
- [ ] Failure injected between publication steps recovers or enters an explicit repair state before further application.
- [ ] Repeating apply does not duplicate changes; concurrent conflicting operations cannot silently overwrite one another.
- [ ] Recovery restores prior affected content while preserving unrelated edits; conflicting later edits stop automatic recovery.
- [ ] Scoped commits exclude unrelated private files and no remote push occurs by default.
- [ ] A disposable-vault demonstration completes ingestion, conflict review, revision, rejection, approval, explicit apply, provenance inspection, and recovery.

### Verification

Inject failures at each journaled write step. Retry after restart and compare exact content, proposal states, and history. Add an unrelated user edit before recovery and confirm that it remains. Run the complete acceptance matrix in the implementation plan and retain results.

### Implementation notes

Git history alone does not provide atomic writes or a backup of every vault file. Avoid destructive whole-vault resets as routine recovery. Do not enable live writes until these checks pass.

## GitHub publication mapping

| Local ID | GitHub issue URL | Status |
|---|---|---|
| ISSUE-001 | N/A | Markdown draft only |
| ISSUE-002 | N/A | Markdown draft only |
| ISSUE-003 | N/A | Markdown draft only |
| ISSUE-004 | N/A | Markdown draft only |
| ISSUE-005 | N/A | Markdown draft only |
