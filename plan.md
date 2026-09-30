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

- one Custom GPT as the content producer;
- GitHub as the permanent source of truth;
- GitHub App for repository reads and writes;
- Web Search for current research and verification;
- no external database, backend, or custom Action unless testing proves it is required.

Fallback order:

1. Custom GPT with GitHub App.
2. Plugin or skill with GitHub App.
3. Custom GPT Action or MCP integration only if direct GitHub writes are unavailable.

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

Status: NEXT

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

### Stage 6 — Command interface

Status: PENDING

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

### Stage 7 — Ready-to-post output

Status: PENDING

Approval must:

- preserve the structured draft record;
- add clean copy to output/ready-to-post.md;
- show the same clean copy in chat.

Marking a post as posted must:

- add its facts to the published fact indexes;
- add the post to data/posts/YYYY-MM.jsonl;
- remove it from the ready queue;
- remove it from active drafts;
- update production state.

### Stage 8 — Acceptance tests

Status: PENDING

Create:

    tests/acceptance-tests.md

Minimum test coverage:

1. Read-only state inspection.
2. Default post generation.
3. Country-specific generation.
4. Exact duplicate rejection.
5. Paraphrase duplicate rejection.
6. Unsafe fact rejection.
7. Full draft-to-posted lifecycle.
8. Persistence across a new conversation.
9. Git SHA conflict handling.
10. Partial-write consistency recovery.

### Stage 9 — Manual installation guide

Status: PENDING

Create:

    docs/gpt-installation.md

The guide must cover:

- connecting GitHub;
- limiting access to viral-producer;
- verifying read and write permissions separately;
- opening GPT Builder;
- creating and naming the GPT;
- copying system/gpt-instructions.md;
- enabling Web Search;
- adding GitHub App;
- configuring conversation starters;
- keeping the first version private;
- read, write, duplicate, and lifecycle tests;
- checking repository changes;
- troubleshooting permissions and connection failures;
- plugin or Action fallback when required.

### Stage 10 — Live test and refinement

Status: PENDING

Process:

1. Install the GPT as private.
2. Run the saved acceptance prompts.
3. Inspect repository changes.
4. Correct instructions or schemas when needed.
5. Repeat all affected tests.
6. Open and review the implementation pull request.
7. Merge only after all required tests pass.

## 6. Definition of Done

The implementation is complete when:

- all planned files exist on build/viral-content-gpt;
- all JSON and JSONL files validate;
- the GPT can read and write the repository;
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
