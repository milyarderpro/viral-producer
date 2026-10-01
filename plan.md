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

Status: APPROVED — IMPLEMENTATION NOT STARTED

Branch implementasi:

    upgrade/viral-producer-v1.1

Branch acceptance test:

    test/viral-producer-v1.1

Production runtime tetap menggunakan main. Branch implementasi dibuat dari main saat revision 100, next_post_number 59, next_fact_number 375, dan 57 active records. Nilai tersebut hanya baseline pembuatan branch; chat implementasi wajib membaca ulang main terbaru sebelum bekerja karena produksi terus berjalan.

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

Status: READY FOR IMPLEMENTATION

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

### Stage 12.2 — Runtime Branch Abstraction

Status: PENDING

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

### Stage 12.3 — Update 1: Fast Approval

Status: PENDING

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

### Stage 12.4 — Update 2: Mixed-Topic Support

Status: PENDING

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

### Stage 12.5 — Update 3: Performance Feedback Loop

Status: PENDING

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

### Stage 12.6 — Update 4: Smart Ready Queue and Content Calendar

Status: PENDING

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

### Stage 12.7 — Update 5: Subject and Angle Cooldown

Status: PENDING

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

### Stage 12.8 — Update 6: Complete Publishing Package

Status: PENDING

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

### Stage 12.9 — Compatibility and Initial Files

Status: PENDING

Compatibility rules:

- missing post_format means themed;
- missing caption/hashtags is allowed only for archived legacy content and active content awaiting controlled backfill;
- missing subject_key uses legacy fallback;
- existing IDs, counters, facts, sources, quality, audits, and archives are preserved;
- performance data and publishing plan may begin empty;
- schema_version remains 1.

Create the empty performance summary, publishing plan, content calendar, and test-plugin guide on the feature branch. Do not modify production snapshots.

### Stage 12.10 — Isolated Acceptance Tests

Status: PENDING

Create test/viral-producer-v1.1 from the current feature branch. Configure a new private plugin named Viral Producer v1.1 Test with the test runtime profile.

Never use the feature branch directly for mutative tests. Never merge test data.

Required test groups:

Runtime isolation:

- test plugin reads and writes only the test branch;
- test plugin refuses main;
- production profile refuses test;
- no implicit default ref.

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

Regression:

- rerun every affected create, revise, replace, approve, ready, posted, persistence, conflict, recovery, legacy, and final consistency test.

### Stage 12.11 — Documentation and Plugin Guide

Status: PENDING

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

### Stage 12.12 — Pull Request, Cutover, and Backfill

Status: PENDING

Before PR:

1. Re-read and synchronize with latest main.
2. Confirm the feature diff contains no stale production snapshots.
3. Confirm test data is absent.
4. Run final read-only consistency and compatibility audit.
5. Open PR only from upgrade/viral-producer-v1.1 to main.
6. Attach the PR to the task.
7. Do not merge until acceptance results pass and the user approves cutover.

Cutover requires a short production-write pause:

1. Pause production writers.
2. Merge the reviewed PR.
3. Update the existing production Viral Producer plugin to version 1.1.0.
4. Use the production runtime profile pointing to main.
5. Start a fresh conversation.
6. Run version and branch verification read-only.
7. Confirm the production plugin refuses test branch writes.
8. Run controlled active caption/hashtag backfill.
9. Run final full consistency audit.
10. Resume production.

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
- all new and affected regression tests pass;
- final consistency audit passes;
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
