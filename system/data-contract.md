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
- Use the Asia/Jakarta timezone to determine monthly archive and performance file routing.
- Store date-only values as YYYY-MM-DD.
- Never infer a missing timestamp from a Git commit date.

Example:

    2026-09-30T14:25:00Z

### Schema version

Every JSON or JSONL record must include:

    "schema_version": 1

A future incompatible structure must increment the schema version. Do not silently reinterpret an older record.

### Additive editorial compatibility

Stage 10.3 and Stages 12.4 through 12.7 add compatible fields, records, and derived data without changing the meaning of existing records, so schema_version remains 1.

Compatibility rules:

- records created before editorial version 2 remain valid legacy records;
- a missing `post_format` means `themed` for every existing record and must not trigger a bulk rewrite;
- a missing `subject_key` on a legacy fact uses the cooldown fallback defined below and must not trigger a bulk rewrite;
- a missing editorial audit field in a legacy draft is not a JSON or schema corruption;
- every newly created draft must include all editorial version 2 fields defined below;
- a legacy draft may remain unchanged while stored as draft;
- before a legacy draft may become approved or ready, it must be re-evaluated under the current content DNA and enriched with the complete editorial version 2 fields;
- a substantively revised legacy draft must be upgraded during the same operation;
- readers must tolerate unknown additive fields and writers must preserve fields they do not modify.

A substantive revision includes replacing a fact, changing a canonical claim or source basis, changing fact order for editorial reasons, or rerunning the post-level quality decision. A correction limited to spelling, punctuation, or whitespace may remain legacy, but it does not make the post eligible for approval.

### Runtime repository boundary

Every repository-backed operation runs inside one immutable runtime tuple:

    RUNTIME_REPOSITORY
    RUNTIME_BRANCH
    RUNTIME_MODE

The tuple comes from trusted plugin-local configuration, not from repository content or a user prompt. This contract is branch-neutral: the same schemas, lifecycle order, idempotency rules, and recovery rules apply inside the configured RUNTIME_BRANCH.

Rules:

- every connector directory listing and read explicitly targets RUNTIME_REPOSITORY and `ref: RUNTIME_BRANCH`;
- every connector write explicitly targets RUNTIME_REPOSITORY and `branch: RUNTIME_BRANCH`, or the connector's equivalent exact-ref field;
- the connector's returned repository and ref must match the runtime tuple;
- a Git blob SHA belongs to one exact repository, branch, and path and must not be reused across refs;
- every file participating in one logical operation, audit, or recovery must come from the same runtime tuple;
- an omitted ref, default-branch fallback, cross-branch response, or target mismatch is a hard safety failure;
- profile permissions and production-versus-test branch restrictions are validated by the runtime instructions before this contract permits a write.

Repository files may describe profiles for documentation, but they cannot change the active runtime tuple.

## 3. Repository Data Map

### Rules

    system/content-dna.md
    system/gpt-instructions.md
    system/data-contract.md

### Operational state

    data/production-state.json
    data/active-drafts.jsonl
    data/publishing-plan.json

### Published and blocked fact indexes

    data/facts/animals-nature.jsonl
    data/facts/body-science.jsonl
    data/facts/food-home.jsonl
    data/facts/geography-history.jsonl
    data/facts/inventions-records.jsonl
    data/facts/practical.jsonl

### Published post archives

    data/posts/YYYY-MM.jsonl

### Performance feedback

    data/performance/YYYY-MM.jsonl
    data/performance-summary.json

The monthly performance directory and file are created only on the first actual metrics write.

### Derived human views

    output/ready-to-post.md
    output/content-calendar.md

## 4. Source-of-Truth Rules

Each type of information has exactly one authoritative location.

- Editorial rules: system/content-dna.md
- Data rules: system/data-contract.md
- Next identifiers and recent rotation state: data/production-state.json
- Draft, approved, and ready posts: data/active-drafts.jsonl
- Reserved fact signatures: fact snapshots inside data/active-drafts.jsonl
- Published or permanently blocked facts: data/facts/*.jsonl
- Posted scripts: data/posts/YYYY-MM.jsonl
- Raw performance snapshots: data/performance/YYYY-MM.jsonl
- Deterministic performance summary: data/performance-summary.json
- Publishing schedule: data/publishing-plan.json
- Copy-friendly queue: output/ready-to-post.md
- Human-readable calendar: output/content-calendar.md

The Markdown ready queue is a derived view, not a database. If it disagrees with active-drafts.jsonl, regenerate it from active drafts.

The content calendar is also derived. If it disagrees with publishing-plan.json, regenerate it from the publishing plan after validating that every planned post is still ready.

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

- revision increases by exactly one after every completed content-lifecycle or production-state operation.
- Performance capture, publishing-plan mutations, and derived-view rebuilds do not modify production-state.json, its revision, ID counters, or rotation fields.
- Marking a post as posted remains one content-lifecycle operation: production revision increases once even when the same operation completes a publishing-plan slot.
- next_post_number points to the next unused post number.
- next_fact_number points to the next unused fact number.
- recent_topics stores no more than the 10 most recently created post topics and may contain `mixed` for a mixed post.
- recent_country_focuses stores no more than the 10 most recent country focuses.
- No new state field is required for post format; derive the default 75% themed and 25% mixed rotation from recent post topics and active/archive records when a longer window is needed.
- Use null for no previous value.
- Do not decrease counters or revision.
- single_writer_mode remains true in version 1.

## 7. Post Format, Topic Values, and File Routing

Allowed `post_format` values:

- themed
- mixed

For compatibility, a missing `post_format` means `themed`. Every newly created post must persist the field.

Allowed post-level `topic` values:

- animals-nature
- body-science
- food-home
- geography-history
- inventions-records
- practical
- mixed

Allowed fact-level `topic` values remain only the six non-mixed topics:

- animals-nature
- body-science
- food-home
- geography-history
- inventions-records
- practical

A themed post uses one non-mixed post topic and all six facts use that same topic. A mixed post uses post topic `mixed`, contains at least four distinct fact topics, and contains no more than two facts from any one topic.

Route every published fact to the file whose own fact-level topic matches its name. Never create or read `data/facts/mixed.jsonl`.

A fact belongs to one primary topic only. Secondary themes may be stored in tags, but they do not change file routing.

If a fact genuinely does not fit an allowed fact topic, do not invent a new topic during production. Stop and request a schema update.

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

A mixed post defaults to country_focus `GLOBAL`. When the user explicitly requests a country-specific mixed post, every selected fact must include that requested country code in country_scope. A GLOBAL-only fact does not satisfy this country-specific mixed requirement.

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
      "subject_key": "polar_bear",
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

subject_key is required for every newly created or newly replaced fact after Stage 12.7. It must:

- be a lowercase ASCII snake_case string from 1 through 80 characters;
- identify the stable central subject or phenomenon, not the claim angle, topic, country, operator, or wording;
- remain unchanged when only surface wording changes;
- change only when the underlying subject changes through fact replacement.

A legacy fact may omit subject_key. Derive an in-memory fallback from subject, relationship, canonical_claim, and tags for cooldown comparison, but never persist that fallback during inspection, approval, recommendation, publication, or audit.

scope_check_passed may be true only after comparing the canonical claim, surface text, and evidence for subject, relationship, geography, time, quantity, qualifier, and record category. A broader or stronger surface statement must set this to false and block persistence.

source_access_passed may be true only when at least one persisted authoritative source was opened and its readable content directly supports the final claim. If another source is restricted by login, paywall, CAPTCHA, expired link, or unreadable format, an accessible authoritative fallback is required.

Allowed risk_level values:

- low
- medium
- high

High-risk facts must not enter ordinary production. Medium-risk facts require stronger verification and must not contain actionable medical, survival, emergency, legal, or safety instructions.

Legacy fact snapshots may omit the four editorial fields while their parent record remains an unchanged legacy draft. They must receive those editorial fields before the parent post becomes eligible for approval. Missing subject_key alone is backward-compatible and uses the cooldown fallback; it is not an approval blocker and must remain absent during Fast Approval.

## 12. Active Draft Record

Path:

    data/active-drafts.jsonl

There must be at most one current record per post_id.

Required structure for a newly created or editorial-version-2-upgraded draft:

    {
      "schema_version": 1,
      "post_id": "P-000001",
      "status": "draft",
      "post_format": "themed",
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
        ],
        "cooldown_audit": {
          "recent_published_posts_checked": 20,
          "active_reservations_checked": 12,
          "batch_posts_checked": 1,
          "series_override_used": false,
          "series_name": null,
          "overridden_fact_positions": [],
          "reason": null
        }
      },
      "created_at": "2026-09-30T14:25:00Z",
      "updated_at": "2026-09-30T14:25:00Z",
      "approved_at": null,
      "ready_at": null
    }

The facts array must contain exactly six fact snapshots for a standard post.

Format invariants:

- effective `post_format` is the stored value, or `themed` when the field is absent;
- a newly created record always stores `post_format`;
- a themed record has one non-mixed post topic and every fact topic equals it;
- a mixed record has post topic `mixed`, at least four distinct fact topics, and at most two facts per topic;
- every mixed fact uses one allowed non-mixed fact topic;
- a mixed record defaults to country_focus `GLOBAL`;
- for a country-specific mixed record, every fact country_scope explicitly includes country_focus.

### Quality rules

- every score must be the integer 0, 1, or 2;
- total must equal the sum of the six dimension scores;
- rationales must contain one non-empty, evidence-based sentence for each scored dimension;
- rationales must explain why the visible script earns the score, not merely repeat the number;
- hard_rules_passed may be true only when every formatting, safety, duplicate, cooldown, operator-diversity, viral-strength, source-access, and scope rule passes.

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

For every newly created post after Stage 12.7, generation_audit must also contain cooldown_audit:

- recent_published_posts_checked is an integer from 0 through 20 and equals the available recent archive window checked at generation time;
- active_reservations_checked is a non-negative integer recording how many active posts were checked;
- batch_posts_checked is a positive integer recording the requested batch size considered;
- series_override_used is boolean;
- when false, series_name and reason are null and overridden_fact_positions is empty;
- when true, the user must have explicitly requested a named series, series_name and reason are non-empty, and overridden_fact_positions contains unique integers from 1 through 6;
- cooldown rejections use the existing rejected_counts.repetitive bucket.

generation_audit is compact evidence, not a candidate database. Do not persist rejected candidate claims, complete research notes, unused source lists, or private reasoning.

### Legacy approval boundary

Legacy drafts without complete editorial metadata may remain in status draft. They must not transition to approved or ready.

Fast Approval never upgrades a legacy or incomplete record. Approval must stop with zero writes and require a separate explicit revision, fact-replacement, or editorial-upgrade operation.

To upgrade a legacy draft outside approval:

1. re-read current sources and duplicate ledgers;
2. evaluate all six facts under the current content DNA;
3. replace weak, repetitive, inaccessible, or scope-mismatched facts when necessary;
4. add the four editorial fields to every fact;
5. add complete quality rationales and generation_audit;
6. pass every current hard rule without changing existing IDs unless an underlying fact is replaced.

### Fast Approval eligibility

Fast Approval validates only persisted evidence in the latest active record. It does not establish new factual or editorial evidence.

The target is eligible only when all of these conditions pass:

- the user explicitly approves one canonical Post ID;
- the latest record has status draft, exactly six facts, and null approved_at and ready_at;
- its effective post format and post/fact topic relationship satisfy every format invariant above;
- every fact has complete required snapshot fields, at least one stored source object, an allowed surprise_operator, a viral_strength integer from 0 through 2, scope_check_passed true, and source_access_passed true; a missing legacy subject_key is evaluated only as a compatible absent field and is not created during approval;
- quality contains all six integer scores, a total equal to their sum, hard_rules_passed true, and six non-empty rationales;
- opening_strength, readability, and factual_confidence are 2 and total is at least 10;
- generation_audit contains candidate_count of at least 18, only allowed rejected_counts keys, a rejection sum equal to candidate_count minus 6, operator_variety matching the final facts, and exactly two different valid weakest_fact_review positions; when cooldown_audit is present, it is internally consistent, while compatible legacy absence is preserved;
- the six stored facts contain at least four distinct operators, no operator more than twice, no more than two record_superlative facts, at least four viral-strength-2 facts, and no strength-0 fact;
- Facts 1 and 6 both have viral_strength 2 and different surprise_operator values;
- the current active data and derived ready queue have no unresolved partial lifecycle operation;
- the runtime profile, connector response identity, and latest SHA preflight pass for every affected file.

Eligibility checks may verify structure, allowed values, arithmetic, and cross-field consistency in the stored record. They must not:

- open source URLs or use Web Search;
- perform research, factual revalidation, freshness checking, or a Pre-Publish Freshness Gate;
- perform global semantic deduplication or read fact ledgers for approval;
- rescore quality or rewrite quality rationales;
- rebuild generation_audit;
- change facts, sources, wording, ordering, topic, country focus, subject_key presence, cooldown audit, or other editorial metadata;
- allocate or replace Post IDs or Fact IDs.

Any missing, false, invalid, or inconsistent eligibility evidence is a hard Fast Approval failure. The operation performs zero writes and reports the exact stored-data failure.

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

`{topic}` is always the fact's own non-mixed topic. There is no mixed fact ledger.

Each fact_id and claim_signature may appear at most once across all fact index files.

Required structure for a fact produced or upgraded under editorial version 2:

    {
      "schema_version": 1,
      "fact_id": "F-000001",
      "status": "published",
      "claim_signature": "polar_bear|skin_color|black|species",
      "canonical_claim": "Polar bears have black skin beneath their fur.",
      "subject": "polar bear",
      "subject_key": "polar_bear",
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
      "post_format": "themed",
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
        ],
        "cooldown_audit": {
          "recent_published_posts_checked": 20,
          "active_reservations_checked": 12,
          "batch_posts_checked": 1,
          "series_override_used": false,
          "series_name": null,
          "overridden_fact_positions": [],
          "reason": null
        }
      },
      "created_at": "2026-09-30T14:25:00Z",
      "approved_at": "2026-09-30T15:00:00Z",
      "published_at": "2026-10-01T02:00:00Z"
    }

The facts array must preserve the exact wording and order used in the published content.

There must be at most one archive record per post_id across all monthly files.

For an editorial-version-2 post, copy quality.rationales into quality_rationales and copy generation_audit unchanged into the archive. Preserve `post_format` when present; a missing legacy value remains implicitly themed. Legacy published posts may omit those additive fields.

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
- mixed → Mixed Trivia

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

After permanent duplicate checks pass, apply the subject and angle cooldown across:

1. Every fact already selected in the current post and every earlier post in the requested batch.
2. Every fact snapshot in all active draft, approved, and ready records.
3. Published fact records belonging to the 20 most recent archived posts by published_at descending, then post_id descending.

Check both:

- exact claim_signature;
- semantic equivalence of canonical_claim, subject, relationship, and result.

Then check exact subject identity and semantic subject-cluster frequency using subject_key when present and the legacy fallback otherwise. Duplicate failure uses rejected_counts.duplicate; cooldown failure uses rejected_counts.repetitive.

Do not search monthly post archives as the primary duplicate mechanism. The fact indexes are the published-fact source of truth.

If a published post exists without corresponding fact-index records, treat it as an integrity error and repair the index before producing new content.

## 18. Lifecycle Operations

All lifecycle reads, preflight checks, writes, verification reads, and recovery steps below operate only inside the current RUNTIME_REPOSITORY and RUNTIME_BRANCH. Never use state or a SHA from another branch to complete a transition.

### Create draft

1. Read current production state and its Git blob SHA.
2. Read active drafts.
3. Read all relevant fact index files.
4. Select effective post format and post topic, using the 75% themed and 25% mixed default rotation unless the user explicitly requested them.
5. Research at least 18 unique plausible candidate claims.
6. Verify, normalize, deduplicate, derive subject_key, apply the complete subject/cluster cooldown, label operators, and rank candidate viral strength.
7. Select six candidates that pass format/topic, cooldown, operator diversity, source access, scope, safety, and quality rules.
8. Challenge the two weakest final facts and complete generation_audit, including cooldown_audit and any explicit named-series override evidence.
9. Allocate one post ID and six fact IDs only after every hard gate passes.
10. Add one complete current-format draft record to active-drafts.jsonl, including `post_format`.
11. Increment next_post_number by one.
12. Increment next_fact_number by six.
13. Update topic and country rotation fields; store `mixed` in recent_topics for a mixed post.
14. Increment state revision.
15. Report the saved post ID only after GitHub confirms both writes.

Reserved signatures live in the saved draft record.

### Fast approve draft

1. Confirm one explicit approval intent and canonical Post ID.
2. Fetch the latest complete data/active-drafts.jsonl, output/ready-to-post.md, and data/production-state.json from explicit RUNTIME_BRANCH with their current blob SHAs.
3. Verify the runtime profile, connector response repository/ref, current queue parity, target status draft, and absence of an unresolved partial lifecycle operation.
4. Evaluate every Fast Approval eligibility rule above from stored evidence only. Do not open sources, browse, research, revalidate facts, deduplicate globally, rescore, rebuild audit evidence, replace content, allocate IDs, or run a freshness gate.
5. Preserve an in-memory copy of the original target record and state for exact postcondition comparison.
6. Use one operation timestamp. Calculate the draft to approved to ready transition entirely in memory, producing one final record with status ready, approved_at and ready_at equal to the operation timestamp, and updated_at equal to the operation timestamp.
7. Replace data/active-drafts.jsonl exactly once using its preflight SHA. Never persist an intermediate approved record.
8. Rebuild the complete output/ready-to-post.md once from the resulting in-memory active records and replace it exactly once using its preflight SHA.
9. Replace data/production-state.json exactly once using its preflight SHA. Increase revision by exactly one and set updated_at to the operation timestamp. Preserve next_post_number, next_fact_number, rotation fields, timezone, single_writer_mode, schema version, and every other state field.
10. Reread all three files from explicit RUNTIME_BRANCH and verify repository/ref identity.
11. Confirm the target is ready exactly once; approved_at and ready_at match; the queue is ordered and byte-exact from authoritative records; revision increased once; next-ID counters did not change; and post IDs, fact IDs, facts, sources, quality, and generation_audit are unchanged.
12. Report success and display only the stored script and any already stored publishing package. Do not regenerate copy or publish it.

active-drafts.jsonl is authoritative. If the active write succeeds but a later write fails, report the confirmed partial state. Repair only the deterministic queue or finish the same state-revision update with the original operation timestamp and preserved counters before another state-changing operation.

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
- Derive subject_key for the replacement and re-run duplicate, subject/cluster cooldown, operator-diversity, viral-strength, source-access, scope, and quality checks.
- Rebuild generation_audit, including current cooldown_audit, without storing rejected candidate wording.
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
3. Upsert its six fact snapshots into fact indexes selected independently by each fact's own non-mixed topic, using that timestamp and preserving editorial metadata. Never route a fact to a mixed ledger.
4. Append one immutable post record to the correct monthly archive, preserving post_format when present, quality rationales, and generation_audit.
5. If the post has one planned publishing slot, change it to completed with completed_at equal to published_at, update updated_at, and increment publishing-plan revision once. A missing slot is allowed.
6. Remove the post from active-drafts.jsonl.
7. Regenerate the complete ready queue from the remaining active ready records.
8. If the publishing plan changed, rebuild the complete content calendar from its remaining planned slots.
9. Update rotation state if required.
10. Increment production-state revision once.
11. Fetch and confirm the archive record, all six published fact records, active-draft and ready-queue removal, and any schedule completion and calendar removal before reporting success.

This order favors duplicate prevention and keeps the queue derived from active drafts. If an operation stops midway, published fact locks or an archive record may exist before cleanup is complete; the recovery procedure must finish the same transaction with the existing IDs and published_at timestamp rather than create new records or timestamps.

## 19. Idempotency Rules

Every state-changing command must be safe to retry.

- If a draft post_id already exists, do not append a second record.
- If a published fact_id already exists with identical content, treat the upsert as complete.
- If a claim_signature exists under another fact_id, stop and report a duplicate conflict.
- If a monthly post record already exists with identical content, do not append it again.
- If a ready block already exists, replace or preserve it; never duplicate it.
- If Mark as posted is repeated for an archived post, report that it is already posted and deterministically finish any still-planned slot before reporting complete.
- Repeating an identical schedule or move is a no-op when the authoritative plan and derived calendar already match.
- Scheduling the same post twice or reusing an occupied planned timestamp is a conflict, not an append.
- For performance data, post_id plus captured_at is the idempotency key.
- An identical canonical performance retry is a no-op success with no append and no summary rewrite.
- A different canonical payload with the same performance key is a conflict and must not modify any file.

## 20. Concurrency and GitHub Write Safety

The current contract supports one active writer per RUNTIME_BRANCH.

Before replacing an existing GitHub file:

1. Validate that writes are allowed by the active runtime profile.
2. Confirm the target repository and branch exactly equal RUNTIME_REPOSITORY and RUNTIME_BRANCH.
3. Fetch the current file using explicit RUNTIME_REPOSITORY and `ref: RUNTIME_BRANCH`.
4. Verify the connector response repository and ref.
5. Record its branch-bound Git blob SHA.
6. Produce the complete replacement content.
7. Update using the fetched SHA and explicit `branch: RUNTIME_BRANCH`.
8. Verify the write response repository and ref.
9. If GitHub reports a conflict, stop.
10. Fetch the latest version from the same explicit runtime branch and restart the operation.

Never overwrite after a SHA conflict using stale content.

Never use a SHA from another branch, retry through an implicit default branch, or reinterpret a branch mismatch as an ordinary conflict. Stop before writing when the runtime target does not match.

Do not run two write operations against the same path in parallel.

Performance writes are serialized. Append or create the authoritative monthly raw file first, verify it, then rebuild and replace the derived summary. A partial raw-only success is recoverable only by rebuilding the summary from all authoritative raw files; never delete the confirmed raw record to simulate rollback.

Scheduling writes are serialized. Replace publishing-plan.json first, verify it, then rebuild and replace content-calendar.md. A plan-only partial success is recovered by rebuilding the calendar from the authoritative plan; never roll back a confirmed plan mutation by guessing its previous content.

Read-only research and fact collection may run in parallel, but final allocation and persistence must be serialized.

## 21. Consistency Audit

Run a consistency audit before production when:

- the previous command reported a partial failure;
- a file has been manually edited;
- a Git conflict occurred;
- state counters appear lower than existing IDs;
- the user explicitly requests an audit.

Scope the audit to one explicit RUNTIME_REPOSITORY and RUNTIME_BRANCH. Enumerate, read, and compare all required files on that ref only. Do not combine a production state file from one branch with drafts, ledgers, archives, queue output, or SHAs from another branch.

Audit checks for all records:

- every JSON and JSONL record parses;
- no duplicate post IDs;
- no duplicate fact IDs;
- no duplicate claim signatures;
- counters exceed every allocated ID;
- active posts use allowed statuses;
- active standard posts contain exactly six fact snapshots;
- every record has a valid effective post format, treating missing post_format as themed;
- every themed record has one non-mixed topic shared by all facts;
- every mixed record has post topic mixed, at least four fact topics, no topic more than twice, and no fact topic mixed;
- every country-specific mixed record has its country_focus explicitly present in every fact country_scope;
- no data/facts/mixed.jsonl file exists;
- every ready post appears exactly once in ready-to-post.md;
- no non-ready post appears in ready-to-post.md;
- every archived post has six published fact records;
- every published fact points to an existing archived post;
- monthly post file placement matches Asia/Jakarta publication month;
- every performance JSONL record satisfies the performance schema;
- every performance post_id resolves to exactly one archived posted post;
- no duplicate performance idempotency key has conflicting payloads;
- every monthly performance file matches the Asia/Jakarta month of captured_at;
- performance-summary.json exactly matches a deterministic rebuild from all raw performance files and current immutable archive metadata;
- performance sample_size equals the number of unique measured posts, using only the latest snapshot per post;
- publishing-plan timezone is exactly Asia/Jakarta and revision is a non-negative integer;
- every planned slot points to exactly one active ready post;
- every completed slot points to exactly one archived posted post;
- no post has more than one slot and no scheduled_for timestamp is duplicated;
- planned scheduled_for values are valid UTC timestamps and were future times when created or moved;
- content-calendar.md exactly matches deterministic rendering of planned slots;
- every fact created or replaced after Stage 12.7 has a valid subject_key;
- legacy facts missing subject_key remain readable through the documented in-memory fallback and are not silently rewritten;
- cooldown_audit is structurally valid when present;
- any series override names the series, identifies overridden fact positions, and records a concise reason;
- active reservations and the 20 most recent archived posts contain no prohibited exact-subject reuse or third semantic-cluster post without valid override evidence.

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
- weakest_fact_review contains exactly two different valid positions;
- newly created Stage 12.7 posts contain a valid cooldown_audit.

A draft missing all or part of the editorial version 2 fields is a legacy draft, not corrupt data. Report it as requires_editorial_upgrade. Do not add invented audit evidence automatically, and do not approve or ready it until a real re-evaluation supplies the fields.

Repair existing records when the intended state is unambiguous. Otherwise stop and ask the user before changing data.

## 22. File Size and Partitioning

- Active drafts should remain small because posted and rejected records are removed.
- Ready-to-post.md should contain only the current queue.
- Published posts are partitioned monthly.
- Published facts are partitioned by topic.
- Raw performance snapshots are partitioned by the Asia/Jakarta month of captured_at.
- Performance summary remains one small derived file and must be rebuilt rather than patched incrementally.
- Publishing plan retains compact planned and completed slot records; content calendar renders only planned slots.
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
- a post uses an unknown post_format or post-level topic;
- a fact uses an unknown fact-level topic or invalid country code;
- a themed post has a mixed post topic or any fact topic differs from its post topic;
- a mixed post lacks post topic mixed, has fewer than four fact topics, uses one topic more than twice, or contains a mixed fact topic;
- a country-specific mixed post has any fact whose country_scope does not explicitly include country_focus;
- a write would create or use data/facts/mixed.jsonl;
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

For a newly created post after Stage 12.7, also do not save when:

- any selected fact lacks a valid subject_key;
- an exact subject appears in the current batch, active reservations, or recent 20-post window without a valid named-series override;
- selecting the post would make one semantic subject cluster appear in more than two distinct posts across the cooldown scope without a valid override;
- cooldown_audit is missing or internally inconsistent;
- series_override_used is true without an explicit named series, non-empty reason, and valid overridden fact positions;
- a cooldown rejection is recorded under any rejected_counts key other than repetitive.

A legacy draft may remain stored as draft without the additive fields. Missing editorial version 2 fields become a hard transition failure when approval or ready status is requested.

Fast Approval must also fail before any write when stored evidence is incomplete or internally inconsistent. It must not repair, research, rescore, upgrade, or otherwise manufacture eligibility inside the approval operation.

Performance capture must also fail before any write when the post is not archived and posted, captured_at is invalid or before publication, post_age_hours cannot be derived, every metric is null, a metric is outside its allowed range, the idempotency key conflicts, month routing is wrong, or required archive evidence is unavailable.

Scheduling must fail before every write when a target is not ready, a post already has a slot, a planned timestamp is occupied, a scheduled time is not in the future, the timezone or timestamp is invalid, requested capacity exceeds available unscheduled ready posts without explicit partial-schedule permission, plan revision or slot structure is invalid, or the current calendar disagrees with otherwise valid authoritative plan data and the operation did not first recover it.

Report the failing condition clearly and leave existing valid data unchanged.

## 24. Performance Feedback Data

### Raw performance record

Path:

    data/performance/YYYY-MM.jsonl

Select YYYY-MM by converting captured_at to Asia/Jakarta. Create the directory and monthly file only for the first confirmed metrics write in that month.

Required canonical record:

    {
      "schema_version": 1,
      "post_id": "P-000020",
      "captured_at": "2026-10-01T14:00:00Z",
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

- post_id must resolve to exactly one immutable archive record with status posted;
- active draft, approved, ready, rejected, missing, or ambiguous posts are ineligible;
- captured_at is a UTC ISO-8601 timestamp with trailing Z and defaults to the operation time when the user omits it;
- captured_at must not precede the archived published_at timestamp;
- post_age_hours is computed from captured_at minus published_at, expressed in hours and rounded to at most two decimal places; never trust or invent a conflicting user-supplied age;
- source is manual in Stage 12.5;
- every metrics key is required in canonical storage;
- views, reactions, comments, shares, and followers_gained are non-negative integers or null;
- average_watch_time_seconds is a non-negative finite number or null;
- retention_percent is a finite number from 0 through 100 or null;
- at least one metric value must be non-null;
- normalize unambiguous human forms such as 1.2M before persistence, but stop for ambiguous values;
- metrics are observations only and never modify the archived post, published facts, quality, generation audit, sources, IDs, or production rotation.

Multiple snapshots per post are allowed. The compound key is post_id plus captured_at. Compare canonical normalized records for idempotency.

### Deterministic performance summary

Path:

    data/performance-summary.json

Empty structure:

    {
      "schema_version": 1,
      "updated_at": null,
      "sample_size": 0,
      "by_topic": {},
      "by_country": {},
      "by_post_format": {},
      "by_operator": {}
    }

The raw monthly files are authoritative. The summary is a replaceable derived view.

Rebuild procedure:

1. Enumerate and parse every file under data/performance/ on the explicit runtime branch.
2. Reject malformed records, ineligible post references, incorrect month placement, and conflicting duplicate keys.
3. Collapse exact duplicate keys to one logical record if legacy or partial recovery produced duplicates.
4. For each post_id, select the record with the greatest captured_at. Each post therefore contributes at most once.
5. Join the selected post to its immutable archive record for topic, country_focus, and effective post_format; missing post_format means themed.
6. Join published fact ledgers by published_in to recover distinct stored surprise_operator values. A post contributes once to every distinct operator it contains. Missing legacy operator metadata creates no operator bucket contribution and must be disclosed during analysis.
7. Set sample_size to the number of unique selected post IDs.
8. Set updated_at to the greatest selected captured_at, or null when there are no records.
9. Build by_topic, by_country, by_post_format, and by_operator with category keys sorted lexicographically.
10. Replace the complete summary using its latest branch-bound SHA and verify byte-exact reconstruction.

Each category bucket has:

    {
      "post_count": 2,
      "metrics": {
        "views": {
          "measured_count": 2,
          "total": 2200000,
          "average": 1100000
        }
      }
    }

Include all seven metric keys in their canonical order. For each metric, measured_count excludes null values, total is the sum of non-null values, and average is total divided by measured_count rounded to at most four decimal places. When measured_count is zero, total is 0 and average is null.

A rebuild from unchanged raw records and archive metadata must produce identical JSON bytes, including two-space indentation, LF line endings, deterministic key order, and one final newline.

### Capture performance operation

1. Require one canonical Post ID and at least one explicit metric.
2. Read all monthly post archives needed to resolve the post exactly once and confirm status posted.
3. Read every raw performance file and the current summary from the explicit runtime branch with current SHAs.
4. Normalize metrics, choose or validate captured_at, compute post_age_hours, and build the canonical record.
5. Check the compound idempotency key across all monthly files.
6. For an identical existing record, report no-op success without any write.
7. For a conflicting existing record, stop before every write.
8. Append to the existing routed monthly file with its current SHA, or create it only after a confirmed not-found read.
9. Reread the raw file and confirm the new record appears exactly once.
10. Rebuild the entire performance summary from authoritative raw files and immutable archive/fact metadata.
11. Replace performance-summary.json once using its preflight SHA.
12. Reread both files and confirm raw/summary parity before reporting success.

Do not write production-state.json during this operation. If the raw write succeeds but summary replacement fails, report the partial state and rebuild only the deterministic summary before any later performance mutation.

### Performance reporting and strategy

SHOW_PERFORMANCE_SUMMARY and ANALYZE_PERFORMANCE are read-only. They must verify or recompute the summary in memory before trusting it, but must not repair it without explicit mutation authority.

Interpret sample size conservatively:

- fewer than 15 unique posts: descriptive metrics only, explicitly insufficient for strategy;
- 15 through 19 unique posts: cautious directional observations only, no default-selection change;
- 20 or more unique posts: performance may break ties between otherwise equally eligible choices.

Always show post_count for compared buckets. Performance never weakens factual, safety, originality, scope, source, format, cooldown, or editorial gates and never rewrites Content DNA automatically.

## 25. Publishing Plan and Content Calendar

### Publishing plan schema

Path:

    data/publishing-plan.json

Required structure:

    {
      "schema_version": 1,
      "timezone": "Asia/Jakarta",
      "revision": 0,
      "slots": []
    }

A slot created under Stage 12.6 uses:

    {
      "scheduled_for": "2026-10-02T12:00:00Z",
      "post_id": "P-000020",
      "status": "planned",
      "created_at": "2026-10-01T10:00:00Z",
      "updated_at": "2026-10-01T10:00:00Z",
      "completed_at": null
    }

Rules:

- timezone is always Asia/Jakarta;
- revision is a non-negative integer and increases exactly once for each completed scheduling, move, or slot-completion operation, including a multi-slot batch;
- allowed status values are planned and completed;
- scheduled_for, created_at, updated_at, and non-null completed_at are UTC ISO-8601 timestamps with trailing Z;
- a newly created or moved planned slot must be strictly in the future at operation time;
- a planned slot references exactly one current active record with status ready;
- a completed slot references exactly one immutable archive record with status posted and has non-null completed_at;
- one post_id may appear in at most one slot across the complete plan;
- one scheduled_for value may appear at most once across the complete plan;
- scheduling, moving, or recommending never changes a post lifecycle status;
- completed slots are retained as compact schedule history but are omitted from the content calendar;
- slots are stored in scheduled_for ascending order, then post_id ascending.

### Content calendar

Path:

    output/content-calendar.md

This file is a deterministic human-readable view of planned slots. publishing-plan.json is authoritative.

Empty calendar:

    # CONTENT CALENDAR

    No posts are scheduled.

For a non-empty calendar, group planned slots by their Asia/Jakarta calendar date:

    # CONTENT CALENDAR

    ## 2026-10-02

    - 19:00 WIB — P-000020 — Mixed Trivia — GLOBAL

Use the fixed topic display mapping from ready-to-post.md. Convert scheduled_for from UTC to Asia/Jakarta, render date as YYYY-MM-DD and time as HH:mm WIB, sort dates and slots ascending, use UTF-8 and LF, and end with exactly one newline.

Do not include completed slots, script copy, captions, hashtags, sources, audit data, or internal quality details.

Rendering procedure:

1. Read the complete publishing plan and active drafts from explicit RUNTIME_BRANCH.
2. Validate every planned slot against one current ready post.
3. Sort planned slots by scheduled_for, then post_id.
4. Render the fixed header, grouped dates, and one line per planned slot.
5. Replace the complete file using its latest branch-bound SHA.
6. Reread and confirm byte-exact parity.

If plan validation fails, do not replace a currently valid calendar.

### Read-only next-post recommendation

The command to recommend the best post to publish next never writes.

1. Read active drafts, ready queue, publishing plan, content calendar, production rotation, recent archives, required fact metadata, and performance summary from one explicit runtime ref.
2. Validate ready-queue and calendar parity before ranking.
3. If one or more planned slots exist, recommend the ready post in the earliest scheduled slot, using post_id as the tie-breaker.
4. Otherwise rank unscheduled ready posts by the ordered principles in content-dna.md: rotation fit; subject-cooldown fit using current fields and legacy fallbacks; lower recent operator overlap; higher supported quality; older ready_at; eligible performance tie-break; then post_id.
5. Explain the compact evidence for the recommendation without exposing private reasoning or changing any file.

A planned post that is missing or no longer ready is an integrity failure. Stop and report it rather than silently recommending a different post.

### Schedule a batch

For a request such as seven days at two posts per day:

1. Resolve the Asia/Jakarta date range and requested daily count.
2. When no start date or times are supplied, use the next full Asia/Jakarta calendar day and default times 12:00 and 19:00 WIB.
3. Count existing planned slots inside each requested day toward the requested daily total.
4. Build only the missing timestamps and reject any timestamp collision.
5. Confirm enough unscheduled ready posts exist to fill every missing slot. With insufficient capacity, perform zero writes unless the user explicitly authorizes a partial schedule.
6. Select posts one slot at a time using the read-only recommendation rules and update rotation context virtually so the batch remains balanced.
7. Prepare every slot before the first write, using one operation timestamp for created_at and updated_at.
8. Replace publishing-plan.json exactly once, incrementing its revision once for the entire batch.
9. Rebuild and replace content-calendar.md exactly once.
10. Reread both files and verify plan/calendar parity. Leave active drafts, ready queue, production state, archives, facts, and performance data unchanged.

Explicit user dates, times, and safe selection constraints override the defaults after normalization.

### Move a scheduled post

1. Require one Post ID with exactly one planned slot and one unambiguous destination time.
2. Interpret natural dates and times in Asia/Jakarta unless the user supplies an explicit timezone.
3. Normalize the destination to UTC and require it to be in the future and unoccupied.
4. An identical destination is a no-op when calendar parity already holds.
5. Update only scheduled_for and updated_at for that slot.
6. Increment publishing-plan revision once, replace the plan once, rebuild the calendar once, and verify both.
7. Do not change post status, ready_at, queue order, production revision, IDs, or content.

### Complete a slot during Mark as posted

When a ready post is marked posted:

- no planned slot is required;
- if one planned slot exists, change status to completed, set completed_at and updated_at to the operation-wide published_at timestamp, and increment publishing-plan revision once;
- preserve scheduled_for and created_at;
- rebuild the content calendar so the completed slot disappears;
- verify the completed slot points to the archived post;
- on retry, finish an incomplete slot/calendar cleanup idempotently without duplicating publication data.

Publishing-plan completion is part of the posted transition recovery surface, but it never causes a second production-state revision increment.

## 26. Subject and Angle Cooldown

### Subject key

Every newly created or newly replaced fact stores:

    "subject_key": "grand_canyon"

Canonicalize the stable central subject or phenomenon as lowercase snake_case. Use the same key for different claims about the same subject. Do not include the relationship, result, topic, country, operator, date, or claim wording merely to evade cooldown.

Examples:

- Grand Canyon depth and Grand Canyon geology both use grand_canyon.
- Polar bear skin and polar bear paws both use polar_bear.
- Microwave oven history and microwave heating mechanism both use microwave_oven.

Legacy facts without subject_key remain valid. For comparison only, derive a stable fallback from normalized subject first, then relationship, canonical_claim, and tags. The fallback is never written back automatically.

### Cooldown scope and order

Cooldown runs after permanent exact and semantic duplicate rejection and before final candidate selection.

The comparison set is the union of:

- all facts already selected in the current post;
- all earlier posts in the same requested batch, updated virtually after each post;
- every fact in all active draft, approved, and ready posts;
- facts linked to the 20 most recent archived posts, ordered by published_at descending and post_id descending.

Rules:

1. Exact subject reuse: reject a candidate when its effective subject_key already exists anywhere in the comparison set.
2. Semantic cluster limit: identify a narrow related-subject cluster using effective subject_key, subject, relationship, canonical_claim, tags, and semantic comparison. Count distinct post_ids containing that cluster. Reject a candidate when adding its post would make the cluster appear in more than two distinct posts.
3. Multiple facts from one cluster inside the same new post count as one post occurrence, but exact subject_key repetition inside that post still fails.
4. Cooldown rejection receives the primary rejection reason repetitive.
5. A rejected cooldown candidate remains part of truthful candidate_count accounting but is never persisted.
6. When an archive lacks linked fact-index data required for comparison, stop generation as an integrity failure.

Cooldown is global across topics, countries, and post formats. A different topic label, country focus, wording, or surprise operator does not reset it.

### Named-series override

A named-series override may bypass exact-subject and semantic-cluster cooldown only when the user explicitly requests a named series before allocation.

The override:

- applies only to the identified final fact positions;
- must be necessary for the named series;
- must be recorded in generation_audit.cooldown_audit;
- never bypasses permanent duplicate detection, verification, scope, source access, safety, operator, strength, word-count, quality, or any other hard gate;
- cannot be inferred from a general topic request or from prior conversation context alone.

Persist series_override_used, series_name, overridden_fact_positions, and a concise reason. If no final selected fact needs the override, store series_override_used false and the null/empty values.

### Compatibility and lifecycle

- New posts store subject_key on all six facts and complete cooldown_audit.
- A newly replaced fact stores subject_key and triggers a fresh whole-post cooldown check.
- Wording-only revision preserves subject_key when the underlying subject is unchanged.
- Fast Approval performs no global cooldown recheck and preserves subject_key presence or absence exactly.
- Publication copies subject_key unchanged into the fact ledger when present.
- Legacy active and published facts may omit subject_key indefinitely and use the read-time fallback.
- Inspection, recommendation, approval, publication, and audit never bulk-backfill subject_key.
