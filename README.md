# scryfall-local

A lightweight, offline command-line tool for searching [Scryfall](https://scryfall.com) Oracle cards from a local cache. No API key, no rate limits, no internet required.

Built for Magic: The Gathering players who want fast card lookups during deckbuilding, theorycrafting, or kitchen-table sessions — especially useful alongside local LLM tools for private brainstorming.

## Features

- 🔍 **Search by name, text, type, colour, mana cost, or keyword**
- 🔮 **Fuzzy name search** — handles typos, misspellings, and partial names
- 🎲 **Random card** generation
- ⚡ **Instant** — loads 38,000+ cards in under a second
- 🔒 **Offline** — works without internet once the cache is downloaded
- 🎨 **Colour-coded output** with emoji symbols (⚪🔵⚫🔴🟢)
- 🏛️ **Format filtering** — Commander, Modern, Legacy, Pauper, Vintage
- 🧩 **Commander colour identity** filtering
- 🔄 **Built-in cache updater** — `scryfall update` fetches the latest data from Scryfall

## Quick Start

### 1. Download the Scryfall Oracle cache

```shell
curl -o ~/.hermes/scryfall/oracle-cards.jsonl \
  "https://data.scryfall.io/oracle-cards/oracle-cards.jsonl"
```

Or use the built-in updater after installation (see [Updating the cache](#updating-the-cache)).

### 2. Install the script

```shell
# Clone this repo
git clone https://github.com/pigarmahdar/scryfall-local.git
cd scryfall-local

# Make executable
chmod +x scryfall

# Optional: add to your PATH
cp scryfall /usr/local/bin/scryfall
```

### 3. Search!

```shell
scryfall name "Sol Ring"
scryfall text "create a treasure token"
scryfall color WG --legal commander
scryfall random
```

## Commands

| Command | Description | Example |
|---------|-------------|---------|
| `name` | Search by card name (partial match) | `scryfall name "Gilded Drake"` |
| `text` | Search by oracle text | `scryfall text "destroy all creatures"` |
| `type` | Search by type line | `scryfall type "enchantment"` |
| `color` | Search by colour identity (WUBRG letters) | `scryfall color WG` |
| `cmc` | Search by mana value (`=`, `<=`, `>=`) | `scryfall cmc <=3` |
| `keyword` | Search by keyword ability | `scryfall keyword flash` |
| `fuzzy` | Fuzzy name search (handles typos, partial names) | `scryfall fuzzy "sol rng"` |
| `random` | Get a random card | `scryfall random` |
| `search` | General search across name + text + type | `scryfall search "2 mana green creature"` |
| `update` | Update the Oracle cache from Scryfall | `scryfall update` |

## Options

| Flag | Description | Example |
|------|-------------|---------|
| `--limit N` | Max results (default: 10) | `--limit 20` |
| `--commander WUBRG` | Filter by commander colour identity | `--commander WU` |
| `--legal <format>` | Only cards legal in a format | `--legal commander` |
| `--exact` | Exact name match only (for `name` command) | `--exact` |

## Examples

```shell
# Find Lightning Greaves
scryfall name "Lightning Greaves"

# All green-white creatures legal in Commander
scryfall type "creature" --commander WG --legal commander

# Cards that create Treasure tokens
scryfall text "create a treasure token"

# Cheap counterspells (CMC 2 or less)
scryfall text "counter target spell" --legal commander --limit 5

# Random card for EDH night
scryfall random

# Fuzzy search — handles typos and partial names
scryfall fuzzy "sol rng"           # → Sol Ring
scryfall fuzzy "thragtuskk"        # → Thragtusk
scryfall fuzzy "jac bele"          # → Jace Beleren etc.

# All flash creatures in blue
scryfall keyword flash --commander U
```

## Updating the Cache

Card data from Scryfall is updated regularly (new sets, legality changes, Oracle text corrections). The tool includes a built-in updater:

```shell
scryfall update
```

This downloads the latest Oracle Cards bulk data (~200 MB) from Scryfall, backs up your existing cache, and replaces it. If anything fails, the backup is restored automatically.

You can also update manually:

```shell
curl -o ~/.hermes/scryfall/oracle-cards.jsonl.gz \
  "https://data.scryfall.io/oracle-cards/oracle-cards-YYYYMMDDHHMMSS.jsonl.gz"
gunzip -f ~/.hermes/scryfall/oracle-cards.jsonl.gz
```

Or use a crontab for automatic monthly updates:

```shell
# Run on the 1st of every month at 3am
0 3 1 * * /path/to/scryfall update
```

## Using with Local LLMs

`scryfall` pairs well with local language models (LM Studio, Ollama, Open WebUI, etc.) for private deckbuilding brainstorming. Run card searches in your terminal while chatting with your local model — no data leaves your machine.

```shell
# Look up cards, then paste the results into your LM session
scryfall text "whenever a creature enters" --legal commander --limit 5
```

### Open Web UI Integration

There is also an [OpenWebUI Tool version](https://github.com/pigarmahdar/scryfall-local/tree/main/openwebui) that brings the same search capabilities directly into your chat interface. See the `openwebui/` folder for installation instructions.

## Fuzzy Search

The `fuzzy` command uses Scryfall's API to handle typos, misspellings, and partial card names. It works in two stages:

1. **Fuzzy match** — queries Scryfall's `/cards/named?fuzzy=` endpoint
2. **Autocomplete** — if fuzzy fails, tries Scryfall's autocomplete API and presents suggestions

You pick a number and it looks up the full card.

```shell
$ scryfall fuzzy "craterhoof behemuth"
  Searching for: "craterhoof behemuth"
  ✓ Found: Craterhoof Behemoth
```

## Data Source

Card data is sourced from [Scryfall](https://scryfall.com), the best open MTG card database. The Oracle cards file contains one entry per unique card (~38,000+ cards).

Bulk data is refreshed every 12–24 hours by Scryfall. Prices become stale after 24 hours, but gameplay data (Oracle text, types, legalities) is stable for weeks.

## Requirements

- Python 3.6+ (pre-installed on macOS and most Linux distributions)
- ~200 MB disk space for the Oracle cards cache

## License

MIT License. See [LICENSE](LICENSE) for details.

Card data is copyright Wizards of the Coast. Scryfall bulk data is provided under [Scryfall's Terms of Service](https://scryfall.com/docs/api/bulk-data).

---

*Built as part of [Kwain.gg](https://kwain.gg/) — a project about generosity, attention, and play.*
