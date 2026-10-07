# scryfall-local

A lightweight, offline command-line tool for searching [Scryfall](https://scryfall.com) Oracle cards from a local cache. No API key, no rate limits, no internet required.

Built for Magic: The Gathering players who want fast card lookups during deckbuilding, theorycrafting, or kitchen-table sessions — especially useful alongside local LLM tools for private brainstorming.

## Features

- 🔍 **Search by name, text, type, colour, mana cost, or keyword**
- 🔮 **Fuzzy name search** — handles typos, misspellings, and partial names
- 💰 **Price lookup** — USD, EUR, and TIX with foil variants
- 🖼️ **Card image download** — grab high-res PNGs straight to your Desktop
- 🎲 **Random card** generation
- ⚡ **Instant** — loads 38,000+ cards in under two seconds
- 🔒 **Offline** — works without internet once the cache is downloaded
- 🎨 **Colour-coded output** with emoji symbols (⚪🔵⚫🔴🟢)
- 🏛️ **Format filtering** — Commander, Modern, Legacy, Pauper, Vintage
- 🧩 **Commander colour identity** filtering
- 📊 **Sorting** — by name, mana value, or price
- 🔄 **Built-in cache updater** — `scryfall update` fetches the latest data from Scryfall
- 🛡️ **Input validation** — clear errors for empty queries and unknown commands

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
| `price` | Search by name with market prices | `scryfall price "Rhystic Study"` |
| `image` | Download a card's image (PNG to Desktop) | `scryfall image "Sol Ring"` |
| `random` | Get a random card | `scryfall random` |
| `search` | General search across name + text + type | `scryfall search "2 mana green creature"` |
| `update` | Update the Oracle cache from Scryfall | `scryfall update` |

## Options

| Flag | Description | Example |
|------|-------------|---------|
| `--limit N` | Max results (default: 10) | `--limit 20` |
| `--commander WUBRG` | Cards whose colour identity fits within these colours | `--commander WU` |
| `--legal <format>` | Only cards legal in a format | `--legal commander` |
| `--exact` | Exact name match only (for `name` command) | `--exact` |
| `--sort <field>` | Sort by `name`, `cmc`, or `price` (descending) | `--sort price` |
| `--size <s\|n\|l\|png>` | Image size for `image`: small, normal, large, png (default: png) | `--size large` |
| `--out <dir>` | Output directory for `image` (default: `~/Desktop`) | `--out ~/Pictures` |
| `--border <color>` | Prefer a specific border colour for `image` (black, silver) | `--border silver` |

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

# Download a card image to your Desktop
scryfall image "Sol Ring"          # → ~/Desktop/Sol Ring.png

# All flash creatures in blue
scryfall keyword flash --commander U
```

## Price Lookup

The `price` command searches by card name and displays market prices prominently — USD, EUR, and TIX (Magic Online tickets), with foil prices shown when available.

```shell
# Look up prices for a specific card
scryfall price "Force of Will"

# Price-sorted results for a partial name
scryfall price "bolt" --limit 5

# Prices for cards in your colour identity
scryfall price "draw" --commander WU --limit 10
```

Example output:

```
  [1] Force of Will  {3}{U}{U}  (CMC 5)
       Instant  •  🔵
       USD $60.09 (foil $71.56)  •  EUR €51.00 (foil €55.44)  •  TIX 16.99
       https://scryfall.com/card/dmr/50/force-of-will
```

The `price` command sorts by USD value (most expensive first) by default. You can also add `--sort price` to any other command to order results by price.

**Note:** Prices come from the Oracle cache and reflect the last `scryfall update`. Run `scryfall update` to refresh if prices seem stale.

## Card Images

The `image` command downloads a card's picture — PNG by default — straight to your **Desktop**:

```shell
# Highest-quality PNG → ~/Desktop/Sol Ring.png
scryfall image "Sol Ring"

# Typo? No problem — resolves via fuzzy match + local fallbacks
scryfall image "sol rng"              # → Sol Ring.png
scryfall image "jace bele"            # → Jace Beleren.png

# Smaller JPEG instead of PNG
scryfall image "Lightning Bolt" --size large

# Save somewhere else
scryfall image "Force of Will" --out ~/Pictures
```

Example output:

```
  Card: Sol Ring
  Source: https://cards.scryfall.io/png/front/8/e/8ee443cc-e17a-493b-9c93-1f9e141a30e4.png
  ✓ Saved: /Users/you/Desktop/Sol Ring.png  (753 KB)
```

How it works:

1. **Name resolution** in three tiers — exact match in your local cache first (instant, offline), then Scryfall's fuzzy API for typos, then a local word-similarity fallback that catches cases where the API returns an unrelated printed-name coincidence
2. **Size selection** — `--size small|normal|large|png` (default `png`, the high-res version). The file extension always matches the real format (`.png` vs `.jpg`)
3. **Collision-safe** — never overwrites existing files; adds ` (1)`, ` (2)`, … as needed
4. **Double-faced cards** — uses the front face's image when a card stores images per face

Unlike the rest of the tool, `image` needs an internet connection to fetch the picture itself (Scryfall hosts the images on their CDN).

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

## Input Validation

The tool validates your input and gives clear feedback:

- **Empty queries** — tells you what's missing instead of dumping all 38,000 cards
- **Unknown commands** — shows the list of valid commands and suggests `scryfall search` as a fallback
- **Unknown sort fields** — warns and suggests valid options (`name`, `cmc`, `price`)

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
