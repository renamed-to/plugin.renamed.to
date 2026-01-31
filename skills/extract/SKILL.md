---
name: extract
description: Extract structured data from documents (PDFs, images, scans). Use when the user wants to pull specific information from documents into JSON or table format.
allowed-tools: Bash, Read, Write, Glob, mcp__renamed-to__extract, mcp__renamed-to__status
argument-hint: [file] [schema description]
---

# Extract Structured Data from Documents

You help users extract structured data from documents using the renamed.to AI extraction service.

## Tool Selection

**Prefer the CLI** (`renamed extract` via Bash) when available — it has richer output and more options.
**Fall back to MCP tools** (`mcp__renamed-to__extract`) if the CLI is not installed and cannot be installed.

## Before You Start

1. **Check the CLI is available** by running `renamed --version` via Bash.
   - If not found, try to install: `brew tap renamed-to/cli && brew install renamed` or `npm install -g @renamed-to/cli`.
   - If installation is not possible, fall back to MCP tools (`mcp__renamed-to__extract`).
2. **Check authentication**: run `renamed doctor` (CLI) or call `mcp__renamed-to__status` (MCP).
   - If not authenticated, direct the user to `renamed auth login`.
3. **Resolve the file path** from `$ARGUMENTS`. First argument is typically the file, remaining text describes what to extract.

## CLI Reference

```
renamed extract [options] <file>

Options:
  -s, --schema <json>        Inline JSON schema defining fields to extract
  -f, --schema-file <path>   Path to JSON schema file
  -p, --parser-id <id>       UUID of a saved parser template
  -i, --instructions <text>  Context for AI extraction
  -o, --output <format>      Output format: json or table (default: table)
```

**Extraction modes:**
- **Discovery** (no schema): AI auto-detects all fields
- **Schema**: Define exact fields via `--schema` or `--schema-file`
- **Parser**: Use a saved template via `--parser-id`

**Field types:** `string`, `number`, `date`, `currency`, `boolean`

## Schema Format

```json
{
  "fields": [
    { "name": "invoice_number", "type": "string" },
    { "name": "total", "type": "currency" },
    { "name": "due_date", "type": "date" }
  ],
  "tables": [
    {
      "name": "line_items",
      "columns": [
        { "name": "description", "type": "string" },
        { "name": "quantity", "type": "number" },
        { "name": "unit_price", "type": "currency" }
      ]
    }
  ]
}
```

## Workflow

### Auto-discovery (simplest)

When the user just wants to see what's in a document:
```bash
renamed extract invoice.pdf
```
This returns all detected fields in table format.

### With a specific schema

When the user knows what they want:

1. Build the schema from their description. If the user says "extract the invoice number, date, and total", create:
   ```bash
   renamed extract invoice.pdf -s '{"fields":[{"name":"invoice_number","type":"string"},{"name":"invoice_date","type":"date"},{"name":"total","type":"currency"}]}'
   ```

2. For complex schemas, write a JSON file and use `--schema-file`:
   ```bash
   renamed extract invoice.pdf -f schema.json
   ```

### JSON output (for saving/piping)

```bash
renamed extract invoice.pdf -o json
renamed extract invoice.pdf -s '...' -o json > extracted.json
```

### Batch extraction

For multiple files with the same schema, loop through them:
```bash
for f in ~/invoices/*.pdf; do renamed extract "$f" -s '...' -o json; done
```

Or write a schema file and process each file:
```bash
renamed extract invoice1.pdf -f schema.json -o json
renamed extract invoice2.pdf -f schema.json -o json
```

Collect results and offer to save as a combined JSON file or CSV.

## When the User Describes What They Want

If the user says something like "extract invoice details", build the schema for them:

1. Ask clarifying questions if needed: "Do you want line items too, or just header fields?"
2. Construct the schema JSON.
3. Run the extraction.
4. Show the results.
5. Offer to save as JSON or CSV using `Write`.

For common document types, proactively suggest schemas:
- **Invoices**: invoice_number, date, vendor, total, line_items
- **Receipts**: date, merchant, total, payment_method
- **Contracts**: parties, effective_date, term, value

## Error Handling

- **CLI not found**: `brew tap renamed-to/cli && brew install renamed` or `npm install -g @renamed-to/cli`. If neither works, use MCP tools as fallback.
- **Not authenticated**: `renamed auth login`
- **Unsupported file type**: Only PDF, JPG, JPEG, PNG, TIFF.
- **File too large**: Max 25MB.
- **Bad schema JSON**: Validate the JSON before passing to `--schema`. Common mistake: forgetting to quote properly in shell.

## Tips

- Auto-discovery mode (`renamed extract file.pdf` with no schema) is great for exploring a document before defining a precise schema.
- Use `-i` instructions to provide context: `renamed extract invoice.pdf -i "This is a German invoice"`.
- For batch extraction, test the schema on one file first before processing the whole batch.
- Pipe JSON output to `jq` for quick filtering: `renamed extract invoice.pdf -o json | jq '.fields'`.
