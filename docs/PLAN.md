# TwoGates — Project Plan

| | |
|---|---|
| Version | 0.1 (draft) |
| Date | 2026-10-13 |
| Assumed start | Monday 19 Oct 2026 = Week 1 (shift every date below if you start later) |
| Planned end | Week 24 = week of 29 Mar 2027 |
| Supervisor | [fill in] |
| Team (4) | [fill in: one owner per lane A–D, plus a named backup for each] |
| Companion docs | [`../README.md`](../README.md) · [`BLUEPRINT.md`](BLUEPRINT.md) · [`../AGENTS.md`](../AGENTS.md) · [`STATUS.md`](STATUS.md) |

Working title: *Two Gates, One Spec: Hardware-Enforced Safety for LLM Agents That Control Physical Devices* (earlier idea name: MicroAgent-WASM).

---

## 1. Goals, success criteria, non-goals

**Goal.** Build and measure a two-layer safety design for LLM agents that drive physical actuators: a Wasm-sandboxed host gate (Tier-1) plus an independent ESP32 interlock (Tier-2), both generated from one safety spec. Answer how often LLMs issue unsafe actuator commands, which layer stops them, what happens under link faults and host compromise, and what the layers cost in latency and energy.

**Success criteria**

| ID | Criterion | By |
|---|---|---|
| SC1 | End-to-end SIM demo: agent → MCP → Wasm → Tier-1 → link → Tier-2 sim → oracle. Conformance vectors give identical verdicts on Tier-1, the Tier-2 core and the oracle. | G1 (end W6) |
| SC2 | FULL hardware demo: spec-hash handshake, heartbeat watchdog to safe state, LED/relay/motor control through both tiers, HIL smoke tests green. | G2 (end W10) |
| SC3 | E3 campaign (link-fault grid + raw-frame attacker): zero oracle-reported violations at the actuators; safe-state entry time measured. | W14 |
| SC4 | E1: at least 520 recorded episodes replayed under D0–D3; E2: at least 150 closed-loop episodes; all with confidence intervals. | G4 (end W19) |
| SC5 | E4 table complete; E5 latency breakdown complete; E6 energy complete or formally cut. | G4 |
| SC6 | Paper A drafted by G3 and submitted by W16 at a venue chosen on cost and deadline; Paper B drafted by W22. | G3 / G5 |
| SC7 | Final report, demo video, public repo with docs, and a reproducible results bundle. | G5 (end W24) |

**Non-goals.** SaaS or fleet control plane; Component Model/WIT; Rust; multi-node; Wi-Fi/OTA on the ESP32; link encryption; hard real-time claims; safety certification; mains-powered loads.

---

## 2. Team, roles and working agreements

| Lane | Owner | Backup | Scope | Main languages |
|---|---|---|---|---|
| **A — Host & Spec** | [name] | [name] | spec compiler, Tier-1, Wasm runtime, MCP server, link layer | Python (+ small C tool modules) |
| **B — Firmware** | [name] | [name] | `tier2_core`, ESP32 HAL, logger firmware, HIL tuning | C / ESP-IDF |
| **C — Evaluation** | [name] | [name] | scenarios, LLM adapters, runner, sim glue, oracle, metrics | Python |
| **D — Hardware, faults, energy, papers** | [name] | [name] | rig, fault/attacker harness, energy, literature review, writing, demo | Python + hardware + writing |

**Working agreements**
- Independence: Tier-1 (lane A), Tier-2 (lane B) and the oracle (lane C) are written by different people. The oracle never imports Tier-1 code.
- Review pairs: A↔B (protocol and core), C↔D (evaluation and faults). Nothing merges without a review from another lane.
- Every lane has a named backup. Pair on the critical-path items (A5 link layer, B1 core).
- Each lane writes the paper sections for its own area; lane D integrates and edits.
- `docs/STATUS.md` is the single status log. Update it at the end of every working session (human or AI).

**Cadence**

| When | What |
|---|---|
| Monday (30 min) | plan the week; confirm owners and blockers |
| Midweek (15 min or async) | integration check |
| Friday (30 min) | demo what works; update STATUS.md |
| Every 2 weeks | supervisor check-in (agenda in §11) |
| Gates | formal review, with the supervisor where possible |

**Tools:** GitHub repo, Projects board (Backlog / In progress / Review / Done; gates as milestones), one issue per task ID, a team chat group.

---

## 3. Timeline

### 3.1 Phases and gates

| Phase | Weeks | Dates | Focus | Gate |
|---|---|---|---|---|
| 0 Spike & setup | W1–W2 | 19 Oct – 1 Nov 2026 | scaffolding, spikes S1–S5, literature kickoff, order stage-1 hardware | **G0** 1 Nov |
| 1 Core in simulation | W3–W6 | 2 Nov – 29 Nov | spec compiler, Tier-1, Wasm runtime, protocol, `tier2_core`, sim + oracle v0 | **G1** 29 Nov |
| 2 Hardware & fault injection | W7–W10 | 30 Nov – 27 Dec | firmware HAL, rig, HIL, watchdog tuning, fault harness v1 | **G2** 27 Dec |
| 3 LLM evaluation v1 & Paper A | W11–W15 | 28 Dec – 31 Jan 2027 | scenarios, adapters, record/replay, first E1 results, E3 campaign, Paper A | **G3** 24 Jan (draft frozen); submission window W15 |
| 4 Full study & extras | W16–W19 | 1 Feb – 28 Feb | full E1/E2, E4, E5, E6 (if on track), analysis | **G4** 28 Feb (results freeze) |
| 5 Papers, report, release | W20–W24 | 1 Mar – 4 Apr | Paper B, final report, demo, video, release, defense prep | **G5** 4 Apr |

Capacity assumption: about 10 h per person per week = 960 h over 24 weeks. Planned work is 825 h, leaving a 14% buffer. Mark exam weeks and holidays on the board and reduce that week's capacity (December–January is the most exposed).

### 3.2 Gate criteria

| Gate | Pass criteria | If it fails |
|---|---|---|
| **G0** | S1–S5 demos work (BLUEPRINT §16). LLM pilot cost is within the estimate. Repo and CI skeleton exist. Hours re-estimated per lane. | S1 fails on the Pi → run the host on a laptop; the Pi is used only for energy and the demo. S3/S4 fail → one extra week, and E6 is dropped first. Two or more spikes fail → re-scope with the supervisor. |
| **G1** | SC1 demo. Vectors identical on three implementations. Property tests I1–I5 green. Tag `v0.1`. Spec language and protocol frozen. | Slip of up to one week allowed (uses buffer). Otherwise cut E4 extras and closed-loop scope. |
| **G2** | SC2 demo on hardware. Heartbeat safe-state time measured. Attacker smoke test shows zero violations. Tag `v0.2`. | Simplify firmware extras (drop logger firmware); run E1/E2 in SIM only. |
| **G3** | Paper A draft complete (design, fault injection, preliminary E1) and internally reviewed. Venue chosen by cost. Scenarios and prompts frozen. | Submit a shorter version or a later venue. Do not cut Paper B. |
| **G4** | E1/E2 complete. E4/E5 complete. E6 complete or formally cut. Analysis plan followed. Results frozen. | Cut E6, then E2 breadth. |
| **G5** | SC7 deliverables. Tag `v1.0`. Reproducible bundle. | Use the buffer. The defense proceeds on whatever was frozen at G4. |

### 3.3 Week by week

| Week | Starts | Focus |
|---|---|---|
| W1 | 19 Oct | Create repo from these docs; fill placeholders; supervisor meeting; order stage-1 hardware; install toolchains; scaffold Makefile/pyproject/CI. A: S1 on laptop. C: S5 pilot design. D: literature search kickoff. |
| W2 | 26 Oct | Hardware arrives. S2–S4 on Pi and ESP32. S5 LLM pilot. **G0 on 1 Nov.** |
| W3 | 2 Nov | A1 spec schema/loader. B1 start (COBS, CRC, parser). C1 scenarios. D1 rig assembly begins. |
| W4 | 9 Nov | A2 Tier-1. B1 state machine and rules. C2 adapters. D3 literature notes. |
| W5 | 16 Nov | A3 Wasm runtime and capabilities. B2 core tests and vectors. C4 sim glue and oracle v0. **Protocol and spec frozen.** |
| W6 | 23 Nov | A4 MCP server. A5 link (SIM). Integration. **G1 on 29 Nov.** |
| W7 | 30 Nov | B3 HAL and tasks. D1 rig complete. A5 serial link. HIL-lite bring-up. |
| W8 | 7 Dec | B3 e-stop and safe outputs. HIL tests. D2 fault harness v1. |
| W9 | 14 Dec | B5 watchdog tuning. A5 hardening. C3 runner and record/replay. E3 dry run. |
| W10 | 21 Dec | **G2 on 27 Dec.** Tag `v0.2`. Light week (holidays). |
| W11 | 28 Dec | C5 metrics/stats skeleton. Pilot E1 on one model. Light week. |
| W12 | 4 Jan | E1 recording runs (small model + local model). E3 campaign. Draft analysis plan. |
| W13 | 11 Jan | E1 more models; replay D1–D3. Paper A drafting (design, spec, implementation). |
| W14 | 18 Jan | Paper A evaluation sections and figures. **G3 on 24 Jan.** |
| W15 | 25 Jan | Paper A polish and submission window; internal review; E4 preparation. If the chosen deadline is 1–4 Feb, submit across the W15/W16 boundary. |
| W16 | 1 Feb | E4 runs (M1–M8). E2 closed-loop start. |
| W17 | 8 Feb | E2 runs. E5 latency. D4 energy rig setup (if on track). |
| W18 | 15 Feb | E6 energy runs (cut line). Analysis. |
| W19 | 22 Feb | Finish analysis. **G4 on 28 Feb.** Results freeze; tag. |
| W20 | 1 Mar | Paper B drafting; figures; report outline. |
| W21 | 8 Mar | Paper B drafting; demo script; video storyboard. |
| W22 | 15 Mar | Paper B draft complete; report draft; rehearsal. |
| W23 | 22 Mar | Repo polish and docs; release candidate; submit Paper B (venue by cost); defense slides. |
| W24 | 29 Mar | **G5 on 4 Apr.** Buffer; final demo rehearsal; tag `v1.0`. |

---

## 4. Work packages (hours are estimates; re-estimate at G0)

### Lane A — Host & Spec (200 h)

| ID | Task | h | Depends on | Done when |
|---|---|---|---|---|
| A1 | Spec schema, loader, validator, code generation | 30 | S1 | `make spec` emits all six artifacts deterministically; invalid specs give clear errors; unit tests |
| A2 | Tier-1 policy engine | 30 | A1 | all vectors pass (guard 0); property tests I2–I4; structured denials |
| A3 | Wasm runtime, capabilities, manifests, signing, limits | 40 | S1 | led/relay/motor modules run; M1–M8 behave as in BLUEPRINT §11.4; signature check enforced |
| A4 | MCP server, tool generation, `get_status`/`wait`/`stop_all` | 15 | A1, A3 | fake-LLM integration test passes; an MCP client can call every tool |
| A5 | Link layer (framing, heartbeat, retries, fault hook) | 30 | protocol freeze (W5) | SIM and serial parity; retries and de-dup tested; fault hook works |
| A6 | Config, logging, CLI | 10 | — | JSONL events match the schema; run manifests written |
| A7 | Integration, hardening, docs | 45 | continuous | CI green; docs match code |

### Lane B — Firmware (190 h)

| ID | Task | h | Depends on | Done when |
|---|---|---|---|---|
| B1 | `tier2_core`: COBS, CRC-16, parser, state machine, rules, watchdog | 60 | A1 tables | vectors identical to Tier-1 and oracle; ASan/UBSan clean |
| B2 | Core tests, vectors, fuzzing | 30 | B1 | at least 10⁶ fuzz iterations without crash; coverage report |
| B3 | ESP32 HAL, tasks, safe outputs, e-stop | 45 | B1, D1 | HIL smoke tests green |
| B4 | Logger firmware (INA219) | 10 | stage-2 parts | CSV stream and START/STOP markers work; calibration run |
| B5 | HIL bring-up and watchdog tuning | 30 | B3 | `t_safe` and jitter report; timeout value chosen |
| B6 | Docs, pin map, cleanup | 15 | — | pin map and wiring match the code |

### Lane C — Evaluation (210 h)

| ID | Task | h | Depends on | Done when |
|---|---|---|---|---|
| C1 | Scenario catalog and success predicates | 25 | — | 26 scenarios, predicate registry, unit tests |
| C2 | LLM adapters (Anthropic, OpenAI-compatible/Ollama, fake) | 25 | — | usage and cost recorded; fake adapter drives CI |
| C3 | Runner, trace schema, record/replay, cost guard | 40 | C1, C2, C4 | `make eval-pilot` works free; paid runs need `--allow-paid`; replay under D1–D3 |
| C4 | Sim glue, virtual actuators, oracle | 40 | B1 | oracle mutation tests; differential tests against Tier-2 sim |
| C5 | Metrics, statistics, report generator | 35 | C3 | tables and figures regenerate from `runs/`; Wilson/bootstrap/McNemar tested |
| C6 | Pilot, full runs, closed-loop runs, analysis | 45 | C3–C5 | SC4 met |

### Lane D — Hardware, faults, energy, papers (225 h)

| ID | Task | h | Depends on | Done when |
|---|---|---|---|---|
| D1 | Rig build, wiring, safety checks | 20 | hardware arrival | bring-up checklist complete; wiring diagram in `hardware/` |
| D2 | Fault-injection and attacker harness; E3/E4 runs | 45 | A5, B3 | SC3 met; M1–M8 outcome table |
| D3 | Literature review and related work | 30 | — | at least 30 sources screened (Scholar, IEEE Xplore, ACM DL); related-work table; novelty statement reviewed by the supervisor |
| D4 | Energy rig and E6 | 35 | B4, A3 | **cut line**: energy table with CIs, or a formal cut at G4 |
| D5 | Paper A | 35 | — | draft frozen at G3 |
| D6 | Paper B | 35 | — | draft complete by W22 |
| D7 | Final report, demo, video, poster | 25 | — | SC7 |

**Total planned effort: 825 h** (A 200, B 190, C 210, D 225) against about 960 h of capacity.

---

## 5. Experiments plan

| ID | Question | Sample size | Where | Lane | Window |
|---|---|---|---|---|---|
| E1 | How often do LLMs emit unsafe actuator calls, and which layer catches them? | 26 scenarios × 4 models × 5 runs = 520 (open-loop, D0, replayed under D1–D3); plus 14 scenarios × 2 models × 3 runs = 84 with limits *not* in the prompt | SIM | C | W11–W17 |
| E2 | What does enforcement cost in task success? | 26 scenarios × 3 models × 3 runs = 234 (closed-loop, D3) | SIM | C | W16–W18 |
| E3 | Does Tier-2 hold under link faults and a compromised host? | fault grid + attacker campaign | HIL-lite / FULL | D (with B) | W12–W14 |
| E4 | What does Wasm add against bad tool code? | modules M1–M8 × Wasm vs native | FULL / SIM | D (with A) | W16 |
| E5 | What does each layer cost in latency? | N ≥ 10,000 calls per config | FULL | A | W17 |
| E6 | What does each layer cost in energy? (scoped extra) | 7–10 repetitions × 5 configs | FULL | D | W18 |

**Rules**
- Scenarios and prompts are frozen at G3; the analysis plan (`experiments/ANALYSIS_PLAN.md`) is frozen before the full runs; results freeze at G4.
- Models are pinned by recorded model ID; prices and the USD→INR rate come from `experiments/models.yaml`.
- Raw logs are never edited. Every table and figure regenerates with `make report` or `make repro`.
- Pilot (S5, week 2): about 50 tool calls with two models to check tool-use reliability and token usage against the estimate.
- Closed-loop E2 is the only part where a model sees denials. Open-loop E1 uses record/replay to keep API cost low.

---

## 6. Publication plan

**6.1 Requirement.** The college requires two conference papers *or* one journal paper. Confirm exactly what counts (accepted vs submitted vs presented; whether workshops and regional IEEE conferences count; the journal threshold list) at the first supervisor meeting.

**6.2 Papers**

| Paper | Working title | Content | Draft | Submit |
|---|---|---|---|---|
| A | Two Gates, One Spec: Hardware-Enforced Safety for LLM Agents That Control Physical Devices | design, spec language, implementation, fault injection (E3), preliminary E1 | G3 (24 Jan) | W15–W16 |
| B | How Often Do LLM Agents Issue Unsafe Actuator Commands, and Which Layer Stops Them? | E1, E2, E4, E5 (and E6 if done) | W22 | W23 |
| Fallback | single journal paper merging A and B (short-letter format) | | W22 | if the department prefers a journal |

**6.3 Outlines**

Paper A: Introduction · Related work · Threat model and design principles · Spec language and compiler · Implementation (host, Wasm sandbox, firmware, protocol) · Evaluation (E3 fault injection, preliminary E1) · Limitations · Conclusion.

Paper B: Introduction · Related work · System summary · Scenarios and method (record/replay, oracle, defense configs) · Results (E1 unsafe-call rates and catch attribution, E2 task impact, E4 sandbox ablation, E5 overhead, E6 energy) · Threats to validity · Conclusion.

**6.4 Venue strategy (cost-aware; decision deferred by the team).** Re-verify every date and fee at the time of choosing. Findings from the October 2026 research:

| Option | What was found | Cost signal |
|---|---|---|
| IEEE Embedded Systems Letters | 4-page letters; overlength charge $125 per page beyond 4; optional open access $2,800 (avoid) | ₹0 if you stay within 4 pages |
| Indian IEEE-sponsored conferences | registration examples ₹4,000–12,000 per paper; some need a full (non-student) registration for IEEE Xplore publication | low |
| EAI SecureComm 2027 | deadline 1 Feb 2027 per its EasyChair call (one aggregator listed 1 Mar); Hefei, China; Springer LNICST; past editions ≈ $620–750 early member rate; one full registration required even for all-student teams | high (registration plus travel) |
| SAIS '27 (USENIX Conference on Secure Agentic-AI Systems) | deadline 4 Feb 2027; 7–8 June 2027, Santa Clara | very high (travel) |
| CPS-IoT Week 2027 workshops | event May 2027, Boulder; 2027 workshop calls not found yet; main-track deadline 5 Nov 2026 is too early | unknown |

Action: shortlist at least four venues by W8 (at least two low-cost), verify deadlines and fees, decide by G3.

**6.5 Integrity rules for papers.** Every number comes from a results file. Every reference is verified (DOI or arXiv ID) before it is cited. Related-work claims are phrased as "we found no work that…" with the search date. Agree author order and a contributions statement early.

---

## 7. Budget and procurement

**Cap: ₹7,000 total, hardware plus API credits.** Publication costs are outside this cap and decided later.

| Item | Stage | Likely ₹ | Range ₹ |
|---|---|---|---|
| Raspberry Pi Zero 2 W | 1 (order now; stock patchy) | 2,000 | 1,500–2,300 |
| ESP32-S3-DevKitC-1 | 1 | 700 | 515–1,190 |
| microSD 32GB (or reuse a 16GB+ card) | 1 | 750 | 0–1,100 |
| 5V 2.5A charger + micro-USB cable (or reuse) | 1 | 300 | 0–400 |
| Jumpers, breadboard, LEDs, resistors, push-button, header | 1 | 300 | 180–500 |
| INA219 power monitor | 2 (after G0) | 300 | 240–350 |
| TB6612FNG driver + small geared motor | 2 | 340 | 250–440 |
| 1-channel relay module (optional) | 2 | 90 | 0–130 |
| Shipping (two orders) | | 200 | 100–300 |
| **Hardware total** | | **≈ 4,980** | **2,800–6,700** |
| Buffer, unspent (replacement ESP32) | | ≈ 750 | |
| API credits | | 100–800 | |
| **Plan total** | | **≈ 5,800–6,500** | |

Stage-1 spend (before G0) is about ₹4,150 (₹3,000 if you reuse an SD card and charger). Prices are from Indian retailer listings in October 2026 and move; re-check at purchase.

**API budget.** At about ₹96 per US$ and roughly 125 episodes per model, one full pass costs about ₹34 on Haiku 5.5 and about ₹670 on Sonnet 5.5 (token-count estimates; check current prices in `experiments/models.yaml`). A free local open-weight model costs nothing. Cap the total at ₹800 with the runner's cost guard.

**Rules**
- **Stop-loss:** if cumulative spend would exceed ₹6,500 without supervisor approval, stop and re-plan.
- Ask the college lab for loans first (Pi, ESP32, multimeter, logic analyzer, bench supply).
- Order from as few shops as possible; check whether GST is included.
- If the Pi is out of stock, run the host on a laptop through Phase 3; the Pi is needed for E5/E6 and the demo. Orange Pi Zero 3 listings seen were pricey (₹3,999 for 1GB, out of stock).
- Log every purchase in `hardware/spend.csv` (date, item, shop, price).

---

## 8. Risk register

| ID | Risk | L | I | Early signal | Mitigation | Owner |
|---|---|---|---|---|---|---|
| R1 | Hardware integration slips (UART/GPIO/ESP32 debugging) | M | H | HIL tests still failing in W9 | SIM-first design; staged purchases; HIL-lite on a laptop; B and D pair up | B / D |
| R2 | wasmtime-py limits (aarch64 wheel, fuel/epoch API, memory limits) | M | M | S1 fails | host on laptop; fall back to subprocess runtime; keep Wasm claims to what is verified | A |
| R3 | Linux jitter causes false heartbeat trips | M | M | spurious FAULT_LATCHED in HIL | heartbeat thread priority; timeout ≥ 250 ms; measure; report jitter as a limitation | A / B |
| R4 | Paper timeline or acceptance falls short of the college rule | H | H | no venue chosen by G3; unclear definition | ask in week 1; two shots; cost-aware venues; journal fallback; start writing in W11 | D |
| R5 | Budget overrun or dead boards | M | M | spend above ₹6,500; ESP32 shorted | buffer; staged buying; wiring checklist; lab loans | D |
| R6 | API cost, rate limits or model retirement | M | M | pilot tokens above estimate | cost guard; config-driven models; local-model fallback; freeze model IDs at G3 | C |
| R7 | Bandwidth and bus factor (Python host lane is the critical path) | H | H | two weeks of missed weekly goals | named backups; pair on A5/B1; AGENTS.md and STATUS.md; cut list | all |
| R8 | Scope creep | M | H | features not in the blueprint appear | freeze at G1; ADR process; cut order (§9) | lead |
| R9 | Novelty challenged (Wassette, Aerogel, AgentSpec, the ASU preprint, hobby projects) | H | M | reviewer feedback; new preprints | honest claim framing; finish the literature review; focus on the combination and the measurement; check arXiv monthly | D |
| R10 | Physical safety incident (shock, burn, short) | L | H | none | RIG rules; low voltage only; no unattended runs | D |
| R11 | Result validity (LLM non-determinism, oracle bug, spec error) | M | H | oracle/Tier mismatches; unstable results | temperature 0 where possible; repeated runs; CIs; oracle independence and mutation tests; vectors; analysis plan | C |
| R12 | Exams and holidays cut capacity | H | M | logged hours below plan | buffer; mark exam weeks; front-load work | all |
| R13 | SIM and hardware behave differently | M | M | HIL results differ from SIM | run conformance vectors on hardware; report differences | B / C |

---

## 9. Scope control

**Freeze points:** spec language and protocol at G1 (changes need an ADR and a version bump); scenarios and prompts at G3; analysis plan before the full runs; results at G4.

**Cut order (first to last):**
1. E6 energy
2. E2 breadth (fewer models)
3. E1 prompt variant "limits not stated"
4. Extra malicious modules (keep M1–M3, M7, M8)
5. Logger firmware
6. Closed-loop recovery analysis

**Never cut:** spec compiler, both tiers, the oracle, E1, E3, at least one paper.

**Scope-add rule:** a new idea becomes an issue labelled `scope-add`. It is accepted only if it replaces equal hours and does not touch the never-cut list, and the supervisor hears about it at the next check-in.

---

## 10. Quality plan and definition of done

- **Code task:** tests (unit plus the relevant property tests or vectors), docs updated, CI green, reviewed by another lane, STATUS.md updated, no TODOs without an issue, no hard-coded limits (principle P6).
- **Experiment:** scripts and config committed; run manifest written; results regenerate via `make`; analysis plan followed; figures reproducible.
- **Paper:** every claim traceable to a results file; references verified; related-work table checked; reproducibility statement; supervisor review.

---

## 11. Communication and supervisor engagement

**Fortnightly check-in agenda:** (1) demo and progress against the plan; (2) risks and decisions needed; (3) asks.

**First-meeting questions**
1. What counts as a "paper" for the requirement: accepted, submitted, or presented? Do workshops and regional IEEE conferences count? Which journals clear the threshold?
2. Is there a project budget, and are registration or travel funded? Is virtual presentation allowed?
3. Can the lab lend a Raspberry Pi, ESP32 boards, a multimeter, a logic analyzer or a bench supply?
4. Is open-sourcing the code allowed under the college's IP policy, and which licence do you prefer?
5. What are the mid-term and final evaluation dates and formats?
6. Who else can review the threat model and the novelty claims?
7. Are there rules for using AI assistants in the project and for reporting that use?

Hold gate reviews with the supervisor at G1, G3 and G4 at least.

---

## 12. First 10 days

- [ ] Create the GitHub repo; add these docs; protect `main`; add the CI skeleton.
- [ ] Fill in the placeholders: team, supervisor, lane owners and backups, final title.
- [ ] Supervisor meeting: ask the questions in §11.
- [ ] Ask the lab for loans.
- [ ] Order stage-1 hardware (Pi Zero 2 W first; stock is patchy).
- [ ] Install toolchains: Python 3.11, clang + lld, CMake, ESP-IDF.
- [ ] Scaffold: `pyproject.toml`, Makefile targets (stubs are fine), a `tier2_core` skeleton with the CRC check value `0x29B1` as its first test, and a spec schema skeleton.
- [ ] Run spike S1 on a laptop (wasmtime + a C-compiled module with a host import).
- [ ] Set up an API account with a spend cap; design the S5 pilot.
- [ ] Start the literature search (Scholar, IEEE Xplore, ACM DL); see the keyword list in Appendix D.
- [ ] Create the Projects board with gates as milestones.

---

## Appendix A — Demo script for the defense

1. Show the rig and the spec (`rig_v1.yaml`).
2. Run a nominal task with a live LLM: the motor runs, then stops.
3. Run an unsafe task ("toggle the relay as fast as you can"): Tier-1 denies with a structured reason.
4. Run the raw-frame attacker, which bypasses Tier-1: Tier-2 rejects the frames and the oracle reports zero violations.
5. Pull the UART or kill the host: the motor stops within the heartbeat timeout.
6. Run the malicious-module test: the native plugin vs the Wasm module.
7. Show the results report: E1 table, latency and (if done) energy plots.

## Appendix B — Final deliverables checklist

- [ ] Project report
- [ ] Two conference papers (or one journal paper) in the state the department requires
- [ ] Public repo: code, docs, spec, results bundle, CI badge
- [ ] Live demo and a recorded video
- [ ] Hardware rig with wiring diagram and BOM
- [ ] Slides and poster

## Appendix C — Meeting agenda template

Date · attendees · progress since last time (link to STATUS.md) · demo · blockers · decisions needed · action items with owners and dates.

## Appendix D — Literature search keywords

`WebAssembly sandbox LLM agent tools`; `Wasm capability access control IoT peripherals`; `MCP server hardware actuator safety`; `runtime enforcement LLM agents`; `LLM-controlled robots safety guardrails`; `deterministic safety gate LLM planner actuation`; `Simplex architecture runtime assurance`; `ESP32 MCP safety firmware`; `prompt injection tool outputs agents physical`; `WebAssembly microcontrollers energy benchmark`. Chase citations forward from Wassette, Aerogel, AgentSpec, RoboGuard and the 2026 "Physical Agentic AI" preprint.
