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
- `main` as the sole production runtime branch for plugin reads and writes;
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

Execution status: COMPLETE — AT-01 through AT-23 passed during Stage 10.

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

Every test defines its prompt, expected chat behavior, repository assertions, and pass condition. Execution evidence is recorded in the same test specification.

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

Status: COMPLETE

Process:

1. Install the workflow as a private Plugin or compatible Custom GPT.
2. Run the saved acceptance prompts.
3. Inspect repository changes.
4. Correct instructions or schemas when needed.
5. Repeat all affected tests.
6. Open and review the implementation pull request.
7. Merge only after all required tests pass.

Result:

- Pull request #1 was opened from `build/viral-content-gpt` to `main`.
- The implementation review found no blocking issue.
- The pull request was merged after all acceptance, regression, and consistency checks passed.
- Merge commit: `c18ad4f42fb94274518d67a7d5e5991b57eddbf0`.

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

## 5B. Stage 11 — User Documentation

Status: COMPLETE

Branch boundary:

- documentation work uses `docs/user-guide`;
- production continues on `main`;
- documentation changes must not modify `data/**` or `output/**`.

Writing standard:

- concise, natural Indonesian;
- technical terms remain in English when clearer;
- short steps and copy-ready prompts;
- no unnecessary schema detail.

Progress:

- Stage 11.1 — Information architecture: COMPLETE.
- Stage 11.2 — Concise user guide: COMPLETE.
- Stage 11.3 — Complete prompt library: COMPLETE.
- Stage 11.4 — README navigation: COMPLETE.
- Stage 11.5 — Documentation QA: COMPLETE.
- Stage 11.6 — User acceptance test: COMPLETE.

Deliverables:

- `docs/user-guide.md`
- `docs/prompt-library.md`
- updated `README.md`

Evidence:

- `57865d69994280c3d623c27c43e7dbbb9efd0b9e` — Add concise Viral Producer user guide.
- `3c7d9f8b6b033a58a053acdc96192b6f25d55a17` — Add complete copy-ready prompt library.
- `d98027a1a110a320bf5f87818da568f79afed29e` — Turn README into a concise documentation landing page.
- `c933a4c688b6f61504bf1942cdcce4a008352e33` — Replace the obsolete Stage 10.7 handoff with post-launch maintenance guidance.
- `a15b1bcb9dbf730b0ecec3e662a98ba1da98c565` — Align installation and smoke-test guidance with the live `main` workflow.
- 2026-10-01 — User reviewed and accepted the documentation; Stage 11.6 passed without production writes.
- `7084c30a654d5ad94b582c9477214103b37d6167` — Merge pull request #3 and publish the Stage 11 documentation to `main`.

## 5C. Stage 12 — Viral Producer 1.1

Status: IN PROGRESS — STAGE 12.11 COMPLETE; AT-24 POST-CUTOVER

Branch implementasi:

    upgrade/viral-producer-v1.1

Branch acceptance test:

    test/viral-producer-v1.1

Production runtime tetap menggunakan main. Branch implementasi dibuat dari main saat revision 100, next_post_number 59, next_fact_number 375, dan 57 active records. Nilai tersebut hanya baseline pembuatan branch; chat implementasi wajib membaca ulang main terbaru sebelum bekerja karena produksi terus berjalan.

Refreshed remote baseline (2026-10-01):

- GitHub main commit observed during the direct update: `ae0464f327cf97ba42e4e39a8cc7f8e82f145105`;
- production state: revision 114, next_post_number 73, next_fact_number 459, single_writer_mode true;
- repository versions before the feature update: Instruction 2.0 — Stage 10.4, Editorial 2.0 — Stage 10.2, Test specification 2.0 — Stage 10.5, schema_version 1;
- production inventory: 71 active records, 426 active fact snapshots, 45 drafts, 26 ready posts, 1 archived post, and 6 published fact records;
- complete current main core files were read through the GitHub connector with explicit `ref: main`;
- current main JSON and JSONL parsing, active ID/signature uniqueness, fact counts, counters, and single-writer invariant passed;
- feature head before the direct GitHub update: `1b35ac8ee37bf605555669f4c6dd6c86c0ccd82d`;
- `02efd0eb5bcf07f22567c7db5bfd8f9182596e07` and `04fe42ed7ba907ab5761e89b9b2494e098cb2d16` synchronize the protected active-draft and production-state snapshots through P-000072;
- the synchronized protected file blobs exactly match main: active drafts `a141f32609812ea6d52902d237d78adcb6bcd430` and production state `0ddb8240c4625b06862256c952390b5789482ed9`;
- main itself was not modified.

Stage 12.4 refreshed remote baseline (2026-10-01):

- GitHub main commit observed before implementation: `bae173883c82f08ea38c1863cab5951293f4eeca`;
- feature head before Stage 12.4: `43aa67fabf9f4c461e41285d59c1cf19a9cf9e02`;
- production state remained revision 121, next_post_number 80, next_fact_number 501, and single_writer_mode true;
- production inventory remained 78 active records;
- current feature versions before Stage 12.4 were Instruction 3.0 — Stage 12, Editorial 2.0 — Stage 10.2, Test specification 3.0 — Stage 12, and schema_version 1;
- protected feature snapshots were already byte-identical to main, so no synchronization write was required;
- protected blobs remained active drafts `f039901aa984aa69266951db39d5f5221c20fe10` and production state `00e56f289b7041db98c2e8840ac88dd7f6621659`;
- main itself was not modified.

Stage 12.5 refreshed remote baseline (2026-10-01):

- GitHub main commit observed before implementation: `bae173883c82f08ea38c1863cab5951293f4eeca`;
- feature head before Stage 12.5: `4c0d502259e8ee9c4f6f623c75cb54c88bc7dfd3`;
- production state remained revision 121, next_post_number 80, next_fact_number 501, and single_writer_mode true;
- production inventory remained 78 active records;
- current feature versions before Stage 12.5 were Instruction 3.0 — Stage 12, Editorial 3.0 — Stage 12, Test specification 3.0 — Stage 12, and schema_version 1;
- protected feature snapshots were already byte-identical to main, so no synchronization write was required;
- protected blobs remained active drafts `f039901aa984aa69266951db39d5f5221c20fe10` and production state `00e56f289b7041db98c2e8840ac88dd7f6621659`;
- main itself was not modified.

Stage 12.6 refreshed remote baseline (2026-10-01):

- GitHub main commit observed before implementation: `bae173883c82f08ea38c1863cab5951293f4eeca`;
- feature head before Stage 12.6: `6d354cf2edb83ad71a313a447a5f3a850bfa732e`;
- production state remained revision 121, next_post_number 80, next_fact_number 501, and single_writer_mode true;
- production inventory remained 78 active records;
- current feature versions before Stage 12.6 were Instruction 3.0 — Stage 12, Editorial 3.0 — Stage 12, Test specification 3.0 — Stage 12, and schema_version 1;
- protected feature snapshots were already byte-identical to main, so no synchronization write was required;
- protected blobs remained active drafts `f039901aa984aa69266951db39d5f5221c20fe10` and production state `00e56f289b7041db98c2e8840ac88dd7f6621659`;
- `data/publishing-plan.json` and `output/content-calendar.md` did not exist before this stage;
- main itself was not modified.

Stage 12.7 refreshed remote baseline (2026-10-01 through 2026-10-02):

- feature head before Stage 12.7: `8d2dc1bb0269c830784e09e3c109135898d9ae99`;
- production state at stage start was revision 122, next_post_number 81, next_fact_number 507, single_writer_mode true, and 79 active records;
- a live production batch continued during implementation and completed at GitHub main commit `af68f8db2d54e6c9d555e66a7c38045cf5f46501`, revision 139, next_post_number 98, next_fact_number 609, and 96 active records;
- the final production snapshot passed JSON/JSONL parsing, six-fact counts, active Post/Fact ID and claim-signature uniqueness, monotonic counters, and single-writer validation;
- the 576 existing active fact snapshots still omit subject_key and remain byte-preserved legacy-compatible data;
- ready-to-post.md remained byte-identical, so only active drafts and production state required synchronization;
- main itself was not modified.

### Tujuan

Meningkatkan Viral Producer menjadi versi 1.1.0 melalui enam update:

1. Fast Approval.
2. Mixed-Topic Support.
3. Performance Feedback Loop.
4. Smart Ready Queue dan Content Calendar.
5. Subject dan Angle Cooldown.
6. Complete Publishing Package dengan caption dan hashtag.

Implementasi harus tetap menggunakan GitHub sebagai source of truth, mempertahankan single_writer_mode, kompatibel dengan data lama, tidak mengganggu produksi aktif di main, dan tidak memasukkan data acceptance test ke main.

### Keputusan final

#### Fast Approval

- Tidak membuka ulang source.
- Tidak menggunakan Web Search atau factual revalidation.
- Tidak menjalankan semantic deduplication dan quality scoring ulang.
- Tidak memiliki Pre-Publish Freshness Gate.
- Tidak membuat candidate pool, mengganti fakta, atau mengalokasikan ID.
- Menggunakan validation evidence yang sudah tersimpan ketika draft dibuat atau direvisi.
- Legacy atau incomplete draft tidak di-upgrade otomatis saat approval; approval harus berhenti dan meminta revisi terpisah.

#### Mixed content

- Default jangka panjang: 75 persen themed dan 25 persen mixed.
- Mixed post berisi minimal empat fact topics.
- Maksimal dua fakta dari fact topic yang sama.
- Existing operator, strength, source, scope, safety, dan quality gates tetap berlaku.

#### Caption dan hashtag

- Caption satu kalimat, idealnya 6–14 kata.
- Caption singkat, padat, jelas, menarik, dan memakai natural American English.
- Caption tidak memakai pertanyaan generik, kalimat trivia, kata trivia dalam prose, pengulangan CTA, emoji default, atau factual claim baru.
- Hashtag berjumlah 4–6, unik, relevan, dan bukan spam.
- Tag #Trivia diperbolehkan; larangan trivia hanya berlaku untuk prose caption.
- On-screen script dan Facebook caption ditampilkan dalam dua code block terpisah.

#### Concurrency

- Produksi tetap single-writer.
- Multi-writer production tidak termasuk scope versi 1.1.
- Plugin test dapat menulis hanya ke configured test branch.

### Target versi

- Plugin: Viral Producer 1.1.0.
- Instruction version: 3.0 — Stage 12.
- Editorial version: 3.0 — Stage 12.
- Test specification version: 3.0 — Stage 12.
- schema_version tetap 1 karena perubahan additive dan backward-compatible.

### Branch dan isolation policy

Struktur:

    main
    ├── upgrade/viral-producer-v1.1
    └── test/viral-producer-v1.1

main:

- tetap menjadi branch produksi;
- tetap dapat menerima produksi selama pengembangan;
- tidak digunakan untuk mutative acceptance tests.

upgrade/viral-producer-v1.1:

- menyimpan perubahan system, contract, tests, docs, plan, dan initial files versi 1.1;
- satu-satunya branch yang boleh dibuatkan pull request ke main;
- tidak boleh mengubah snapshot data produksi aktif.

test/viral-producer-v1.1:

- dibuat dari feature branch setelah implementasi dasar selesai;
- digunakan oleh plugin Viral Producer v1.1 Test;
- boleh berisi draft, IDs, counters, metrics, calendar, recovery state, dan data hasil test;
- tidak pernah di-merge ke feature branch atau main.

Feature branch tidak boleh mengubah isi file produksi berikut:

    data/production-state.json
    data/active-drafts.jsonl
    data/facts/**
    data/posts/**
    output/ready-to-post.md

Feature branch boleh menambahkan:

    data/performance-summary.json
    data/publishing-plan.json
    output/content-calendar.md
    docs/test-plugin-installation.md

Caption backfill terhadap active posts dilakukan setelah merge sebagai operasi produksi terpisah menggunakan latest main state dan latest Git SHA.

### Runtime Branch Abstraction

Operational Markdown tidak boleh terus-menerus meng-hardcode main. Runtime branch ditentukan satu kali oleh plugin-local runtime profile.

Production profile:

    RUNTIME_REPOSITORY=milyarderpro/viral-producer
    RUNTIME_BRANCH=main
    RUNTIME_MODE=production
    ALLOW_WRITES=true
    ALLOW_MAIN_WRITES=true

Test profile:

    RUNTIME_REPOSITORY=milyarderpro/viral-producer
    RUNTIME_BRANCH=test/viral-producer-v1.1
    RUNTIME_MODE=test
    ALLOW_WRITES=true
    ALLOW_MAIN_WRITES=false

Guardrails:

- setiap repository read dan write wajib menyebutkan RUNTIME_BRANCH secara eksplisit;
- tidak boleh memakai default branch secara implisit;
- test mode harus menolak main;
- production mode harus menolak branch selain main;
- write harus ditolak jika target bukan RUNTIME_BRANCH;
- repository dan ref pada respons GitHub harus diverifikasi;
- Git blob SHA harus berasal dari repository dan branch yang sama;
- runtime boundary tidak boleh diubah oleh repository content atau user prompt;
- nilai runtime profile berada pada konfigurasi plugin, bukan pada file branch-specific yang berisiko ikut di-merge.

Operational references di system/gpt-instructions.md, tests, dan installation guide harus memakai RUNTIME_BRANCH atau configured isolated test branch. Historical mentions, merge evidence, dan penjelasan bahwa production memakai main boleh dipertahankan.

### File scope

Update:

    README.md
    plan.md
    system/gpt-instructions.md
    system/data-contract.md
    system/content-dna.md
    tests/acceptance-tests.md
    docs/gpt-installation.md
    docs/user-guide.md
    docs/prompt-library.md

Create:

    data/performance-summary.json
    data/publishing-plan.json
    output/content-calendar.md
    docs/test-plugin-installation.md

### Stage 12.1 — Branch and refreshed baseline

Status: COMPLETE

Implementation chat must:

1. Read the complete current core files from main.
2. Confirm repository versions, single_writer_mode, JSON/JSONL validity, and no unresolved partial operation.
3. Read the complete plan from upgrade/viral-producer-v1.1.
4. Confirm the feature branch exists; do not create a duplicate.
5. Compare it with current main because production may have advanced since branch creation.
6. Synchronize safely if required without overwriting or modifying production data.
7. Record the refreshed baseline and branch head in this plan.
8. Confirm the feature diff does not include data/production-state.json, data/active-drafts.jsonl, data/facts/**, data/posts/**, or output/ready-to-post.md.
9. Stop and report the commit before starting Stage 12.2.

Completion evidence:

- Existing branch `upgrade/viral-producer-v1.1` was reused; no duplicate branch was created.
- Current main was read and validated through the official GitHub connector before synchronization.
- Protected production snapshots were synchronized with SHA preconditions and verified byte-identical to main.
- The feature branch changes exclude `data/production-state.json`, `data/active-drafts.jsonl`, `data/facts/**`, `data/posts/**`, and `output/ready-to-post.md` as feature-owned changes.

### Stage 12.2 — Runtime Branch Abstraction

Status: COMPLETE

Implement RUNTIME_REPOSITORY, RUNTIME_BRANCH, RUNTIME_MODE, ALLOW_WRITES, and ALLOW_MAIN_WRITES across active instructions and tests.

Requirements:

- eliminate operational main hardcoding;
- retain legitimate historical and production-profile mentions;
- make every connector read/write explicit-ref;
- add startup repository/ref verification;
- block test-on-main and production-on-test mismatches;
- update tests to use {{RUNTIME_BRANCH}} or configured isolated test branch;
- document separate production and test profiles;
- keep data contract branch-neutral while making SHA, lifecycle, and recovery rules apply inside one RUNTIME_BRANCH.

Acceptance:

- test profile cannot write main;
- production profile cannot write test;
- no default-branch fallback exists;
- branch mismatch blocks before write.

Completion evidence:

- `d0572ff85df34b2cbb35174efacdccd70a3eaac7` adds the immutable runtime profile and explicit-ref enforcement to `system/gpt-instructions.md`.
- `1c48373c4c873852c0da211a5d4d1361cd847d5d` makes lifecycle, SHA, concurrency, audit, and recovery rules branch-neutral within one RUNTIME_BRANCH.
- `040d9c2940e318fb054088960f250af1e710f3e9` upgrades the test specification to 3.0 — Stage 12 and adds AT-24 through AT-27 for runtime isolation.
- `94bd3cb24efe53478be2bccc259a70e65ad2eb34` documents separate production and test profiles plus explicit connector refs.
- Instruction version is 3.0 — Stage 12 and Test specification version is 3.0 — Stage 12. Editorial version remains 2.0 — Stage 10.2 until later Stage 12 editorial work changes content DNA.
- Production/test profile mismatches, target-branch mismatches, default-branch fallback, connector response mismatches, and cross-branch SHA reuse are hard failures before write.
- Static validation passed for all five profile fields, explicit read/write refs, protected-file isolation, JSON/JSONL validity, and whitespace errors.
- AT-25 through AT-27 are specified for isolated Stage 12.10 execution.
- AT-24 is a production-runtime smoke test and is intentionally deferred to Stage 12.12 after the PR is merged and the production plugin is updated to v1.1; a Stage 10 production runtime cannot satisfy its version/profile preconditions.
- Stage 12.3 remains pending and was not started.

### Stage 12.3 — Update 1: Fast Approval

Status: COMPLETE

Eligibility:

- explicit Post ID and approval intent;
- current status draft;
- exactly six facts;
- complete current editorial metadata;
- every stored scope_check_passed and source_access_passed is true;
- quality.hard_rules_passed is true;
- quality rationales and generation_audit are complete and internally consistent;
- operator and viral-strength gates pass from stored metadata;
- Fact 1 and Fact 6 are strength 2 with different operators;
- no unresolved partial lifecycle operation;
- latest SHA preflight passes.

Fast Approval must not:

- open source URLs;
- use Web Search;
- run fresh research or factual revalidation;
- perform global semantic dedup research;
- rescore editorial quality;
- rebuild generation_audit;
- replace facts;
- allocate IDs;
- run a Pre-Publish Freshness Gate.

Write flow:

1. Fetch latest affected files and SHAs.
2. Calculate final draft-to-approved-to-ready record in memory.
3. Persist active-drafts.jsonl once with approved_at and ready_at.
4. Rebuild and write the complete ready-to-post.md once.
5. Increment production-state revision once.
6. Leave next_post_number and next_fact_number unchanged.
7. Reread and verify exact queue parity.
8. Display the stored script and publishing package without regeneration.

Legacy or incomplete drafts must fail Fast Approval and require a separate revision or upgrade command.

Completion evidence:

- Current main was read and validated through the GitHub connector at observed commit `bae173883c82f08ea38c1863cab5951293f4eeca`, revision 121, next_post_number 80, next_fact_number 501, and 78 active records.
- Main JSON/JSONL parsing, global active/published ID and signature uniqueness, counters, archive linkage, ready-queue parity, single_writer_mode, and partial-operation checks passed before implementation.
- `1df7520c59aa8663792b9327da0ff73d4ca308c2` replaces approval-time re-evaluation with stored-evidence-only Fast Approval in `system/gpt-instructions.md`.
- `4c2f3f9b495f6b7ee0a985593f24c3fdb94dcb71` defines complete Fast Approval eligibility, forbidden operations, one-write-per-file sequencing, counter preservation, postconditions, and partial recovery in `system/data-contract.md`.
- `c0bc0f4bda4bebe172ba3ea4484d1cab0e80cb1f` expands AT-10 and adds AT-28 and AT-29 for zero-revalidation approval, incomplete-record rejection, strict write counts, exact parity, and stale-SHA restart behavior.
- Fast Approval now performs no source opening, Web Search, research, factual revalidation, freshness gate, global semantic deduplication, rescoring, audit rebuilding, content replacement, or ID allocation.
- An eligible approval writes active drafts once, the complete ready queue once, and production state once; revision increases once while both next-ID counters and stored editorial evidence remain unchanged.
- Legacy, incomplete, inconsistent, mismatched-ref, partial-operation, or stale-preflight conditions block before write or trigger contract-defined recovery.
- `eb09f1f1c62c7d8fa11cff7d2a4563b8321f77ba` and `c6a631f8db5abbda85a7a4ed10def3f8f1475ca8` synchronize the protected feature snapshots with main through P-000079 and revision 121.
- Protected snapshot blobs match main: active drafts `f039901aa984aa69266951db39d5f5221c20fe10` and production state `00e56f289b7041db98c2e8840ac88dd7f6621659`.
- The new Fast Approval tests are specified but remain unexecuted until the isolated Stage 12.10 run.
- Stage 12.4 remains pending and was not started.

### Stage 12.4 — Update 2: Mixed-Topic Support

Status: COMPLETE

Add post_format:

    themed
    mixed

Backward compatibility:

- missing post_format means themed;
- existing records do not require bulk rewrite.

Post-level topic values:

    animals-nature
    body-science
    food-home
    geography-history
    inventions-records
    practical
    mixed

Fact-level topic values remain the six existing non-mixed topics. Never create data/facts/mixed.jsonl.

Themed rules:

- all facts use the themed post topic unless the live contract explicitly allows a documented exception.

Mixed rules:

- at least four fact topics;
- no fact topic appears more than twice;
- post topic is mixed;
- each fact preserves its own fact topic;
- publication routes every fact to its own topic ledger;
- default country_focus is GLOBAL;
- country-specific mixed requests require every selected fact to support the requested country;
- existing operator, strength, opening/closing, source, scope, safety, word-count, audit, and quality rules remain mandatory;
- long-term default rotation is 75 percent themed and 25 percent mixed.

Ready heading for mixed posts:

    Mixed Trivia

Completion evidence:

- `8111f7c4cb184eef4a3977052646055b64f3d1e1` upgrades Content DNA to Editorial 3.0 — Stage 12, defines the 75/25 default format rotation, and makes themed and mixed editorial gates explicit.
- `e90408859c2b8c7f7e74efd3c3a5bc0d3b1695a4` adds the additive `post_format` contract, themed backward compatibility, mixed topic constraints, country-specific coverage, per-fact publication routing, and the prohibition on `data/facts/mixed.jsonl`.
- `2c7a66f28ea71d80d15bb635e4210cd67e1ab475` implements format normalization, selection, generation, approval validation, publication, audit, and user-facing reporting in the active instructions.
- `0b4cd8a1b8c7d9253f380dae74f0d39ee7c2b220` adds global format gates plus AT-30 through AT-32 for mixed generation, multi-ledger publication, and themed backward compatibility.
- Mixed ready blocks use the fixed heading `Mixed Trivia`; fact-level topics remain limited to the six existing ledgers.
- Missing `post_format` is interpreted as themed without a bulk or incidental rewrite, including during Fast Approval.
- Static validation passed for required format rules, correct ready heading, independent fact routing, default rotation, JSON/JSONL parsing, and the new acceptance-test specifications.
- Protected feature snapshots remain byte-identical to main: active drafts `f039901aa984aa69266951db39d5f5221c20fe10` and production state `00e56f289b7041db98c2e8840ac88dd7f6621659`.
- AT-30 through AT-32 are specified but remain unexecuted until the isolated Stage 12.10 run.
- Stage 12.5 remains pending and was not started.

### Stage 12.5 — Update 3: Performance Feedback Loop

Status: COMPLETE

Raw storage:

    data/performance/YYYY-MM.jsonl

Derived summary:

    data/performance-summary.json

Performance record minimum:

    {
      "schema_version": 1,
      "post_id": "P-000020",
      "captured_at": "ISO-8601",
      "post_age_hours": 24,
      "source": "manual",
      "metrics": {
        "views": 1200000,
        "reactions": 84000,
        "comments": 2300,
        "shares": 15000,
        "average_watch_time_seconds": 8.4,
        "retention_percent": null,
        "followers_gained": 3200
      }
    }

Rules:

- metrics are accepted only for archived posted posts;
- multiple snapshots per post are allowed;
- post_id plus captured_at is the idempotency key;
- an identical retry is a no-op success;
- a different payload with the same key is a conflict;
- metrics never modify the published post;
- summary is rebuilt deterministically;
- sample size below 15–20 posts must not drive strong strategic conclusions;
- performance may act as a tie-breaker but never weaken factual, safety, originality, or editorial gates;
- Content DNA is never automatically rewritten from one successful post.

Commands:

    Catat performa P-000020: ...
    Tampilkan ringkasan performa konten.
    Analisis topic, format, country, dan operator dengan performa terbaik.

Initial performance-summary.json:

    {
      "schema_version": 1,
      "updated_at": null,
      "sample_size": 0,
      "by_topic": {},
      "by_country": {},
      "by_post_format": {},
      "by_operator": {}
    }

The monthly performance directory/file is created on the first actual metrics write.

Completion evidence:

- `3e682fa8fcfbcb63d1bb9666d25f0aee055d8a4f` adds performance-feedback guardrails to Content DNA, including the descriptive, directional, and tie-breaker sample thresholds.
- `c0d071902110418401becd5a637ceecceff1b927` defines raw monthly storage, posted-only eligibility, canonical metrics, compound-key idempotency, deterministic aggregation, recovery, audit, and hard failures.
- `188ac724988888cee49d13018d50f07ebf75cdc6` implements record, summary, and analysis commands plus the serialized raw-then-summary persistence flow.
- `ccdd18e4deb57aae641b5d4c3ec7d64a9615f7cb` adds global performance gates and AT-33 through AT-38 for valid snapshots, posted-only enforcement, retry, conflict, deterministic rebuilding, and small-sample restraint.
- `39e57c4a8f8642c4fda1e4dad5fb71e6db9a81cf` removes legacy trailing whitespace from the acceptance specification so the Stage 12.5 static check is clean.
- `e2e78b80e09e1b3dca386ea94ed05f11ac9879c1` creates the empty deterministic `data/performance-summary.json` baseline.
- Raw performance uses `data/performance/YYYY-MM.jsonl`, routed from `captured_at` in Asia/Jakarta; the directory and monthly file remain absent until the first real metrics write.
- The summary counts unique measured posts from their latest snapshots and aggregates topic, country, effective post format, and distinct operator buckets with explicit post counts.
- Fewer than 15 posts supports descriptive reporting only, 15–19 supports cautious directional observations, and at least 20 posts is required before performance may act as a tie-breaker.
- Performance writes do not modify archived posts, published facts, active drafts, ready queue, production-state revision, IDs, counters, rotation, or Content DNA.
- Static validation passed for the empty summary schema, raw/summary rules, deterministic latest-snapshot aggregation, command coverage, AT-33 through AT-38, JSON/JSONL parsing, and protected-file parity.
- Protected feature snapshots remain byte-identical to main: active drafts `f039901aa984aa69266951db39d5f5221c20fe10` and production state `00e56f289b7041db98c2e8840ac88dd7f6621659`.
- AT-33 through AT-38 are specified but remain unexecuted until the isolated Stage 12.10 run.
- Stage 12.6 remains pending and was not started.

### Stage 12.6 — Update 4: Smart Ready Queue and Content Calendar

Status: COMPLETE

Existing output/ready-to-post.md remains the deterministic complete ready queue.

Add a read-only recommendation command:

    Rekomendasikan post terbaik untuk diposting berikutnya.

Recommendation considers:

- scheduled slot;
- topic and country rotation;
- themed/mixed alternation;
- subject cooldown;
- operator repetition;
- quality;
- ready age;
- performance summary only after the minimum sample threshold.

Recommendation must not mutate repository state.

Create data/publishing-plan.json:

    {
      "schema_version": 1,
      "timezone": "Asia/Jakarta",
      "revision": 0,
      "slots": []
    }

Slot minimum:

    {
      "scheduled_for": "ISO-8601",
      "post_id": "P-000020",
      "status": "planned"
    }

Rules:

- only ready posts may be scheduled;
- one post cannot occupy multiple active slots;
- scheduling does not change post lifecycle;
- timezone is Asia/Jakarta;
- marking a post as posted must finish or remove its active schedule entry;
- output/content-calendar.md is rebuilt deterministically from the publishing plan;
- empty plan and empty calendar are valid.

Commands:

    Susun jadwal posting tujuh hari, dua post per hari.
    Tampilkan content calendar.
    Pindahkan P-000020 ke jadwal besok pukul 19.00 WIB.

Completion evidence:

- `684031f86f3946e8e04119d295a6a05c6ebce255` defines deterministic smart-queue recommendation priorities without repository mutation.
- `78349883ec5581d9280d62958511b0356e017d02` defines the publishing-plan schema, WIB/UTC routing, calendar renderer, scheduling and move flows, publication cleanup, idempotency, recovery, audit, and hard failures.
- `e5cd65a8d21636d3b50d3bdf422e3c23773e9fd9` implements recommendation, scheduling, calendar display, and move commands in the active instructions.
- `25a2aeeeeafd43d5073bbe773a1536ca58db03f4` aligns the scheduled Mark-as-posted write order with the authoritative lifecycle contract.
- `3ed0319a97eba81e2dc8fde658b5ab776c3c1567` creates the empty `data/publishing-plan.json` baseline at revision 0.
- `4e9bc916d5e51edb0c29d5c363a4de394d24b7e5` creates the deterministic empty `output/content-calendar.md`.
- `ef3d9aeec2737fe36ced387bea384763b548232d` adds global scheduling gates and AT-39 through AT-44 for read-only recommendation, seven-day scheduling, ready-only enforcement, conflicts, WIB moves, and publication cleanup.
- Recommendations choose only ready posts, honor the earliest planned slot, and otherwise apply the ordered rotation, cooldown, operator, quality, ready-age, eligible-performance, and Post-ID rules.
- Default seven-day scheduling starts on the next full Asia/Jakarta day at 12:00 and 19:00 WIB, stores UTC timestamps, and prepares the whole batch before one plan write and one calendar rebuild.
- Publishing-plan mutations use their own revision and do not change post lifecycle, ready queue order, production revision, IDs, counters, content, archives, facts, or performance data.
- A scheduled post marked as posted becomes a completed plan slot using the same published_at timestamp and disappears from the active calendar while production-state revision still increases only once.
- Production advanced during final verification to revision 122, next_post_number 81, next_fact_number 507, and 79 active records; uniqueness, counters, six-fact records, and unchanged ready-queue parity passed before synchronization.
- `38166692688b2c8d0d2ef3a0cbfdd27e61003c59` and `a7d27d63a243b6e52e6b5ea100c8443eef082117` synchronize the protected feature snapshots with the latest main without writing to main.
- Static validation passed for initial JSON/Markdown files, planned-to-ready and completed-to-archive linkage rules, unique post/time constraints, command coverage, AT-39 through AT-44, JSON/JSONL parsing, whitespace, and final protected-file parity.
- Protected feature snapshots are byte-identical to main: active drafts `60adad6cb25bec4810fbcf85baf7421c665c2a30` and production state `3bcb7a8f8671f578165a72c6939a3b04f67f75bd`.
- AT-39 through AT-44 are specified but remain unexecuted until the isolated Stage 12.10 run.
- Stage 12.7 remains pending and was not started.

### Stage 12.7 — Update 5: Subject and Angle Cooldown

Status: COMPLETE

Add subject_key for new facts:

    "subject_key": "grand_canyon"

Rules:

- the same subject cannot be reused within the 20 most recent posts;
- one semantic subject cluster may appear at most twice within those 20 posts;
- the current batch, all active reservations, and recent published posts are included;
- subject similarity uses subject_key, subject, relationship, canonical claim, tags, and semantic comparison;
- legacy facts without subject_key use the existing fields as fallback;
- named-series override requires explicit user instruction and persisted generation_audit evidence;
- cooldown rejection counts as repetitive;
- no legacy bulk migration is required.

Completion evidence:

- `c80ce419ad8375a1c7a309119c4c874691e4aaee` separates permanent duplicate prevention from temporary subject/cluster cooldown and defines the named-series exception boundary.
- `891d6c2b16829444018a0a47c2e74da7f2c24606` adds subject_key, cooldown_audit, the 20-post plus active/batch comparison scope, legacy fallback, override evidence, audit rules, and lifecycle compatibility to the data contract.
- `862ed1fc28ed2601aff009b46913a8bfafed1b90` makes cooldown part of hard_rules_passed and validates stored cooldown evidence during Fast Approval without running a new global check.
- `66c744264c678409db85098f815118b9640dbd26` implements subject-key derivation, duplicate-before-cooldown ordering, generation/replacement checks, repetitive rejection accounting, override handling, Fast Approval preservation, and audit behavior.
- `c441f994f420759a57b3d94060fb3078abc8403e` ensures the weakest-fact challenge reruns cooldown checks after replacement.
- `97bfd2f91a62a8e3d7e56585a6bb1700d740daa0` adds global cooldown gates and AT-45 through AT-48 for exact-subject rejection, semantic-cluster limits, legacy fallback, and explicit named-series override evidence.
- Every newly created or replaced fact requires a stable lowercase snake_case subject_key; legacy missing keys remain valid and use an in-memory fallback without repository rewrite.
- Permanent exact and semantic duplicate rejection runs before cooldown and cannot be bypassed by a series override.
- Cooldown spans the current post and requested batch, every active reservation, and fact records linked to the 20 most recent archived posts.
- Exact subject reuse is blocked and one narrow semantic cluster may appear in at most two distinct posts unless an explicit named-series override covers the required final positions.
- Cooldown rejection uses rejected_counts.repetitive and new posts persist compact cooldown_audit evidence; no rejected candidate wording or private reasoning is stored.
- Production advanced through intermediate snapshots while the stage was open; final synchronization commits `a2ab2ff5e64b124fb8dbe0a604cbba29f432c210` and `ee1c74abdec6de0e56d5d0870d21e7cf7d45600b` copy the completed batch snapshots to the feature branch without writing to main.
- Protected feature snapshots are byte-identical to main: active drafts `192c3da563ebdb2e3b21d663b04ab21bfaefada3` and production state `6f4ab56799024273d6d63c7272b695c4a76e402e`.
- Static validation passed for schema examples, sequential workflow numbering, duplicate-before-cooldown order, 576 legacy fact snapshots remaining unmodified, AT-45 through AT-48, JSON/JSONL parsing, whitespace, ready-queue parity, and protected-file parity.
- AT-45 through AT-48 are specified but remain unexecuted until the isolated Stage 12.10 run.
- Stage 12.8 remains pending and was not started.

### Stage 12.8 — Update 6: Complete Publishing Package

Status: COMPLETE

Add to active post records:

    "caption": "Nature has a talent for making the impossible look ordinary.",
    "hashtags": [
      "#DidYouKnow",
      "#AmazingFacts",
      "#AnimalFacts",
      "#NatureFacts",
      "#LearnSomethingNew"
    ]

Caption rules:

- exactly one sentence;
- ideal length 6–14 words;
- natural American English;
- concise, clear, and attractive;
- not a generic question;
- no "Which fact surprised you?";
- no prose use of trivia;
- no "Here are six facts";
- no restatement of the six facts;
- no new factual claim;
- no duplicate CTA;
- no citation;
- no emoji by default.

Hashtag rules:

- 4–6 unique tags;
- relevant to topic, post format, and supported country scope;
- no misleading, unrelated, or spam tags;
- PascalCase when appropriate;
- #Trivia is allowed as a hashtag;
- hashtags never enter the on-screen script.

Behavior:

- generate caption and hashtags after the final six facts pass;
- persist them with the draft;
- approval must not regenerate them;
- wording-only revision may preserve them when still relevant;
- fact replacement, topic change, country change, or post-format change must recheck them;
- archive future posted posts with the same caption and hashtags.

Chat and queue output use two separate plain-text code blocks:

1. ON-SCREEN SCRIPT.
2. FACEBOOK CAPTION containing the caption, one blank line, and the hashtags.

Completion evidence:

- Latest production baseline observed before completion remained GitHub `main` commit `af68f8db2d54e6c9d555e66a7c38045cf5f46501`; the feature branch was 57 commits ahead and 0 behind after the Stage 12.8 specification writes.
- `e2ba23744b5b3da3ae3561cd920284ef3880f370` defines the complete publishing-package editorial rules in Content DNA, including caption/hashtag constraints, lifecycle preservation, recheck triggers, two-surface output, and the explicit no-backfill boundary.
- `35c74dfbe80784998025cf656e6051250eefb0ce` extends the additive schema contract with active and archived `caption`/`hashtags`, Fast Approval preservation, deterministic two-block ready rendering, recheck rules, hard failures, audit coverage, and archive parity.
- `b0660262e7f5374b3cbf0d80878a9659a29235e1` implements runtime generation, persistence, revision/replacement rechecks, Fast Approval no-regeneration behavior, archival preservation, consistency checks, and separate ON-SCREEN SCRIPT / FACEBOOK CAPTION chat output.
- `9b8588bb01d317f78cc387c9885bdee02cc87e94` adds publishing-package global gates and AT-49 through AT-53 for generation/style validation, recheck triggers, Fast Approval preservation, ready/chat two-block parity, and archive parity.
- Static validation passed for required package sections, generation-before-ID ordering, Fast Approval no-regeneration, fact/topic/country/format recheck coverage, separate code-block surfaces, archive preservation, no-backfill instructions, AT-49 through AT-53 PENDING status, acceptance-range updates, and trailing-whitespace checks.
- JSON parsing passed for `data/production-state.json`, `data/performance-summary.json`, and `data/publishing-plan.json`. JSONL parsing passed for 96 active records, all six fact ledgers (6 published fact records total), and the one monthly archive record. No raw performance JSONL file exists yet.
- Protected files remained byte-identical to `main`: production state blob `6f4ab56799024273d6d63c7272b695c4a76e402e`, active drafts blob `192c3da563ebdb2e3b21d663b04ab21bfaefada3`, ready queue blob `99249fa32468766e9380f0172ca22147b9f3d76a`, every fact-ledger blob, and archive blob `7dac2f275f23083c00d8f2b4ba5cc67a1def3515` all matched `main`.
- The feature diff contains no `data/production-state.json`, `data/active-drafts.jsonl`, `data/facts/**`, `data/posts/**`, or `output/ready-to-post.md` changes.
- No active production post was backfilled and no mutative acceptance test was executed. AT-49 through AT-53 remain specification-only until Stage 12.10.
- `main` was not modified. Stage 12.9 remains pending and was not started.

### Stage 12.9 — Compatibility and Initial Files

Status: COMPLETE

Compatibility rules:

- missing post_format means themed;
- missing caption/hashtags is allowed only for archived legacy content and active content awaiting controlled backfill;
- missing subject_key uses legacy fallback;
- existing IDs, counters, facts, sources, quality, audits, and archives are preserved;
- performance data and publishing plan may begin empty;
- schema_version remains 1.

Create the empty performance summary, publishing plan, content calendar, and test-plugin guide on the feature branch. Do not modify production snapshots.

Completion evidence:

- Latest production baseline observed during Stage 12.9 remained GitHub `main` commit `af68f8db2d54e6c9d555e66a7c38045cf5f46501`; before the closing plan write the feature branch was 62 commits ahead and 0 behind.
- `f0d8e3a2f709b2d5ad2227612c95de5cd0f957bd` defines the explicit Stage 12.9 additive compatibility boundary in `system/data-contract.md`: missing `post_format` means themed, legacy missing `subject_key` uses fallback, permitted missing publishing-package fields remain readable, existing IDs/counters/content/audits/archives are preserved, empty additive stores are valid, and schema_version remains 1.
- `d2fcd84b339ab19f62f2beee229dd91dfd6f97a4` implements the same compatibility and migration-safety behavior in `system/gpt-instructions.md`, including no implicit backfill, no compatibility-only renumbering/rescoring/rewriting, and valid empty performance/scheduling baselines.
- `d98b4caf3af0bd4cd81fdedd2ec681696ec52fa4` adds AT-54 and AT-55 for legacy publishing-package compatibility and empty additive-store validity. Both remain PENDING until Stage 12.10.
- `9b8602506b6543925200bcecf21fa92f21109680` creates `docs/test-plugin-installation.md` with the isolated test runtime profile, explicit-ref rules, Plugin Creator bootstrap prompt, version/read/write smoke checks, explicit main-rejection check, acceptance-test ordering, troubleshooting, cleanup boundaries, and the instruction never to merge the test branch.
- The existing empty initial files were reused without churn: `data/performance-summary.json` blob `7015c52e15104099a6b1d84a30664b6f9e6ce96b`, `data/publishing-plan.json` blob `a3843d9c7737d37f83a5928a3dd8c4f26bc82d10`, and `output/content-calendar.md` blob `d548070e221e5157692b76edb9fdfd2fff423377` exactly match their canonical empty structures.
- JSON parsing passed for `data/production-state.json`, `data/performance-summary.json`, and `data/publishing-plan.json`. JSONL parsing passed for 96 active records, all six fact ledgers with 6 published fact records total, and the one monthly archive record.
- Static validation passed for every Stage 12.9 compatibility rule, empty-store canonical bytes, test runtime profile, no feature-branch mutative testing, never-merge boundary, AT-54/AT-55 PENDING status, acceptance-range updates, and trailing whitespace.
- Protected production paths remain absent from the feature diff: `data/production-state.json`, `data/active-drafts.jsonl`, `data/facts/**`, `data/posts/**`, and `output/ready-to-post.md`. Their verified blobs remained identical to `main`.
- Search confirmed that `test/viral-producer-v1.1` does not yet exist. Stage 12.9 did not create the test branch or run any mutative acceptance test.
- No caption/hashtag backfill was performed and `main` was not modified.
- Stage 12.10 remains pending and was not started.

### Stage 12.10 — Isolated Acceptance Tests

Status: COMPLETE — PRE-CUTOVER GATE PASSED; AT-24 DEFERRED TO STAGE 12.12

Create test/viral-producer-v1.1 from the current feature branch. Configure a new private plugin named Viral Producer v1.1 Test with the test runtime profile.

Never use the feature branch directly for mutative tests. Never merge test data.

Stage 12.10 is the isolated pre-cutover gate. It covers AT-25 through AT-55 (31 tests) plus the final full consistency audit on the test branch. AT-24 is excluded from this stage because it validates the installed production profile on merged `main`; it is a mandatory post-cutover smoke test in Stage 12.12.

Required test groups:

Runtime isolation:

- AT-25: test plugin reads and writes only the test branch and refuses `main`;
- AT-26: production profile refuses the test branch;
- AT-27: all repository calls use explicit repository/ref and reject mismatched responses;
- AT-24 is not executed here; it runs against the refreshed production plugin during Stage 12.12.

Fast Approval:

- zero web/source calls;
- zero new IDs;
- counters unchanged;
- one active write, one queue write, one state write;
- exact queue parity;
- legacy/incomplete rejection.

Mixed:

- minimum four fact topics;
- maximum two per fact topic;
- correct post topic and fact topics;
- correct multi-ledger publication routing;
- themed backward compatibility.

Performance:

- posted-only enforcement;
- valid snapshots;
- idempotent retry;
- conflict rejection;
- deterministic summary;
- small-sample restraint.

Smart queue/calendar:

- read-only recommendation;
- ready-only scheduling;
- no duplicate schedule;
- Asia/Jakarta handling;
- no lifecycle mutation from scheduling;
- posted cleanup.

Cooldown:

- exact subject rejection;
- semantic cluster limit;
- legacy fallback;
- explicit series override evidence.

Publishing package:

- caption length and style;
- no generic trivia prose or new factual claim;
- 4–6 unique relevant hashtags;
- no hashtags in script;
- active/queue/chat/archive parity;
- approval preservation.

Compatibility and initial stores:

- permitted legacy missing caption/hashtags remains readable with zero implicit backfill;
- missing post_format remains themed without rewrite;
- missing subject_key continues to use legacy fallback;
- empty performance summary is valid;
- absent raw performance files before first metrics write are valid;
- revision-0 empty publishing plan and deterministic empty calendar are valid;
- existing IDs, counters, facts, sources, quality, audits, ledgers, and archives remain preserved.

Regression:

- rerun every affected create, revise, replace, approve, ready, posted, persistence, conflict, recovery, legacy, and final consistency test.

Completion evidence:

- `test/viral-producer-v1.1` recorded PASS for AT-25 through AT-55: 31 of 31 pre-cutover Stage 12 tests.
- The final full consistency audit on the isolated test branch passed after the Stage 12.10 test run, with no unresolved partial state accepted as complete.
- The production-side AT-24 attempt remained unclassified because the installed production plugin and `main` still exposed Stage 10 versions; this is the expected pre-cutover state, not an acceptance failure.
- AT-24 remains mandatory for the Stage 12 Definition of Done and must pass after merge plus production plugin v1.1 refresh, before any backfill or production resumption.
- Test fixtures, counters, snapshots, and commits remain confined to the disposable test branch and are never merged or copied into the feature branch or `main`.

### Stage 12.11 — Documentation and Plugin Guide

Status: COMPLETE

Update README, user guide, prompt library, production installation guide, and plan. Add docs/test-plugin-installation.md with:

- Plugin Creator prompt;
- test runtime profile;
- GitHub connection;
- version check;
- branch read/write smoke test;
- explicit main rejection test;
- acceptance-test sequence;
- troubleshooting;
- instruction never to merge the test branch.

Documentation language is concise natural Indonesian except technical identifiers.

Completion evidence:

- `c691969d5edb6b55063c9f2bbc67663238db8ddd` updates all Stage 12.11 documentation deliverables: `README.md`, `docs/user-guide.md`, `docs/prompt-library.md`, `docs/gpt-installation.md`, and `docs/test-plugin-installation.md`.
- README, user guide, and prompt library now document the Version 1.1 production workflow: themed/mixed generation, Fast Approval, subject/angle cooldown, two-surface publishing package, smart recommendation, content calendar, performance feedback, legacy compatibility, and serialized write safety.
- Production documentation clearly targets **Viral Producer** on `main`; test documentation clearly targets **Viral Producer v1.1 Test** on `test/viral-producer-v1.1`. Mutative acceptance testing is explicitly excluded from the production plugin.
- Production installation guidance now reports Instruction, Editorial, and Test Specification versions as `3.0 — Stage 12`, keeps the production runtime profile on `main`, and points test-plugin setup to the separate test guide.
- Test-plugin guidance is updated to Stage 12.11, records the completed Stage 12.10 result of AT-25 through AT-55 PASS (31/31), preserves the never-merge test-data boundary, and treats future test activity as an isolated authorized rerun.
- AT-24 remains consistently classified as the mandatory Stage 12.12 post-cutover production smoke test: merge reviewed PR → refresh production plugin to v1.1 → new chat/version check → AT-24 PASS → controlled backfill → final production audit → resume production.
- Documentation validation passed for balanced Markdown fences, final newlines, no trailing whitespace, production/test runtime separation, current Editorial version, Stage 12.10 evidence wording, and AT-24 ordering.
- The documentation commit changes only the five documentation files above. No `data/**`, `output/**`, `system/**`, or acceptance-test file changed in Stage 12.11.
- During validation, `main` remained at `af68f8db2d54e6c9d555e66a7c38045cf5f46501` and `test/viral-producer-v1.1` remained at `07eeb5831a852ff6df384c230f3a14ed85aa5c18`; neither branch was modified.
- Stage 12.12 remains PENDING and was not started.

### Stage 12.12 — Pull Request, Cutover, and Backfill

Status: IN PROGRESS — PRE-PR AUDIT COMPLETE; CUTOVER NOT STARTED

Before PR:

1. Re-read and synchronize with latest main.
2. Confirm the feature diff contains no stale production snapshots.
3. Confirm test data is absent.
4. Run final read-only consistency and compatibility audit.
5. Open PR only from upgrade/viral-producer-v1.1 to main.
6. Attach the PR to the task.
7. Do not merge until the Stage 12.10 pre-cutover gate passes (AT-25 through AT-55 plus the isolated test-branch final consistency audit) and the user approves cutover. AT-24 is intentionally not a pre-merge gate.

Pre-PR checkpoint — 2026-10-02:

- Feature checkpoint before this plan update: `1f931b92d66e92e8cf7987a60b5418798cd3dbfa`.
- Latest production `main`: `af68f8db2d54e6c9d555e66a7c38045cf5f46501`.
- Latest isolated test branch observed read-only: `07eeb5831a852ff6df384c230f3a14ed85aa5c18`.
- Git compare reports the feature branch 66 commits ahead and 0 behind `main`; `main` is the exact merge base, so no synchronization commit is required.
- Full feature core files were re-read: `plan.md`, `system/gpt-instructions.md`, `system/content-dna.md`, `system/data-contract.md`, and `tests/acceptance-tests.md`.
- The feature diff contains 13 implementation/documentation paths only: `README.md`, `data/performance-summary.json`, `data/publishing-plan.json`, `docs/gpt-installation.md`, `docs/prompt-library.md`, `docs/test-plugin-installation.md`, `docs/user-guide.md`, `output/content-calendar.md`, `plan.md`, `system/content-dna.md`, `system/data-contract.md`, `system/gpt-instructions.md`, and `tests/acceptance-tests.md`.
- Protected production snapshots are absent from the diff: no changes to `data/production-state.json`, `data/active-drafts.jsonl`, `data/facts/**`, `data/posts/**`, or `output/ready-to-post.md`.
- The three additive Stage 12 initial stores are canonical empty values: performance summary schema version 1 with sample_size 0, publishing plan schema version 1 with Asia/Jakarta/revision 0/no slots, and an empty deterministic content calendar.
- No test fixture, test counter, test performance snapshot, test publishing plan, or test lifecycle data is present in the feature diff.
- Latest `main` read-only consistency audit passed: revision 139, next_post_number 98, next_fact_number 609, single_writer_mode true, 96 active records, 26 ready records, 6 published facts, and 1 archive.
- Main JSON/JSONL parsing, unique Post IDs, unique Fact IDs, unique claim signatures, six-fact active/archive records, counter monotonicity, and exact 26-post ready-queue ID parity all passed.
- Compatibility audit passed with the current legacy baseline: 96 active records may omit post_format and publishing package fields pending controlled backfill, 576 active fact snapshots may omit subject_key, and the legacy archive/published facts remain readable without implicit migration.
- Static merge audit passed for current Stage 12 versions, balanced Markdown fences, final newlines, no trailing whitespace, runtime-profile explicit-ref rules, production/test-plugin separation, and no operational `ref: main` hardcoding in active runtime instructions.
- Stage 12.10 remains COMPLETE: AT-25 through AT-55 are PASS (31/31) and the isolated test-branch final consistency audit is PASS.
- Stage 12.11 remains COMPLETE.
- AT-24 status remains **PENDING POST-CUTOVER**. It must run only after the PR is merged and the production plugin is refreshed to v1.1, and it must PASS before controlled backfill or production resumption.
- Neither `main` nor `test/viral-producer-v1.1` was modified during the pre-PR audit.
- Cutover, production-plugin refresh, AT-24 execution, backfill, final production audit, and production resumption have not started.

Cutover requires a short production-write pause:

1. Pause production writers.
2. Merge the reviewed PR.
3. Update the existing production Viral Producer plugin to version 1.1.0.
4. Use the production runtime profile pointing to `main`.
5. Start a fresh conversation.
6. Run version and branch verification read-only.
7. Run AT-24 as the mandatory post-cutover production smoke test and record PASS.
8. If AT-24 does not pass, keep production paused; do not start backfill and do not resume production.
9. Confirm the refreshed production plugin still refuses test-branch overrides.
10. Run controlled active caption/hashtag backfill.
11. Run final full consistency audit on production.
12. Resume production.

Caption/hashtag backfill:

- targets current draft and ready records only;
- excludes archived posts;
- uses latest main content and SHA;
- does not open sources or use Web Search;
- does not change facts, IDs, counters other than one revision increment, quality, generation audit, or sources;
- treats missing post_format as themed without requiring a bulk format rewrite;
- prepares and validates all captions/hashtags before the first write;
- writes active records, rebuilds ready queue, increments revision once, and verifies parity.

### Definition of Done for Stage 12

Stage 12 is complete only when:

- production and test runtime profiles are isolated;
- test plugin cannot write main;
- no test data reaches main;
- Fast Approval performs no web/source recheck;
- mixed posts generate, persist, approve, and publish correctly;
- performance snapshots and deterministic summary work;
- smart recommendation is read-only;
- publishing calendar persists safely;
- subject cooldown works with legacy fallback;
- captions and hashtags are stored and copy-ready;
- active, queue, chat, and archive parity pass;
- old records remain readable;
- IDs and counters remain monotonic;
- all Stage 12.10 pre-cutover tests (AT-25 through AT-55) and the isolated test-branch final consistency audit pass;
- AT-24 passes as the mandatory Stage 12.12 post-cutover smoke test before backfill or production resumption;
- final production consistency audit passes;
- documentation and plugin guides are complete;
- production backfill completes successfully;
- production resumes on main.

### Execution Protocol for a New Chat

The implementation chat must:

1. Read this entire plan and the live Viral Producer skill.
2. Read complete core files from current main and complete plan.md from the feature branch.
3. Use current repository evidence, not conversation memory.
4. Start only with Stage 12.1.
5. Work one stage at a time.
6. Update this plan after every completed stage.
7. Record commits and validation evidence.
8. Stop after each stage and wait for the user to say "lanjutkan".
9. Never create a duplicate feature branch.
10. Never run mutative tests on main.
11. Never create a PR or merge before all acceptance tests pass.

Suggested first prompt for the new chat:

    Implementasikan Stage 12.1 dari plan.md pada repository milyarderpro/viral-producer. Main masih aktif digunakan untuk produksi. Baca core files terbaru dari main, baca seluruh Stage 12 pada branch upgrade/viral-producer-v1.1, refresh baseline, sinkronkan feature branch secara aman jika diperlukan, pastikan production data tidak berubah, update plan.md, commit hasilnya, lalu berhenti. Jangan mengerjakan Stage 12.2 sebelum saya mengatakan "lanjutkan".

## 6. Definition of Done

The implementation is complete when:

- all planned files exist on `main`;
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
- Opened and reviewed pull request #1, confirmed it was mergeable with no blocking findings, merged the validated implementation into `main`, and marked Stage 10 complete.
- Promoted `main` to the sole production runtime branch, retired operational use of `build/viral-content-gpt`, and prepared the renamed Viral Producer 1.0.0 plugin package.
- Started Stage 11 on `docs/user-guide` and added a concise Indonesian user guide without changing production data.
- Added the Stage 11.3 prompt library with clearly labeled read-only and repository-writing commands.
- Completed Stage 11.4 with concise README navigation and two-way links between the user guide and prompt library.
- Completed Stage 11.5 documentation QA: verified navigation, links, prompt intent coverage, Markdown structure, branch references, and production isolation; removed obsolete Stage 10.7 instructions and restricted mutative acceptance tests to isolated test branches.
- Completed Stage 11.6 after the user reviewed and accepted the concise Indonesian documentation; no production test or data mutation was required.
- Merged pull request #3, published the documentation on `main`, and verified that production state and active drafts were unchanged.
