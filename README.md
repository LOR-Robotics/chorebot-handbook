# ChoreBot handbook

> Public entrypoint. **GitHub Wiki tab** will mirror this once seeded (private `chorebot-docs` cannot enable Wikis on GitHub Free org plans).

- Wiki (once seeded): https://github.com/LOR-Robotics/chorebot-handbook/wiki
- Private deep docs: https://github.com/LOR-Robotics/chorebot-docs

# ChoreBot wiki home

**Org:** LOR-Robotics · **Docs repo:** `chorebot-docs`  
**Stack (locked):** ROS 2 Jazzy · Gazebo Harmonic · Nav2 · LLM → skill JSON → executor  
**G0 chore (locked):** dog-poop scoop (`scoop_poop`)  
**Updated:** 25 Sep 2026 (BST) · Systems

Welcome. Short sentences. Terms defined once.

---

## Getting started

1. Read the org plan: which repo does what (`chorebot-sim`, `chorebot-skills`, `chorebot-firmware`, `chorebot-docs`).
2. Skim process: [PROCESS](PROCESS) — need → requirement → design → build → verify → evidence.
3. Pick a lane (Tech, Sim, CI, Systems, Perception, Nav, **Mobile**) and an Issue.
4. Develop using [DEV_AND_SIM](DEV_AND_SIM): Ubuntu/WSL or Docker; never skip the Issue for non-trivial work.
5. Open a PR. Answer: purpose, what changed, how tested, residual risk.

**Local mirror:** `/workspace/chorebot/ros_ws/` plus architecture docs beside it.

```text
chorebot-sim      → worlds, URDF, bringup (G0 clip path)
chorebot-skills   → schemas, executors, llm_bridge
chorebot-firmware → empty until G1
chorebot-docs     → this handbook, ADRs, wiki
```

---

## Gates G0–G2

| Gate | Proof | Unlocks |
|------|--------|---------|
| **G0 — Virtual** | 2–3 min sim clip: natural-language chore → plan → execute → “done” | Pitch meetings |
| **G1 — Barn** | Same control stack on a rolling chassis (teleop + one LLM chore) | Hardware credibility |
| **G2 — Field** | One outdoor chore on Ryan’s acres, logged, recoverable fail | Customer-testable MVP |

Ship **G0 first**. Do not wait on company formation or metal for G0.

### G0 demo definition of done

1. Utterance → inspectable `scoop_poop` skill JSON (mock LLM OK).
2. Execute in Gazebo: approach `poop_marker` → scoop stub → bin → dump stub → home → “done”.
3. Artefacts: **2–3 min clip** linked; **`skills-unit` green**; **headless sim smoke** green when ready; evidence on Issues (see matrix).
4. Hard rule: LLM path never publishes `/cmd_vel`.

**Verification matrix:** [`docs/verification/g0-scoop-poop-matrix.md`](docs/verification/g0-scoop-poop-matrix.md)

### App lane (parallel — not a G0 blocker)

**ChoreBot Mobile** owns the **operator mobile app** (iPhone first, TestFlight). The G0 demo does **not** require the app; Mobile is a parallel operator-UX track. Systems ADR: [`docs/adr/0002-operator-mobile-app.md`](docs/adr/0002-operator-mobile-app.md).

### Hardware applicability (G1 pack — does not block G0)

**ChoreBot Hardware** lane: keep G0 software metal-applicable. Pack index: [`docs#26`](https://github.com/LOR-Robotics/chorebot-docs/issues/26). Soft-note: G0 remains sim.

| Doc / Issue | Role |
|-------------|------|
| [G0↔G1 software↔hardware interface](docs/interfaces/g0-g1-software-hardware.md) | Tech contract Hardware must honour (frames, skill JSON, `/chorebot/estop`, compute class) |
| [ADR 0005](docs/adr/0005-g1-compute-and-connectivity.md) | **Accepted** — J401 + Orin Nano 8GB (DK OK); 2× ODrive S1; traction 19–54 V; Wi‑Fi → Tailscale → LTE |
| [`hardware/`](hardware/) briefs ([PR #38](https://github.com/LOR-Robotics/chorebot-docs/pull/38)) | G1 architecture, compute, connectivity, Ryan ask list |
| [ODrive diagnostics gap](hardware/odrive-diagnostics-gap.md) ([PR #43](https://github.com/LOR-Robotics/chorebot-docs/pull/43)) | **G1 unsupervised barn gated** on side-channel `/diagnostics` (ADR 0005 SKUs unchanged) |
| HW-COMPUTE-001 [#32](https://github.com/LOR-Robotics/chorebot-docs/issues/32) · HW-CONN-001 [#33](https://github.com/LOR-Robotics/chorebot-docs/issues/33) · HW-ESTOP-001 [#34](https://github.com/LOR-Robotics/chorebot-docs/issues/34) · HW-GEO-001 [#35](https://github.com/LOR-Robotics/chorebot-docs/issues/35) · HW-POWER-001 [#36](https://github.com/LOR-Robotics/chorebot-docs/issues/36) · HW-SCOOP-ICD-001 [#37](https://github.com/LOR-Robotics/chorebot-docs/issues/37) | REQ pack |

### G0 status (locate path)

| Item | Where | Role |
|------|--------|------|
| SIM-001 | [`chorebot-sim#1`](https://github.com/LOR-Robotics/chorebot-sim/issues/1) | Sim bringup / smoke / clip track |
| PERC-001 | [`chorebot-sim#2`](https://github.com/LOR-Robotics/chorebot-sim/issues/2) · [docs#11](https://github.com/LOR-Robotics/chorebot-docs/issues/11) | G0 critical: `poop_marker` → TF `poop` for `locate` |
| PERC-002 | [`chorebot-sim#3`](https://github.com/LOR-Robotics/chorebot-sim/issues/3) | Optional RGB clip candy — **non-blocking** |
| PERC-003 | [`chorebot-docs#1`](https://github.com/LOR-Robotics/chorebot-docs/issues/1) | **Non-goal for G0:** field CV / real poop detector — **not a CI gate** |
| SAFE-001 | [`docs#8`](https://github.com/LOR-Robotics/chorebot-docs/issues/8) | LLM never `/cmd_vel`; `skills-unit` |
| G0-001 | [`docs#9`](https://github.com/LOR-Robotics/chorebot-docs/issues/9) | End-to-end clip DoD |

G0 perception = ground-truth / model pose in sim. Field CV is post-G0 (PERC-003). No perception required check until PERC-001 smoke exists.

---

## How we test

- **Every PR (`chorebot-skills`):** schema + unit tests **without** Gazebo (`skills-unit`).
- **Every PR (`chorebot-sim`):** Docker build + **headless** Gazebo launch smoke (ADR 0004).
- **Nightly:** longer scenarios, bags, screenshots when stable.
- **Evidence:** logs, bags, screenshots, JUnit — link them on the Issue/PR ([verification guide](docs/verification/README.md)).

Hard rule: the LLM path never publishes `/cmd_vel`. E-stop and geofence sit above the skill executor.

---

## Repos at a glance

| Repo | One-liner |
|------|-----------|
| `chorebot-sim` | Robot model + yard world + bringup |
| `chorebot-skills` | What the LLM is allowed to ask the robot to do |
| `chorebot-firmware` | Placeholder for MCU / e-stop hardware (G1) |
| `chorebot-docs` | Process, ADRs, wiki, verification |

Optional later: `chorebot-fleet`. Mobile app repo TBD (Tech ACK; ADR 0002). No other product robot repos without Tech.

---

## ADRs

| ADR | Status |
|-----|--------|
| [0001 Record ADRs](docs/adr/0001-record-architecture-decisions.md) | Accepted |
| [0002 Operator mobile app](docs/adr/0002-operator-mobile-app.md) | Proposed |
| [0003 Skill transport WS/HTTP](docs/adr/0003-skill-transport-ws-http.md) | Proposed |
| [0004 Docker sim runtime](docs/adr/0004-docker-sim-runtime.md) | Accepted |
| [0005 G1 compute + connectivity](docs/adr/0005-g1-compute-and-connectivity.md) | Accepted |

Index: [`docs/adr/README.md`](docs/adr/README.md).

---

## Glossary

| Term | Meaning |
|------|---------|
| **ADR** | Architecture Decision Record — short note of a choice and why |
| **CI** | Continuous integration — automated build/test on GitHub Actions |
| **Executor** | Code that runs a skill step-by-step |
| **LLM** | Large language model — planner that emits skill JSON |
| **Nav2** | ROS 2 navigation stack |
| **PR** | Pull request |
| **Skill** | Named chore contract (JSON schema) — e.g. `scoop_poop` |
| **Smoke test** | Short “does it start?” check |
| **Stub** | Fake stand-in until real hardware or Nav2 is wired |

---

## Key doc index

- Architecture: `tech-architecture-v0.md`
- G0 skill contract: `g0-scoop-poop-skill.md`
- G0 status: `G0-STATUS.md`
- G0 verification matrix: `docs/verification/g0-scoop-poop-matrix.md`
- Process: `PROCESS.md`
- Develop & sim: `DEV_AND_SIM.md`
- Org / repo map: `ORG_PLAN.md`
- Investor diligence: `investor/` (do not casually edit)

---

*Inspired by aerospace discipline. Not formal certification. Keep Issues honest and evidence findable.*
