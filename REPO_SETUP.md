# Repository setup

This document explains the two-repository arrangement used for NoUploadAI and how to maintain it
without leaking implementation code into the public showcase.

## The two repositories

| | [`NoUploadAI-SRC`](https://github.com/avartan007/NoUploadAI-SRC) | [`NoUploadAI`](https://github.com/avartan007/NoUploadAI) |
|---|---|---|
| Visibility | Private | Public |
| Purpose | Complete application implementation | Recruiter-facing documentation and media |
| Contents | `src/`, `public/`, build config, lockfile | Markdown docs, PNG/GIF/MP4/SVG media, `studio/` demos |
| History | 21 incremental feature commits | Documentation commits only |
| Documentation | `docs/` is git-ignored and absent from every commit | This is the docs repository |

## Why they are separate

The implementation is deliberately kept out of the public repository so it cannot be copied into
someone else's project and presented as theirs. Recruiters still get a complete, visual and
technical picture from this repository: architecture, feature map, privacy analysis, real
screenshots, a recorded product tour, the launch film and runnable companion demos.

## Local layout

```text
build-nouploadai-local-rag/          →  git remote: NoUploadAI-SRC (private)
├── .gitignore                       # includes `docs/`
├── index.html
├── package.json / package-lock.json
├── src/                             # application source
├── public/                          # icon, manifest, service worker
├── dist/                            # generated, ignored
└── docs/                            # IGNORED by the parent repo
    ├── .git/                        # nested repository → NoUploadAI (public)
    ├── README.md
    ├── FEATURES.md
    ├── MEDIA.md
    ├── PRIVACY.md
    ├── PROJECT_SUMMARY.md
    ├── REPO_SETUP.md
    ├── SHOWCASE_NOTICE.md
    ├── TOPICS.txt
    ├── hero.png
    ├── landing-tour.gif
    ├── launch-film.gif / .mp4
    ├── logo.svg
    ├── studio-*.png / .gif
    └── studio/                      # runnable no-build demos
```

`docs/` is ignored from the very first commit of the private repository. It has never been staged
there, and no path under `docs/` appears anywhere in that repository's history.

## Verification

From the source checkout:

```bash
# docs/ must be ignored
git check-ignore -v docs/README.md

# must print nothing
git log --all --pretty=format: --name-only | grep '^docs/'

# must contain only application files
git ls-files
```

If any of those checks fail, stop and fix the ignore rule before pushing.

## Pushing the source repository

```bash
cd build-nouploadai-local-rag
git remote -v                 # git@github.com:avartan007/NoUploadAI-SRC.git
git add -A                    # docs/ is skipped automatically
git commit -m "feat: ..."
git push origin main
```

**Never** use `git add -f docs/...` in this repository. Do not remove the `docs/` line from
`.gitignore`, even temporarily.

## Pushing the showcase repository

```bash
cd build-nouploadai-local-rag/docs
git remote -v                 # git@github.com:avartan007/NoUploadAI.git
git add -A
git commit -m "docs: ..."
git push origin main
```

Because the docs repository lives inside an ignored folder of the source checkout, the two
histories cannot mix. `git status` in the parent never reports changes under `docs/`.

## Repository metadata

The public repository description and topics are set with the GitHub CLI:

```bash
gh repo edit avartan007/NoUploadAI \
  --description "Local-first browser RAG: chat with PDFs using hybrid search, cited answers and verifiable zero-upload privacy. Docs, demos and media." \
  --add-topic rag --add-topic local-first --add-topic privacy ...
```

The canonical topic list is kept in [TOPICS.txt](TOPICS.txt) so it can be reapplied consistently.
Topics fill the GitHub limit of 20; replace rather than accumulate if the list changes.

## Updating the media

1. Rebuild and serve the app (`npm run build`, `npm run preview`).
2. Serve the docs folder (`python3 -m http.server 4174 --directory docs`) for the studio demos.
3. Capture new frames with headless Chromium and encode GIF/MP4 with FFmpeg.
4. Replace the asset in `docs/`, update [MEDIA.md](MEDIA.md), then commit **in the docs repository**.

The application has no references to these media files, so this workflow never touches the source
history.

## Access requests

The private source repository can be shared with recruiters and hiring teams on request via
<https://github.com/avartan007>. Do not mirror, fork or copy its contents into the public
repository.
