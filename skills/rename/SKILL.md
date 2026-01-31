---
name: rename
description: Rename files intelligently using AI content analysis. Use when the user wants to rename files based on their content, batch rename documents, or organize file names.
allowed-tools: Bash, Read, Glob
argument-hint: [files or directory]
---

# Rename Files by Content

You help users rename files using the `renamed` CLI, which sends files to the renamed.to AI service for content analysis and intelligent renaming.

## Before You Start

1. **Check the CLI is available** by running `renamed --version`. If not found, tell the user to install it: `npm install -g @renamed-to/cli` or `brew install renamed-to/cli/renamed`.
2. **Check authentication** by running `renamed doctor`. If not authenticated, direct them to `renamed auth login`.
3. **Resolve file paths** from `$ARGUMENTS`. If the user provided a directory, use `Glob` to find supported files (PDF, JPG, JPEG, PNG, TIFF).

## CLI Reference

```
renamed rename [options] <files...>

Options:
  -a, --apply                 Apply the rename (without this, it only previews)
  -o, --output-dir <dir>      Output directory
  -p, --prompt <instruction>  Custom AI instruction for filename format
  -s, --strategy <name>       Folder organization strategy
  -t, --template <name>       Predefined filename template
  -l, --language <code>       Output language (en, de, fr, es, ...)
  --overwrite                 Overwrite existing files
  --on-conflict <mode>        fail (default), skip, or suffix
  --json                      Machine-readable JSON output
```

**Strategies:** `by_date`, `by_issuer`, `by_type`, `by_date_issuer`, `by_date_type`, `by_issuer_type`, `by_all`, `root`, `follow_custom_prompt`

**Templates:** `standard`, `date_first`, `company_first`, `minimal`, `detailed`, `department_focus`

## Workflow

### Single File

1. **Preview first** — always run without `-a` to show the suggested name:
   ```bash
   renamed rename invoice.pdf
   ```
2. Show the user the before/after preview from the output.
3. If the user approves, apply the rename:
   ```bash
   renamed rename -a invoice.pdf
   ```

### Batch Rename

1. **Preview the batch:**
   ```bash
   renamed rename *.pdf
   ```
   Or for a directory of mixed files:
   ```bash
   renamed rename ~/Documents/inbox/*.pdf ~/Documents/inbox/*.jpg
   ```
2. Show the full preview table to the user.
3. If approved, apply:
   ```bash
   renamed rename -a ~/Documents/inbox/*.pdf ~/Documents/inbox/*.jpg
   ```

### With JSON output (for programmatic use)

```bash
renamed rename --json invoice.pdf
renamed rename --json -a *.pdf
```

## Options to Offer

Only mention these if the user asks or the context calls for it:

- **Custom prompt**: `renamed rename -a -p "Format: YYYY-MM-DD_CompanyName_Type" invoice.pdf`
- **Strategy**: `renamed rename -a -s by_date -o ~/Documents invoice.pdf`
- **Template**: `renamed rename -a -t date_first invoice.pdf`
- **Language**: `renamed rename -a -l de invoice.pdf`
- **Conflict handling**: `renamed rename -a --on-conflict suffix *.pdf`

## Error Handling

- **CLI not found**: `npm install -g @renamed-to/cli`
- **Not authenticated**: `renamed auth login`
- **Unsupported file type**: Tell the user which types are supported (PDF, JPG, JPEG, PNG, TIFF).
- **File too large**: Max 25MB per file.

## Tips

- Always preview before applying. The CLI defaults to preview mode (no `-a`), which is safe.
- For large batches, use `--json` output to parse results programmatically if needed.
- In a git repo, mention that `git mv` may be preferred to preserve history.
- The `--on-conflict suffix` flag is useful for batches where name collisions are likely.
