---
name: web-pdf-reader
description: Read PDF content from web URLs that WebFetch cannot handle. Use when a user provides a URL ending in .pdf, or when WebFetch returns binary/garbled content from a PDF URL. Handles arxiv papers, academic PDFs, documentation PDFs, and any other PDF hosted on the web. Also triggers when the user asks to "read this paper", "summarize this PDF", or provides a link to a PDF document.
---

# Web PDF Reader

WebFetch converts HTML to markdown — it cannot parse binary PDF files. This skill works around that limitation by downloading the PDF locally and using the Read tool, which has native PDF support.

## Workflow

1. Download the PDF to `/tmp/` using curl:
   ```bash
   curl -sL -o /tmp/<descriptive-name>.pdf "<url>"
   ```
   - Use `-L` to follow redirects (common on arxiv, Google Drive, etc.)
   - Use `-s` for silent mode
   - Name the file descriptively (e.g., `memgpt-paper.pdf` not `file.pdf`)

2. Read the PDF using the Read tool:
   ```
   Read /tmp/<descriptive-name>.pdf
   ```
   - For large PDFs (>10 pages), use the `pages` parameter to read in chunks (e.g., `pages: "1-10"`)
   - Max 20 pages per Read call

3. Clean up after processing:
   ```bash
   rm /tmp/<descriptive-name>.pdf
   ```

## URL Patterns

Common PDF URL patterns to detect:
- `*.pdf` — direct PDF links
- `arxiv.org/pdf/*` — arxiv papers
- `*.pdf?*` — PDFs with query params
- Google Drive/Dropbox export links that resolve to PDF

## Tips

- For arxiv, prefer the `/pdf/` URL (full paper) over `/abs/` (just metadata) when the user wants the full content
- If curl fails (403/404), try adding a browser-like User-Agent: `curl -sL -H "User-Agent: Mozilla/5.0" -o /tmp/paper.pdf "<url>"`
- For very large PDFs, read the first few pages to get the structure, then target specific sections
