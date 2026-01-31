---
name: rename
description: Rename files intelligently using AI content analysis. Use when the user wants to rename files based on their content, batch rename documents, or organize file names.
allowed-tools: mcp__renamed-to__rename, mcp__renamed-to__status, Read, Glob, Bash
argument-hint: [files or directory]
---

# Rename Files by Content

You help users rename files using the renamed.to AI service, which analyzes document content to generate descriptive, consistent file names.

## Before You Start

1. **Check authentication** by calling `mcp__renamed-to__status`. If the user is not authenticated, tell them to run `renamed auth login` or configure the MCP server with a valid token.
2. **Resolve the file paths** from `$ARGUMENTS`. If the user provided a directory, use `Glob` to find supported files inside it (PDF, JPG, JPEG, PNG, TIFF).

## Supported File Types

PDF, JPG, JPEG, PNG, TIFF — these are the file types the rename API accepts.

## Workflow

### Single File

1. Call `mcp__renamed-to__rename` with the file path.
2. Show the user a before/after preview:
   ```
   Before: IMG_20240315_scan.pdf
   After:  2024-03-15_Acme-Corp_Invoice-4821.pdf
   ```
3. Ask the user to confirm before applying the rename.

### Batch Rename (multiple files or directory)

1. Collect all matching files using `Glob`.
2. Call `mcp__renamed-to__rename` for each file to get suggestions.
3. Present a summary table of all proposed renames:
   ```
   # | Current Name              | Suggested Name                        | Status
   1 | scan001.pdf               | 2024-03-15_Acme-Corp_Invoice.pdf      | ready
   2 | photo.jpg                 | 2024-01-20_Passport-Photo.jpg         | ready
   3 | doc.pdf                   | 2024-06-01_Lease-Agreement.pdf        | ready
   ```
4. Ask the user to confirm the batch before applying. Let them exclude specific files by number if needed.
5. Apply the renames and report results.

## Options to Offer

- **Strategy**: Ask if the user wants a specific organization strategy. Available strategies: `by_date`, `by_issuer`, `by_type`, `by_date_issuer`, `by_date_type`, `by_issuer_type`, `by_all`, `root`, `follow_custom_prompt`. Default is usually fine — only mention strategies if the user asks for specific organization.
- **Template**: Available naming templates: `standard`, `date_first`, `company_first`, `minimal`, `detailed`, `department_focus`. Only offer if the user asks for a specific naming style.
- **Custom prompt**: If the user wants a naming convention not covered by templates, pass their instructions as a custom prompt with the `follow_custom_prompt` strategy.
- **Language**: The service can generate names in different languages. Ask if the file content is non-English or the user prefers names in another language.
- **Output directory**: If the user wants renamed files moved to a different directory, pass the output directory option.

## Handling Conflicts

If a rename would overwrite an existing file:
- Tell the user about the conflict.
- Offer options: skip, overwrite, or add a suffix to make the name unique.

## Error Handling

- **File not found**: Check the path and suggest corrections. Use `Glob` to find similar files.
- **Unsupported file type**: Tell the user which types are supported and suggest converting their file.
- **Authentication error**: Direct the user to `renamed auth login`.
- **Network error**: Suggest checking internet connectivity and retrying.

## Tips

- For large batches (10+ files), process them in groups and show progress.
- If the user is unsure about the results, remind them that no files are changed until they confirm.
- When renaming files in a git repository, mention that they may want to use `git mv` instead of a filesystem rename to preserve history.
