# Project summary

**NoUploadAI** — a browser-native, private-by-architecture document assistant.

## In one paragraph

NoUploadAI turns a browser tab into a complete RAG workspace: it parses PDFs and text files,
chunks and embeds them locally, retrieves evidence with a hybrid BM25 + dense search, and produces
answers whose claims link back to the exact source passage. Optionally, a small local language
model can run on WebGPU or WASM instead of the built-in extractive composer. Documents, vectors,
chats and settings live in IndexedDB; a live network monitor, upload counter and lockdown switch
make the privacy behaviour visible rather than declarative. The build is a self-contained web app
with no server component.

## Problem

Typical "chat with your documents" products require the user to upload sensitive material to a
third party. That blocks adoption in legal, healthcare, research and enterprise contexts, and it
makes privacy a promise the user cannot verify. Even well-intentioned tools rarely expose what
they send, when, and to whom.

## Solution

Move every stage of the pipeline into the client:

```text
file → parse → chunk → embed → IndexedDB
                                  ↓
question → BM25 + cosine → reciprocal rank fusion → grounded answer → citations
                                  ↓
                     network proof + lockdown + receipts
```

The interface pairs that pipeline with a three-pane workspace (library, chat, inspector), a
seven-scene launch film rendered in Canvas, six themes, a command palette and installable PWA
behaviour. A separate public showcase repository carries documentation and real screen media;
the implementation stays private.

## Key technical decisions

| Decision | Rationale |
|---|---|
| React 19 + TypeScript + Vite 7 | Fast iteration, strict typing, modern tooling |
| Tailwind 4 with CSS-variable theming | Six themes and five accents without shipping six stylesheets |
| Zustand store | Single action surface for ingest, retrieval, jobs, dialogs and privacy state |
| Raw IndexedDB | Five purpose-built stores; no ORM dependency for the data volume involved |
| Hand-written BM25, tokenizer, stemmer, hashing embedder and extractive composer | A credible zero-download mode with no model CDN dependency |
| Transformers.js at runtime from a CDN | Keeps the single-file build small; models are cached by the browser after first use |
| Web Worker for ML | Keeps embedding and generation off the UI thread |
| Embeddings pinned to WASM/q8, WebGPU for LLM only | Predictable cross-device behaviour with an optional accelerator path |
| Reciprocal rank fusion | Combines semantic and keyword evidence without hand-tuned score normalization |
| `vite-plugin-singlefile` | One portable `index.html` that can be opened or hosted anywhere |
| Separate private/public repositories | Implementation is not exposed for reuse; recruiters still get a complete, visual story |

## What is implemented today

- PDF text extraction with page previews and an explicit "no selectable text" error (no OCR)
- Eleven-plus text-like extensions, HTML stripping and pseudo-pagination
- Sentence-aware chunking with configurable size/overlap and page-scoped chunks
- Three embedding choices: built-in hashing plus two small neural models, with automatic upgrade of hashed chunks
- Four optional local LLMs, WebGPU/WASM selection, streaming and interruption
- Hybrid dense + lexical retrieval with rank fusion, summary shortcuts and metadata shortcuts
- Cited, streamed answers; clickable source cards with scores and neighbouring chunks
- Network monitor, upload counter, request log, lockdown switch and per-answer receipts
- Chat export as Markdown, two-step data erasure, storage estimates
- Landing page with interactive answer explorer, launch film and capability sections
- PWA manifest, service worker, offline shell and install prompts
- Responsive layout, drawers, drag & drop, paste, keyboard shortcuts, themes and reduced motion

## Scope boundaries

- No OCR, no server sync, no account system, no import/backup UI.
- Neural modes require one-time internet access to a model CDN and Hugging Face.
- The privacy proof is in-page instrumentation, not an OS-level sandbox.
- Bundled sample "PDFs" are text fixtures without original bytes, so page previews are unavailable for them.
- Numeric metrics printed inside the landing showcase and the launch film are illustrative presentation copy.

## Stack

`React 19` · `TypeScript 5.9` · `Vite 7` · `Tailwind CSS 4` · `Zustand 5` · `Framer Motion 14` ·
`pdf.js 5` · `Transformers.js 4` (runtime CDN) · `WebGPU / WASM` · `IndexedDB` · `Web Workers` ·
`Canvas 2D` · `WebAudio` · `Service Workers` · `vite-plugin-singlefile`.

## Repository model

| Repo | Visibility | Role |
|---|---|---|
| `NoUploadAI-SRC` | Private | Full source, 21 incremental commits, `docs/` excluded via `.gitignore` and absent from all history |
| `NoUploadAI` | Public | This showcase: README, features, privacy model, media and standalone studio demos |

Details are in [REPO_SETUP.md](REPO_SETUP.md) and [SHOWCASE_NOTICE.md](SHOWCASE_NOTICE.md).

## Author

Avartan Athlay — <https://github.com/avartan007>
