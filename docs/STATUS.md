https://github.com/IndicKangaroo/TwoGates.git

# STATUS

> Newest entry first. Update the snapshot, then add a log entry, at the end of every working session (human or AI). Keep entries short and factual, and link issues and PRs.

## Snapshot

| | |
|---|---|
| Date | 2026-10-13 |
| Phase / week | Phase 0, week 1 (assuming Week 1 = week of 19 Oct 2026) |
| Next gate | G0 on 1 Nov 2026 (spikes S1–S5) |
| Build state | nothing implemented yet; docs drafted |
| Hardware | none ordered |
| Hardware spend | ₹0 of ₹7,000 |
| API spend | ₹0 |

## Decisions in force

- Project registered as the team's major project. Working title: *Two Gates, One Spec: Hardware-Enforced Safety for LLM Agents That Control Physical Devices* (earlier idea name: MicroAgent-WASM). Confirm the registered title.
- No Rust. Python host with `wasmtime`; C for Wasm tool modules and firmware.
- Core Wasm modules; no Component Model/WIT.
- Budget cap ₹7,000 total (hardware plus API credits); single node; staged purchases.
- Energy analysis (E6) is kept as a scoped extra with a cut line (first thing cut if gate G3 slips).
- Publication costs and venue are decided later (by G3). The college requires two conference papers or one journal paper; the exact definition is still to be confirmed.

## Open questions

- Supervisor name; confirmed start date; lane owners and backups; final title; licence.
- What counts as a "paper" (accepted, submitted or presented)? Do workshops and regional IEEE conferences count? Is registration or travel funded? Is virtual presentation allowed?
- Can the college lab lend a Pi, ESP32 boards, a multimeter or a logic analyzer?

## Log

### 2026-10-13: docs drafted
- **Done:** README, AGENTS.md, PLAN, BLUEPRINT and this status log drafted.
- **In progress:** none.
- **Next:** create the repo and push these files; fill the placeholders (team, supervisor, lanes); supervisor meeting; order stage-1 hardware (Pi Zero 2 W first); scaffold Makefile, pyproject and CI; run spike S1 on a laptop.
- **Blockers:** none yet.
- **Decisions needed:** see Open questions.
