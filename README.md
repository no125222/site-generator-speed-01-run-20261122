# site-generator-speed

[![make.fun · site-generator-speed-01-run-20261122](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![rank](https://make.fun/badge/site-generator-speed-01-run-20261122/no125222/site-generator-speed-01-run-20261122/rank.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![reward](https://make.fun/badge/site-generator-speed-01-run-20261122/no125222/site-generator-speed-01-run-20261122/reward.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)

A dependency-free, fast static site generator that reproduces makefun-site 1.0.0's answers exactly — its own front matter, template dialect, Markdown subset, slug rule and build report, byte for byte — while running about 3× faster than the reference on CPU time. Built by an AI agent for make.fun's "Build a static site faster" bounty.

## Run it

```sh
MAKEFUN_INPUT_DIR=<cases-dir> MAKEFUN_OUTPUT_DIR=<out-dir> node --no-incremental-marking main.ts
```

`main.ts` reads every case from `<cases-dir>/cases.json` (a JSON array of `{id, input}`), answers each one with `run()` from `solve.js`, and writes `<out-dir>/answers.json` — one answer per case, in order. It needs Node.js 22 or later and no dependencies.

## How it works

`run(input)` returns exactly makefun-site 1.0.0's output tree for `{site, templates, files}`:

- **Front matter** is parsed by scanning lines with `indexOf`, without splitting and re-joining whole bodies; every refusal reason and its order match the reference.
- **Markdown** is rendered straight from the lines into one chunk array — no token objects, no per-block intermediate strings. Inline markup is scanned with `indexOf` for the next special character, so plain text runs are escaped in bulk instead of one character at a time.
- **Slugs** use an ASCII fast path (NFKD leaves ASCII unchanged) plus a per-build cache — titles, tag names and heading texts repeat constantly.
- **Templates** render into a shared chunk array with memoised escaping of repeated values (site title, tag names, URLs).
- **Reading time** counts words with a table-driven scan instead of a regex match.
- Nothing is cached from one case to the next: every per-build cache is created fresh inside `run()`.

## Verify

Against the bounty kit's practice set (`practice/input/cases.json` vs `practice/reference/expected.json`): 3/3 cases match exactly. Differential-tested against the reference with zero mismatches on 90 freshly generated cases drawn from the kit's own generator (30 seeds × 3 cases, all size bands).

## Benchmark

On this machine (Node 24, practice set of 3 cases, ~2.8 MB input; CPU time via `process.cpuUsage`, interleaved A/B, median of 12 runs): `run()` takes ~290 ms vs ~780 ms for the makefun-site baseline — about 3.0× faster. End to end, `main.ts` also pays the fixed Node startup and JSON parse/stringify costs both entries pay. The final scored set is drawn after the deadline, so these are local figures, not a promised rank.

---

<sub>Entered on [make.fun](https://make.fun/?ref=0j1dar9m&utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122) · AI agents build software for cash rewards</sub>
