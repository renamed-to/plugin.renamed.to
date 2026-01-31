---
name: pdf-split
description: Split PDF documents by content or topic using AI. Use when the user wants to break apart multi-document PDFs, split scanned batches, or separate PDF sections.
allowed-tools: mcp__renamed-to__pdf_split, mcp__renamed-to__status, Read, Glob, Bash
argument-hint: [pdf file]
---

# Split PDFs by Content

You help users split PDF documents using the renamed.to AI service. This is useful for breaking apart multi-document scans, separating PDF sections by topic, or splitting large PDFs into manageable pieces.

## Before You Start

1. **Check authentication** by calling `mcp__renamed-to__status`. If the user is not authenticated, tell them to run `renamed auth login` or configure the MCP server with a valid token.
2. **Resolve the file path** from `$ARGUMENTS`. Verify it is a PDF file.

## Split Modes

### Smart (AI-powered) — default
The AI analyzes document content and determines natural split points. Best for:
- Scanned document batches (multiple documents in one PDF)
- Mixed-content PDFs (invoices followed by contracts followed by receipts)
- Any PDF where you want intelligent content-based splitting

### By Bookmarks
Splits at existing PDF bookmark boundaries. Best for:
- Well-structured PDFs with a table of contents
- eBooks or reports with chapter bookmarks

### Every N Pages
Splits at fixed page intervals. Best for:
- Uniformly structured documents (e.g., every 2 pages is a separate form)
- Simple mechanical splitting

## Workflow

1. Confirm the file is a PDF and within the 100MB size limit.
2. Ask which split mode the user wants if not obvious from context. Default to **smart** mode.
3. If using smart mode, ask if the user has any instructions for how to split (e.g., "split by invoice" or "separate each letter"). This is optional — the AI works well without instructions.
4. Call `mcp__renamed-to__pdf_split` with the file and options.
5. The split is an async job. Poll `mcp__renamed-to__status` for progress and keep the user informed.
6. When complete, show the results:

```
Split complete! 5 documents created:

# | Filename                              | Pages
1 | 2024-03-15_Acme-Corp_Invoice.pdf      | 1-3
2 | 2024-03-20_GlobalTech_Contract.pdf     | 4-8
3 | 2024-04-01_Smith-LLC_Receipt.pdf       | 9
4 | 2024-04-05_Acme-Corp_Statement.pdf     | 10-12
5 | 2024-04-10_Insurance-Claim.pdf         | 13-15
```

7. The split files are available for download. Ask where the user wants them saved (default: same directory as the source file).
8. After downloading, offer to rename the split files using the `/rename` skill if the auto-generated names are not ideal.

## Options to Offer

- **Instructions**: Free-text guidance for the AI on how to split. Only relevant for smart mode. Examples: "Each invoice is a separate document", "Split at each cover page", "Separate by company name".
- **Pages per split**: Only for `every-n-pages` mode. Ask the user how many pages per chunk.
- **Output directory**: Where to save split files. Default is the same directory as the source PDF.

## Error Handling

- **Not a PDF**: Tell the user this tool only works with PDF files.
- **File too large**: The PDF split service accepts files up to 100MB. Suggest using a local tool to reduce file size first.
- **Job failed**: The AI could not determine how to split the document. Suggest trying with explicit instructions or a different split mode.
- **Authentication error**: Direct the user to `renamed auth login`.
- **Timeout**: Large PDFs can take a while to process. Reassure the user and continue polling. If it takes more than a few minutes, suggest they check back later.

## Tips

- Smart mode works best when documents in the PDF have visually distinct boundaries (different headers, logos, or layouts).
- For scanned document batches, smart mode is almost always the right choice.
- If the user has a large batch of PDFs to split, process them sequentially rather than in parallel to avoid rate limits.
- After splitting, suggest using `/rename` to ensure the split files have descriptive names, and `/extract` to pull data from individual documents.
