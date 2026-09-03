# Corpus Audit Challenge

## Background

You've joined the data engineering team supporting a legacy records system. Every record in that system carries exactly one ledger ID — a plain integer between **1 and 20,000 inclusive**. Every ID in that range is supposed to appear exactly once, somewhere, in the nightly export.

The export job that produces `data/corpus.csv` is old and slightly broken. It doesn't write one ID per line — instead it batches many IDs into a row, delimited by `/`, because commas were apparently needed for "real" CSV columns. It's also inconsistent about zero-padding: some IDs are written as bare integers, others are zero-padded to a wider fixed field width, left over from when the field was a fixed-width string in the old system. `42`, `042`, and `00042` are all the same ID, just written three different ways.

Because of a defect in the export job, the corpus doesn't match reality:

- **Some IDs never made it into the export at all.**
- **Some IDs were written more than once** (sometimes with different padding each time).

Your job is to audit the export: parse it, and report exactly which IDs are missing and which are duplicated.

## The data

`data/corpus.csv` has two columns:

| column | meaning |
|---|---|
| `batch_id` | Sequential batch number. No significance beyond ordering. |
| `payload` | A batch of ledger ID tokens joined with `/`. |

To recover the full list of raw ID tokens: concatenate every row's `payload` (row order doesn't matter) and split on `/`. Every token is a non-negative base-10 integer string, optionally zero-padded (e.g. `7`, `007`, and `0000007` all mean ID `7`). There is no other punctuation, whitespace, or non-numeric junk to handle — every token you get from splitting on `/` is a clean digit string.

Token order within and across batches is randomized and carries no meaning.

## What to build

A program (any language you like) that reads `data/corpus.csv` and produces a `report.json` in the repo root, shaped like this:

```json
{
  "total_tokens_parsed": 0,
  "unique_valid_ids": 0,
  "missing_ids": [],
  "duplicate_ids": [{ "id": 0, "count": 0 }]
}
```

- `total_tokens_parsed` — how many `/`-delimited tokens you parsed out of the whole file.
- `unique_valid_ids` — how many distinct IDs in `[1, 20000]` appeared at least once.
- `missing_ids` — every ID in `[1, 20000]` that never appears, sorted ascending.
- `duplicate_ids` — every ID that appears more than once, sorted ascending by `id`, with the total number of times it occurred (including the first occurrence).

`example/` has a tiny (`1`–`20`) worked version of the exact same problem — `example/corpus.csv` alongside the `example/report.json` it should produce — so you can check your parser's output format and logic before pointing it at the real data.

## Also write up (a few paragraphs, in `NOTES.md`)

1. **Your approach** — how you parse the corpus and compute the two lists.
2. **Complexity** — time and space complexity of your solution, in terms of the range size `N` (20,000 here) and the token count `T`.
3. **Scaling it up** — suppose the range were `1` to `20,000,000,000` instead of `20,000`, and the corpus was many gigabytes spread across thousands of batch files. Your current approach almost certainly doesn't fit in memory anymore. Sketch (no need to implement it) how you'd change your approach to handle that — streaming, external sort, bitsets, sampling, whatever you'd reach for, and why.

## Constraints & assumptions

- Any language/runtime is fine. Tell us how to run it — a section in your own notes, a `Makefile` target, a `package.json` script, whatever's natural for your language.
- No external service calls; this should run offline against the file in this repo.
- Don't hand-solve it against this specific file (e.g. by hardcoding IDs) — your code should work unchanged if we swapped in a different `data/corpus.csv` generated the same way.
- Budget roughly 2–3 hours. We're not expecting production polish, but we are expecting it to be correct, and for the write-up to reflect real thinking rather than padding.

## What we're evaluating

- **Correctness** — do `missing_ids` and `duplicate_ids` exactly match reality?
- **Code quality** — is it readable, reasonably structured, and tested?
- **Judgment under scale** — is the write-up's answer to "scaling it up" actually sound, or just buzzwords?
- **Communication** — could a teammate understand your solution and notes without you in the room?

## Submitting

Fork this repository, do your work in the fork, and open a pull request back here when you're done. If you'd rather not use GitHub for the submission, a zipped folder (excluding any dependency folders like `node_modules`) works too — just make sure it includes your code, `report.json`, and `NOTES.md`.
