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
qmd search "project timeline"

# Semantic search (requires embeddings)
qmd vsearch "how to deploy"

# Hybrid search (best quality, requires embeddings)
qmd query "quarterly planning process"
```

### Common Options

```bash
-n <num>              # Number of results (default: 5)
-c <collection>       # Search specific collection only
--all                 # Return all matches (use with --min-score)
--min-score <num>     # Minimum score threshold (0.0-1.0)
--full               # Show full document content
--json               # JSON output
--files              # Output: docid,score,filepath,context
--md                 # Markdown output
--csv                # CSV output
--xml                # XML output
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
qmd get "meetings/2024-01-15.md"

# By docid (from search results)
qmd get "#abc123"

# Specific lines
qmd get "agents.md:50" -l 20    # From line 50, show 20 lines

# Get multiple documents by glob pattern
qmd multi-get "journals/2025-05*.md"

# Get multiple documents by comma-separated list
qmd multi-get "doc1.md, doc2.md, #abc123"

# Limit multi-get to files under 20KB
qmd multi-get "docs/*.md" --max-bytes 20480
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

# Export all matches for an agent
qmd search "API" --all --files --min-score 0.3

# Full document content
qmd search "config" --full

# Search within specific collection
qmd search "API" -c notes
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

## Installation

### Install QMD Globally

```bash
bun install -g https://github.com/tobi/qmd
```

### Setup Workflow

1. **Install QMD**
   ```bash
   bun install -g https://github.com/tobi/qmd
   ```

2. **Create collections for your content**
   ```bash
   qmd collection add ~/notes --name notes
   qmd collection add ~/Documents/meetings --name meetings
   qmd collection add ~/work/docs --name docs
   ```

3. **Add context to help with search results**
   ```bash
   qmd context add qmd://notes/ "Personal notes and ideas"
   qmd context add qmd://meetings/ "Meeting transcripts and notes"
   qmd context add qmd://docs/ "Work documentation"
   ```

4. **Generate embeddings for semantic search**
   ```bash
   qmd embed
   ```

5. **Start searching**
   ```bash
   qmd search "project timeline"        # Fast keyword search
   qmd vsearch "how to deploy"          # Semantic search
   qmd query "quarterly planning"       # Hybrid + reranking
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
# Add collection from specific path
qmd collection add ~/notes --name notes
qmd collection add ~/Documents/meetings --name meetings
qmd collection add ~/work/docs --name docs

# Add with custom glob mask
qmd collection add <path> --name <name> --mask "**/*.md"
```

### Add Context to Collections

Context helps QMD understand what each collection contains:

```bash
qmd context add qmd://notes/ "Personal notes and ideas"
qmd context add qmd://meetings/ "Meeting transcripts and notes"
qmd context add qmd://docs/ "Work documentation"

# Add context to subdirectories
qmd context add qmd://docs/api/ "API documentation"
```

### Update Index

```bash
qmd update              # Re-index all collections
qmd update --pull       # Git pull first, then re-index
```

### Check Status

```bash
qmd status
```

### List Collections

```bash
qmd collection list
```

### Remove Collection

```bash
qmd collection remove <name>
```

### Rename Collection

```bash
qmd collection rename <old-name> <new-name>
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
qmd get "meetings/2024-01-15.md"
qmd get "#abc123"
```

### Get Multiple Documents

```bash
qmd multi-get "journals/2025-05*.md"
qmd multi-get "docs/*.md" --max-bytes 20480
```

### Search with JSON Output

```bash
qmd search "API" --json -n 10
```

### Export All Matches for Agent Processing

```bash
qmd search "API" --all --files --min-score 0.3
```

### Search Across Everything

```bash
qmd search "project timeline"
qmd vsearch "how to deploy"
qmd query "quarterly planning process"
```

## Agent Usage Pattern

When user asks to search their notes/docs:

1. Determine search query from user's request
2. Choose search type:
   - `qmd search` - Fast keyword search (always available)
   - `qmd vsearch` - Semantic search (requires embeddings)
   - `qmd query` - Hybrid search with reranking (best quality, requires embeddings)
3. Choose collection if specified (workspace, daily-logs, or omit for all)
4. Run search with appropriate options
5. Parse and present results to user
6. If user wants full content, use `qmd get` with docid or filename
7. For multiple documents, use `qmd multi-get` with glob patterns

Example:
```
User: "搜索关于 model provider 的内容"
→ Run: qmd search "model provider" -n 5
→ Present: Top results with titles, scores, and snippets
→ If needed: qmd get <docid> for full content
```

Example with multi-get:
```
User: "获取所有 2025 年 5 月的日志"
→ Run: qmd multi-get "journals/2025-05*.md"
→ Present: All matching documents
```

Example with export:
```
User: "导出所有关于 API 的文档"
→ Run: qmd search "API" --all --files --min-score 0.3
→ Present: File list with scores for further processing
```
