# scryfall-local

A lightweight, offline command-line tool for searching [Scryfall](https://scryfall.com) Oracle cards from a local cache. No API key, no rate limits, no internet required.

Built for Magic: The Gathering players who want fast card lookups during deckbuilding, theorycrafting, or kitchen-table sessions — especially useful alongside local LLM tools for private brainstorming.

## Features

- 🔍 **Search by name, text, type, colour, mana cost, or keyword**
- 🎲 **Random card** generation
- ⚡ **Instant** — loads 38,000+ cards in under a second
- 🔒 **Offline** — works without internet once the cache is downloaded
- 🎨 **Colour-coded output** with emoji symbols (⚪🔵⚫🔴🟢)
- 🏛️ **Format filtering** — Commander, Modern, Legacy, Pauper, Vintage
- 🧩 **Commander colour identity** filtering

## Quick Start

### 1. Download the Scryfall Oracle cache

```bash
curl -o ~/.hermes/scryfall/oracle-cards.jsonl \
  "https://data.scryfall.io/oracle-cards/oracle-cards.jsonl"
```

Or use the [Scryfall bulk data API](https://scryfall.com/docs/api/bulk-data) to get the latest download URL.

### 2. Install the script

```bash
# Clone this repo
git clone https://github.com/pigarmahdar/scryfall-local.git
cd scryfall-local

# Make executable
chmod +x scryfall

# Optional: add to your PATH
cp scryfall /usr/local/bin/scryfall
```

### 3. Search!

```bash
scryfall name "Sol Ring"
scryfall text "create a treasure token"
scryfall color WG --legal commander
scryfall random
```

## Usage

```
scryfall <command> <query> [options]
```

### Commands

| Command | Description | Example |
|---------|-------------|---------|
| `name` | Search by card name (partial match) | `scryfall name "Gilded Drake"` |
| `text` | Search by oracle text | `scryfall text "destroy all creatures"` |
| `type` | Search by type line | `scryfall type "enchantment"` |
| `color` | Search by colour identity | `scryfall color UG` |
| `cmc` | Search by mana value | `scryfall cmc <=3` |
| `keyword` | Search by keyword ability | `scryfall keyword flash` |
| `random` | Get a random card | `scryfall random` |
| `search` | General search (name + text + type) | `scryfall search "2 mana counter"` |

### Options

| Flag | Description | Example |
|------|-------------|---------|
| `--limit N` | Max results (default: 10) | `--limit 20` |
| `--commander WUBRG` | Filter by commander colour identity | `--commander WU` |
| `--legal <format>` | Only cards legal in a format | `--legal commander` |
| `--exact` | Exact name match only | `--exact` |

### Colour Codes

| Code | Colour |
|------|--------|
| W | White ⚪ |
| U | Blue 🔵 |
| B | Black ⚫ |
| R | Red 🔴 |
| G | Green 🟢 |

Combine letters for multicolour: `WG` (Selesnya), `UB` (Dimir), `RUG` (Temur), `WUBRG` (5-colour)

## Examples

```bash
# Find Lightning Greaves
scryfall name "Lightning Greaves"

# All green-white creatures legal in Commander
scryfall type "creature" --commander WG --legal commander

# Cards that create Treasure tokens
scryfall text "create a treasure token"

# Cheap counterspells (CMC 2 or less)
scryfall text "counter target spell" --cmc <=2

# Random card for EDH night
scryfall random

# All flash creatures in blue
scryfall keyword flash --commander U
```

## Using with Local LLMs

`scryfall` pairs well with local language models (LM Studio, Ollama, etc.) for private deckbuilding brainstorming. Run card searches in your terminal while chatting with your local model — no data leaves your machine.

```bash
# Look up cards, then paste the results into your LM session
scryfall text "whenever a creature enters" --legal commander --limit 5
```

## Data Source

Card data is sourced from [Scryfall](https://scryfall.com), the best open MTG card database. The Oracle cards file contains one entry per unique card (~38,000+ cards).

### Updating the cache

Download a fresh copy periodically to stay current with new sets:

```bash
curl -o ~/.hermes/scryfall/oracle-cards.jsonl \
  "https://data.scryfall.io/oracle-cards/oracle-cards.jsonl"
```

Bulk data is refreshed every 12–24 hours by Scryfall.

## Requirements

- Python 3.6+ (pre-installed on macOS and most Linux distributions)
- ~190 MB disk space for the Oracle cards cache

## License

MIT License. See [LICENSE](LICENSE) for details.

Card data is copyright Wizards of the Coast. Scryfall bulk data is provided under [Scryfall's Terms of Service](https://scryfall.com/docs/api/bulk-data).

---

*Built as part of [Kwain.gg](https://kwain.gg) — a project about generosity, attention, and play.*
