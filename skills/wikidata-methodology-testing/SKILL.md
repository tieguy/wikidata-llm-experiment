---
name: wikidata-methodology-testing
description: Test SIFT methodology accuracy by reading production Wikidata, proposing claims, and logging for human verification (no execution)
---

# Wikidata Methodology Testing

## Purpose

Measure SIFT methodology accuracy by:
1. Reading entities from production Wikidata
2. Running full SIFT verification pipeline
3. Proposing claims (WITHOUT executing)
4. Logging for human verification

**This skill does NOT write to Wikidata.** All proposed claims are logged only.

## Safety

**READ-ONLY:** This skill reads from production Wikidata but never writes.

```python
# Read from production Wikidata
site = pywikibot.Site('wikidata', 'wikidata')  # Production for reading
repo = site.data_repository()
# NO write operations - logging only
```

## Workflow

### Step 1: Select Test Entity

Load entity from `docs/test-entities.yaml` or accept Q-id as argument.

```
/wikidata-methodology-testing Q42
```

### Step 2: Fetch Entity from Production Wikidata

```python
import pywikibot

site = pywikibot.Site('wikidata', 'wikidata')
repo = site.data_repository()
item = pywikibot.ItemPage(repo, '[Q-id]')
item.get()

# Extract current claims, labels, descriptions
```

### Step 3: System Selects Properties

Based on entity type (detected from P31), propose properties to verify:

**For humans (P31 = Q5):**
- P569 (date of birth)
- P570 (date of death)
- P27 (country of citizenship)
- P106 (occupation)
- P21 (sex or gender)

**For organizations (P31 = Q43229 or subclasses):**
- P571 (inception)
- P576 (dissolved/abolished)
- P17 (country)
- P159 (headquarters location)
- P112 (founded by)

**For creative works (P31 = Q7725634 or subclasses):**
- P577 (publication date)
- P50 (author)
- P123 (publisher)
- P495 (country of origin)
- P136 (genre)

Log which properties were selected and why.

### Step 4: Run SIFT Verification

For each selected property, follow the full SIFT methodology from `docs/wikidata-methodology.md`:

1. **Source Discovery:** Use WebSearch to find sources
2. **Stop:** Question assumptions
3. **Investigate:** Assess source authority
4. **Find:** Cross-reference multiple sources
5. **Trace:** Locate primary sources

### Step 5: Propose Claim (No Execution)

For each verified property, create a proposed claim:

```yaml
proposed_claim:
  property: P569
  property_label: date of birth
  proposed_value: "1952-03-11"
  value_type: time
  precision: day
  references:
    - url: "https://example.com/source"
      retrieved: 2026-01-20
  confidence: high
  confidence_reasoning: "Primary source confirmed"
```

**DO NOT EXECUTE.** This is proposal only.

### Step 6: Log for Human Verification

Write to `logs/wikidata-methodology-testing/[date]-[Q-id]-[P-id].yaml`:

```yaml
test_date: [YYYY-MM-DD]
entity: [Q-id]
entity_label: [label]
entity_type: [human|organization|creative_work]

property: [P-id]
property_label: [label]
property_selected_by: system

sources_consulted:
  - url: "[url]"
    name: "[name]"
    type: [primary|secondary]
    reliability: [1-5]
    useful_for: "[description]"

sift_verification:
  stop: "[what was questioned]"
  investigate: "[source assessment]"
  find_better: "[cross-reference findings]"
  trace: "[primary source status]"

proposed_claim:
  value: "[proposed value]"
  value_type: [item|string|time|quantity]
  precision: [if applicable]
  confidence: [high|medium|low]
  confidence_reasoning: "[reasoning]"
  references:
    - url: "[url]"
      retrieved: [date]

# HUMAN FILLS IN AFTER REVIEW:
human_verification:
  reviewed_by: null
  review_date: null
  sift_correct: null  # true/false
  proposed_value_correct: null  # true/false
  actual_value: null  # if different from proposed
  failure_mode: null  # hallucinated_source|misread_source|wrong_property|incorrect_value|other
  notes: null
```

### Step 7: Present for Human Verification

After logging, present the proposed claim to human:

```
Proposed claim for [Entity Label] ([Q-id]):

Property: [P-id] ([Property Label])
Proposed Value: [value]
Confidence: [high/medium/low]

Sources consulted:
1. [source 1] (reliability: X/5)
2. [source 2] (reliability: X/5)

SIFT Analysis:
- Stop: [what was questioned]
- Investigate: [source assessment]
- Find: [cross-references]
- Trace: [primary source]

Log file: logs/wikidata-methodology-testing/[filename].yaml

Please verify and update the human_verification section of the log file.
```

## Metrics Tracked

After testing completes, analyze logs for:
- **Property selection appropriateness:** Did system select sensible properties?
- **Source fetchability rate:** What % of sources were accessible?
- **SIFT accuracy rate:** What % of proposed claims were correct?
- **Failure mode distribution:** Which error types are most common?
- **Accuracy by entity type:** Do humans/orgs/works differ?
- **Accuracy by property type:** Which properties are harder?
