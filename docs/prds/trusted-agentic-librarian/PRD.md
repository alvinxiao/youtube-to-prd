# Trusted agentic librarian

## Source

- Original video: https://youtu.be/_bieksxg6oY?si=pJ_3xd_0Z04IgdoG
- Canonical video: https://www.youtube.com/watch?v=_bieksxg6oY
- Title: What Is An Agentic Librarian? Full Hermes & Obsidian Workflow (HITL)
- Creator: Wanderloots, Callum. The supplied automatic transcript sometimes renders the name incorrectly.
- Published: August 29, 2026, verified from the YouTube page metadata.
- Duration: 42:48, verified from page metadata.
- Evidence: full timestamped transcript supplied by the user, supplemented by page metadata and description. Captions could not be retrieved directly. No video frames, kit source, or running software were inspected.
- Context: standalone; repository integration assumptions are provisional. This is a requirements export, not an implementation or installation of the creator's kit.
- Status: draft for review. Product scope and implementation choices below are proposals, not approved decisions.

## Summary

**Proposed:** Build a local Markdown knowledge library that separates raw sources, review proposals, and approved wiki pages. Let an AI agent synthesize candidate changes while a deterministic publisher admits only the exact revision a human approved.

**Observed:** The tutorial presents three operating paths: direct ingestion, a review companion around the built-in skill, and a librarian with code-based validation. It recommends choosing the simplest path that meets the user's needs, rather than requiring all three paths, at 4:10–7:09.

## Problem and goals

**Observed, 0:09–0:35 and 17:11–18:29:** An agent can propagate false or conflicting information across a wiki before a person can intervene. Detecting a contradiction after writing it does not provide an admission boundary.

**Inferred:** The primary user is a person curating a knowledge vault who wants synthesis assistance without surrendering editorial control.

**Proposed success criteria:** In acceptance fixtures, all published changes reference an approved revision; pending, rejected, and deferred proposals change no compiled pages. Every published page has source references. An unchanged rerun makes zero model calls and zero duplicate writes. A recovery drill restores the previous published content without discarding unrelated user work.

## Non-goals

**Proposed:** Exclude autonomous truth certification, automatic promotion of agent memories, a replacement for Obsidian, hosted multi-user collaboration, and a production research crawler. Exclude implementing all three operating paths, a second visual review application, scheduled automation, and molecular Zettelkasten from the initial release. Do not reproduce or redistribute the membership kit without permission.

## Tutorial assessment

| Path | Transcript evidence | Assessment |
|---|---|---|
| Built-in LLM Wiki | 10:15–11:23 initialization; 12:49–14:39 synthesis, hashes, metadata, links; 14:56–17:04 contested concepts | Observed: fast ingestion can preserve disagreement, but changes reach the wiki before review. Useful as a comparison baseline. |
| Review companion | 19:36–21:46 wrapper; 22:25–24:24 durable proposals and feedback; 25:18–26:50 separate decision and application | Observed: review adds editorial control without editing the upstream skill. At 27:02–27:28 the narrator explains that instructions alone cannot prove enforcement. |
| Deterministic librarian | 28:18–28:43 preflight and validation; 35:38–38:45 proposals, decisions, checked revisions, Git | Observed: code controls admission and records history. Source code was not inspected, so bypass resistance and crash behavior remain unverified. |

**Proposed:** Use the third path as the reference MVP design, with Markdown review as its first interface. Preserve the companion's explicit decision/application separation. This is an independent design, not a claim about the kit's internals.

**Observed, 40:48–41:57:** Agent memory and curated world knowledge serve different purposes. The semantic engine can be replaced, but each model must handle context, structured output, filenames, and citations adequately.

**Proposed:** Deterministic validation checks structure, authorization, and exact content. Human approval expresses a curatorial judgment; neither guarantees factual truth.

## Functional requirements

“Observed” means described in the supplied transcript, not independently exercised. Proposed criteria make the behavior testable without claiming the tutorial demonstrated every edge case. N/A means a new proposal without tutorial evidence.

| ID | Requirement | Acceptance criteria | Evidence type | Source timestamp |
|---|---|---|---|---|
| FR-001 | Initialize separate Raw, Review, and Wiki areas with a schema, index, and action log. | Initialization creates documented areas; repeating it preserves existing content. Idempotence is Proposed. | Observed; Proposed criterion | 3:23–4:02; 10:15–10:58 |
| FR-002 | Discover local source files and record source identity, content hash, and processing state. | New or changed fixtures become candidates; an unchanged rerun creates no duplicate proposal and makes no model call. Do not overwrite raw body content, a Proposed constraint. | Observed; Proposed constraint | 11:28–12:44; 13:44–14:14; 41:05–41:22 |
| FR-003 | Generate source-grounded, interlinked Markdown changes. | Candidate content identifies its sources and proposed targets; a fixture with two related sources produces linked concepts. Missing source references fail validation, a Proposed check. | Observed; Proposed criterion | 13:08–14:39; 35:38; 37:40–38:11 |
| FR-004 | Surface conflicting positions for review without silently choosing a winner. | Opposing-source fixture produces an attributed comparison and proposed contested metadata. Wiki content remains unchanged until approval. Detection completeness is not guaranteed. | Observed | 14:56–18:29 |
| FR-005 | Persist review proposals with revision, status, source, target, proposed content, and human feedback. | Restarting retains proposals; a reviewer can inspect create/update changes and submit feedback; revised content receives a new revision. | Observed | 23:18–24:24; 36:01–36:47 |
| FR-006 | Support approval, rejection, deferral, and revision, individually and in a batch. Record decisions separately from application. | Each action persists; approve does not itself publish; explicit apply publishes only selected approved proposals. A revised proposal requires a fresh decision, a Proposed rule. | Observed; Proposed revision rule | 23:01–23:12; 25:18–26:13; 36:36–37:17 |
| FR-007 | Validate proposals in code and publish only the approved revision. | Unapproved, malformed, or revision-mismatched proposals fail without wiki writes. Published bytes match the approved payload. Changes to source or target after approval invalidate application, a Proposed extension. | Observed; Proposed checks | 6:18–7:03; 27:15–28:43; 38:16–38:23 |
| FR-008 | Update the wiki, links, index, proposal status, and source coverage after successful publication. | A successful fixture links its sources, appears in the index, marks its proposal applied, and reports coverage. Failed or rejected candidates do not count as covered. Coverage semantics are Proposed. | Observed; Proposed criterion | 26:26–26:50; 37:17–38:51 |
| FR-009 | Record Git checkpoints for librarian changes and provide a documented recovery procedure. | Each successful changed run has a recoverable checkpoint; a drill restores prior wiki content and preserves unrelated edits. Failure-recovery details are Proposed. | Observed; Proposed criterion | 33:36–33:53; 38:16–38:35 |
| FR-010 | Provide a Markdown review inbox and summary of pending changes and source coverage. | Counts match stored proposal states and source records after restart and apply. Decisions remain usable in Obsidian or a text editor. | Observed; Proposed criterion | 30:34–31:40; 33:11–33:29; 38:40–39:08 |

## Nonfunctional requirements

| ID | Requirement | Verification | Evidence type |
|---|---|---|---|
| NFR-001 | Restrict the synthesis worker from writing the approved wiki or decision records; use one publisher for mutation. | Attempt direct writes and forged approvals from the worker; both fail under configured permissions. Audit CLI/API paths for the same authorization checks. | Proposed, motivated by 27:02–28:43 |
| NFR-002 | Reject path traversal, symlink escapes, malformed payloads, and source instructions that attempt to authorize publication. | Adversarial fixtures cannot write outside permitted paths or manufacture a valid human decision. | Proposed, motivated by 21:03–21:28 |
| NFR-003 | Make publication recoverable and safe to retry; serialize conflicting writes. | Inject failure between writes, restart, and reconcile to one recorded result without duplicate content. Concurrent apply attempts cannot overwrite each other. | Proposed |
| NFR-004 | Keep vault files local by default and require explicit provider configuration before source content leaves the device. | With no provider configured, no outbound synthesis request occurs; diagnostics omit secrets and private source bodies. | Proposed |
| NFR-005 | Keep all review actions available through readable Markdown and a keyboard-operable command interface. | Complete create, inspect, revise, reject, defer, approve, and apply without a mouse or color-only cues. | Proposed |
| NFR-006 | Make semantic providers replaceable through a structured proposal contract. | Two fake providers pass the same contract tests; malformed output fails before publication. Qualify a real model with citations and context fixtures before use. | Proposed, motivated by 41:22–42:02 |

## UX and data considerations

**Proposed states:** needs_review → approved → applied; needs_review → rejected or deferred; revision_requested → needs_review with a new revision. Reopening a deferred proposal returns it to review. Validation errors leave content unpublished and show a corrective message. Approval, validation result, and apply result are distinct records.

**Proposed records:** Source has stable ID, origin, hash, and coverage state. Proposal has ID, revision, source references, source hashes, target paths and base hashes, exact changes, rationale, and feedback. Decision has proposal ID, revision, payload digest, actor, and time. Run has inputs, validation results, affected paths, checkpoint, and outcome. Index and dashboard are derived views, not separate decision stores.

**Proposed authority rule:** Editing a proposal or adding feedback does not approve it. A local owner action records approval through the publisher. Synthesis and review run with separate capabilities. Manual wiki edits remain possible, but invalidate affected pending approvals. Rejection and deferral preserve evidence without changing gold content.

**Observed:** The tutorial also shows a decision studio over the same records at 31:29–31:40 and 33:11–33:29. **Proposed:** Defer that second interface until Markdown review is reliable.

## Dependencies and constraints

**Proposed:** Use a local filesystem, Markdown, Git, one semantic provider, and an optional Obsidian interface. Language, packages, supported OS versions, file size limits, and exact Hermes API/configuration remain Unresolved. No existing application architecture was inspected. The tutorial's environment-variable names are not treated as a verified installation contract.

**Observed, 7:20–7:33 and 42:28–42:38:** The companion is offered through an article, while the full kit is membership material. **Unresolved:** Redistribution terms, dependency licenses, and compatibility with current Hermes and Obsidian versions require verification before implementation. This export does not assume access to those assets.

## Risks and alternatives

- **Inferred:** Human review can become a bottleneck or a rubber stamp. **Proposed:** Show concise changes and evidence, with deferral and selective approval.
- **Inferred:** Correct citations can still support mistaken claims. **Proposed:** Preserve uncertainty, contested positions, and raw sources; never label validation as truth certification.
- **Inferred:** A skill wrapper can be bypassed if the worker retains broad write access. **Proposed:** Enforce capabilities outside the model. Docker alone does not prove this boundary.
- **Inferred:** Git can include private notes and does not cover untracked content automatically. **Proposed:** Keep history local, scope commits to intended files, and test backups separately.
- **Proposed alternatives:** Start with a manual companion workflow for lower setup cost, or use retrieval directly over sources to avoid maintaining a compiled wiki. The latter comparison is discussed at 16:05–16:33.

## Open questions and decisions

| Item | Status | Impact |
|---|---|---|
| Standalone Markdown export; no GitHub issue creation | Approved by user | Governs this delivery. |
| Deterministic librarian MVP with Markdown review | Proposed | Confirm before implementation; companion-only delivery would reduce scope and enforcement. |
| Execution language, OS, provider, Hermes integration | Unresolved | Blocks concrete dependency selection and runtime setup. |
| Local owner approval and separate worker permissions | Proposed | Choose the actual process/identity boundary before claiming bypass resistance. |
| Existing vault migration and manual-edit policy | Unresolved | Inventory paths, metadata, and permissions before touching a live vault. |
| Automatic promotion and scheduled runs | Proposed exclusion | Tutorial permits policy exceptions at 28:31–28:43; this draft requires review for all publication. |

## Definition of done

**Proposed:** All requirements pass their acceptance checks in a disposable vault. A reviewer can ingest two sources, inspect conflicting proposals, revise one, reject another, explicitly approve and apply a revision, inspect provenance and coverage, and recover the prior wiki through a documented checkpoint. Unapproved and stale content cannot publish. Runtime choices and unresolved deployment boundaries are resolved before production use.
