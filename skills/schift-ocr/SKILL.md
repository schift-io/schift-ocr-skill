---
name: schift-ocr
description: Read PDFs and document images through Schift OCR before answering. Use whenever the user gives or points to a PDF, scan, photo of a document, screenshot of a form, receipt, invoice, contract, table, chart, handwritten note or exam paper (Korean or English), or asks for OCR, text extraction, 표 추출, 손글씨 인식, 스캔본/PDF 읽기, 문서 인식, 영수증·계약서·시험지 읽기. Covers which Schift OCR tool to call, private (ZDR) vs shared mode, batch for long documents, polling, tables and errors.
---

# Schift OCR

When a PDF or an image of a document arrives, read it with Schift OCR first. Do not transcribe the image yourself, and do not answer from your own reading of the pixels. Answer from the OCR result.

This applies to scans, phone photos of paper, forms, receipts, invoices, contracts, tables, charts, formulas, handwriting and exam papers. Skip OCR only for images that are not documents (a photo of a cat, a UI mockup with no text to extract) or when the user says not to use it.

## Tools (MCP server `schift-ocr`)

| Tool | Use |
|---|---|
| `ocr_document` | Read one PNG, JPEG or PDF. Exactly one of `image_url` (public https only) or `image_base64`. Optional: `task` (default `markdown`), `options`, `zdr`, `wait` (default true, waits up to 300 s), `batch`, `filename`. |
| `get_result(job_id)` | Fetch a job that was not finished when `ocr_document` returned. Your own jobs from the last 24 h only. |
| `list_recent_jobs(limit=10)` | Job metadata (no content). Use it to find a lost `job_id`. |
| `list_tasks()` | Task names and their required `options`. |
| `get_usage()` | Credit left, pages left per mode, prices, and `account_default_mode`. |

Limits per request: 10 MB, and a PDF may have at most 20 pages.

## How to send the file

1. The file is at a public https URL: `ocr_document(image_url=..., filename=...)`.
2. The file is on disk and you have a shell (Claude Code): send it with the REST API, so the bytes never pass through your context. Same key, same prices, same modes.

   ```bash
   curl -sS -X POST https://schift.io/v1/run \
     -H "Authorization: Bearer $SCHIFT_OCR_API_KEY" \
     -F "image=@/path/to/file.pdf" -F "task=markdown"
   # add -F "zdr=true" for private mode, -F "batch=true" for batch
   ```

   The response has `result` (first page), `pages` (every page, when more than one), `job_id` and `usage`. On `504 timeout`, or for `batch=true` (`202` with `poll`), fetch with `curl -sS https://schift.io/v1/jobs/JOB_ID -H "Authorization: Bearer $SCHIFT_OCR_API_KEY"` (or `get_result(job_id)`).
3. The file is small (a single image of a few hundred KB) and there is no shell: `ocr_document(image_base64=...)`. Do not base64 large files into a tool call.
4. The image exists only as a chat attachment, with no path or URL you can reach: you cannot forward its bytes. Say so and ask the user for a file path or a public link. Do not fall back to reading it yourself without telling the user.

Never print, echo or log the API key.

## Task

- `markdown` (default): whole page as Markdown, with tables and formulas. Use this unless the user asks for something specific.
- `structured`: typed blocks in reading order with bboxes (JSON). Use when the user needs positions or block types.
- `kie`: key fields from receipts, invoices, forms. Needs `options.keys` (list of field names) or `options.schema`.
- `formula`, `chart`, `words`, `layout`, `caption`, `region` (`options.region=[x_min,y_min,x_max,y_max]`), `classify` (`options.target`, `options.labels`).

Call `list_tasks` if unsure; it is the source of truth for names and required options.

## Mode: shared vs private (ZDR)

- 학습 허용 (shared, `zdr=false`): 10원/page, 50 free pages per day (KST). The page and result may be used to train the model.
- 비공개 · ZDR (`zdr=true`): 50원/page, needs paid credit. Never used for training; the result is deleted on the server after it is delivered.

Default: omit `zdr`, so the user's account setting applies (`get_usage` → `account_default_mode`).

Override to `zdr=true` when the document looks sensitive: ID cards, passports, resident registration numbers, contracts, medical records, bank or tax documents, HR files, anything the user calls confidential. Tell the user in one line that you used private mode and why, and what it costs. If ZDR fails with `[zdr_requires_paid]`, stop and ask the user: add credit, or explicitly allow shared mode. Never silently drop to shared mode for a sensitive document.

## Long documents and batch

- A PDF over 20 pages is refused (`too_many_pages`). Split it into parts of at most 20 pages (and at most 10 MB each), for example `qpdf in.pdf --split-pages=20 part-%d.pdf` or pypdf, and send each part.
- More than 20 pages in total: send every part with `batch=true`. Batch is 20% cheaper (shared 8원, ZDR 40원/page) and finishes within 48 h. It returns `job_id` and `deadline_at` at once.
- Keep a list of `part -> job_id`. Tell the user the job ids and the deadline. Do not busy-wait for hours; check with `get_result` a few times, then report status and let the user come back. If the user needs the text now, send without `batch` and accept the full price.
- Up to 20 pages: one normal call. A large PDF that may take over 300 s: `wait=false`, then poll.

## Polling `get_result`

- Pending jobs return `status`, `pages`, `pages_done` and a `next` hint. Wait about 10 to 30 s between calls for normal jobs, a few minutes for batch.
- `done` returns one text block per page (`<!-- page i/n -->`) plus a JSON summary with `charged`, `failed_pages` and `mode`.
- ZDR results can be fetched exactly once, and an unfetched ZDR result is deleted 10 min after the job finishes. Fetch promptly, and save or use the text the first time; a second call says it was already delivered.
- `resubmit: true` means the server restarted. Pages without a result were refunded; send the document again.
- A page that failed shows `[page failed: ...]` and was not charged. Say which pages failed.

## Presenting the result

- Quote the OCR text as returned. Do not "fix" names, numbers or amounts from your own reading of the image.
- Tables that come back as HTML (`<table>`, `rowspan`, `colspan`) stay HTML. Do not flatten them to Markdown pipe tables; merged cells would be lost. If the user wants CSV or Excel, convert from the HTML and say so.
- Keep LaTeX formulas as LaTeX.
- For multi-page documents keep page markers so the user can find things.
- After the job, mention pages and credits charged in one line (from the summary's `charged`).

## Errors

- HTTP 401 or the server does not connect: the key is missing or wrong. Tell the user to create a key at https://schift.io/dashboard and set `SCHIFT_OCR_API_KEY` (then restart the client). Keys start with `sk-ocr-`.
- `[quota_exceeded]`, HTTP 402, or `[cap]`: not enough credit, or the monthly cap of 20,000 pages. Show the message and link https://schift.io/dashboard (credit top-up) and https://schift.io/pricing. Do not retry.
- `[zdr_requires_paid]`: see Mode above.
- HTTP 429 / `rate_limited` / `too_many_concurrent_jobs`: the usage tier limit. Wait `retry_after_sec` seconds (REST response body) before retrying. If the reply has no `retry_after_sec` (the MCP tool currently returns only `request refused: rate_limited`), wait 60 s. Do not retry in a tight loop.
- `queue_full`, `batch_queue_full`, `not_started` (503): wait a minute and retry once or twice.
- `batch_unavailable`: send without `batch`, after telling the user it costs the normal price.
- `unsupported file type`: only PNG, JPEG and PDF. Convert (for example HEIC or TIFF to PNG) first.
- `image_url refused`: the URL is not public https or points to a private address. Download it and send the file instead.
- Anything else: show the error text to the user. Do not invent a result.
