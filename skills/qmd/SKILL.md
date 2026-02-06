---
name: qmd
description: Search and retrieve documents from indexed markdown collections using QMD (Query Markup Documents). Use when the user asks to search their notes, documentation, memory logs, or any indexed markdown files. Supports keyword search (BM25), semantic search (vector), and hybrid search with reranking. Use for queries like "search my notes for X", "find documents about Y", "what did I write about Z", or "retrieve information from my knowledge base".
---

# QMD - Query Markup Documents

Local search engine for markdown notes, documentation, and knowledge bases. All searches run locally without external API calls.

## Quick Reference

### Search Commands

```bash
# Keyword search (BM25) - Fast, no vectors needed
qmd search "query text" -n 5

# Semantic search (requires embeddings)
qmd vsearch "query text" -n 5

# Hybrid search (best quality, requires embeddings)
qmd query "query text" -n 5
```

### Common Options

```bash
-n <num>              # Number of results (default: 5)
-c <collection>       # Search specific collection only
--min-score <num>     # Minimum score threshold (0.0-1.0)
--full               # Show full document content
--json               # JSON output
--files              # Output: docid,score,filepath,context
```

## Available Collections

Current indexed collections:
- **workspace** - OpenClaw configuration, skills, and documentation (9 files)
- **daily-logs** - Memory logs and daily notes (memory/ folder)

## Common Workflows

### 1. Search Across All Collections

```bash
qmd search "model provider" -n 5
```

### 2. Search Specific Collection

```bash
qmd search "heartbeat" -c workspace
qmd search "meeting notes" -c daily-logs
```

### 3. Get Document Content

```bash
# By filename
qmd get "agents.md"

# By docid (from search results)
qmd get "#abc123"

# Specific lines
qmd get "agents.md:50" -l 20    # From line 50, show 20 lines
```

### 4. List Collection Files

```bash
qmd ls workspace
qmd ls daily-logs
```

### 5. Search with Filters

```bash
# High-quality results only
qmd search "authentication" --min-score 0.5 -n 10

# All matches above threshold
qmd search "API" --all --min-score 0.3

# Full document content
qmd search "config" --full
```

### 6. Different Output Formats

```bash
# JSON (for programmatic use)
qmd search "query" --json

# File list with scores
qmd search "query" --files

# Markdown format
qmd search "query" --md
```

## Implementation Notes

### Windows Path Configuration

QMD requires Git Bash on Windows. Always set PATH before running qmd commands:

```powershell
$env:Path = "C:\Program Files\Git\bin;" + [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")
qmd search "query"
```

### QMD Binary Location

- Windows: `C:\Users\Administrator\.bun\bin\qmd.exe`
- Installed via: `bun install -g https://github.com/tobi/qmd`

### Index Location

- Database: `C:/Users/Administrator/.cache/qmd/index.sqlite`
- Models cache: `C:/Users/Administrator/.cache/qmd/models/`

### Vector Embeddings Status

Current status: **Embeddings not generated** (keyword search only)

To enable semantic/hybrid search:
```bash
qmd embed
```

Note: Embedding generation may take several minutes and requires:
- embeddinggemma-300M (~300MB)
- qwen3-reranker (~640MB)
- qmd-query-expansion (~1.1GB)

## Collection Management

### Add New Collection

```bash
qmd collection add <path> --name <name> --mask "**/*.md"
```

### Update Index

```bash
qmd update              # Re-index all collections
qmd update --pull       # Git pull first, then re-index
```

### Add Context

```bash
qmd context add qmd://<collection>/ "Description of this collection"
```

### Check Status

```bash
qmd status
```

## Score Interpretation

- **0.8 - 1.0**: Highly relevant
- **0.5 - 0.8**: Moderately relevant
- **0.2 - 0.5**: Somewhat relevant
- **0.0 - 0.2**: Low relevance

## Troubleshooting

### Command Hangs or Times Out

QMD commands may be slow on first run (model loading). Use reasonable timeouts (15-30 seconds for search, 5+ minutes for embed).

### No Results Found

- Check collection exists: `qmd status`
- Verify files are indexed: `qmd ls <collection>`
- Try broader search terms
- Lower `--min-score` threshold

### Embeddings Failed

Vector embedding generation may fail on Windows. Keyword search (BM25) works without embeddings and is often sufficient for most use cases.

## Examples

### Search for Configuration Info

```bash
qmd search "model configuration" -c workspace -n 3
```

### Find Recent Memory Logs

```bash
qmd search "meeting" -c daily-logs --min-score 0.3
```

### Get Specific Document

```bash
qmd get "agents.md" --full
```

### Search with JSON Output

```bash
qmd search "API" --json -n 10
```

## Agent Usage Pattern

When user asks to search their notes/docs:

1. Determine search query from user's request
2. Choose collection if specified (workspace, daily-logs, or omit for all)
3. Run `qmd search` with appropriate options
4. Parse and present results to user
5. If user wants full content, use `qmd get` with docid or filename

Example:
```
User: "搜索关于 model provider 的内容"
→ Run: qmd search "model provider" -n 5
→ Present: Top results with titles, scores, and snippets
→ If needed: qmd get <docid> for full content
```
