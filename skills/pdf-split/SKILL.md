---
name: pdf-split
description: Split PDF documents by content or topic using AI. Use when the user wants to break apart multi-document PDFs, split scanned batches, or separate PDF sections.
allowed-tools: Bash, Read, Glob, mcp__renamed-to__pdf_split, mcp__renamed-to__status
argument-hint: [pdf file]
---

# Split PDFs by Content

You help users split PDF documents using AI to detect document boundaries, or by bookmarks/page count.

## Tool Selection

**Prefer the CLI** (`renamed pdf-split` via Bash) when available — it handles job polling and file downloads automatically.
**Fall back to MCP tools** (`mcp__renamed-to__pdf_split`) if the CLI is not installed and cannot be installed.

## Before You Start

1. **Check the CLI is available** by running `renamed --version` via Bash.
   - If not found, try to install: `brew tap renamed-to/cli && brew install renamed` or `npm install -g @renamed-to/cli`.
   - If installation is not possible, fall back to MCP tools (`mcp__renamed-to__pdf_split`).
2. **Check authentication**: run `renamed doctor` (CLI) or call `mcp__renamed-to__status` (MCP).
   - If not authenticated, direct the user to `renamed auth login`.
3. **Resolve the file path** from `$ARGUMENTS`. Verify it's a PDF.

## CLI Reference

```
renamed pdf-split [options] <file>

Options:
  -m, --mode <mode>               smart (AI, default), every-n-pages, or by-bookmarks
  -i, --instructions <text>       AI instructions for smart mode
  -n, --pages-per-split <number>  Pages per split (for every-n-pages mode)
  -o, --output-dir <dir>          Directory to save split files (default: .)
  -w, --wait                      Wait for completion and download files
  --json                          Machine-readable JSON output
```

**Modes:**
- **smart** (default): AI analyzes content to find document boundaries. Best for scanned batches, mixed documents.
- **every-n-pages**: Fixed page intervals. Best for uniform forms.
- **by-bookmarks**: Split at PDF bookmark boundaries. Best for books, manuals.

## Workflow

### Basic split (AI-powered)

Always use `--wait` so the CLI polls for completion and downloads the files:

```bash
renamed pdf-split scanned-batch.pdf --wait
```

This outputs split files to the current directory.

### With output directory

```bash
renamed pdf-split scanned-batch.pdf --wait -o ./split-output
```

### With AI instructions

Guide the AI on how to split:
```bash
renamed pdf-split invoices.pdf --wait -i "Split by invoice number"
renamed pdf-split contract-bundle.pdf --wait -i "Split at each cover page"
```

### By bookmarks

```bash
renamed pdf-split book.pdf --wait -m by-bookmarks
```

### Every N pages

```bash
renamed pdf-split forms.pdf --wait -m every-n-pages -n 2
```

### JSON output

```bash
renamed pdf-split scanned-batch.pdf --wait --json
```

## Choosing the Right Mode

Ask the user about their document if the mode isn't obvious:

- "Is this a batch of scanned documents?" → **smart** mode
- "Does the PDF have bookmarks/chapters?" → **by-bookmarks**
- "Are documents always the same page count?" → **every-n-pages**
- Default to **smart** if unsure — it works for most cases.

## After Splitting

1. Show the results — filenames, page ranges, and count.
2. **Offer to rename** the split files using `/rename` if the auto-generated names aren't descriptive enough.
3. **Offer to extract** data from the split documents using `/extract`.

## Error Handling

- **Not a PDF**: This tool only works with PDF files.
- **File too large**: Max 100MB for PDF split.
- **Job failed**: AI couldn't determine split points. Suggest trying with `-i` instructions or a different mode.
- **CLI not found**: `brew tap renamed-to/cli && brew install renamed` or `npm install -g @renamed-to/cli`. If neither works, use MCP tools as fallback.
- **Not authenticated**: `renamed auth login`
- **Timeout**: Large PDFs take time. The `--wait` flag handles polling automatically.

## Tips

- Smart mode works best when documents have visually distinct boundaries (different headers, logos, layouts).
- For large batches of PDFs, process sequentially to avoid rate limits.
- Without `--wait`, the CLI returns a job ID immediately. The user would need to check status manually — always use `--wait` for interactive use.
- Chain with rename: `renamed pdf-split batch.pdf --wait -o ./split && renamed rename -a ./split/*.pdf`
