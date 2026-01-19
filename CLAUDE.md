Instructions for Claude Code when working on this project.

## Project Purpose

This is a research experiment exploring whether LLM-assisted fact-checking can reliably support Wikidata contributions. The goal is to understand the practical constraints—not to maximize edit volume, but to understand what “correct” looks like.

## Core Principles

### 1. No edits to production Wikidata

All pywikibot operations target `test.wikidata.org` only. The config enforces this, but double-check before any edit operation. If you’re uncertain whether something targets test or production, stop and verify.

### 2. Every claim needs a verifiable reference

Wikidata’s standard: claims should be verifiable from reliable published sources. This means:

- Primary sources (official websites, government records) are preferred
- Secondary sources (news, encyclopedias) are acceptable with appropriate caution
- No claim should rely solely on LLM “knowledge”—always fetch and cite an actual source

### 3. Log everything

Before making any edit, log:

- The proposed claim (item, property, value)
- Sources consulted and their URLs
- Confidence assessment
- Any ambiguity or uncertainty encountered

Write logs to `logs/` with timestamps. This provenance chain is the main research output.

### 4. Respect Wikidata’s data model

When adding claims, include:

- **References**: At minimum, `stated in` (P248) or `reference URL` (P854) with `retrieved` (P813)
- **Precision**: Dates need appropriate precision (year vs. day). Don’t claim false precision.
- **Qualifiers**: Use when relevant (e.g., `start time`, `end time` for things that change)

## Working with pywikibot

### Reading an item

```python
import pywikibot
site = pywikibot.Site('test', 'wikidata')
repo = site.data_repository()
item = pywikibot.ItemPage(repo, 'Q42')
item.get()
# item.claims, item.labels, item.descriptions now populated
```

### Adding a claim with reference

```python
claim = pywikibot.Claim(repo, 'P31')  # instance of
target = pywikibot.ItemPage(repo, 'Q5')  # human
claim.setTarget(target)

# Add reference
ref = pywikibot.Claim(repo, 'P854')  # reference URL
ref.setTarget('https://example.com/source')
retrieved = pywikibot.Claim(repo, 'P813')  # retrieved
retrieved.setTarget(pywikibot.WbTime(year=2025, month=1, day=19))
claim.addSources([ref, retrieved])

item.addClaim(claim, summary='Adding claim with reference')
```

### SPARQL queries

```python
from pywikibot import pagegenerators
query = '''SELECT ?item WHERE { ?item wdt:P31 wd:Q5 . } LIMIT 10'''
generator = pagegenerators.WikidataSPARQLPageGenerator(query, site=site)
```

## Fact-Checking Protocol

Adapt from the CheckPlease methodology:

1. **Identify the atomic claim** — break complex statements into individual verifiable facts
1. **Search for authoritative sources** — prefer primary sources, official records
1. **Verify the specific fact** — not just “this source talks about the topic” but “this source confirms this specific claim”
1. **Assess source reliability** — is this source authoritative for this type of claim?
1. **Note any caveats** — temporal limitations, geographic scope, definitional ambiguity
1. **Record confidence level** — high/medium/low with reasoning

For Wikidata specifically, also consider:

- Does this property exist? Is it the right property for this claim?
- What’s the conventional way to model this in Wikidata? (Check similar items)
- Are there existing claims that conflict?

## Useful Commands

```bash
# Test pywikibot connection
python -c "import pywikibot; print(pywikibot.Site('test', 'wikidata'))"

# Run a script with throttling (pywikibot handles this, but be aware)
python -m pywikibot.scripts.login  # interactive login if needed
```

## What Success Looks Like

The research output is understanding, not volume. Success is:

- Clear documentation of what fact-checking steps are necessary
- Realistic estimates of time/effort per claim
- Identified failure modes (where does LLM fact-checking fall short?)
- A refined prompt methodology tuned for Wikidata’s norms
- Enough logged examples to analyze patterns

## Questions to Explore

As you work, note observations about:

- Which types of claims are easy vs. hard to verify?
- Where do source quality judgments get tricky?
- What’s the gap between “LLM thinks this is true” and “verifiable from cited source”?
- How often do you find conflicting information?
- What would a sustainable human-in-the-loop workflow look like?
