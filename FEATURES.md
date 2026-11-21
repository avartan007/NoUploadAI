# Features

A functional map of NoUploadAI, written against the shipping implementation. File names refer to
the private source repository; they are included so reviewers can map claims to code.

## 1. Document ingestion

| Capability | Implementation |
|---|---|
| PDF parsing | `pdfjs-dist` 5.6 with an inlined worker; per-page `getTextContent()` and a y-coordinate heuristic that restores paragraph breaks (`lib/pdf.ts`) |
| Text formats | `File.text()` for TXT, MD, CSV, TSV, JSON, LOG, HTML, XML, YAML, TEX and RST; HTML is stripped with `DOMParser` (`lib/ingest.ts`, `lib/files.ts`) |
| Flat-file pagination | Plain text is split into ~2,800-character pseudo-pages so citations still point to a location (`lib/chunk.ts`) |
| Page previews | Original PDF bytes are kept as a Blob; `renderPdfPage()` draws a chosen page to canvas at up to 2× DPR in the Inspector |
| Scanned PDFs | Detected and rejected with an explicit message: selectable text is required, OCR is not implemented |
| Progress | Serial job queue with `reading → parsing → chunking → model → embedding → saving → done` phases and a live progress ring |
| Input methods | File picker, full-window drag & drop with a full-screen overlay, and clipboard paste |

Accepted extensions are broader than the UI's "PDF · TXT · Markdown · CSV · JSON · HTML" shorthand.

## 2. Chunking and indexing

- Sentence-aware chunker: normalize whitespace → split on blank lines → split on sentence boundaries → hard-split over-long sentences → greedily pack to 900 characters with a 140-character, word-aligned overlap (`lib/chunk.ts`, `lib/types.ts`).
- Chunking is per page, so **a chunk never crosses a page boundary**; citations stay precise.
- Both chunk size and overlap are adjustable in Settings (400–1600 and 0–400 respectively). Top-K is adjustable from 2 to 10.
- IndexedDB keeps five stores: `docs`, `chunks` (indexed by document), `blobs` (original PDF bytes), `chats` and `kv` (settings). A mirrored in-memory chunk store keeps search fast (`lib/db.ts`).
- On load, the app can resume a cached model and re-embed chunks that were indexed with the fallback hash embedder, in batches of 24. It never downgrades better embeddings.

## 3. Embeddings and generation

### Embedders

| Option | Size | Notes |
|---|---|---|
| Instant hashing | 0 MB | Built-in 384-dim FNV feature hashing. No network request at all |
| `Xenova/all-MiniLM-L6-v2` | ~23 MB | Default neural embedder, 384-dim |
| `Xenova/bge-small-en-v1.5` | ~34 MB | 384-dim, uses a query-prefix instruction |

Neural embeddings run in a Blob-backed Web Worker through Transformers.js 4.3.2 loaded from jsDelivr. Model weights come from Hugging Face and are cached by the browser. Embedding inference is pinned to **WASM/q8**; WebGPU is reserved for the optional LLM. If the neural model cannot load, the app falls back to hashing and tells the user.

### Generators

- **Instant · Extractive (default, 0 MB)** — a hand-written composer scores candidate sentences by IDF-weighted query overlap, position and source rank, deduplicates them and emits up to four cited bullets. The answer is streamed in small increments for a responsive feel (`lib/lite.ts`).
- **Local LLM (optional)** — `SmolLM2-360M-Instruct` (~230 MB), `Qwen2.5-0.5B-Instruct` (~400 MB), `Llama-3.2-1B-Instruct-q4f16` (~950 MB) or `Qwen2.5-1.5B-Instruct` (~1.2 GB). The worker probes WebGPU (including `shader-f16` support) before falling back to WASM q4/q8, streams with `TextStreamer`, supports interruption, and reports tokens/sec. If loading fails, generation falls back to extractive with a warning.
- Prompts instruct the model to answer **only** from the numbered context passages and cite them as `[1]`, `[2][3]`. The last four non-error messages are included as conversation context.

### Shortcuts

- Metadata questions ("how many pages?", "is it a markdown file?") are answered from document metadata with no embedding and zero network requests.
- Summary-style questions sample evenly spaced chunks from each scoped document instead of running pairwise retrieval.

## 4. Hybrid retrieval

`lib/search.ts` implements dense + lexical retrieval with reciprocal rank fusion:

1. A BM25 index (k1 = 1.4, b = 0.75) is cached per scope and chunk version, using a hand-written tokenizer and stemmer.
2. Dense scores are cosine similarities between the query vector and stored vectors that share the same embedding model.
3. Dense rank is weighted 1.15 for neural embeddings and 0.7 for hash embeddings; lexical rank is weighted 1.0; each contributes `1 / (60 + rank)`.
4. Final score blends the fused dense score (55%) with a normalized lexical score (45%). Low-dense, zero-lexical tail results are pruned after the first two hits.
5. The top-K results (default 5) become the numbered sources attached to the answer.

## 5. Citations and the Inspector

- Each source chip shows a number, document name and page. Clicking one opens the Sources tab.
- The Sources tab shows the exact passage, dense and lexical match scores, adjacent chunks and — for uploaded PDFs whose bytes were kept — a rendered page preview.
- The **Proof** tab shows the large "documents uploaded: 0" counter, the full request log with host classification, and the lockdown toggle.
- The **Engine** tab shows WebGPU availability, device/adapter details, storage usage/quota and PWA install state.
- Assistant messages carry a receipt: engine, retrieval time, generation time, tokens, tokens/sec, device and the number of requests observed during that answer.

> Bundled sample documents are text-based even when named `.pdf`, so they do not have original bytes and cannot show a page preview. Uploaded PDFs can.

## 6. Privacy tooling

- The network monitor starts before React renders and patches `fetch`, `XMLHttpRequest.open/send` and `navigator.sendBeacon` on the main thread, plus `fetch` inside the ML worker. A `PerformanceObserver` records other resource loads (`lib/netproof.ts`).
- Hosts are classified as **app**, **model** (Hugging Face), **runtime** (jsDelivr) or **other**. Only requests with a body or a non-GET/HEAD/OPTIONS method that were not blocked increment the upload counter.
- **Lockdown mode** makes the patched APIs reject or throw and tells the worker to refuse model downloads. It is available from the Proof tab, the command palette and the privacy CTA.
- The top bar always shows the live upload count. The Settings dialog offers a two-step "erase all data" that clears IndexedDB and Cache API entries.
- See [PRIVACY.md](PRIVACY.md) for the honest scope and limitations of this instrumentation.

## 7. Interface and interaction

- **Two views** — a marketing/explainer landing page with an interactive answer explorer, and the three-pane workspace (library / chat / inspector).
- **Six themes plus Auto** — Ink (aurora), Slate, Mono, Clay, Forest and Paper — with five accent colours and a reduce-motion setting.
- **Command palette** — `⌘/Ctrl + K` groups fuzzy-filtered commands for new chat, file add, samples, export, film, proof, lockdown, themes, per-document scope, Model Center, Settings and erase data.
- **Keyboard** — `/` focuses the question box; `Enter` sends and `Shift+Enter` inserts a newline; the film player adds space/K, arrows, M and F.
- **Responsive** — the library becomes a drawer below `lg`, the inspector below `xl`; safe-area padding, backdrop blur and animated springs throughout.
- **Accessibility** — skip link, labelled controls, dialog/tab semantics, focus-visible outlines, live-region toasts and focus handoff between views.
- **Chat export** — the active conversation, including its sources, downloads as Markdown.

## 8. PWA and offline behaviour

- Manifest with `standalone` display and an SVG app icon; service worker registration is guarded by HTTPS/localhost checks.
- Separate caches: `nouploadai-shell-v1` for the app shell and `nouploadai-runtime-v1` for CDN/Hugging Face runtime and model GETs.
- Engine panel reports connection, service-worker and install state. The service worker is optional: the app still runs as a normal web page, and a single-file `dist/index.html` can be opened directly (with reduced offline-model support).

## 9. Launch film

A 45-second, 1280×720 film rendered entirely from Canvas 2D primitives, with seven scenes:

1. The problem — cloud upload animation
2. The idea — crossed-out cloud, a browser tab
3. Meet NoUploadAI — kinetic wordmark and tech pills
4. Drop & embed — parse/chunk/embed and vector clusters
5. Ask & cite — typed question, streamed answer, source card
6. Proof — cleared network tab, a large zero, airplane badge
7. Stack — technology tiles and closing logo

The score is synthesized with WebAudio oscillators and noise buffers — no audio files. The player
offers an inline looping preview, a full-screen modal, a clickable scene scrubber, keyboard
controls and a save-to-video action using `canvas.captureStream()` + `MediaRecorder` (MP4/H.264
preferred, WebM fallback). Numeric labels in the film are presentation copy, not live telemetry.

## 10. Companion studio demos (this repository only)

`studio/` in the public showcase repository contains three standalone, no-build HTML pages that
demonstrate the same local-only approach: **Compress** (canvas resize/re-encode with byte-size
comparison), **Scrub** (real EXIF parsing and pixel-only rebuild) and **Optimize SVG** (comment,
metadata and whitespace cleanup with live preview). They are demonstrators of a design philosophy,
not features of the NoUploadAI application build.

---

## Known limitations

- No OCR for scanned PDFs.
- No backup/import UI; export is Markdown-only.
- Neural mode needs one-time internet access for the runtime and model weights; CDN operators see IP and requested URLs.
- The privacy proof is in-page instrumentation, not a sandbox. It does not intercept WebSockets, EventSource, service-worker-internal fetches or direct resource loads.
- Erasing data clears storage and caches, but models already resident in a worker remain until reload.
- The landing page's pipeline card says a 120-character overlap; the actual default is 140.
