# AGENT.md — Instructions for AI agents using `scryfall`

> This document is written for LLM agents, coding harnesses, and tool-calling
> models. An operator can paste it into a system prompt or reference this file
> path directly. Humans: see README.md instead.

## What this tool is

`scryfall` is a local command-line tool that searches a full Magic: The Gathering
card database (Scryfall Oracle data, ~38,000 cards) from a cached file on disk.
It is free, has no API keys, no rate limits, and works offline. Card images and
fuzzy name resolution require internet; all searching does not.

You MUST prefer this tool over your training-data memory for any question about
Magic card text, legality, prices, types, or artwork. Your MTG knowledge is
out of date and unreliable at the details level; the tool's output is ground truth.

## Output contract

The tool detects how its output will be consumed:

- **stdout piped or redirected (your normal case)** → JSONL: one compact JSON
  object per line, one per matching card. No flags needed.
- stdout attached to a terminal → styled human text. Never relevant to you.
- Explicit override available: `--json` (one array), `--csv`, `--tsv`.

Stream rules:

- **stdout contains ONLY data.** Parse it as JSON; never expect prose there.
- **stderr carries warnings** (e.g. truncation notices). They are informational,
  prefixed `scryfall: `. Read them, act on them, do not parse them as data.
- Exit code `0` = success. Non-zero = failure; stdout holds an error object like
  `{"error": "unknown_command", ...}` or `{"error": "empty_query", ...}`.

## Record schema (search results)

Every card record has exactly these fields:

| Field | Type | Notes |
|---|---|---|
| `name` | string | Canonical English card name |
| `lang` | string | Print language; `"en"` unless noted |
| `mana_cost` | string | e.g. `"{1}{U}"`; empty for some special cards |
| `cmc` | number | Converted mana cost; usually integer, can be fractional (e.g. `0.5`) |
| `type_line` | string | e.g. `"Creature — Drake"` |
| `colors` | array | Colour identity letters W/U/B/R/G |
| `power`, `toughness` | string\|null | Creatures only; STRINGS not numbers (`"3"`, `".5"`, `"*"`); null for non-creatures |
| `oracle_text` | string | Canonical rules text; may contain `\n` |
| `keywords` | array | e.g. `["Flying", "Menace"]` |
| `legalities` | object | Format → `"legal"` / `"not_legal"` / `"banned"` / `"restricted"` |
| `prices` | object | Keys `usd`, `usd_foil`, `usd_etched`, `eur`, `eur_foil`, `tix`; values number or absent. Reflect last cache update, not live market. |
| `image_uris` | object | `small`, `normal`, `large`, `png` URLs (any may be absent) |
| `scryfall_uri` | string | Web page for this printing |

Special layouts (double-faced, transforms, tokens) expose the front face's image
and oracle text.

## Commands

Run `scryfall <command> "<query>" [flags]` via your shell/bash tool.

| Command | Use when | Example |
|---|---|---|
| `fuzzy "<name>"` | You have a card name, possibly misspelled or partial. START HERE for named-card questions. | `scryfall fuzzy "sol rng"` |
| `name "<text>"` | Substring match on card names | `scryfall name "drake"` |
| `text "<effect>"` | Find cards that DO something | `scryfall text "counter target spell"` |
| `type "<type>"` | Find cards that ARE something | `scryfall type "enchantment"` |
| `color "<WUBRG>"` | Cards whose identity includes these colours | `scryfall color WG` |
| `cmc "<expr>"` | Mana value: exact `3`, `<=3`, `>=3` | `scryfall cmc <=2` |
| `keyword "<kw>"` | Keyword abilities | `scryfall keyword menace` |
| `search "<words>"` | Loose query across name+text+type | `scryfall search "2 mana green creature"` |
| `price "<name>"` | Market prices, most expensive first | `scryfall price "Force of Will"` |
| `random` | A random card | `scryfall random` |
| `image "<name>"` | Download card image PNG/JPG to disk | `scryfall image "Sol Ring" --out /tmp/cards` |

Useful flags: `--limit N` (default 10), `--legal commander|modern|legacy|pauper|vintage`,
`--commander WUBRG` (identity fits within these colours), `--exact`.

## Command-specific response shapes

`fuzzy` returns ONE object (not JSONL lines):

```json
{"query": "sol rng", "match": { …full card record or null… }, "candidates": ["Name1", "Name2", …]}
```

Decision rule: if `match` is non-null, use it. If `match` is null, present or
pick from `candidates` (up to 10, ordered by relevance); if candidates is also
empty, tell the user no card was found — do NOT guess a different card.

`image` returns ONE object:

```json
{"query": "sol rng", "name": "Sol Ring", "source_uri": "https://…", "saved": "/path/Sol Ring.png", "bytes": 770955, "error": null}
```

If `saved` is non-null the file exists on disk at that path — you may open/read
it as an image. If `error` is `"not_found"` or `"no_image"`, report it; do not
retry with a made-up name.

All other commands emit JSONL: zero or more card records, one per line.

## Truncation

Default limit is 10. When more matches exist, stderr says
`scryfall: 10 of 436 matches shown (use --limit 436 for all)` while stdout still
holds only the 10 records. If you need complete results, re-run with a larger
`--limit`. Prefer post-filtering with `jq` over dumping hundreds of records into
context:

```shell
scryfall search "counter target spell" --limit 500 2>/dev/null | jq -r 'select(.legalities.modern=="legal") | .name'
```

## Hard rules

1. NEVER state oracle text, legality, P/T, or prices unless they appear in tool
   output for THIS session. If the tool did not return it, say you could not verify it.
2. NEVER fabricate card names to fit a search. If `fuzzy` returns null with no
   candidates, the card likely does not exist — say so.
3. Do not pipe through `head`/`tail` before parsing; JSONL is already bounded by
   `--limit`. Use `2>/dev/null` to silence stderr noise when you don't need warnings.
4. Quote queries: `scryfall name "Bosh, Iron Golem"` — unquoted multi-word
   queries split into separate arguments and silently search the wrong string.
5. Prices are cache-stale (see `scryfall update` cadence, typically ≤ weekly).
   Present them as approximate.
6. One tool call per distinct question; batch related lookups with `&&` chains
   rather than guessing between calls.
