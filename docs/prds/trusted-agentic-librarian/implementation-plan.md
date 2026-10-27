# Implementation plan: trusted agentic librarian

- PRD: [PRD.md](./PRD.md)
- Issues: [issues.md](./issues.md)
- Original video: https://youtu.be/_bieksxg6oY?si=pJ_3xd_0Z04IgdoG
- Mode: standalone. Status: proposed, not implementation authorization.
- Evidence: user-supplied full transcript and verified video metadata. No code, kit, frames, or live tutorial workflow were inspected.

## Existing architecture

**Observed in transcript:** Raw, schema, wiki, log, and index are initialized at 10:15–10:58. Durable review proposals appear at 22:25–24:24. Code-based preflight, validation, decisions, and Git checkpoints are described at 28:18–28:43 and 33:36–38:35.

**Unresolved:** No host application has been selected. Existing files in this workspace are not assumed to implement this product. All paths and interfaces below are Proposed standalone design choices.

## Proposed design

**Proposed layout:** `raw/` stores source bodies; `review/` stores editable proposals and feedback; `wiki/` stores published pages; a protected control area stores decisions, source manifests, run journals, and checkpoints. A documented schema governs the records. Markdown serves as the portable review interface, with derived index and inbox summaries.

**Proposed flow:** Discover changed sources → preflight → bounded synthesis → deterministic validation → persist proposal → human decision → explicit apply → revalidation → checkpoint and publish → rebuild derived summaries.

The synthesis worker reads permitted sources and existing wiki content and produces structured changes. It cannot publish or record human decisions. The publisher checks proposal revision, payload digest, source hashes, target base hashes, authorization, and target paths. It resolves permitted links against existing pages and pages in the same approved batch. Block unresolved internal links under the initial Proposed policy.

The local review command records a decision on exact content, rather than accepting an agent-written `approved` field as authority. A proposal edit creates a new revision and invalidates the previous decision. Approval remains distinct from application, reflecting 25:18–25:38.

Use a single publisher per vault. Stage writes and keep a recovery journal before mutating multiple files. Git checkpoints provide history; they do not make filesystem writes atomic. On interruption, recover or stop with an explicit repair state before another apply. Do not silently overwrite unrelated changes.

Keep provider calls behind a structured proposal contract. Skip synthesis when hashes and pending work show no change, consistent with 41:05–41:22. Do not claim a specific runtime command or package exists yet.

## Milestones and dependencies

| Milestone | Deliverable and exit check | Requirements | Issue | Depends on |
|---|---|---|---|---|
| M1 | Select runtime and authority boundary; initialize disposable vault, schemas, source manifest, and no-change discovery. Reinitialization preserves content. | FR-001, FR-002, NFR-004 | ISSUE-001 | None |
| M2 | Provider contract produces grounded changes and conflict proposals without touching wiki files. Test valid and malformed responses. | FR-003, FR-004, NFR-006 | ISSUE-002 | M1 |
| M3 | Durable inbox, revisions, feedback, and owner decisions work after restart. Approval alone leaves wiki unchanged. | FR-005, FR-006, FR-010, NFR-005 | ISSUE-003 | M2 |
| M4 | Protected publisher rejects unauthorized or stale changes, applies exact content, and updates provenance and coverage. | FR-007, FR-008, NFR-001, NFR-002 | ISSUE-004 | M3 |
| M5 | Checkpoints, interruption recovery, retry safety, and documented rollout pass a complete disposable-vault drill. | FR-009, NFR-003 | ISSUE-005 | M4 |

**Proposed sequencing:** Establish records and authority before provider integration. Implement review before publication. Do not enable production writes until M5 passes. Estimates depend on runtime and existing vault constraints, so no unsupported dates are assigned.

## Migration, rollout, and rollback

**Proposed migration:** Inventory a copy of the target vault and map existing raw sources, wiki pages, and metadata to the new schema. Preserve original paths where practical. Mark existing wiki content as legacy/unreviewed rather than retroactively approved. Produce a migration report before changing originals. Do not overwrite existing control files or assume the creator's kit schema.

**Proposed rollout:** Start in a disposable vault with model fixtures, then a real provider using non-sensitive sources. Run against a copied vault in proposal-only mode. Review diffs and privacy settings before enabling the protected publisher on live content. External Git remotes and automatic ingestion remain outside this release.

**Proposed recovery:** Before publication, record the affected paths and their prior content and create a scoped checkpoint. After successful publication, record a completion checkpoint and journal outcome. On failure, reconcile only those paths after checking for later edits. If user edits conflict, stop and show a recovery diff. Exercise recovery before release; do not use a whole-vault destructive reset as the default procedure.

## Verification strategy

All checks below are Proposed work for implementation, not claims of completed software tests.

| Check | Expected result |
|---|---|
| Initialization and source discovery | Repeating setup preserves files; modified source is detected; unchanged run makes zero provider calls. |
| Semantic fixtures | Related sources yield linked proposals; opposing sources retain both attributions; absent citations or invalid structured output fail validation. |
| Review lifecycle | Revision, rejection, deferral, and batch decisions survive restart. No decision alone mutates compiled pages. |
| Approval binding | Alter payload, raw source, or target after approval; apply fails without wiki writes and requests fresh review. |
| Authority and path attacks | Worker cannot publish or forge owner decisions; traversal, symlink escape, and injected source instructions do not gain authority. |
| Publication | Approved exact bytes, links, index, applied status, and source coverage agree; rejected content remains outside the wiki. |
| Concurrency and interruption | Overlapping apply operations serialize; injected failures recover without duplicate writes or false completed status. |
| Git recovery | Recover prior published content while preserving unrelated edits; private files outside the change set are not added accidentally. |
| UX and provider qualification | Complete the workflow using keyboard commands and Markdown; qualify the chosen real model for context, filenames, and citation handling. |

Visual parity with the tutorial's decision studio is outside scope. Before implementing optional Obsidian Bases views, verify the supported file format and version against official documentation. No screenshots were analyzed for this export.

## Unknowns

- **Unresolved:** Runtime, OS, provider, storage limits, and package licensing. Resolve in M1 before selecting dependencies.
- **Unresolved:** How local processes and filesystem identities isolate synthesis from owner approval. Resolve before M4 implementation.
- **Unresolved:** Actual Hermes skill APIs, environment configuration, and compatibility. Inspect official documentation and chosen versions during implementation.
- **Unresolved:** Existing vault schema and manual edits. Inspect a copy before migration.
- **Proposed exclusions:** Decision studio, multi-bot coordination, memory integration, scheduled runs, automatic promotion, and Zettelkasten expansion. These can become later requirements without weakening approval controls.
