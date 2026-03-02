# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a static content archive containing **303 episode transcripts** from Lenny's Podcast, with an AI-generated topic index (88 topics) for discovery. There is no build system, test suite, or application code — the repository is purely content (markdown) plus one Bash script for index generation.

## Structure

```
├── CLAUDE.md                        # This file — AI assistant guidance
├── README.md                        # Project overview and usage docs
├── episodes/                        # 303 guest directories (~26 MB total)
│   └── {guest-name}/
│       └── transcript.md            # YAML frontmatter + full transcript
├── index/                           # AI-generated topic index (~262 KB)
│   ├── README.md                    # Master topic listing with episode counts
│   └── {topic}.md                   # 88 topic files (e.g., product-management.md)
└── scripts/
    └── build-index.sh               # Index generator (requires Claude CLI + jq)
```

### Naming conventions

- Episode directories use lowercase, hyphenated guest names: `brian-chesky`, `adam-fishman`
- Some guests have multiple episodes with suffixed directories: `wes-kao`, `wes-kao-20`
- Topic index files use lowercase, hyphenated topic names: `product-management.md`, `ab-testing.md`

## Transcript Format

Each `episodes/{guest}/transcript.md` has two parts:

### 1. YAML Frontmatter (between `---` delimiters)

Fields:
- `guest` — Guest name(s)
- `title` — Full episode title
- `youtube_url` — Link to the YouTube video
- `video_id` — YouTube video identifier
- `description` — Episode description (may be multi-line)
- `duration_seconds` — Episode length as a float
- `duration` — Human-readable duration (e.g., `'1:05:46'`)
- `view_count` — Views at time of archival
- `channel` — Always `Lenny's Podcast`
- `keywords` — List of topic tags (e.g., `product-market fit`, `growth`, `retention`)

### 2. Transcript Content

After the closing `---`, the transcript follows with:
- An H1 heading matching the episode title
- An H2 `## Transcript` section
- Timestamped speaker dialogue: `Speaker Name (HH:MM:SS):`

File sizes range from ~64 lines (short teasers) to ~2,400 lines (long interviews). Most are 400–800 lines.

## Index

The `index/` directory contains 88 AI-generated topic files plus a `README.md` entry point.

- **`index/README.md`** — Lists all topics with episode counts (last generated: 2026-01-14, covering 269 of 303 episodes)
- **Topic files** — Each lists episodes tagged with that topic as markdown links back to transcripts

Largest topics: Product Management (142 episodes), Leadership (73), Entrepreneurship (52), Product Strategy (52), Product Development (46).

**Note:** 34 episodes added after the last index build are not yet indexed. Run `./scripts/build-index.sh` to index them.

## Working with Large Transcript Files

Transcript files are large (often 25,000+ tokens). Use these strategies:

### 1. Use Grep for targeted searches (preferred)
```
# Search for specific topics across all transcripts
Grep pattern="product.market fit" path="episodes/"

# Search with context lines for better understanding
Grep pattern="early stage" path="episodes/" output_mode="content" -C=5
```

### 2. Read frontmatter first (lines 1-15)
Get metadata before deciding to read more:
```
Read file_path="episodes/guest-name/transcript.md" limit=15
```

### 3. Read in chunks when needed
For sequential reading, use offset/limit:
```
Read file_path="..." offset=1 limit=500    # First chunk
Read file_path="..." offset=500 limit=500  # Second chunk
```

### 4. Use Agent tool with Explore agent
For research across multiple transcripts:
```
Agent subagent_type="Explore" prompt="Find insights about X across transcripts"
```

### 5. Handle persisted output
When Read returns a persisted output path like:
`Output saved to: ~/.claude/.../tool-results/xxx.txt`
Read that file to access the full content.

## Rebuilding the Index

```bash
./scripts/build-index.sh
```

**Requirements:** Claude CLI (`claude` command) and `jq` must be available on PATH.

**How it works:**
1. Iterates over every `episodes/*/transcript.md`
2. Skips episodes already referenced in existing topic files (idempotent)
3. Sends each transcript to Claude Sonnet to extract 4–6 broad topic keywords as JSON
4. Appends episodes to the corresponding `index/{topic}.md` files, creating new files as needed
5. Regenerates `index/README.md` with updated topic counts

The script is idempotent and incremental — it can be interrupted and safely rerun. It includes a 1-second delay between API calls to avoid rate limiting.

**Environment variables:**
- `EPISODES_DIR` — Episode source directory (default: `episodes`)
- `OUTPUT_DIR` — Index output directory (default: `index`)
- `TEMP_DIR` — Temporary working directory (default: auto-created via `mktemp`)
