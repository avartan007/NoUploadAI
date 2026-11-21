# Showcase notice

## What this repository is

This is the **public showcase** for NoUploadAI. It contains project documentation, captured
product media, the brand mark and three standalone, runnable companion demos. It exists so that
recruiters, hiring managers and technical evaluators can understand the project without receiving
the implementation.

## What this repository is not

It is not the application source repository. It contains no TypeScript/React application code, no
build configuration and no implementation history. The complete implementation lives in the
private [`NoUploadAI-SRC`](https://github.com/avartan007/NoUploadAI-SRC) repository, which is
shared with hiring teams on request.

The only executable code here is the self-contained HTML/CSS/JavaScript under `studio/`: three
small demonstrators of the same local-only philosophy. They are not bundled into the NoUploadAI
application and should not be read as features of it.

## Accuracy of the material

- Screenshots, GIFs and the MP4 are real captures of the running application or of the bundled
  studio demos. See [MEDIA.md](MEDIA.md) for the source of each file.
- Feature descriptions in [FEATURES.md](FEATURES.md) and the privacy analysis in
  [PRIVACY.md](PRIVACY.md) are written against the implementation, including limitations.
- Hard-coded numbers that appear inside the launch film and the landing showcase (retrieval time,
  tokens/sec, embedding counters) are illustrative presentation copy in the product UI, not
  benchmark results measured during the captures.
- Where a common claim would overstate the code — for example around WebGPU usage, lockdown
  coverage or offline availability — the documentation states the precise behaviour instead.

## Intended use

You are welcome to view, link to and discuss this repository for evaluation purposes. Please do
not republish the documentation or media as your own project, and do not treat the studio demos as
a licensed component library. Source access requests should go through
<https://github.com/avartan007>.

## Author

Avartan Athlay — <https://github.com/avartan007>
