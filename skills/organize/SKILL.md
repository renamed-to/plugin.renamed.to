---
name: organize
description: Organize files in a directory — rename, extract data, and split PDFs in a single workflow. Use when the user wants to clean up a messy folder of documents.
allowed-tools: Bash, Read, Write, Glob, mcp__renamed-to__rename, mcp__renamed-to__pdf_split, mcp__renamed-to__watch, mcp__renamed-to__status
argument-hint: [directory]
---

# Organize a Directory of Files

You help users organize messy directories — combining PDF splitting, AI renaming, and optional watch mode into a guided workflow.

## Tool Selection

**Prefer the CLI** (`renamed` commands via Bash) when available — richer output, more options, handles job polling automatically.
**Fall back to MCP tools** if the CLI is not installed and cannot be installed.

## Before You Start

1. **Check the CLI is available** by running `renamed --version` via Bash.
   - If not found, try to install: `brew tap renamed-to/cli && brew install renamed` or `npm install -g @renamed-to/cli`.
   - If installation is not possible, fall back to MCP tools.
2. **Check authentication**: run `renamed doctor` (CLI) or call `mcp__renamed-to__status` (MCP).
   - If not authenticated, direct the user to `renamed auth login`.
3. **Resolve the directory** from `$ARGUMENTS`. Verify it exists.

## Workflow

### Step 1: Scan the directory

Use `Glob` to inventory the directory. Report what you find:

```
Scanning ~/Documents/inbox...

Found 23 files:
  12 PDFs (3 large files that may contain multiple documents)
   8 images (JPG, PNG)
   2 TIFF files
   1 unsupported file (README.txt — will skip)
```

Flag large PDFs with generic names (scan001.pdf, batch.pdf) as likely multi-document scans.

### Step 2: Split multi-document PDFs

If any PDFs look like multi-document scans:

1. Ask the user which ones to split.
2. For each approved PDF:
   ```bash
   renamed pdf-split scan001.pdf --wait -o ~/Documents/inbox
   ```
3. Show results:
   ```
   Split 3 PDFs into 14 individual documents:
     scan001.pdf → 5 documents
     scan002.pdf → 6 documents
     office_batch.pdf → 3 documents
   ```

### Step 3: Rename all files

1. **Preview first** — always preview before applying:
   ```bash
   renamed rename ~/Documents/inbox/*.pdf ~/Documents/inbox/*.jpg ~/Documents/inbox/*.png
   ```
2. Show the full rename plan to the user.
3. If approved, apply:
   ```bash
   renamed rename -a ~/Documents/inbox/*.pdf ~/Documents/inbox/*.jpg ~/Documents/inbox/*.png
   ```

For folder organization, add a strategy:
```bash
renamed rename -a -s by_type -o ~/Documents ~/Documents/inbox/*.pdf
```

### Step 4: Summary

Show what was accomplished:
```
Organization complete!

  3 PDFs split into 14 documents
  22 files renamed

All files are now in ~/Documents/inbox/ with descriptive names.
```

### Step 5: Offer watch mode

If the directory receives files regularly:

"Want me to set up automatic processing? New files will be renamed automatically."

```bash
renamed watch ~/Documents/inbox -o ~/Documents --split-pdfs
```

For a dry run first:
```bash
renamed watch ~/Documents/inbox -o ~/Documents --dry-run
```

## Watch Mode Options

```bash
# Basic: rename new files automatically
renamed watch ~/inbox -o ~/Documents

# With PDF splitting
renamed watch ~/inbox -o ~/Documents --split-pdfs

# Specific file types only
renamed watch ~/inbox -o ~/Documents -p "*.pdf" "*.jpg"

# Pipeline mode (never block on failures)
renamed watch ~/inbox -o ~/Documents --passthrough

# Docker/NFS volumes
renamed watch /data -o /output --poll
```

## Handling Large Directories

For 20+ files:
- Process in batches — split PDFs first, then rename in groups of ~10.
- Show progress between batches.
- Let the user stop or adjust between batches.

## Error Handling

- **Empty directory**: Check the path.
- **No supported files**: Supported types are PDF, JPG, JPEG, PNG, TIFF.
- **Partial failures**: Continue with remaining files. Show failures in summary and offer to retry.
- **CLI not found**: `brew tap renamed-to/cli && brew install renamed` or `npm install -g @renamed-to/cli`. If neither works, use MCP tools as fallback.
- **Not authenticated**: `renamed auth login`

## Tips

- Always start with a scan overview so the user knows what to expect.
- Be conservative — preview before splitting, preview before renaming. Get confirmation.
- If the user seems overwhelmed: "I can rename everything with sensible defaults. Want me to go ahead?"
- Always mention watch mode for directories that receive new files regularly.
