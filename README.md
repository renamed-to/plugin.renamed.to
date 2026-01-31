# plugin.renamed.to

Claude Code plugin for [renamed.to](https://www.renamed.to) — AI-powered file organization. Rename files by content, extract structured data from documents, and split PDFs by topic.

## Prerequisites

- A [renamed.to](https://www.renamed.to/sign-up) account
- The `@renamed-to/mcp` server configured in your Claude Code MCP settings

## Installation

Add the plugin marketplace and install:

```bash
claude plugin add renamed-to/plugin.renamed.to
```

Or add the repository URL directly in your Claude Code plugin settings.

## Skills

| Skill | Command | Description |
|-------|---------|-------------|
| **rename** | `/rename [files]` | Rename files using AI content analysis. Supports batch operations, custom naming strategies, and dry-run previews. |
| **extract** | `/extract [file] [schema]` | Extract structured data from PDFs, images, and scans into JSON or CSV. Define schemas or let the AI suggest one. |
| **pdf-split** | `/pdf-split [file]` | Split PDFs by content (AI), bookmarks, or fixed page counts. Ideal for scanned document batches. |
| **organize** | `/organize [directory]` | Full directory organization workflow — split multi-document PDFs, rename everything by content, and optionally extract data. |

## How It Works

The plugin teaches Claude how to use the renamed.to MCP server tools effectively. Each skill provides:

- Step-by-step workflows for common file organization tasks
- Previews and confirmations before any files are modified
- Error handling with actionable suggestions
- Batch processing with progress reporting

## Links

- [renamed.to](https://www.renamed.to) — the file organization service
- [@renamed-to/mcp](https://github.com/renamed-to/mcp.renamed.to) — the MCP server
- [@renamed-to/cli](https://github.com/renamed-to/renamed.to-cli) — the CLI

## License

MIT - see [LICENSE](LICENSE) for details.
