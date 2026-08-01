<div align="center">

# 🎓 Education Catalog Audit

**An AI agent that opens a real university course catalog, reads a department's subject listings, and reports units, prerequisites and terms offered — then films itself doing it.**

[![CI](https://github.com/coasty-ai/coasty-education-catalog-audit/actions/workflows/ci.yml/badge.svg)](https://github.com/coasty-ai/coasty-education-catalog-audit/actions/workflows/ci.yml)
[![Node](https://img.shields.io/badge/node-%E2%89%A520.11-339933?logo=node.js&logoColor=white)](https://nodejs.org)
[![Dependencies](https://img.shields.io/badge/dependencies-0-brightgreen)](package.json)
[![Runs offline](https://img.shields.io/badge/runs%20offline-%240.00-blue)](#try-it-in-30-seconds)
[![License](https://img.shields.io/badge/license-MIT-black)](LICENSE)

<img src="media/demo.gif" alt="The agent browsing the course catalog and extracting units and prerequisites" width="820">

<sub>Every frame above is a **real screenshot the model actually saw** — pulled from the run's own model-input frames, not a reconstruction.</sub>

</div>

---

## What this is

A complete, production-grade [Coasty](https://coasty.ai) computer-use automation for **course catalog auditing**. It gives an AI agent one goal in plain English, and the agent drives a real browser on a real cloud desktop to accomplish it — no selectors, no scraping rules, no DOM parsing to maintain.

Registrars, advising teams, transfer-credit evaluators and curriculum tools all need the same facts: what a department actually offers, how many units each subject carries, what it requires first, and which terms it runs in. Catalogs are the authoritative source for that, and almost none of them ship an API — they are HTML, they are re-authored every academic year, and each institution renders them differently. A scraper is a per-catalog selector set that breaks on the next redesign. An agent reads the page the way an advisor does, so the same prompt works across a re-skin and across institutions.

**Zero dependencies. Runs offline for $0 on a fresh clone. ~$0.70 to run for real.**

```
"Go to http://catalog.mit.edu and open the subject listing for Course 6,
 Electrical Engineering and Computer Science. From that listing, take the
 first three undergraduate subjects shown and record for each one: its
 subject number, its full title, its total units, its prerequisites exactly
 as written, and the terms in which it is offered. Then report those three
 subjects with all five values, and state how many of the three list at
 least one prerequisite."
```

That prompt *is* the automation. When the site redesigns, the prompt still works.

## Try it in 30 seconds

No API key. No account. No install. No spend.

```bash
git clone https://github.com/coasty-ai/coasty-education-catalog-audit
cd coasty-education-catalog-audit
npm start
```

That boots a bundled offline mock in-process and runs the whole agent loop against it. Then render the demo video from the run's own frames:

```bash
npm run demo     # needs ffmpeg; writes media/demo.mp4 + demo.gif + poster.jpg
```

Check your setup any time with `npm run doctor`.

## Run it for real

```bash
export COASTY_API_KEY=sk-coasty-test-...      # sandbox keys never bill
export COASTY_BASE_URL=https://coasty.ai/v1
export COASTY_ALLOW_LIVE=1                     # destination consent
npm start -- --live --confirm-cost-cents 120   # cost consent
```

Both consents are required and they are deliberately separate. A live key alone will not spend; a base URL alone will not spend. See [Safety](#safety).

| | |
|---|---|
| Expected cost | **70¢** (14 steps × 5 credits) |
| Worst case | **120¢** (24-step cap) |
| Model-input frames | **free** |
| Machine runtime | Coasty provisions and destroys its own VM |

`npm run estimate` prints this before anything runs.

## How it works

```
POST /v1/tasks                          Coasty provisions its own ephemeral VM,
                                        drives the agent, and destroys the VM
GET  /v1/runs/{id}                      poll to a terminal state
GET  /v1/runs/{id}/screenshots          the exact frames the model saw — free
GET  /v1/runs/{id}/events               per-step narration (SSE)
ffmpeg                                  frames → demo.mp4 + demo.gif + poster
```

The demo video is a **byproduct of running the automation**, not a separate artifact to author and keep in sync. There is no storyboard, no HTML mock, and nothing that can drift from reality — if the agent did something different, the video shows something different.

Verification is intrinsic and runs without a human watching:

```
✓ frames captured              14 frames
✓ frame count matches steps    14 frames vs 14 steps
✓ not all frames degraded      0 degraded
✓ frames are distinct          14/14 unique
✓ duration matches pacing      9.60s vs 9.60s expected
✓ stream width correct         1280x720
✓ video is non-trivial         288 packets
```

## Safety

This repo is built so that **accidental spend is structurally impossible**, not merely discouraged:

- **Fail-closed destination.** An unset `COASTY_BASE_URL` resolves to the bundled offline mock. Production is never a default.
- **Two independent consents.** `COASTY_ALLOW_LIVE=1` authorises the *destination*; `--confirm-cost-cents N` authorises the *cost*, and N must equal the server-computed worst case exactly.
- **Idempotency by default.** The submit key is derived from the prompt, so a retried submit returns the original run instead of provisioning a second machine.
- **A hard cap per unit.** A worst case above `capCents` in [`automation.json`](automation.json) is refused before any request is made.
- **No credentials, ever.** This automation targets a public catalog. Nothing here reads a password, a token, or a cookie — and nothing here touches a student record.

## Project layout

```
automation.json      the entire unit definition — prompt, target, budget, caps
src/client.mjs       Coasty client: fail-closed target, retry, idempotency
src/capture.mjs      model-input frames → mp4/gif/poster, with sanity checks
src/cli.mjs          run · demo · estimate
tools/mock.mjs       the bundled offline Coasty (real 1280×720 PNG frames)
tools/doctor.mjs     preflight
test/                25 tests, zero dependencies, fully offline
```

Adding a new automation is one `automation.json` and one prompt — `src/` never forks. See [AGENTS.md](AGENTS.md) for the authoring contract used by Claude Code and Codex.

## Tests

```bash
npm test     # node --test, no install, no network, no key
```

## Related

Part of the **Coasty automation catalog** — production-grade computer-use automations across 12 industries. See [the index](https://github.com/coasty-ai) for finance, healthcare, legal, logistics, energy, public sector, HR, retail, manufacturing, nonprofit and e-commerce.

- [Coasty docs](https://coasty.ai/docs) · [API reference](https://coasty.ai/docs/llms.txt)
- [computer-use-cookbook](https://github.com/coasty-ai/computer-use-cookbook) — the API, by endpoint, in 4 languages
- [open-cowork](https://github.com/coasty-ai/open-cowork) — the open-source AI coworker

## License

MIT © Coasty
