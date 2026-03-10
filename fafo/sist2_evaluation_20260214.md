# 🔬 FAFO: sist2 as Content Search Layer
### Date: 2026-02-14

---

## What Is It?

**sist2** (Simple Incremental Search Tool) — a C binary that crawls filesystems, extracts text from 30+ file formats, and indexes into either **Elasticsearch** or **SQLite FTS5**.

## The Two Deployment Paths

### Path A: SQLite Backend (Lightweight) ⭐ RECOMMENDED

| Aspect | Detail |
|--------|--------|
| **Memory** | ~20MB |
| **Setup** | Zero dependencies — single binary + SQLite file |
| **Query syntax** | SQLite FTS5 (same as our keyword_index!) |
| **Speed** | Fast for medium indices (our ~25K files = medium) |
| **Machine-gun safe?** | ✅ SQLite handles concurrent reads natively |
| **Fuzzy search** | ❌ No |
| **Embeddings** | ✅ Yes (O(n) brute-force, fine for our scale) |

### Path B: Elasticsearch Backend (Heavy)

| Aspect | Detail |
|--------|--------|
| **Memory** | >500MB (Java) |
| **Setup** | Need ES 7.17+ running separately |
| **Query syntax** | ES query_string (most powerful) |
| **Speed** | Blazing at any scale |
| **Machine-gun safe?** | ✅✅✅ Born for it |
| **Fuzzy search** | ✅ |
| **Embeddings** | ✅ KNN search O(log n) |

## Format Coverage (Key Ones for Us)

| Format | Text Extract | Notes |
|--------|:---:|-------|
| `.py`, `.jl`, `.md`, `.txt` | ✅ | Via `text/plain` handler |
| `.html`, `.xml`, `.json` | ✅ | Parsed, tags stripped |
| `.pdf`, `.epub` | ✅ | MuPDF + optional OCR |
| `.docx`, `.xlsx`, `.pptx` | ✅ | Office format support |
| `.zip`, `.tar`, `.7z`, `.rar` | ✅ | Scans INSIDE archives recursively |
| Images | OCR | With `--ocr-images --ocr-lang eng` |

## ⚠️ Critical Constraint: Not Native Windows

sist2 is a **Linux binary**. Options for our Windows PC:

1. **Docker** (recommended) — `docker-compose up` and mount `C:\cee_v25_Pre_Final` as volume
2. **WSL2** — Run natively in WSL, mount Windows drives via `/mnt/c/`
3. **VPS deployment** — Run sist2 on VPS, sync files via rclone (already in use!)

## Workflow

```bash
# 1. Scan filesystem → creates .sist2 index file
sist2 scan --threads 4 --content-size 32768 /mnt/c/cee_v25_Pre_Final -o ./cee.sist2

# 2. Index into SQLite (lightweight) or Elasticsearch
sist2 sqlite-index ./cee.sist2 --search-index ./search.sist2

# 3. Start web server (has REST API)
sist2 web --search-index ./search.sist2 --bind 0.0.0.0:4090 ./cee.sist2

# 4. Incremental re-scan (only new/modified files)
sist2 scan --incremental /mnt/c/cee_v25_Pre_Final -o ./cee.sist2
```

## vs AnyTXT

| | AnyTXT | sist2 (SQLite) | sist2 (ES) |
|---|---|---|---|
| Machine-gun queries | 💀 | ✅ | ✅✅✅ |
| Setup complexity | Install .exe | Docker/WSL | Docker + ES |
| Native Windows | ✅ | ❌ | ❌ |
| Format coverage | Good | Excellent | Excellent |
| Incremental scan | Auto | `--incremental` flag | Same |
| API for AI | JSON-RPC (beta) | REST via web server | ES REST API |

## My Recommendation

> **Start with sist2 + SQLite on Docker.** It's the lightest footprint, handles concurrent queries, and we already have Docker Desktop. If we outgrow SQLite, we swap to ES backend without re-scanning.
>
> Keep AnyTXT for human GUI searches. Use sist2 for the AI machine-gun layer.

## Open Questions

1. Docker vs WSL2? Docker is simpler but WSL2 gives native filesystem access (faster I/O)
2. Run locally or on VPS? Local = faster scan, VPS = always-on with rclone mirror
3. How to handle the `netsh` port forward for the web API?
