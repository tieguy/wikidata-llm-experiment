# wikidata-factcheck-experiment

An experiment by [Luis Villa](https://meta.wikimedia.org/wiki/User:LuisVilla) exploring LLM-assisted fact-checking for Wikidata contributions. I do not expect anyone other than myself will use this, but I provide it here in the interests of transparency.

## Research Questions

1. **Throughput**: If every claim is rigorously fact-checked before submission, what’s a realistic rate for creating/updating Wikidata claims?
1. **Validation requirements**: What level of source verification is necessary to meet Wikidata’s community standards? Where do LLM confidence levels map to Wikidata’s reference requirements?
1. **Prompt engineering**: How should fact-checking prompts (like [CheckPlease](https://checkplease.neocities.org)) be adapted for Wikidata’s specific data model and sourcing norms?
1. **Next-generation ideation**: If there is going to be a [Gas Town](https://steveklabnik.com/writing/how-to-think-about-gas-town/) for "a world in which every single human being can freely share in the sum of all knowledge", what could/would/should that look like?

## Approach

This project uses Claude Code as a research agent that:

- Identifies claims to add or update
- Applies structured fact-checking before any edit
- Records the full provenance chain (sources consulted, confidence levels, verification steps)
- Submits edits via pywikibot with proper references

All development and testing happens against **test.wikidata.org** until the methodology is validated.

## Setup

### Prerequisites

- Python 3.9+
- A Wikidata test instance account (create at https://test.wikidata.org)

### Installation

```bash
python -m venv venv
source venv/bin/activate  # or `venv\Scripts\activate` on Windows
pip install pywikibot
```

### Configuration

1. Create `user-config.py` in the project root:

```python
family = 'wikidata'
mylang = 'test'
usernames['wikidata']['test'] = 'YourTestUsername'
```

1. Generate bot credentials at https://test.wikidata.org/wiki/Special:BotPasswords
- Grant permissions: Edit existing pages, Create/edit/move pages, High-volume editing
- Save the credentials securely
1. Create `user-password.py` (gitignored):

```python
('YourTestUsername', BotPassword('YourBotName', 'YourBotPassword'))
```

1. Test the connection:

```bash
python -c "import pywikibot; site = pywikibot.Site('test', 'wikidata'); print(site.logged_in())"
```

## Project Structure

```
├── README.md
├── CLAUDE.md              # Instructions for Claude Code
├── user-config.py         # pywikibot site configuration
├── user-password.py       # credentials (gitignored)
├── src/
│   ├── factcheck/         # fact-checking logic
│   ├── wikidata/          # pywikibot wrapper utilities  
│   └── experiments/       # experimental scripts
├── logs/                  # edit logs and provenance records
└── docs/                  # methodology notes
```

## Status

🚧 **Early exploration** — not yet making edits, just establishing tooling and methodology.

## License

CC0
