---
name: extract
description: Extract structured data from documents (PDFs, images, scans). Use when the user wants to pull specific information from documents into JSON or CSV format.
allowed-tools: mcp__renamed-to__extract, mcp__renamed-to__status, Read, Write, Glob
argument-hint: [file] [schema description]
---

# Extract Structured Data from Documents

You help users extract structured data from documents using the renamed.to AI extraction service. This turns unstructured PDFs, images, and scans into clean JSON or tabular data.

## Before You Start

1. **Check authentication** by calling `mcp__renamed-to__status`. If the user is not authenticated, tell them to run `renamed auth login` or configure the MCP server with a valid token.
2. **Resolve the file path** from `$ARGUMENTS`. The first argument is typically the file, and any remaining text describes what to extract.

## Supported File Types

PDF, JPG, JPEG, PNG, TIFF.

## Schema Definition

The extraction API accepts a schema that defines what data to pull from the document. A schema has two parts:

### Fields (key-value pairs)
Individual pieces of data to extract:
```json
{
  "fields": [
    { "name": "invoice_number", "type": "string" },
    { "name": "total_amount", "type": "currency" },
    { "name": "invoice_date", "type": "date" },
    { "name": "vendor_name", "type": "string" }
  ]
}
```

### Tables (rows of data)
Tabular data like line items:
```json
{
  "tables": [
    {
      "name": "line_items",
      "columns": [
        { "name": "description", "type": "string" },
        { "name": "quantity", "type": "number" },
        { "name": "unit_price", "type": "currency" },
        { "name": "total", "type": "currency" }
      ]
    }
  ]
}
```

Available types: `string`, `number`, `date`, `currency`, `boolean`.

Each field or column can include an `instruction` property for extraction hints (e.g., `"instruction": "Look in the top-right corner"`).

## Workflow

### When the user describes what they want

1. If the user says something like "extract invoice details from this PDF", build the schema for them based on their description. Ask clarifying questions if needed ("Do you also want line items, or just the header fields?").
2. Call `mcp__renamed-to__extract` with the file and schema.
3. Display the results in a clear table format.
4. Offer to save as JSON or CSV.

### When the user provides a schema

1. If the user provides a schema (JSON or describes fields explicitly), validate it and pass it directly.
2. Call `mcp__renamed-to__extract` with the file and schema.
3. Display results and offer to save.

### Batch Extraction

1. Use `Glob` to find all matching files.
2. Extract from each file using the same schema.
3. Present results in a combined table or as a list of JSON objects.
4. Offer to save as a single JSON array or CSV file.

## Displaying Results

### Single file — show a formatted table:
```
Field            | Value                  | Confidence
-----------------+------------------------+-----------
Invoice Number   | INV-2024-4821          | 0.98
Total Amount     | $1,250.00              | 0.95
Invoice Date     | 2024-03-15             | 0.97
Vendor Name      | Acme Corporation       | 0.99
```

### Tables — show as formatted rows:
```
Line Items:
# | Description          | Qty | Unit Price | Total
1 | Widget Pro           |  10 |    $100.00 | $1,000.00
2 | Widget Pro Shipping  |   1 |    $250.00 |   $250.00
```

### Low confidence values
Flag any values with confidence below 0.8 so the user can verify them manually.

## Saving Results

When the user wants to save:
- **JSON**: Use `Write` to save the structured data as a JSON file.
- **CSV**: Convert the extracted fields/tables to CSV format and use `Write` to save.
- Default filename suggestion: `{original_name}_extracted.json` or `{original_name}_extracted.csv`.

## Error Handling

- **No data extracted**: The document may not contain the requested information. Suggest broadening the schema or checking that the file is readable (not a scanned image with poor quality).
- **Schema errors**: Validate field types before calling the API. Suggest corrections for typos in type names.
- **Authentication error**: Direct the user to `renamed auth login`.
- **File too large**: The service has a 25MB file size limit. Suggest compressing or splitting large files.

## Tips

- When extracting from invoices, receipts, or contracts, offer common schemas proactively rather than asking the user to define every field.
- For batch extraction, confirm the schema works well on a single file before processing the entire batch.
- Remind users that extraction accuracy depends on document quality — clean scans work better than photos taken at an angle.
