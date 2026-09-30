# Viral Producer — Implementation Plan

## 1. Objective

Build one AI content producer for English-language Facebook Reels trivia that:

- follows the editorial DNA derived from the reference dataset;
- targets the US primarily, followed by Canada, the UK, and Australia;
- researches and verifies every fact before use;
- prevents reuse of the same underlying fact, including paraphrases;
- stores durable production state in GitHub;
- produces clean scripts that are ready to copy into Facebook content.

## 2. Version 1 Architecture

Version 1 uses:

- one private ChatGPT Plugin with a reusable instruction skill when Plugin Creator is available;
- an existing Custom GPT as a compatibility path when GPT Builder remains available;
- GitHub as the permanent source of truth;
- a GitHub app, plugin, or workspace connection for repository reads and writes;
- Web Search for current research and verification;
- no external database, backend, custom MCP server, or Action unless testing proves it is required.

Fallback order:

1. Private Plugin with the available GitHub connection.
2. Private Custom GPT with GitHub App where GPT Builder remains available.
3. Plugin with a restricted custom MCP server.
4. Legacy Custom GPT Action or dedicated backend only when supported and necessary.

## 3. Final Repository Structure

    viral-producer/
    ├── README.md
    ├── plan.md
    ├── dataset-reference.md
    ├── system/
    │   ├── gpt-instructions.md
    │   ├── content-dna.md
    │   └── data-contract.md
    ├── data/
    │   ├── production-state.json
    │   ├── active-drafts.jsonl
    │   ├── facts/
    │   │   ├── animals-nature.jsonl
    │   │   ├── body-science.jsonl
    │   │   ├── food-home.jsonl
    │   │   ├── geography-history.jsonl
    │   │   ├── inventions-records.jsonl
    │   │   └── practical.jsonl
    │   └── posts/
    │       └── YYYY-MM.jsonl
    ├── output/
    │   └── ready-to-post.md
    ├── docs/
    │   └── gpt-installation.md
    └── tests/
        └── acceptance-tests.md

The monthly posts directory is populated when the first post is marked as posted. Permanent one-file-per-post storage is not used.

## 4. Authoritative Design Decisions

### Content format

- Standard post: exactly six facts.
- Hook: "Did you know?"
- CTA: "Enjoyed these facts? Like the video and follow for more!"
- Ideal fact length: 12–15 English words.
- Acceptable fact length: 11–18 words when accuracy or natural phrasing requires it.
- Final copy uses simple conversational American English.
- The reference dataset is a style reference, not a factual authority.

### Audience and topics

Country weighting for country-specific posts:

- US: 60%
- Canada: 15%
- UK: 15%
- Australia: 10%

Default topic weighting:

- Geography and history: 30%
- Animals and nature: 20%
- Human body and everyday science: 15%
- Food and household knowledge: 15%
- Inventions, firsts, and unusual records: 15%
- Safe practical knowledge: 5%

### Identifiers

Use six-digit permanent IDs:

- Posts: P-000001
- Facts: F-000001

IDs are never reused, including after rejection.

### Fact lifecycle

Logical lifecycle:

    candidate → reserved → published

Storage behavior:

- candidate exists only during research;
- reserved facts live inside data/active-drafts.jsonl;
- published facts live in data/facts/*.jsonl;
- blocked facts may be stored in the fact index to prevent unsafe, false, or unverifiable claims from being proposed again.

### Post lifecycle

    draft → approved → ready → posted

- Active records live in data/active-drafts.jsonl.
- Ready copy is rendered into output/ready-to-post.md.
- Posted records move to data/posts/YYYY-MM.jsonl.
- Posted records are removed from active drafts and the ready queue.

### Duplicate prevention

Duplicate checks use:

- human-readable claim_signature;
- canonical claim;
- subject, relationship, and result;
- semantic comparison, not exact wording alone;
- reserved facts in active drafts;
- published and blocked fact indexes.

Paraphrases, translations, reordered wording, and changed surface copy do not make an old fact new.

### Write safety

Version 1 uses single-writer mode.

Every write must:

- fetch the current Git blob SHA;
- update using that SHA;
- stop and re-read after a conflict;
- avoid concurrent writes to the same path;
- be idempotent when retried;
- report success only after GitHub confirms the write.

## 5. Implementation Stages

### Stage 1 — Repository initialization

Status: COMPLETE

Deliverables:

- initialize the default branch;
- add README.md;
- upload dataset-reference.md;
- create branch build/viral-content-gpt;
- keep later implementation changes on the branch;
- open a pull request after the implementation is ready for review.

Completed commits:

- fdd201b — Initialize Viral Producer repository
- cb9e790 — Add viral trivia reference dataset

### Stage 2 — Editorial DNA

Status: COMPLETE

Deliverable:

    system/content-dna.md

Required coverage:

- target audiences and country distribution;
- topic rotation;
- six-fact structure;
- language and sentence construction;
- surprise mechanisms;
- fact ordering and emotional mix;
- source standards and freshness checks;
- originality and duplicate prevention;
- medical, safety, survival, and misinformation boundaries;
- 12-point quality score and final checklist.

Completed commit:

- 287581f — Define viral trivia content DNA

### Stage 3 — Data contract

Status: COMPLETE

Deliverable:

    system/data-contract.md

Required coverage:

- post and fact ID formats;
- JSON and JSONL schemas;
- topic routing;
- source object structure;
- claim signatures and canonical claims;
- draft, fact, and post records;
- lifecycle operations;
- monthly archives;
- ready queue rendering;
- duplicate checks;
- idempotency, SHA conflicts, recovery, and consistency audits.

Completed commit:

- b05855e — Define repository data contract and lifecycle
- ea2e172 — Fix approval transition ordering

### Stage 4 — Initial database

Status: COMPLETE

Create:

    data/production-state.json
    data/active-drafts.jsonl
    data/facts/animals-nature.jsonl
    data/facts/body-science.jsonl
    data/facts/food-home.jsonl
    data/facts/geography-history.jsonl
    data/facts/inventions-records.jsonl
    data/facts/practical.jsonl
    output/ready-to-post.md

Acceptance criteria:

- every JSON file is valid;
- all JSONL files are valid and initially empty;
- next post and fact numbers both start at 1;
- revision starts at 0;
- single_writer_mode is true;
- the ready queue contains no post;
- every fact category is ready for use.

Completed commit:

- bbdd604 — Initialize production data stores

### Stage 5 — Main GPT instructions

Status: COMPLETE

Create:

    system/gpt-instructions.md

The instructions must define:

- GPT identity and responsibilities;
- authoritative repository and branch;
- required files to read;
- mandatory GitHub and Web Search usage;
- default topic selection;
- collection of at least 18 candidates per six-fact post;
- verification and source capture;
- canonicalization and semantic deduplication;
- writing, ordering, scoring, and saving;
- approval, rejection, revision, and posting;
- consistency recovery;
- prohibition against claiming a write succeeded before confirmation.

Core generation flow:

    Read state
    → Choose topic
    → Research candidates
    → Verify claims
    → Canonicalize
    → Check duplicates
    → Select six
    → Write script
    → Score script
    → Reserve facts
    → Save draft

Completed commits:

- b7c63c6 — Add main GPT production instructions
- 020b241 — Align GPT approval flow with data contract

### Stage 6 — Command interface

Status: COMPLETE

The GPT must recognize natural-language commands such as:

    Create 5 posts.
    Create 3 posts about Australia.
    Show P-000001.
    Replace fact 4 in P-000001.
    Approve P-000001.
    Reject P-000001.
    Show the next ready-to-post script.
    Mark P-000001 as posted.
    Audit the fact database.

Indonesian production requests must still produce final English scripts unless explicitly requested otherwise.

Implementation requirements:

- resolve natural Indonesian and English by intent rather than exact wording;
- normalize post IDs, supported countries, and topic synonyms;
- distinguish read-only inspection from state-changing commands;
- preserve explicit confirmation boundaries for approval, rejection, and posted status;
- serialize multiple repository mutations;
- ask one concise question only for material ambiguity;
- keep the interface inside system/gpt-instructions.md to avoid duplicated behavioral rules.

Completed commit:

- 7bb8773 — Define natural-language command interface

### Stage 7 — Ready-to-post output

Status: COMPLETE

Approval must:

- preserve the structured draft record;
- add clean copy to output/ready-to-post.md;
- show the same clean copy in chat.

Marking a post as posted must:

- add its facts to the published fact indexes;
- add the post to data/posts/YYYY-MM.jsonl;
- remove it from active drafts;
- regenerate the ready queue from the remaining active records;
- update production state.

Implementation requirements:

- rebuild the complete Markdown queue deterministically instead of patching individual blocks;
- order ready posts by ready_at and then post_id;
- use fixed human-readable topic headings;
- keep metadata outside the clean Facebook copy;
- display paste-ready copy in one plain-text code block;
- verify exact parity between chat copy, the active record, and the queue;
- make partial posted transitions resume with the existing IDs and operation timestamp.

Completed commits:

- d9a7659 — Define deterministic ready-to-post rendering
- ef210c1 — Specify ready-to-post output behavior

### Stage 8 — Acceptance tests

Status: COMPLETE

Specification status: COMPLETE

Execution status: NOT RUN — scheduled for Stage 10 after private GPT installation.

Created:

    tests/acceptance-tests.md

Test coverage:

1. Read-only state inspection.
2. Default post generation.
3. Country-specific generation.
4. Command and ID normalization.
5. Exact duplicate rejection.
6. Paraphrase duplicate rejection.
7. Unsafe fact rejection.
8. Surface wording revision.
9. Underlying fact replacement.
10. Approval and ready-queue parity.
11. Persistence across a new conversation.
12. Full ready-to-posted lifecycle.
13. Idempotent posted retry.
14. Git SHA conflict handling.
15. Partial-write consistency recovery.
16. Final full-database consistency audit.

Every test defines its prompt, expected chat behavior, repository assertions, and pass condition. Results remain NOT RUN until executed and evidenced during Stage 10.

Completed commit:

- ab866bd — Add Viral Producer acceptance test suite

### Stage 9 — Manual installation guide

Status: COMPLETE

Created:

    docs/gpt-installation.md

The guide covers:

- the current Custom GPT to Plugin transition;
- a recommended private Plugin path and compatible GPT Builder path;
- connecting GitHub with least-privilege access;
- limiting the connected account to viral-producer;
- verifying read and write permissions separately;
- installing the complete system/gpt-instructions.md workflow;
- enabling Web Search or equivalent browser research;
- configuring conversation starters or saved prompts;
- keeping the first version private;
- read, write, duplicate, and lifecycle smoke tests;
- checking repository commits and file changes;
- troubleshooting permissions, branches, stale SHAs, and connection failures;
- restricted MCP, Action, or backend fallback only when required.

Completed commits:

- cc3a2c3 — Add ChatGPT installation and setup guide
- fc468e6 — Clarify legacy Action fallback

### Stage 10 — Live test and refinement

Status: IN PROGRESS

Process:

1. Install the workflow as a private Plugin or compatible Custom GPT.
2. Run the saved acceptance prompts.
3. Inspect repository changes.
4. Correct instructions or schemas when needed.
5. Repeat all affected tests.
6. Open and review the implementation pull request.
7. Merge only after all required tests pass.

## 5A. Stage 10 Editorial Refinement Plan

This refinement keeps the Version 1 architecture unchanged. It strengthens editorial selection, claim fidelity, scoring, and testing without adding a backend, external database, or one-file-per-post storage.

Existing P-000001 through P-000003 remain preserved as pre-refinement test evidence. They must not be approved until they pass the revised rules or are explicitly revised through the normal lifecycle.

### Stage 10.1 — Live-output baseline audit

Status: COMPLETE

Scope:

- inspect the first three generated drafts and production state;
- verify IDs, counters, JSONL structure, sources, word counts, and queue behavior;
- compare the scripts with dataset-reference.md and content-dna.md;
- identify editorial and claim-fidelity failures.

Findings:

- repository workflow and persistence are functioning correctly;
- P-000002 is the strongest DNA match;
- P-000001 overuses record and superlative facts and contains one geographic scope drift;
- P-000003 is accurate but too textbook-like and low in surprise;
- all three receiving 12/12 shows that quality scoring is too permissive;
- the transient candidate pool cannot currently be audited from repository evidence.

### Stage 10.2 — Viral DNA v2

Status: COMPLETE

Update:

    system/content-dna.md

Required changes:

- define a controlled surprise-operator taxonomy;
- require at least four distinct operator families per standard post;
- allow no more than two selected facts from the same operator family;
- add an anti-textbook rule for definitions and ordinary classroom facts;
- require at least four strong facts and no weak filler fact;
- strengthen opening and closing requirements;
- add exact claim-scope preservation for geography, time, quantity, and qualifiers;
- add source-accessibility and fallback-source rules;
- replace loose scoring guidance with anchored 0, 1, and 2 definitions;
- reserve 12/12 for exceptional posts that satisfy explicit evidence-based conditions.

Acceptance criteria:

- a six-record post fails the diversity gate;
- a factual but textbook-only post fails the viral gate;
- wording cannot broaden United States into America or North America;
- every score of 2 has an objective rubric justification.

Completed commit:

- ed91651 — Strengthen viral trivia editorial DNA

### Stage 10.3 — Additive editorial audit contract

Status: COMPLETE

Update:

    system/data-contract.md

Add backward-compatible fields for newly created or substantively revised drafts:

- per-fact editorial metadata: surprise_operator, viral_strength, scope_check_passed, and source_access_passed;
- post-level generation_audit: candidate_count, rejection counts, operator variety, and weakest-fact review;
- compact quality rationales for every scored dimension.

Rules:

- keep schema_version 1 because the change is additive;
- legacy drafts may omit the new fields until revised;
- every new draft must contain the new fields;
- candidate_count must be at least 18 for a standard six-fact post;
- do not persist full rejected candidate text or create a new candidate database;
- do not modify existing production IDs, counters, or historical facts during this stage.

Completed commit:

- 5c35c0f — Add editorial audit metadata contract

### Stage 10.4 — Generation and self-critique guardrails

Status: COMPLETE

Update:

    system/gpt-instructions.md

Required workflow:

    Research at least 18 candidates
    → verify direct source support
    → canonicalize and deduplicate
    → label surprise operators
    → rank viral strength
    → select a diverse six
    → compare surface wording with source scope
    → challenge the two weakest facts
    → replace weak or repetitive facts
    → score with written rationales
    → persist only after every hard gate passes

Additional behavior:

- opening and closing must be selected from the strongest facts;
- inaccessible or restricted sources require a second accessible authoritative source;
- absolute and record terms receive an explicit scope check;
- a 12/12 draft receives an additional adversarial review before saving;
- failure to meet the editorial gate triggers candidate replacement rather than score inflation.

Completed commits:

- 80506f6 — Add generation and self-critique guardrails
- f3c4c0f — Align runtime instructions with audit field names

### Stage 10.5 — Editorial regression tests

Status: COMPLETE

Update:

    tests/acceptance-tests.md

Add tests for:

- operator monoculture rejection;
- geographic and temporal scope drift;
- textbook-only candidate rejection;
- quality-score inflation;
- inaccessible-source fallback;
- persisted generation audit metadata;
- opening and closing strength;
- backward compatibility with the three existing drafts.

After these tests are added, rerun all earlier acceptance tests affected by generation, revision, approval, persistence, and recovery.

Completed commit:

- e33efce — Add editorial regression test suite

### Stage 10.6 — Plugin refresh guide

Status: COMPLETE

Update:

    docs/gpt-installation.md

Document:

- how to update the existing private Plugin or compatible GPT;
- which instruction and reference files must be replaced or refreshed;
- how to start a new conversation to avoid stale instructions;
- how to verify the installed instruction version;
- three saved regression prompts for geography, animals, and body science;
- how to compare the new outputs with the pre-refinement baseline.

Completed commit:

- d6a9e8b — Add plugin refresh and validation guide

### Stage 10.7 — Controlled live validation

Status: COMPLETE

Process:

1. Refresh the private Plugin or GPT with the updated files.
2. Run read-only state verification.
3. Generate three new test posts covering geography-history, animals-nature, and body-science.
4. Verify candidate audit metadata, operator diversity, scope checks, scores, and sources in GitHub.
5. Run the new editorial regression tests.
6. Run all affected lifecycle tests.
7. Record evidence and commit identifiers in tests/acceptance-tests.md.
8. Revise P-000001 and P-000003 only through normal commands if they are still intended for publication.
9. Approve content only after it passes the revised rules.

Exit criteria:

- no selected script contains scope broadening;
- every new standard post has at least four operator families and no more than two facts from one family;
- every new post records at least 18 researched candidates in compact audit metadata;
- weak textbook filler is absent;
- score rationales support every value and 12/12 is exceptional rather than automatic;
- sources are authoritative, directly supportive, and accessible or backed by an accessible fallback;
- all required acceptance tests and the final consistency audit pass.

Validation results:

- AT-01 through AT-23 passed and are recorded in tests/acceptance-tests.md.
- Geography-history, animals-nature, and body-science v2 outputs were produced as P-000005, P-000006, and P-000008.
- P-000008 completed the body-science regression with 24 candidates, 18 documented rejections, five operator families, six strength-2 facts, complete audit metadata, and a calibrated 12/12 score.
- Compared with legacy P-000003, P-000008 removes textbook filler, strengthens opener and closer selection, adds diverse surprise mechanisms, and stores direct source/scope evidence.
- AT-14 preserved concurrent state correctly. Its Writer A draft later exposed an endpoint-operator collision; Fact 6 was replaced through the normal lifecycle, and the repaired record passed all v2 gates.
- AT-15 proved deterministic recovery of a missing ready-queue block without changing authoritative post or fact content.
- The final consistency audit passed on production snapshot `bbb08d718c89162515ffc09a86d8a47b22c8b289`.
- The validated snapshot contains 7 active posts, 42 active fact snapshots, 6 published facts, 1 archived post, and 1 ready post.
- Counters, rotation history, global fact/signature uniqueness, archive linkage, version-2 gates, and deterministic ready-queue parity all passed.
- All Stage 10.7 exit criteria are satisfied.

Evidence:

- `ab13fbfc58b24eb9011192bb3d6eea19c2217d3f` — Record AT-01 through AT-23 and the earlier full consistency audit.
- `ab360da3a906cea8e33b3dc869b31fee1e650d3a` — Persist the P-000008 body-science regression draft.
- `bbb08d718c89162515ffc09a86d8a47b22c8b289` — Complete P-000008 state recovery and final validated production snapshot.
- `af3c09b6a69ebb3926aaf8ce529a39b096baf1eb` — Record body-science regression and final consistency evidence in the acceptance results.

## 6. Definition of Done

The implementation is complete when:

- all planned files exist on build/viral-content-gpt;
- all JSON and JSONL files validate;
- the installed Plugin or compatible GPT can read and write the repository;
- research produces traceable sources;
- exact and semantic duplicates are rejected;
- unsafe facts are rejected;
- a post completes the full lifecycle;
- a new conversation preserves state through GitHub;
- ready-to-post.md contains clean copy only;
- acceptance tests pass;
- installation documentation is usable without developer assistance;
- the implementation pull request is reviewed and merged.

## 7. Plan Maintenance Rule

This file is the authoritative implementation checklist.

After every stage or approved design change:

1. update the relevant decision or stage;
2. record the stage status;
3. add the resulting commit identifier;
4. update acceptance criteria when behavior changes;
5. keep examples consistent with the current schema.

Do not rely on conversation history as the only record of the plan.

## 8. Refinement Log

### 2026-09-30

- Replaced the planned 11–16-word rule with a 12–15-word ideal and 11–18-word accepted range based on the reference dataset.
- Standardized post and fact IDs to six digits.
- Stored reserved facts inside active drafts instead of published fact indexes.
- Added blocked facts for false, unsafe, or unverifiable claims.
- Added single-writer mode, Git SHA conflict protection, idempotency, and consistency recovery.
- Confirmed that permanent posts will be stored in monthly JSONL archives, not one file per post.
- Corrected approval ordering so active drafts reach ready status before the derived ready queue is regenerated.
- Embedded the bilingual command interface in the main GPT instructions so intent handling and lifecycle rules remain synchronized.
- Made ready-to-post rendering deterministic and corrected posted cleanup so the queue is always rebuilt from authoritative active records.
- Defined acceptance tests as a reusable specification with objective repository assertions; actual execution remains reserved for the installed-GPT test stage.
- Added a Plugin-first installation path in response to OpenAI's Custom GPT transition while retaining GPT Builder compatibility where available.
- Recorded the first live-output audit and added the Stage 10 editorial refinement plan for DNA diversity, claim-scope fidelity, honest scoring, compact audit metadata, regression tests, and controlled revalidation.
- Implemented Viral DNA v2 with eight controlled surprise operators, hard diversity and viral-strength gates, anti-textbook filtering, source-access checks, exact scope preservation, and anchored quality scoring.
- Added the backward-compatible editorial audit contract with per-fact operator and validation fields, compact candidate accounting, quality rationales, legacy approval boundaries, publication preservation, and hard consistency checks.
- Implemented runtime generation and self-critique guardrails with live candidate accounting, opened-source verification, operator and strength selection, exact scope checks, weakest-fact challenges, 12/12 adversarial review, legacy upgrades, and schema-aligned persistence.
- Expanded the acceptance suite from 15 to 23 tests with version-2 global gates and regressions for operator monoculture, scope drift, textbook filler, score inflation, inaccessible sources, persisted audit metadata, opening and closing strength, and legacy compatibility.
- Added the manual Editorial Version 2 refresh guide for the existing private Plugin or compatible GPT, including identity-preserving updates, read-only version checks, fresh-conversation verification, three saved content prompts, baseline comparison, and stale-instruction troubleshooting.
- Completed AT-01 through AT-23 and a full consistency audit on the installed Plugin v2; recorded the passing evidence and the repaired P-000007 endpoint-operator defect.
- Completed the P-000008 body-science regression, verified its six direct sources and v2 editorial metadata, compared it with legacy P-000003, reran the full consistency audit, and marked Stage 10.7 complete.
