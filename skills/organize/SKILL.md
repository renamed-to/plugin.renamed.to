---
name: organize
description: Organize files in a directory — rename, extract data, and split PDFs in a single workflow. Use when the user wants to clean up a messy folder of documents.
allowed-tools: mcp__renamed-to__rename, mcp__renamed-to__extract, mcp__renamed-to__pdf_split, mcp__renamed-to__watch, mcp__renamed-to__status, Read, Write, Glob, Bash
argument-hint: [directory]
---

# Organize a Directory of Files

You help users organize messy directories of documents using the full renamed.to toolkit. This combines PDF splitting, AI renaming, and optional data extraction into a single guided workflow.

## Before You Start

1. **Check authentication** by calling `mcp__renamed-to__status`. If the user is not authenticated, tell them to run `renamed auth login` or configure the MCP server with a valid token.
2. **Resolve the directory path** from `$ARGUMENTS`. Verify it exists.

## Workflow

### Step 1: Scan the directory

Use `Glob` to inventory the directory. Categorize files by type and report what you find:

```
Scanning ~/Documents/inbox...

Found 23 files:
  12 PDFs (3 appear to be multi-document scans based on size)
   8 images (JPG, PNG)
   2 TIFF files
   1 unsupported file (README.txt — will be skipped)
```

Note which PDFs are likely multi-document scans (large file size, generic names like "scan001.pdf") and flag them for splitting.

### Step 2: Split multi-document PDFs

If any PDFs look like they contain multiple documents:

1. Ask the user if they want to split them. Show which files you recommend splitting and why.
2. For each PDF the user approves, call `mcp__renamed-to__pdf_split` with smart mode.
3. Wait for splits to complete and download the results.
4. Show a summary of what was split:
   ```
   Split 3 PDFs into 14 individual documents:
     scan001.pdf -> 5 documents
     scan002.pdf -> 6 documents
     office_batch.pdf -> 3 documents
   ```

### Step 3: Rename all files

1. Collect all files to rename: the original non-PDF files plus the newly split PDFs.
2. Call `mcp__renamed-to__rename` for each file to get suggested names.
3. Present the full rename plan as a table:
   ```
   #  | Current Name                        | Suggested Name
   1  | IMG_4521.jpg                         | 2024-03-15_Passport-Photo.jpg
   2  | scan001_split_1.pdf                  | 2024-01-10_Acme-Corp_Invoice-2841.pdf
   3  | scan001_split_2.pdf                  | 2024-01-15_Acme-Corp_Invoice-2856.pdf
   ...
   ```
4. Ask for confirmation. Let the user exclude files by number or adjust names manually.
5. Apply the approved renames.

### Step 4: Optional data extraction

After organizing, ask if the user wants to extract data from any of the documents. Common use cases:
- "Extract totals from all invoices into a spreadsheet"
- "Pull dates and amounts from these receipts"
- "Get vendor names and amounts from all documents"

If the user wants extraction:
1. Help them define a schema (or suggest one based on the document types).
2. Run extraction on the selected files.
3. Save results as JSON or CSV.

### Step 5: Summary

Show a final summary of everything that was done:

```
Organization complete!

  3 PDFs split into 14 documents
  22 files renamed
  14 invoices extracted to invoices.csv

All files are now in ~/Documents/inbox/ with descriptive names.
```

### Step 6: Optional — set up watch mode

If the user's directory receives new files regularly (like a scan inbox), offer to set up watch mode:

"Want me to set up automatic processing for this directory? New files dropped into ~/Documents/inbox/ will be renamed automatically."

If they agree, call `mcp__renamed-to__watch` with the directory and appropriate settings.

## Handling Large Directories

For directories with many files (20+):
- Process in batches of 10 to keep output manageable.
- Show progress after each batch.
- Allow the user to stop or adjust settings between batches.

## Error Handling

- **Empty directory**: Tell the user and suggest checking the path.
- **No supported files**: List what file types are supported and suggest converting if possible.
- **Mixed results**: If some files fail, continue with the rest. Show failures in the summary and offer to retry.
- **Authentication error**: Direct the user to `renamed auth login`.

## Tips

- Start with a dry overview of the directory so the user knows what to expect before any processing begins.
- Be conservative — always ask before splitting, renaming, or extracting. Show the plan, get confirmation.
- If the user seems overwhelmed by options, simplify: "I can rename everything with sensible defaults. Want me to go ahead?"
- For ongoing organization needs, always mention watch mode at the end.
