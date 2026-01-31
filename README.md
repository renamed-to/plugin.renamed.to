# plugin.renamed.to

Claude Code plugin for [renamed.to](https://www.renamed.to) — AI-powered file organization. Rename files by content, extract structured data from documents, and split PDFs by topic.

## Prerequisites

- A [renamed.to](https://www.renamed.to/sign-up) account
- The `renamed` CLI installed: `npm install -g @renamed-to/cli` or `brew install renamed-to/cli/renamed`
- Authenticated: `renamed auth login`

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
| **extract** | `/extract [file] [schema]` | Extract structured data from PDFs, images, and scans into JSON or CSV. Define schemas or let the AI auto-discover fields. |
| **pdf-split** | `/pdf-split [file]` | Split PDFs by content (AI), bookmarks, or fixed page counts. Ideal for scanned document batches. |
| **organize** | `/organize [directory]` | Full directory organization workflow — split multi-document PDFs, rename everything by content, and optionally extract data. |

## How It Works

The plugin teaches Claude how to use the `renamed` CLI effectively. Each skill provides:

- Step-by-step workflows with the correct CLI flags and options
- Previews and confirmations before any files are modified
- Error handling with actionable suggestions
- Batch processing with progress reporting
- Chaining between skills (split → rename → extract)

Skills invoke the CLI via `Bash` — no MCP server required. Just install the CLI and authenticate.

## Links

- [renamed.to](https://www.renamed.to) — the file organization service
- [@renamed-to/cli](https://github.com/renamed-to/cli.renamed.to) — the CLI tool
- [@renamed-to/mcp](https://github.com/renamed-to/mcp.renamed.to) — MCP server (for Cursor, Claude Desktop, etc.)
- [@renamed-to/sdk](https://github.com/renamed-to/renamed-sdk) — multi-language SDKs

## License

MIT - see [LICENSE](LICENSE) for details.
