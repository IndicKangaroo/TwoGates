# AGENTS.md — working agreement for AI assistants on TwoGates

**How to use this file**
- **Coding agents** (Codex, Cursor, Claude Code, and similar): keep it in the repo root. For Claude Code, add a `CLAUDE.md` containing the single line `@AGENTS.md` (check your tool's docs for the import syntax).
- **Chat sessions** (claude.ai and similar): paste this file plus the top of `docs/STATUS.md` at the start, and attach `docs/BLUEPRINT.md` when the task touches design. Use a starter prompt from §14.
- **Humans:** update §3–§5 whenever reality changes. A stale instruction is worse than none.

---

## 1. Project in 60 seconds

TwoGates puts two independent safety gates between an LLM agent and physical actuators. Both gates are generated from one YAML safety spec.

- **Tier-1 (host, Python):** MCP server → Wasm tool modules (deny-by-default capabilities) → policy gate.
- **Tier-2 (ESP32-S3, C):** an independent interlock that re-enforces the same rules in integer arithmetic, with a heartbeat watchdog and an e-stop.
- **Link:** UART, COBS + CRC-16 frames, absolute set-points only.
- **Sim + oracle:** the same Tier-2 C core runs on a laptop with virtual time. An independent oracle checks what the pins actually did.
- **Context:** college major project, research track. Team of 4, 24 weeks. Output: two conference papers or one journal paper, a public repo, a demo. Budget cap ₹7,000. **No Rust** (nobody on the team knows it).

## 2. Source of truth (in this order)

1. `docs/BLUEPRINT.md`: design, semantics, protocol, requirements.
2. `docs/PLAN.md`: scope, schedule, gates, budget, risks.
3. `docs/STATUS.md`: what is true right now.
4. Code and tests.

If these disagree, **stop and ask**. Do not silently pick one. Design changes need an ADR row in BLUEPRINT §15.

## 3. Session protocol

**At the start**
1. Read this file, the newest entries in `docs/STATUS.md`, and the BLUEPRINT sections your task touches.
2. Restate the task, the acceptance criteria and the files you expect to touch. Ask at most three questions, and only if the task is truly ambiguous.
3. Check `git status` and the branch. Never work directly on `main`; use `lane/<topic>`.
4. Run the baseline (`make lint test`, if those targets exist) so you know what was already broken.

**At the end**
1. Run `make lint test` (and `make test-fw` if C changed). Report results honestly, including failures and skipped tests.
2. Add a dated entry at the **top** of `docs/STATUS.md`: done, in progress, next, blockers, decisions needed.
3. If you made a design decision, add or update the ADR row. Do not leave it only in a commit message.
4. Summarize: files changed, tests run, risks, follow-ups.

## 4. Repo map (planned; check STATUS.md for what exists)

```
README.md  AGENTS.md  Makefile  pyproject.toml
docs/        PLAN.md  BLUEPRINT.md  STATUS.md  decisions/
spec/        schema.json  examples/rig_v1.yaml  tests/vectors.yaml
twogates/    spec/ host/ sim/ eval/ faults/ energy/        (Python package)
tools/       src/ manifests/ build/                        (C → Wasm tool modules)
firmware/    components/tier2_core/  components/hal_esp32/  main/  logger/
hardware/    wiring.md  pinout.md  spend.csv
experiments/ scenarios/ prompts/ models.yaml ANALYSIS_PLAN.md   (runs/ is git-ignored)
tests/       unit/ property/ integration/ hil/
```

## 5. Commands (target interface)

If a target is missing, implement it as part of the task or say so. Do not pretend it ran.

| Command | Purpose | Costs money | Needs hardware |
|---|---|---|---|
| `make setup` | virtualenv + pinned dependencies | no | no |
| `make lint` | ruff + mypy | no | no |
| `make test` | pytest: unit, property, integration (excludes `hil` and `paid`) | no | no |
| `make test-fw` | C core tests with ASan/UBSan (host build) | no | no |
| `make test-hil` | `pytest -m hil` | no | **yes** |
| `make spec` | compile the spec → `build/generated/` | no | no |
| `make tools` | build and sign the Wasm tool modules | no | no |
| `make fw-build` | ESP-IDF build | no | no |
| `make fw-flash PORT=…` | flash the ESP32 | no | **yes** |
| `make sim-demo` | scripted end-to-end demo in SIM | no | no |
| `make eval-pilot` | whole pipeline with the fake LLM | no | no |
| `make eval-record` | record LLM episodes (needs `ALLOW_PAID=1` and `BUDGET_INR=…`) | **yes** | no |
| `make eval-replay` | replay recorded calls under D1/D2/D3 | no | no |
| `make report` | tables and figures from `runs/` | no | no |
| `make repro EXP=…` | regenerate one experiment's results | maybe | maybe |

## 6. Architecture invariants (do not violate)

1. **The spec is the only source of limits.** No numeric limits in host code, firmware, tests or docs except via generated files, or quoted from the spec.
2. **`tier2_core` is portable and deterministic:** C11, no ESP-IDF headers, no `malloc` after init, no blocking calls, integer arithmetic only, time injected, wrap-safe `u32` millisecond comparisons.
3. **Tier-1 never touches actuators directly.** Only the link to Tier-2 does. (The Pi's GPIO is used only in the Phase-0 spike.)
4. **A denial never changes state.** Safe-direction commands are always accepted when armed. Commands are absolute set-points (no toggles, no relative changes).
5. **Tier-2 is authoritative and Tier-1 is stricter** (guard band). Never loosen Tier-2 to make a test pass.
6. **Wasm modules get only the imports their manifest grants.** No WASI. Unsigned modules are refused.
7. **The oracle is independent.** It must not import Tier-1 policy code, and it is written and reviewed by a different person than Tier-1.
8. **The enforcement-off switch exists only in sim builds** (`T2_SIM_BUILD`). It must never compile into firmware.
9. **ESP32 radios stay off; no OTA.**
10. **Protocol and spec semantics change only with** an ADR, a version bump (`ver` byte or `spec_version`), and updated vectors.
11. **Reason codes, evaluation precedence and the event schema are shared** by Tier-1, Tier-2 and the oracle (BLUEPRINT §5.3, §6.8, §7.4).
12. **Time:** nothing calls wall-clock time directly; everything uses the `Clock` abstraction.

## 7. Hardware, secrets and cost rules

- Low voltage only (≤ 6 V). Never mains. Never instruct anyone to connect mains loads.
- Never print, log or commit API keys. `.env` is git-ignored; keep `.env.example` current.
- Paid LLM calls only with `ALLOW_PAID=1` and a budget cap. Estimate tokens first and state the expected ₹ before running.
- Do not run anything that actuates hardware unless the human confirmed the rig is wired, supervised, and the e-stop is within reach.
- Do not suggest purchases outside PLAN §7 without flagging the ₹7,000 cap.

## 8. Coding conventions

- **Python:** 3.11+, type hints everywhere, `ruff` (lint and format), `mypy`. Small modules, docstrings on public functions, no global state, inject `Clock` and `Link`. No `print` in library code; use events and logging.
- **C (firmware):** C11, `-Wall -Wextra -Werror`, `t2_` prefix, fixed-width integers, no recursion, no dynamic allocation in the core, a unit test for every rule, sanitizers in host builds.
- **Wasm tools:** freestanding C, imports only from `hw.*`, at most 64 KiB of memory.
- **Commits:** Conventional Commits (`feat:`, `fix:`, `test:`, `docs:`, `refactor:`, `chore:`), one logical change each, with the task ID (for example `A2`) in the PR title.
- **Branches and PRs:** `lane/<topic>`; review by someone from another lane; CI must be green.
- **Dependencies:** pinned. Ask before adding one.

## 9. Testing rules

- Every rule change needs a vector in `spec/tests/vectors.yaml`, a Tier-1 test, a core test and an oracle test.
- Properties I1–I5 (BLUEPRINT §14.2) must stay green.
- HIL tests carry `@pytest.mark.hil` and paid tests carry `@pytest.mark.paid`; neither runs by default.
- Fix the code, not the test, unless the spec says the test is wrong. If you change a test's expectation, say why in the PR.
- Report failures verbatim. Never claim a test passed without running it.

## 10. Research-integrity rules

- **Never invent** results, numbers, citations or measurements. Every number in docs or papers comes from files under `experiments/results/**` or `runs/**` generated by scripts, or is labeled *estimate*, *target* or `[verify]`.
- Do not hand-edit raw logs or results. Re-run instead.
- Do not cite a reference you have not verified (DOI or arXiv ID). Mark unverified ones `[verify]`.
- Scenarios, prompts and the analysis plan are frozen at the gates (PLAN §5). Record any later change.
- Keep claims honest: write "we found no work that…" with the search date, never "first to…".
- Report negative and null results.

## 11. Stop and ask the human when…

- the docs disagree with each other or with the code;
- a change would alter spec semantics, the protocol, the threat model or an invariant;
- the task needs hardware actions, spending money or paid API calls;
- you would add a dependency, a new tool, or a scope item that is not in the plan;
- test results contradict a documented guarantee;
- you are unsure whether something is safe for the rig.

## 12. Task playbooks by lane

**A — Host & Spec.** Key files: `twogates/spec/`, `host/policy.py`, `runtime.py`, `link.py`, `mcp_server.py`. Order: schema → compiler → Tier-1 against vectors → runtime and capabilities → MCP → link. Pitfalls: wasmtime-py API drift (check the installed version), commit Tier-1 state only after Tier-2's ACK, apply the guard band only in Tier-1, inject clocks.

**B — Firmware.** Key files: `firmware/components/tier2_core/`, `hal_esp32/`, `main/`. Order: CRC and COBS → parser → rules and state machine against vectors → fuzzing → HAL. Pitfalls: strapping pins, wrap-safe timestamps, safe outputs at boot, no `malloc`, no ESP-IDF includes in the core.

**C — Evaluation.** Key files: `twogates/eval/`, `sim/oracle.py`, `experiments/`. Order: scenarios → fake adapter → runner (virtual time) → oracle → record/replay → metrics → real adapters. Pitfalls: oracle independence, cost guard first, record model IDs, never mix prompt variants within one run ID.

**D — Hardware, faults, energy, papers.** Key files: `hardware/`, `twogates/faults/`, `twogates/energy/`, `docs/`. Order: bring-up checklist → fault injector → attacker → malicious modules → energy rig. Pitfalls: common ground, separate motor supply, calibrate the INA219, every claim must trace to a results file.

## 13. Glossary

- **Tier-1 / Tier-2:** host-side policy gate / ESP32 firmware interlock.
- **MCP:** Model Context Protocol, how agents call tools.
- **Wasm:** WebAssembly; here, core modules compiled from C and run in wasmtime.
- **Capability:** a host function a module is explicitly allowed to import.
- **Activation:** an accepted transition from the safe value to a non-safe value.
- **Safe-direction command:** one that moves toward the safe value (always accepted when armed).
- **Safe state:** every output at its safe value (LEDs off, relay off, motor coasting).
- **Spec hash:** first 8 bytes of SHA-256 over the canonical spec; checked in the handshake.
- **Guard band:** extra margin that makes Tier-1 stricter than Tier-2.
- **Heartbeat:** periodic host message; if it stops, Tier-2 enters the safe state.
- **SIM / HIL-lite / FULL:** laptop with virtual time / laptop + real ESP32 / Pi + real ESP32.
- **Oracle:** independent checker of event logs against the spec; the ground truth for violations.
- **D0–D3:** defense configurations: none, Tier-1 only, Tier-2 only, both.
- **E1–E6:** experiments (BLUEPRINT §10.1). **G0–G5:** project gates (PLAN §3.2). **ADR:** architecture decision record.

## 14. Starter prompts (copy-paste)

**General**
> Read AGENTS.md and the newest entries of docs/STATUS.md. Today's task: <task ID and description>. Follow the session protocol. Before coding, restate the acceptance criteria and list the files you will touch.

**Review**
> Review this diff against the AGENTS.md §6 invariants and BLUEPRINT §5 and §7. List violations first, then risks, then nits.

**Design question**
> Using docs/BLUEPRINT.md as the baseline, propose options for <question>. Give trade-offs, a recommendation, and the ADR row you would add. Do not change files.

**Experiment analysis**
> Given the files in runs/<id>, compute <metric> exactly as defined in BLUEPRINT §10.6, with the intervals from §10.7. Show the script, not just the numbers.

**Writing**
> Draft section <X> of Paper A using only claims backed by files in experiments/results/ or by docs/BLUEPRINT.md. Mark anything unverified [verify].

**Debugging**
> Here is the failing test output and the relevant files. Find the root cause before proposing a fix, and do not weaken tests.
