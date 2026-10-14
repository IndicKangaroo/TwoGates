# TwoGates

**Hardware-enforced safety for LLM agents that control physical devices.**

> Working title: *Two Gates, One Spec: Hardware-Enforced Safety for LLM Agents That Control Physical Devices*
> Earlier idea name: MicroAgent-WASM · Status: **planning → scaffolding** (see [`docs/STATUS.md`](docs/STATUS.md))
> Research prototype for a college major project. **Not certified or intended for safety-critical use.**

Nothing described under "Quick start" exists yet: the commands are the target interface we are building towards. `docs/STATUS.md` says what actually works today.

---

## 1. What is this? (plain English)

AI assistants are starting to press real buttons: switch a relay, spin a motor, turn on a heater. A wrong command in a chat is harmless. A wrong command to a machine can break it or hurt someone.

TwoGates puts **two independent safety checkpoints** between an AI agent and a machine, both generated from **one written safety specification**:

1. **Gate 1 (host).** Every tool the AI uses runs inside a locked-down WebAssembly sandbox, and every command is checked against the rules (value ranges, switching rates, minimum delays, forbidden combinations) before it leaves the computer.
2. **Gate 2 (microcontroller).** A small ESP32 next to the machine re-checks the *same* rules in firmware and forces a safe state if the computer stops talking to it. It cannot be argued with, so even if the computer is compromised, the machine stays within its limits.

We then crash-test the system. Real LLMs control a small low-voltage rig while we measure how often they issue unsafe commands, which gate stops them, what happens when the link is corrupted or the host is attacked, and what the safety layers cost in latency and energy.

## 2. The problem

LLM agents are being wired to real hardware through tool protocols such as MCP. Sandboxing protects the computer, but it does not stop an in-range-but-dangerous command (a relay toggled 50 times a second, a motor reversed at full speed). The safeguards that exist today are mostly ad hoc and rarely measured. The questions we want to answer with evidence are:

- Does the safety rule have to live **outside the model**? (Models still make unsafe calls, even when told the limits.)
- Does it have to live **outside the host computer too**? (A compromised or buggy host can lie.)
- What does each layer **cost**, and what does each layer **catch** that the other does not?

## 3. The idea: two gates, one spec

```
 ┌───────────────────────┐
 │ LLM agent (off-board) │   cloud API / local model / scripted fake
 └──────────┬────────────┘
            │ MCP (tool calls)
 ┌──────────▼───────────────────────────────────────────────┐
 │ HOST  Raspberry Pi Zero 2 W (or a laptop in sim mode)    │
 │  MCP server → Wasm tool modules (sandbox, no ambient     │
 │  authority) → TIER-1 policy gate (Python)                │
 └────────────────────────────────────┬─────────────────────┘
                                      │ UART, COBS + CRC-16 frames, heartbeat
 ┌────────────────────────────────────▼─────────────────────┐
 │ ESP32-S3  TIER-2 interlock (portable C core + HAL)       │
 │  same rules, integer-only, safe state, e-stop, watchdog  │
 └────────┬───────────────┬────────────────┬────────────────┘
          ▼               ▼                ▼
        LEDs            relay          DC motor (low voltage)
```

One YAML safety spec → compiler → `host_policy.json` + `spec_tables.h` (firmware) + `tool_schemas.json` + `actuator_ids.h` + conformance test vectors + a spec hash. The host refuses to arm unless the firmware reports the same hash.

## 4. What we will show

| RQ | Question |
|---|---|
| RQ1 | How often do current LLMs emit unsafe actuator calls (out of range, too frequent, bad sequencing), including under prompt injection? |
| RQ2 | What does each layer catch, and what is caught only by the ESP32 (host compromise, link faults)? |
| RQ3 | What does Wasm isolation add beyond a policy gate (for example against a malicious third-party tool module), and what does the full stack cost in latency and energy? |

Planned contributions: (1) the spec language and compiler that targets both tiers; (2) an open-source reference implementation on hardware costing under ₹7,000; (3) a measured study with real LLM calls, fault injection and an independent oracle.

## 5. Guarantees and limits (short version)

We aim to demonstrate, not assume:
- **G1** Whatever bytes reach the ESP32, outputs never violate the spec's per-actuator rules.
- **G2** Outputs are safe at boot, and within about one heartbeat timeout after the host stops responding or the e-stop is pressed.
- **G3** Under nominal timing, anything Tier-1 accepts, Tier-2 also accepts (measured, with disagreements logged).
- **G4** A tool module can only reach the host functions its manifest grants; its CPU and memory are bounded.
- **G5** Host and firmware were built from the same spec (hash handshake).

Limits: someone (or something) operating *inside* the allowed envelope is not stopped; we do not check that the spec is right; no protection against physical tampering or power faults; no real-time guarantee on Linux; not certified. Full threat model: [`docs/BLUEPRINT.md`](docs/BLUEPRINT.md) §4.

## 6. Repository layout (planned)

```
twogates/
├── README.md  AGENTS.md  LICENSE  Makefile  pyproject.toml
├── docs/            PLAN.md  BLUEPRINT.md  STATUS.md  decisions/
├── spec/            schema.json  examples/rig_v1.yaml  tests/vectors.yaml
├── twogates/        spec/  host/  sim/  eval/  faults/  energy/      (Python package)
├── tools/           src/ (C → Wasm)  manifests/  build/
├── firmware/        components/tier2_core/  components/hal_esp32/  main/  logger/
├── hardware/        wiring.md  pinout.md  spend.csv
├── experiments/     scenarios/  prompts/  models.yaml  ANALYSIS_PLAN.md  (runs/ is git-ignored)
├── tests/           unit/  property/  integration/  hil/
└── .github/workflows/ci.yml
```

## 7. Hardware (build budget ≤ ₹7,000)

One node, bought in two stages. Likely prices in October 2026; re-check when you buy.

| Item | Stage | Likely ₹ |
|---|---|---|
| Raspberry Pi Zero 2 W (stock is patchy) | 1 | 2,000 |
| ESP32-S3-DevKitC-1 | 1 | 700 |
| microSD 32GB (or reuse) | 1 | 750 |
| 5V charger + micro-USB cable (or reuse) | 1 | 300 |
| Jumpers, breadboard, LEDs, button, header | 1 | 300 |
| INA219 power monitor | 2 | 300 |
| TB6612FNG driver + small geared motor | 2 | 340 |
| 1-channel relay module (optional) | 2 | 90 |
| Shipping | | 200 |
| **Total** | | **≈ 4,980** |

A buffer of about ₹750 stays unspent for a replacement ESP32; API credits are about ₹100–800. Details, power architecture and the proposed pin map: [`docs/BLUEPRINT.md`](docs/BLUEPRINT.md) §13.

**Rig safety rules**
- Low voltage only (≤ 6 V). Never mains.
- The relay switches only low-voltage loads (LED, buzzer).
- The motor has its own supply, with a common ground; never power it from the Pi's 5V pin.
- Check continuity and shorts with a multimeter before every power-up.
- Keep the e-stop within reach; never leave LLM runs on hardware unattended.

## 8. Quick start (target interface)

### A. Simulation: no hardware, no cost

```bash
git clone <repo-url> twogates && cd twogates
make setup        # virtualenv + pinned dependencies
make spec         # compile spec/examples/rig_v1.yaml → build/generated/
make tools        # build and sign the Wasm tool modules (needs clang + lld)
make test         # unit, property and SIM integration tests
make sim-demo     # scripted agent end-to-end in simulation
```

### B. Hardware

```bash
make fw-build                         # ESP-IDF build of firmware/ (needs ESP-IDF 5.x)
make fw-flash PORT=/dev/ttyUSB0      # flash the ESP32 over USB (radios are disabled; no OTA)
# on the Pi (64-bit Raspberry Pi OS Lite, UART enabled: see BLUEPRINT §13.3):
python -m twogates.host.cli serve --port /dev/serial0     # planned CLI; names may change
make test-hil                         # hardware-in-the-loop smoke tests (rig connected)
```

### C. LLM evaluation

```bash
make eval-pilot                                        # free: scripted fake LLM, drives the whole pipeline
ALLOW_PAID=1 BUDGET_INR=500 make eval-record MODEL=<name-in-experiments/models.yaml>
make eval-replay RUN=<run-id>                          # replay recorded calls under Tier-1 only / Tier-2 only / both
make report                                            # tables and figures from runs/
```

Paid calls need `ALLOW_PAID=1` and a budget cap. API keys go in a git-ignored `.env` (see `.env.example`).

## 9. The safety spec in 60 seconds

```yaml
spec_version: 1
heartbeat: {period_ms: 50, timeout_ms: 250}
actuators:
  relay1:
    kind: digital
    pin: 6
    safe: 0
    rules: {min_off_ms: 1000, max_activations: {count: 2, window_ms: 10000}, max_on_ms: 5000}
  motor1:
    kind: signed_pct
    pins: {in1: 7, in2: 15, pwm: 16, stby: 8}
    safe: 0
    rules: {range: [-60, 60], reverse_dead_time_ms: 500, min_off_ms: 500, max_on_ms: 8000,
            max_activations: {count: 6, window_ms: 10000}}
mutex: [[motor1, relay1]]
```

| Rule | Meaning |
|---|---|
| `range` | allowed set-points |
| `min_off_ms` | minimum rest before the next activation |
| `max_activations` | at most N activations per sliding window |
| `max_on_ms` | after this long non-safe, the firmware forces the safe value |
| `reverse_dead_time_ms` | a motor must stop and wait before reversing; direct flips are refused |
| `mutex` | members cannot be active together |

Commands that move *toward* the safe value are always accepted. Denials never change state. Retries are safe because every command is an absolute set-point. Semantics and reason codes: [`docs/BLUEPRINT.md`](docs/BLUEPRINT.md) §5.

## 10. Writing a tool module (Wasm, in C)

```c
// tools/src/led.c: freestanding C, no libc, no WASI
#include "actuator_ids.h"      // generated: ACT_LED1, ACT_LED2, ...

__attribute__((import_module("hw"), import_name("actuate")))
extern int hw_actuate(int actuator_id, int value);   // returns 0 or -reason_code

__attribute__((export_name("tool_set_led")))
int tool_set_led(int led_index, int on) {
    if (led_index < 0 || led_index > 1) return -3;    // UNKNOWN_ACTUATOR
    int id = (led_index == 0) ? ACT_LED1 : ACT_LED2;
    return hw_actuate(id, on ? 1 : 0);
}
```

```toml
# tools/manifests/led.toml
name = "led"
exports = ["tool_set_led"]
capabilities = ["actuator:led1", "actuator:led2"]
max_memory_pages = 4
max_call_ms = 50
```

A module that imports anything other than what its manifest grants fails to instantiate. Modules are signed (Ed25519) and verified before they are compiled. Build flags live in `tools/Makefile` (roughly `clang --target=wasm32 -nostdlib -Wl,--no-entry`; adjust as needed).

## 11. Evaluation at a glance

| ID | What | Where |
|---|---|---|
| E1 | Unsafe-call study: 26 scenarios × 4 models × 5 runs, recorded with no defenses, replayed under Tier-1 only / Tier-2 only / both | SIM |
| E2 | Closed-loop impact of enforcement on task success | SIM |
| E3 | Fault injection: bit flips, truncation, drops, heartbeat loss, a raw-frame attacker | hardware |
| E4 | Sandbox ablation: eight malicious modules, Wasm vs native | hardware / SIM |
| E5 | Latency breakdown per layer | Pi |
| E6 | Energy per layer (a scoped extra, first to be cut if we slip) | Pi + INA219 |

The ground truth comes from an **independent oracle** that checks what the pins actually did against the spec. It shares no code with Tier-1. Statistics: Wilson intervals for proportions, bootstrap for latencies, McNemar for paired comparisons. See [`docs/BLUEPRINT.md`](docs/BLUEPRINT.md) §10–12.

## 12. Roadmap

- [ ] **G0** (1 Nov 2026): spikes S1–S5 pass: Wasm on the Pi, GPIO, UART + watchdog, LLM pilot
- [ ] **G1** (29 Nov): end-to-end simulation; spec and protocol frozen
- [ ] **G2** (27 Dec): hardware demo, fault-injection smoke test
- [ ] **G3** (24 Jan 2027): Paper A draft; first E1 results
- [ ] **G4** (28 Feb): full study; results frozen
- [ ] **G5** (4 Apr): report, demo, release `v1.0`

Dates assume Week 1 starts 19 Oct 2026. Full plan: [`docs/PLAN.md`](docs/PLAN.md).

## 13. Related work (what exists, what we add)

Preliminary web search, October 2026. A Google Scholar / IEEE Xplore / ACM DL pass is still to do; verify every reference before citing it.

- **Wassette** (Microsoft, 2025): runs Wasm components as MCP tools on Wasmtime with deny-by-default permissions (storage, network, environment variables); no hardware or actuator permissions; early-stage when we looked. <https://github.com/microsoft/wassette>
- **Aerogel** (Liu, Garcia, Srivastava, IEEE/ACM SEC 2021): fine-grained access control for Wasm on bare-metal IoT peripherals; no LLM agents.
- **Van Kenhove et al.** (IEEE/IFIP NOMS 2025): WASI interfaces for I²C and USB with drivers in Wasm; WASI I²C was at proposal Phase 2, GPIO/SPI at Phase 1 when we looked.
- **AgentSpec** (2025): a DSL for runtime enforcement of LLM-agent actions, including embodied agents and driving.
- **RoboGuard** (IEEE RA-L, accepted Feb 2026): temporal-logic guardrails for LLM-enabled robots.
- **"Physical Agentic AI"** (arXiv 2608.22657, 2026): a deterministic orchestrator between an LLM planner and robot actuation; informed planners still dispatched 23–29% of faulted steps, while the deterministic layer cut false dispatch to 0% in their testbeds.
- **Hobby and open-source ESP32 + MCP projects**: Thingwire (allowlists, rate limits, deadman switch), Animus (limits compiled into firmware, auto-off), AIoT Sentinel (typed tools, interlocks, hash-chained audit trail); Espressif's ESP-Claw (2026).
- **Runtime assurance / Simplex**: the classic untrusted-controller, trusted-monitor pattern we apply to LLM tool calls.
- **Wasm on embedded hardware, benchmarks:** Has et al. (2025: WAMR and wasm3 vs native on three MCUs), treVM (2026), a Raspberry Pi 3B energy study (EASE 2023). Because of these, runtime-vs-native speed and energy is *not* our novelty.

**What we add:** one spec compiled to both enforcement points (host gate and independent MCU interlock), capability-scoped Wasm tools, and a measured, oracle-checked study with real LLM calls and fault injection on cheap hardware. We found no paper doing exactly this combination; that is a search result, not a proof.

## 14. Safety, ethics and security notes

- Rig rules are in §7. Spec limits are conservative, so violating one in a test must never damage hardware: do not test destructive limits.
- Never commit API keys. Paid LLM calls need an explicit flag and a budget cap.
- The ESP32 has radios disabled and no OTA; reflashing requires physical USB access. Physical tampering is out of scope.
- No personal data is collected. LLM prompts and logs contain only synthetic tasks.

## 15. Contributing and AI assistants

Read [`AGENTS.md`](AGENTS.md) first. It is the working agreement for humans and AI assistants: session start/end checklists, invariants, commands and conventions. Record every design decision in the ADR table (BLUEPRINT §15). Update [`docs/STATUS.md`](docs/STATUS.md) at the end of each session.

## 16. Team

| Lane | Owner | Backup |
|---|---|---|
| A: Host & Spec | [name] | [name] |
| B: Firmware | [name] | [name] |
| C: Evaluation | [name] | [name] |
| D: Hardware, faults, energy, papers | [name] | [name] |

Supervisor: [name]. Institution: [college, department, year].

## 17. License and citation

License: Apache-2.0 (proposed; confirm against the college's IP policy before the first public release).

```bibtex
@misc{twogates2027,
  title  = {Two Gates, One Spec: Hardware-Enforced Safety for LLM Agents That Control Physical Devices},
  author = {[authors]},
  year   = {2027},
  note   = {Project repository, [url]}
}
```
