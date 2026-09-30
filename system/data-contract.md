# Viral Producer Data Contract

## 1. Purpose

This document defines the machine-readable data model, file ownership rules, identifiers, lifecycle transitions, validation requirements, and recovery behavior for Viral Producer.

All automation must follow this contract before reading or writing production data.

The goals are:

- prevent duplicate facts;
- preserve reliable publication history;
- keep the repository understandable to humans;
- avoid thousands of individual post files;
- make writes idempotent and recoverable;
- keep the first version simple enough for a single producer.

## 2. General Conventions

### Encoding and line endings

- Use UTF-8.
- Use LF line endings.
- End Markdown and JSON files with a newline.
- JSONL means one complete JSON object per physical line.
- Do not put comments, trailing commas, or blank records inside JSONL files.
- Keep object keys in the documented order when practical.

### Date and time

- Use ISO 8601 timestamps.
- Store full operational timestamps in UTC with a trailing Z.
- Use the Asia/Jakarta timezone to determine the monthly archive file.
- Store date-only values as YYYY-MM-DD.
- Never infer a missing timestamp from a Git commit date.

Example:

    2026-09-30T14:25:00Z

### Schema version

Every JSON or JSONL record must include:

    "schema_version": 1

A future incompatible structure must increment the schema version. Do not silently reinterpret an older record.

### Additive editorial compatibility

Stage 10.3 adds optional fields without changing the meaning of existing fields, so schema_version remains 1.

Compatibility rules:

- records created before editorial version 2 remain valid legacy records;
- a missing editorial audit field in a legacy draft is not a JSON or schema corruption;
- every newly created draft must include all editorial version 2 fields defined below;
- a legacy draft may remain unchanged while stored as draft;
- before a legacy draft may become approved or ready, it must be re-evaluated under the current content DNA and enriched with the complete editorial version 2 fields;
- a substantively revised legacy draft must be upgraded during the same operation;
- readers must tolerate unknown additive fields and writers must preserve fields they do not modify.

A substantive revision includes replacing a fact, changing a canonical claim or source basis, changing fact order for editorial reasons, or rerunning the post-level quality decision. A correction limited to spelling, punctuation, or whitespace may remain legacy, but it does not make the post eligible for approval.

## 3. Repository Data Map

### Rules

    system/content-dna.md
    system/gpt-instructions.md
    system/data-contract.md

### Operational state

    data/production-state.json
    data/active-drafts.jsonl

### Published and blocked fact indexes

    data/facts/animals-nature.jsonl
    data/facts/body-science.jsonl
    data/facts/food-home.jsonl
    data/facts/geography-history.jsonl
    data/facts/inventions-records.jsonl
    data/facts/practical.jsonl

### Published post archives

    data/posts/YYYY-MM.jsonl

### Human copy queue

    output/ready-to-post.md

## 4. Source-of-Truth Rules

Each type of information has exactly one authoritative location.

- Editorial rules: system/content-dna.md
- Data rules: system/data-contract.md
- Next identifiers and recent rotation state: data/production-state.json
- Draft, approved, and ready posts: data/active-drafts.jsonl
- Reserved fact signatures: fact snapshots inside data/active-drafts.jsonl
- Published or permanently blocked facts: data/facts/*.jsonl
- Posted scripts: data/posts/YYYY-MM.jsonl
- Copy-friendly queue: output/ready-to-post.md

The Markdown ready queue is a derived view, not a database. If it disagrees with active-drafts.jsonl, regenerate it from active drafts.

Do not use conversation memory as a source of truth.

## 5. Identifier Format

### Post ID

Use six digits:

    P-000001
    P-000002

The numeric portion comes from production-state.json field next_post_number.

### Fact ID

Use six digits:

    F-000001
    F-000002

The numeric portion comes from production-state.json field next_fact_number.

### Rules

- IDs are case-sensitive.
- IDs are never reused.
- Gaps are allowed.
- Rejecting a draft does not return its IDs to the available pool.
- Changing surface wording does not create a new fact ID.
- Replacing the underlying claim creates a new fact ID.
- A published post ID and fact ID are immutable.

## 6. Production State Schema

Path:

    data/production-state.json

Required structure:

    {
      "schema_version": 1,
      "revision": 0,
      "next_post_number": 1,
      "next_fact_number": 1,
      "operational_timezone": "Asia/Jakarta",
      "single_writer_mode": true,
      "last_topic": null,
      "last_country_focus": null,
      "recent_topics": [],
      "recent_country_focuses": [],
      "updated_at": null
    }

### Field rules

- revision increases by exactly one after every completed state-changing operation.
- next_post_number points to the next unused post number.
- next_fact_number points to the next unused fact number.
- recent_topics stores no more than the 10 most recently created post topics.
- recent_country_focuses stores no more than the 10 most recent country focuses.
- Use null for no previous value.
- Do not decrease counters or revision.
- single_writer_mode remains true in version 1.

## 7. Topic Values and File Routing

Allowed topic values:

- animals-nature
- body-science
- food-home
- geography-history
- inventions-records
- practical

Route facts to the file whose name matches the topic.

A fact belongs to one primary topic only. Secondary themes may be stored in tags, but they do not change file routing.

If a fact genuinely does not fit an allowed topic, do not invent a new topic during production. Stop and request a schema update.

## 8. Country Scope

Allowed country codes:

- US
- CA
- UK
- AU
- GLOBAL
- OTHER

country_scope is always an array because one fact may apply to multiple countries.

Examples:

    ["US"]
    ["CA", "US"]
    ["GLOBAL"]

Use OTHER only when a country-specific fact falls outside the four target markets. Record the country name in tags or canonical_claim.

## 9. Claim Signature

claim_signature is a short, human-readable semantic identity for the underlying fact.

Format:

    subject|relationship|object_or_result|scope

Example:

    polar_bear|skin_color|black|species
    ohio_flag|shape|non_rectangular|us_states
    first_alarm_clock|ring_time|4_am|documented_design

Normalization rules:

- lowercase;
- use underscores instead of spaces;
- use singular nouns when natural;
- remove articles;
- describe meaning rather than surface wording;
- include a scope when it changes truth;
- include a time period only when it defines the claim;
- do not include promotional adjectives.

The signature is a deduplication aid, not the only duplicate test. A semantic comparison of canonical claims is still required.

Two candidates are duplicates when they communicate the same underlying relationship, even if their signatures or wording differ.

## 10. Source Object

Every persisted fact must contain at least one source object.

Required source structure:

    {
      "url": "https://example.gov/page",
      "title": "Source page title",
      "publisher": "Publisher name",
      "source_type": "government",
      "accessed_at": "2026-09-30T14:25:00Z",
      "supports": "Short description of the part of the claim this source supports"
    }

Allowed source_type values:

- government
- university
- museum
- scientific-organization
- peer-reviewed
- official-organization
- reference
- reputable-reporting

Do not persist a social post, trivia list, search result page, or AI answer as the authoritative source.

For a time-sensitive claim, add valid_as_of to the fact record.

## 11. Fact Snapshot in Active Drafts

Candidate facts are transient and are not written to the repository.

When a draft is saved, its six accepted facts become reserved through fact snapshots stored inside the draft record. A reserved fact does not yet appear in the published fact index.

Required fact snapshot for a newly created or editorial-version-2-upgraded draft:

    {
      "fact_id": "F-000001",
      "claim_signature": "polar_bear|skin_color|black|species",
      "canonical_claim": "Polar bears have black skin beneath their fur.",
      "subject": "polar bear",
      "relationship": "skin color",
      "object_or_result": "black",
      "topic": "animals-nature",
      "country_scope": ["GLOBAL"],
      "surface_text": "A polar bear's skin is black beneath its thick, transparent-looking fur",
      "word_count": 12,
      "surprise_operator": "visual_biology",
      "viral_strength": 2,
      "scope_check_passed": true,
      "source_access_passed": true,
      "sources": [
        {
          "url": "https://example.gov/page",
          "title": "Source page title",
          "publisher": "Publisher name",
          "source_type": "government",
          "accessed_at": "2026-09-30T14:25:00Z",
          "supports": "Confirms skin color beneath the fur"
        }
      ],
      "verified_at": "2026-09-30T14:25:00Z",
      "valid_as_of": null,
      "risk_level": "low",
      "tags": ["animal anatomy"]
    }

Allowed surprise_operator values:

- belief_reversal
- hidden_mechanism
- visual_biology
- scale_number
- historical_origin
- geographic_quirk
- everyday_consequence
- record_superlative

viral_strength must be the integer 0, 1, or 2 using the anchors in system/content-dna.md.

scope_check_passed may be true only after comparing the canonical claim, surface text, and evidence for subject, relationship, geography, time, quantity, qualifier, and record category. A broader or stronger surface statement must set this to false and block persistence.

source_access_passed may be true only when at least one persisted authoritative source was opened and its readable content directly supports the final claim. If another source is restricted by login, paywall, CAPTCHA, expired link, or unreadable format, an accessible authoritative fallback is required.

Allowed risk_level values:

- low
- medium
- high

High-risk facts must not enter ordinary production. Medium-risk facts require stronger verification and must not contain actionable medical, survival, emergency, legal, or safety instructions.

Legacy fact snapshots may omit the four editorial fields while their parent record remains an unchanged legacy draft. They must receive the fields before the parent post becomes eligible for approval.

## 12. Active Draft Record

Path:

    data/active-drafts.jsonl

There must be at most one current record per post_id.

Required structure for a newly created or editorial-version-2-upgraded draft:

    {
      "schema_version": 1,
      "post_id": "P-000001",
      "status": "draft",
      "topic": "animals-nature",
      "country_focus": "GLOBAL",
      "hook": "Did you know?",
      "facts": [],
      "cta": "Enjoyed these facts? Like the video and follow for more!",
      "quality": {
        "opening_strength": 2,
        "surprise_quality": 2,
        "concrete_detail": 2,
        "shareability": 2,
        "readability": 2,
        "factual_confidence": 2,
        "total": 12,
        "hard_rules_passed": true,
        "rationales": {
          "opening_strength": "Fact 1 is strength 2, immediate, and one of the three strongest facts.",
          "surprise_quality": "Five facts are strength 2 across five distinct operators.",
          "concrete_detail": "Every fact contains a memorable image, number, mechanism, or consequence.",
          "shareability": "At least four facts create a clear tell-someone reaction.",
          "readability": "All six lines are natural, balanced, and within the word limits.",
          "factual_confidence": "All final claims preserve scope and have directly readable authoritative support."
        }
      },
      "generation_audit": {
        "candidate_count": 20,
        "rejected_counts": {
          "duplicate": 2,
          "weak": 6,
          "scope": 1,
          "source": 2,
          "unsafe": 0,
          "repetitive": 3,
          "other": 0
        },
        "operator_variety": 5,
        "weakest_fact_review": [
          {
            "fact_position": 2,
            "result": "retained",
            "reason": "Adds a familiar human connection while remaining specific and surprising."
          },
          {
            "fact_position": 4,
            "result": "replacement_passed",
            "reason": "The original filler was replaced and the final fact passed a second review."
          }
        ]
      },
      "created_at": "2026-09-30T14:25:00Z",
      "updated_at": "2026-09-30T14:25:00Z",
      "approved_at": null,
      "ready_at": null
    }

The facts array must contain exactly six fact snapshots for a standard post.

### Quality rules

- every score must be the integer 0, 1, or 2;
- total must equal the sum of the six dimension scores;
- rationales must contain one non-empty, evidence-based sentence for each scored dimension;
- rationales must explain why the visible script earns the score, not merely repeat the number;
- hard_rules_passed may be true only when every formatting, safety, duplicate, operator-diversity, viral-strength, source-access, and scope rule passes.

### Generation audit rules

candidate_count is the number of unique plausible candidate claims actually considered before final selection. It must be at least 18 for a standard six-fact post.

Every non-selected candidate receives exactly one primary rejection reason. Therefore:

    sum(rejected_counts values) = candidate_count - 6

Allowed rejected_counts keys are:

- duplicate
- weak
- scope
- source
- unsafe
- repetitive
- other

operator_variety must equal the number of distinct surprise_operator values across the final six facts. It must be at least 4 and no greater than 6.

weakest_fact_review must:

- contain exactly two entries;
- refer to two different final fact positions from 1 through 6;
- use result retained or replacement_passed;
- contain a concise reason based on the current final fact;
- never store full rejected candidate wording.

generation_audit is compact evidence, not a candidate database. Do not persist rejected candidate claims, complete research notes, or unused source lists.

### Legacy approval boundary

Legacy drafts without complete editorial metadata may remain in status draft. They must not transition to approved or ready.

To upgrade a legacy draft:

1. re-read current sources and duplicate ledgers;
2. evaluate all six facts under the current content DNA;
3. replace weak, repetitive, inaccessible, or scope-mismatched facts when necessary;
4. add the four editorial fields to every fact;
5. add complete quality rationales and generation_audit;
6. pass every current hard rule without changing existing IDs unless an underlying fact is replaced.

Allowed active status values:

- draft
- approved
- ready
- rejected

### Status meaning

- draft: generated and waiting for user review.
- approved: user approved it, but the ready queue has not yet been confirmed.
- ready: present in output/ready-to-post.md and ready to copy.
- rejected: transient state during cleanup; remove it after releasing the reservation.

Posted records do not remain in active-drafts.jsonl.

## 13. Published Fact Record

Path:

    data/facts/{topic}.jsonl

Each fact_id and claim_signature may appear at most once across all fact index files.

Required structure for a fact produced or upgraded under editorial version 2:

    {
      "schema_version": 1,
      "fact_id": "F-000001",
      "status": "published",
      "claim_signature": "polar_bear|skin_color|black|species",
      "canonical_claim": "Polar bears have black skin beneath their fur.",
      "subject": "polar bear",
      "relationship": "skin color",
      "object_or_result": "black",
      "topic": "animals-nature",
      "country_scope": ["GLOBAL"],
      "surface_text": "A polar bear's skin is black beneath its thick, transparent-looking fur",
      "surprise_operator": "visual_biology",
      "viral_strength": 2,
      "scope_check_passed": true,
      "source_access_passed": true,
      "sources": [],
      "verified_at": "2026-09-30T14:25:00Z",
      "valid_as_of": null,
      "risk_level": "low",
      "tags": ["animal anatomy"],
      "published_in": "P-000001",
      "published_at": "2026-10-01T02:00:00Z"
    }

Allowed status values:

- published
- blocked

Use blocked only for a claim that should not be proposed again because it is false, materially misleading, unsafe, or impossible to verify reliably.

A blocked record uses published_in and published_at as null and should include:

    "block_reason": "Concise reason"

Editorial metadata is optional for a blocked record and for a published legacy fact. When the source active draft contains editorial metadata, copy it unchanged into the published fact record.

Do not delete published or blocked fact records.

## 14. Published Post Record

Path:

    data/posts/YYYY-MM.jsonl

The YYYY-MM file is selected using the Asia/Jakarta calendar month of published_at.

Required structure for a post produced or upgraded under editorial version 2:

    {
      "schema_version": 1,
      "post_id": "P-000001",
      "status": "posted",
      "topic": "animals-nature",
      "country_focus": "GLOBAL",
      "hook": "Did you know?",
      "facts": [
        {
          "fact_id": "F-000001",
          "text": "Final published surface text",
          "word_count": 12
        }
      ],
      "cta": "Enjoyed these facts? Like the video and follow for more!",
      "quality_total": 12,
      "quality_rationales": {
        "opening_strength": "Fact 1 is strength 2, immediate, and one of the three strongest facts.",
        "surprise_quality": "Five facts are strength 2 across five distinct operators.",
        "concrete_detail": "Every fact contains a memorable image, number, mechanism, or consequence.",
        "shareability": "At least four facts create a clear tell-someone reaction.",
        "readability": "All six lines are natural, balanced, and within the word limits.",
        "factual_confidence": "All final claims preserve scope and have directly readable authoritative support."
      },
      "generation_audit": {
        "candidate_count": 20,
        "rejected_counts": {
          "duplicate": 2,
          "weak": 6,
          "scope": 1,
          "source": 2,
          "unsafe": 0,
          "repetitive": 3,
          "other": 0
        },
        "operator_variety": 5,
        "weakest_fact_review": [
          {
            "fact_position": 2,
            "result": "retained",
            "reason": "Adds a familiar human connection while remaining specific and surprising."
          },
          {
            "fact_position": 4,
            "result": "replacement_passed",
            "reason": "The original filler was replaced and the final fact passed a second review."
          }
        ]
      },
      "created_at": "2026-09-30T14:25:00Z",
      "approved_at": "2026-09-30T15:00:00Z",
      "published_at": "2026-10-01T02:00:00Z"
    }

The facts array must preserve the exact wording and order used in the published content.

There must be at most one archive record per post_id across all monthly files.

For an editorial-version-2 post, copy quality.rationales into quality_rationales and copy generation_audit unchanged into the archive. Legacy published posts may omit those additive fields.

Published archive records are immutable except to correct proven data corruption. A wording change after publication must be recorded as a new post.

## 15. Ready-to-Post Markdown

Path:

    output/ready-to-post.md

This file is a copy-friendly derived view. data/active-drafts.jsonl remains authoritative.

### Empty queue

When no active records have status ready, the complete file must be:

    # READY TO POST

    No approved posts are waiting to be published.

### Topic display names

Render topic values in headings using this fixed mapping:

- animals-nature → Animals and Nature
- body-science → Body and Everyday Science
- food-home → Food and Household Knowledge
- geography-history → Geography and History
- inventions-records → Inventions, Firsts, and Records
- practical → Safe Practical Knowledge

### Ready block

Render one block for every active record whose status is ready.

Required block:

    ## P-000001 — Animals and Nature

    Did you know?

    [Fact 1]

    [Fact 2]

    [Fact 3]

    [Fact 4]

    [Fact 5]

    [Fact 6]

    Enjoyed these facts? Like the video and follow for more!

    ---

The heading identifies the post but is not part of the Facebook copy.

The clean copy consists only of hook, six surface_text values in stored order, and CTA. Separate each component with exactly one blank line.

Do not include sources, fact IDs, numbering, bullets, quality scores, internal status, audit notes, hashtags, or production guidance in the clean copy.

hook, every surface_text value, and cta must each be a single line without embedded carriage returns or line feeds.

### Deterministic rendering

Every queue-changing operation must rebuild the complete file rather than append, remove, or patch an individual Markdown block.

Rendering procedure:

1. Read the latest active-drafts.jsonl and its Git blob SHA.
2. Select only records whose status is ready.
3. Validate that each selected record has a non-null ready_at, exactly six facts, and all required copy fields.
4. Sort by ready_at ascending, then post_id ascending as the tie-breaker.
5. Render the required header, all ready blocks, and separators.
6. Use UTF-8, LF line endings, and exactly one final newline.
7. Replace output/ready-to-post.md using its latest Git blob SHA.
8. Fetch the result and confirm every ready post appears exactly once and no other post appears.

If validation fails, do not replace a currently valid queue. Report the inconsistent record and repair the authoritative active data first.

### Chat parity

After approval or a SHOW_NEXT_READY request, reconstruct the clean copy from the authoritative active record. The hook, six facts, ordering, punctuation, and CTA shown in chat must exactly match the clean-copy portion of the corresponding queue block.

Place the clean copy in one plain-text code block for convenient copying. Keep the post ID, status, and repository confirmation outside that code block.

## 16. Word Count Rule

Count words by splitting the final visible fact on whitespace.

- A hyphenated expression counts as one word.
- A number counts as one word.
- A contraction counts as one word.
- Punctuation does not create an additional word.
- Store the calculated result in word_count.

Target 12 to 15 words. Accept 11 to 18 words when accuracy or natural phrasing requires it.

## 17. Duplicate Check Order

Before saving a draft, check candidate facts against:

1. Other candidates in the same requested batch.
2. Every fact snapshot in data/active-drafts.jsonl whose status is draft, approved, or ready.
3. Every published or blocked record in data/facts/*.jsonl.
4. Other facts already selected for the same post.

Check both:

- exact claim_signature;
- semantic equivalence of canonical_claim, subject, relationship, and result.

Do not search monthly post archives as the primary duplicate mechanism. The fact indexes are the published-fact source of truth.

If a published post exists without corresponding fact-index records, treat it as an integrity error and repair the index before producing new content.

## 18. Lifecycle Operations

### Create draft

1. Read current production state and its Git blob SHA.
2. Read active drafts.
3. Read all relevant fact index files.
4. Research at least 18 unique plausible candidate claims.
5. Verify, normalize, deduplicate, label operators, and rank candidate viral strength.
6. Select six candidates that pass operator diversity, source access, scope, safety, and quality rules.
7. Challenge the two weakest final facts and complete generation_audit.
8. Allocate one post ID and six fact IDs only after every hard gate passes.
9. Add one complete editorial-version-2 draft record to active-drafts.jsonl.
10. Increment next_post_number by one.
11. Increment next_fact_number by six.
12. Update topic rotation fields.
13. Increment state revision.
14. Report the saved post ID only after GitHub confirms both writes.

Reserved signatures live in the saved draft record.

### Approve draft

1. Confirm the post exists with status draft.
2. Confirm all six facts contain complete editorial metadata.
3. Confirm quality rationales and generation_audit are complete and internally consistent.
4. Re-run current hard validation, including source access and claim-scope checks.
5. If the record is legacy, upgrade it before approval; do not bypass missing fields.
6. Change status to approved and set approved_at.
7. Change status to ready and set ready_at.
8. Regenerate output/ready-to-post.md from active drafts whose status is ready.
9. Confirm the post appears exactly once in the Markdown queue.
10. Increment state revision.
11. Report success only after all writes are confirmed.

active-drafts.jsonl is authoritative. If queue regeneration fails after the status becomes ready, report a partial failure and regenerate the derived Markdown queue before starting another state-changing operation.

### Revise wording

- Keep the same fact ID when only surface_text changes.
- Recalculate word_count, scope_check_passed, quality scores, and rationales.
- Reverify that wording preserves the source claim.
- If the revision reruns the post-level editorial decision, upgrade a legacy record completely.
- Regenerate the ready queue if the post is ready.

### Replace a fact

- Allocate a new fact ID.
- Replace the complete fact snapshot.
- Increment next_fact_number.
- Never reuse the removed fact ID.
- Upgrade the complete post to editorial version 2.
- Re-run duplicate, operator-diversity, viral-strength, source-access, scope, and quality checks.
- Rebuild generation_audit without storing rejected candidate wording.
- Regenerate the ready queue when necessary.

### Reject draft

1. Confirm the post is not posted.
2. Change status to rejected.
3. Regenerate the ready queue without the post.
4. Remove the rejected record from active-drafts.jsonl.
5. Increment state revision.
6. Keep all allocated IDs consumed.

Because the rejected fact snapshots were never added to the published fact index, their underlying claims may be researched again later unless separately blocked.

### Mark as posted

1. Confirm the post exists with status ready.
2. Record one operation-wide published_at timestamp.
3. Upsert its six fact snapshots into the correct fact index files as published records using that timestamp, preserving editorial metadata.
4. Append one immutable post record to the correct monthly archive, preserving quality rationales and generation_audit.
5. Remove the post from active-drafts.jsonl.
6. Regenerate the complete ready queue from the remaining active ready records.
7. Update rotation state if required.
8. Increment state revision.
9. Fetch and confirm the archive record, all six published fact records, absence from active drafts, and absence from the ready queue before reporting success.

This order favors duplicate prevention and keeps the queue derived from active drafts. If an operation stops midway, published fact locks or an archive record may exist before cleanup is complete; the recovery procedure must finish the same transaction with the existing IDs and published_at timestamp rather than create new records or timestamps.

## 19. Idempotency Rules

Every state-changing command must be safe to retry.

- If a draft post_id already exists, do not append a second record.
- If a published fact_id already exists with identical content, treat the upsert as complete.
- If a claim_signature exists under another fact_id, stop and report a duplicate conflict.
- If a monthly post record already exists with identical content, do not append it again.
- If a ready block already exists, replace or preserve it; never duplicate it.
- If Mark as posted is repeated for an archived post, report that it is already posted.

## 20. Concurrency and GitHub Write Safety

Version 1 supports one active writer.

Before replacing an existing GitHub file:

1. Fetch the current file.
2. Record its Git blob SHA.
3. Produce the complete replacement content.
4. Update using the fetched SHA.
5. If GitHub reports a conflict, stop.
6. Fetch the latest version and restart the operation.

Never overwrite after a SHA conflict using stale content.

Do not run two write operations against the same path in parallel.

Read-only research and fact collection may run in parallel, but final allocation and persistence must be serialized.

## 21. Consistency Audit

Run a consistency audit before production when:

- the previous command reported a partial failure;
- a file has been manually edited;
- a Git conflict occurred;
- state counters appear lower than existing IDs;
- the user explicitly requests an audit.

Audit checks for all records:

- every JSON and JSONL record parses;
- no duplicate post IDs;
- no duplicate fact IDs;
- no duplicate claim signatures;
- counters exceed every allocated ID;
- active posts use allowed statuses;
- active standard posts contain exactly six fact snapshots;
- every ready post appears exactly once in ready-to-post.md;
- no non-ready post appears in ready-to-post.md;
- every archived post has six published fact records;
- every published fact points to an existing archived post;
- monthly file placement matches Asia/Jakarta publication month.

Additional checks for a record containing editorial version 2 fields:

- all six facts contain allowed surprise_operator values;
- all viral_strength values are integers from 0 through 2;
- scope_check_passed and source_access_passed are true;
- at least four facts have viral_strength 2 and none has 0;
- at least four distinct operators exist;
- no operator occurs more than twice;
- record_superlative occurs no more than twice;
- Facts 1 and 6 are strength 2 and use different operators;
- quality total equals its six scores;
- all six quality rationales are present;
- candidate_count is at least 18;
- rejected_counts uses only allowed keys and sums to candidate_count minus 6;
- operator_variety equals the computed distinct-operator count;
- weakest_fact_review contains exactly two different valid positions.

A draft missing all or part of the editorial version 2 fields is a legacy draft, not corrupt data. Report it as requires_editorial_upgrade. Do not add invented audit evidence automatically, and do not approve or ready it until a real re-evaluation supplies the fields.

Repair existing records when the intended state is unambiguous. Otherwise stop and ask the user before changing data.

## 22. File Size and Partitioning

- Active drafts should remain small because posted and rejected records are removed.
- Ready-to-post.md should contain only the current queue.
- Published posts are partitioned monthly.
- Published facts are partitioned by topic.
- Do not create one permanent file per post.
- Do not merge all historical records into one global file.

If any fact index grows beyond practical GitHub or tool limits, introduce a new schema version and subdivide that topic by year or alphabetical subject range. Do not change partitioning during a production operation.

## 23. Hard Validation Failures

Do not save or publish any record when:

- JSON is invalid;
- a universally required field is missing;
- an ID is duplicated;
- a claim signature is duplicated;
- fewer or more than six facts are present in a standard post;
- a fact lacks an acceptable source;
- quality hard_rules_passed is false;
- opening_strength is below 2;
- factual_confidence is below 2;
- readability is below 2;
- total quality is below 10;
- a fact uses an unknown topic or invalid country code;
- an unsafe high-risk fact is present;
- the current Git SHA changed during the write operation.

For every newly created or editorial-version-2-upgraded draft, also do not save, approve, ready, or publish when:

- any fact lacks surprise_operator, viral_strength, scope_check_passed, or source_access_passed;
- a surprise_operator value is unknown;
- a viral_strength value is outside 0 through 2;
- scope_check_passed or source_access_passed is false;
- fewer than four facts have viral_strength 2;
- any fact has viral_strength 0;
- fewer than four distinct operators are present;
- any operator occurs more than twice;
- record_superlative occurs more than twice;
- Facts 1 and 6 use the same operator or either has viral_strength below 2;
- any quality rationale is missing or empty;
- candidate_count is below 18;
- rejected_counts contains an unknown key or does not sum to candidate_count minus 6;
- operator_variety does not match the final six facts;
- weakest_fact_review does not contain exactly two different valid final positions;
- generation_audit contains full rejected candidate wording.

A legacy draft may remain stored as draft without the additive fields. Missing editorial version 2 fields become a hard transition failure when approval or ready status is requested.

Report the failing condition clearly and leave existing valid data unchanged.
