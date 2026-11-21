# Privacy model

NoUploadAI is designed around a simple rule: **your documents are data, not traffic**. This page
describes what the code actually does, what can leave the device, and where the guarantees stop.
It intentionally avoids marketing shorthand like "nothing ever touches the network".

## What stays on the device

| Data | Where it lives | Notes |
|---|---|---|
| Uploaded PDFs and text files | IndexedDB (`docs`, `blobs`, `chunks` stores) | Original PDF bytes are retained only for page previews |
| Extracted text, chunk vectors and BM25 index | IndexedDB + in-memory mirror | Never serialized into a request |
| Questions, answers, citations and chat history | IndexedDB (`chats`) | Export is a local Markdown download |
| Settings, theme, selected models | IndexedDB (`kv`) | No preference sync |
| Model files, once downloaded | Browser Cache API + service-worker caches | Reused on later visits |

There is no backend endpoint, no account system, no analytics module, no telemetry code and no
advertising SDK in the application source. Document text and questions are never placed in URLs,
headers or request bodies by application code.

## Network activity that does happen

The pure local path — **Instant hashing** embedder plus **Instant · Extractive** generator — does
not require any network request beyond loading the app itself.

When neural features are selected, the browser performs GET requests for:

1. the Transformers.js ESM bundle from `cdn.jsdelivr.net`;
2. ONNX runtime binaries and model/tokenizer/weight files from Hugging Face hosts (`huggingface.co`, `hf.co`, CDN/LFS hosts);
3. same-origin GETs for the app shell, manifest, icon and service worker.

These are downloads of code and model weights, not uploads of your files. However, the CDN and
model hosts still see your IP address, request timing and which model files were requested.
"No document data leaves" is therefore not the same as "no network activity" or "anonymity".

## The Network Proof

The proof is an observer, not a claim. Before React renders, `main.tsx` starts
`startNetworkMonitor()`, which wraps:

- `window.fetch`, `XMLHttpRequest.open` / `send`, `navigator.sendBeacon` on the main thread;
- `fetch` inside the ML worker, which forwards entries back to the page;
- a `PerformanceObserver` for `"resource"` entries that the wrappers do not see.

Entries are classified by host:

| Class | Example | Meaning |
|---|---|---|
| `app` | same origin | App shell, manifest, service worker probe |
| `runtime` | `cdn.jsdelivr.net` | Transformers.js bundle |
| `model` | Hugging Face hosts | Weights, tokenizer, ONNX binaries |
| `other` | anything else | Surfaced explicitly rather than hidden |

The counter increments only for requests that (a) carry a request body or use a method other than
GET/HEAD/OPTIONS and (b) were not blocked. This is why the badge reads **0 uploads** while a
model download is in progress: a download is a GET and is not an upload.

Every assistant answer also records a receipt — engine, timings, device and the number of requests
observed while answering — so a reviewer can see whether a question triggered outbound traffic.

## Lockdown mode

Enabling lockdown sets a global flag:

- patched main-thread `fetch`, XHR and beacon calls reject/throw with a clear error;
- the worker receives the flag, blocks its own non-blob/data fetches and refuses to download the ML runtime;
- the mode is exposed in the Proof tab, the command palette and the privacy CTA, and is reflected in the top-bar badge.

### Scope of the protection

Lockdown is **in-page instrumentation**, not a sandbox. It patches selected JavaScript APIs in the
page and worker. It does not intercept WebSockets, `EventSource`, service-worker-internal fetches,
direct `<img>`/CSS resource loads, or a reference to `fetch` captured before the patch. Resource
entries from the observer are logged as GET with zero body bytes and therefore never count as
uploads. Treat lockdown as a strong guard against accidental application-level uploads, not as a
network firewall. For that, use the OS firewall or a browser extension.

## Storage and erasure

- Documents, chunks, chats and settings persist in IndexedDB. If storage is unavailable (some private-browsing modes), the app warns and continues in memory only.
- **Settings → Erase all data** clears all IndexedDB stores and deletes every Cache API cache for the origin. It asks for confirmation twice.
- Two consequences worth knowing: already-loaded models remain in worker memory until the page is reloaded, and clearing all Cache API caches also removes the service worker's app-shell cache, so offline support needs a fresh visit to repopulate it.

## Companion studio demos

The pages under `studio/` in this public repository read files with the File API, draw them to a
canvas and encode the result locally. They do not call `fetch` for user content (the only fetch is
for the bundled sample image) and contain no analytics. They are demonstrators, not part of the
NoUploadAI application.

## Claim vs implementation

| Common shorthand | Precise behaviour |
|---|---|
| "Embeddings run on WebGPU" | Embeddings are pinned to WASM/q8. WebGPU is attempted only for the optional LLM |
| "Works fully offline" | The hashing + extractive path does; neural modes need a one-time download and cached weights |
| "Lockdown blocks all network APIs" | It blocks patched fetch/XHR/beacon paths in page and worker only |
| "0 uploads proves nothing leaves" | It counts non-GET, body-bearing requests observed by those patches; GET downloads and IP-level metadata still exist |
| "Erase removes models immediately" | Storage and caches are cleared; worker-held models persist until reload |

## Reporting a concern

If you find a request or behaviour that contradicts this document, open an issue on the public
showcase repository or contact the author. Reproducibility matters more than a flattering claim.
