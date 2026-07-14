# Federated-AI-Commons-Model

An agent-based simulation (Mesa) of resource allocation under scarcity vs.
abundance, crisis governance, citizen agency (tested against Ostrom's
commons-governance principles), and — as of this revision — a full second
economic layer: a Raw Material Conversion economy with a structurally blind
Ledger, community-level barter, and ecological cost feedback on both
material-delivery systems.

**Read `FINDINGS.md` before tuning parameters blind.** Several defaults only
make sense in light of bugs and calibration issues already found and fixed
— reusing intuitive-looking values without that context will likely
reproduce mistakes already made once. This file is the map; `FINDINGS.md`
is the full experimental record, including every bug caught and how.

**Run `pytest test_federated_ai_commons_model.py` before trusting any change.**
A persisted regression suite now exists specifically because this project
hit real, avoidable mistakes from *not* having one — including once
starting to build a duplicate of a feature that already existed a few
hundred lines away.

---

## Quick start

```python
!pip install -q mesa pytest   # if running in Colab

from federated_ai_commons_model import FederatedAICommonsModel

m = FederatedAICommonsModel(200, 8, seed=1, coupled_governance=True,
                            rehabilitation_enabled=True, graduation_enabled=True)
for _ in range(300):
    m.step()

df = m.datacollector.get_model_vars_dataframe()
df.tail(10)
```

Pasting the file into a notebook cell instead of uploading it? Use
`%%writefile federated_ai_commons_model.py` as the cell's first line and
run it in its own cell before importing — 2,300+ lines pasted as
directly-executed code is fragile.

---

## Architecture

- **`Citizen`** — strategy, `resources`, `reputation`, `contribution`,
  `affinities`, `relapse_count`, `experience` (used by the peer
  rehabilitation mechanic). Pays `effort_cost = contribution ** 2` unless
  `post_labor_economy_enabled`.
- **`CommunityNode`** — policy, `care_load`, `crisis_severity`,
  `emergency_declared`, `local_reserve`/`trust` (federated mode),
  `distance`/`history` (latency mode), and — new this revision —
  `raw_materials`, `local_particulars`, `production_strategy`,
  `assimilated_by`/`consecutive_empty_steps` (ghost-town lifecycle).
- **`SystemLedger`** — two genuinely separate roles: `get_system_status()`
  for `informed_migration` (unrelated to allocation), and
  `ship_raw_materials()`, which is **structurally blind** to everything
  except `raw_materials` levels — verified by direct source inspection in
  the test suite, not just by docstring claim.
- **`FederatedAICommonsModel`** — ~132 opt-in parameters across 28
  independently-toggleable subsystems, all defaulted to preserve the
  original unmodified model's behavior.

---

## What's new this revision: the Raw Material Conversion economy

A second, parallel economic layer, entirely separate from the flat
automation dividend (`post_scarcity_enabled`) — confirmed by direct code
inspection that neither system touches the other's mechanics except
through one deliberate shared bridge (ecological cost, below).

- **Two-tier structure**: a "blind" global Ledger ships raw materials
  based only on `raw_materials` levels (`ship_raw_materials`), each
  community sovereignly converts its own supply into `local_particulars`
  via an evolvable `production_strategy`, and communities barter
  particulars directly with each other — genuine bilateral trade subject
  to the real "double coincidence of wants" problem (Jevons), not a
  central clearinghouse.
- **`particulars_consumption_enabled`** closes a real gap: for a while,
  none of this reached individual citizens at all. Fixed with a
  diminishing-returns (not flat) value function — confirmed necessary
  directly: the flat version made *not* trading a strictly dominant
  strategy for citizen welfare, tested and reversed.
- **`citizen_philanthropy_enabled`** — individual resources for reputation,
  no currency. Found and fixed a real reputation-laundering exploit where
  free-riding strategies could convert unearned dividend money into
  reputation; now gated on genuine contribution.
- **`ledger_ecological_cost_enabled`** — the one deliberate bridge between
  the dividend and the raw-material economy: both get throttled by the
  same accumulating extraction-damage signal. Tested directly (see
  `FINDINGS.md`): default parameters are aggressive enough to collapse
  the dividend to near-zero for everyone, which is itself a real finding,
  not just a calibration note.
- **Ghost towns handled realistically, in two stages**: immediate,
  cost-free wind-down (production pauses, nothing lost, resumes instantly
  on repopulation) followed only by delayed assimilation to a trusted
  neighbor if abandonment is genuinely sustained (30+ steps) — not a
  single harsh event.

## Also resolved this revision: both long-standing dead-code items

- **`peer_rehabilitation_enabled`** — the experience-based individual
  rehabilitation mechanic, dormant for many revisions, is now a real,
  tested, mutually-exclusive alternative to the community-investment
  mechanic. Finding: both converge to the same long-run flourishing
  ceiling, but peer rehabilitation gets there measurably faster, via a
  genuine compounding-skill dynamic the original mechanic can't produce.
- **`socialize_new_agent`** — confirmed permanently unreachable (no birth
  mechanism exists anywhere in the model) and deleted outright.

---

## Parameter reference (condensed — see `FINDINGS.md` for full detail)

| category | key params | default |
|---|---|---|
| Core | `N_citizens`, `N_communities`, `seed` | required / `None` |
| Coupled / Gradual Crisis Governance | `coupled_governance`, `gradual_crisis_response_enabled` | `False` |
| Rehabilitation | `rehabilitation_enabled`, `graduation_enabled`, **`peer_rehabilitation_enabled`** | `False` |
| Post-Scarcity/Automation | `post_scarcity_enabled`, `automation_level_enabled`, `automation_care_absorption` | `False` / `0.0` |
| Post-Labor Economy | `post_labor_economy_enabled` | `False` |
| Allocation (centralized) | `flat_allocation`, `population_weighted_allocation`, `memory_decay_enabled` | `False` |
| Citizen Agency | `caregiver_choice_enabled`, `policy_voting_enabled`, `relapse_history_penalty` | `False` |
| Contested Sites / Ecology | `contested_sites_enabled`, `ecological_impact_enabled` | `False` |
| **Raw Material Economy** | `raw_material_economy_enabled`, `particulars_consumption_enabled`, `strategy_fitness_metric` | `False` / `"total_particulars"` |
| **Ledger Ecological Cost** | `ledger_ecological_cost_enabled`, `ecological_cost_per_unit` | `False` |
| **Citizen Philanthropy** | `citizen_philanthropy_enabled`, `philanthropy_min_contribution` | `False`, `0.3` |
| **Community Assimilation** | `abandonment_threshold` | `30` |
| Federated Commons | `federated_mode`, `local_reserve_target`, `trust_initial` | `False` |
| Communication Latency | `communication_latency_enabled`, `min_distance`, `max_distance` | `False` |
| Privacy | `privacy_enabled` | `False` |
| Automated Provisioning | `automated_provisioning_enabled`, `provisioning_detects_inequality`, `individual_floor_enabled` | `False` |
| Debug | `debug_assertions`, `log_allocation_details` | `False` |

**Bold rows are new this revision.**

---

## The short version of what's been found, cumulative

- The default labor economy is structurally broken (~150x cost/payout
  mismatch) unless `post_scarcity_enabled` or properly-calibrated
  `automation_level_enabled` is on. **Still no runtime warning for this —
  open item.**
- `coupled_governance` produces *permanent* emergency rule by default;
  `gradual_crisis_response_enabled` fixes the *legitimacy* of how it's
  invoked (consent vs. force), not the underlying frequency — that still
  needs `automation_care_absorption`.
- The project's founding philosophical thesis reproduces mechanically:
  once labor decouples from survival, resource inequality collapses while
  reputation inequality persists as the durable axis of status.
- **Citizen agency is necessary but not sufficient for good governance** —
  democratic voting can and does select the most exploitable policy when
  free-riders form a majority.
- **Every reward mechanism built this project needed adversarial testing
  before it could be trusted** — philanthropy laundered dividend money
  into reputation until gated; flat particulars valuation made trading
  strictly worse than isolation until fixed; the "Big Push" turned out to
  be optional once the environment was properly fixed first.
- A persisted test suite now exists specifically because ad-hoc,
  one-off regression scripts already caused two real mistakes this
  project — a near-duplicated feature and a false regression alarm from
  comparing incompatible sample sizes.

**For the full experimental record, the Ostrom-principle analysis, every
bug and how it was found, and current known issues, see `FINDINGS.md`.**
