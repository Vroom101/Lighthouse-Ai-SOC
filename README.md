<div align="center">

```
 ██▓     ██▓  ▄████  ██░ ██ ▄▄▄█████▓ ██░ ██  ▒█████   █    ██   ██████ ▓█████
▓██▒    ▓██▒ ██▒ ▀█▒▓██░ ██▒▓  ██▒ ▓▒▓██░ ██▒▒██▒  ██▒ ██  ▓██▒▒██    ▒ ▓█   ▀
▒██░    ▒██▒▒██░▄▄▄░▒██▀▀██░▒ ▓██░ ▒░▒██▀▀██░▒██░  ██▒▓██  ▒██░░ ▓██▄   ▒███
▒██░    ░██░░▓█  ██▓░▓█ ░██ ░ ▓██▓ ░ ░▓█ ░██ ▒██   ██░▓▓█  ░██░  ▒   ██▒▒▓█  ▄
░██████▒░██░░▒▓███▀▒░▓█▒░██▓  ▒██▒ ░ ░▓█▒░██▓░ ████▓▒░▒▒█████▓ ▒██████▒▒░▒████▒
░ ▒░▓  ░░▓   ░▒   ▒  ▒ ░░▒░▒  ▒ ░░    ▒ ░░▒░▒░ ▒░▒░▒░ ░▒▓▒ ▒ ▒ ▒ ▒▓▒ ▒ ░░░ ▒░ ░
░ ░ ▒  ░ ▒ ░  ░   ░  ▒ ░▒░ ░    ░     ▒ ░▒░ ░  ░ ▒ ▒░ ░░▒░ ░ ░ ░ ░▒  ░ ░ ░ ░  ░
  ░ ░    ▒ ░░ ░   ░  ░  ░░ ░  ░       ░  ░░ ░░ ░ ░ ▒   ░░░ ░ ░ ░  ░  ░     ░
    ░  ░ ░        ░  ░  ░  ░          ░  ░  ░    ░ ░     ░           ░     ░  ░
```

**AI-Powered Security Operations Centre**

A real-time **hybrid network intrusion detection system** that fuses three independent
detection layers — supervised ML on flow telemetry, an O(1) volumetric rate aggregator,
and Suricata signatures — and correlates them with Wazuh host alerts, scoring every
event on a NIST SP 800-30 risk scale and acting on it autonomously.

[![Python](https://img.shields.io/badge/Python-3.11+-3776AB?style=flat-square&logo=python&logoColor=white)](https://python.org)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.111-009688?style=flat-square&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com)
[![React](https://img.shields.io/badge/React-18-61DAFB?style=flat-square&logo=react&logoColor=black)](https://react.dev)
[![Zeek](https://img.shields.io/badge/Zeek-primary_sensor-EF7B26?style=flat-square)](https://zeek.org)
[![Suricata](https://img.shields.io/badge/Suricata-signatures-EE3424?style=flat-square&logo=suricata&logoColor=white)](https://suricata.io)
[![ML](https://img.shields.io/badge/CIC_2017-macro_F1_0.99-FF6600?style=flat-square)](https://xgboost.readthedocs.io)
[![Wazuh](https://img.shields.io/badge/Wazuh-host_fusion-3578E5?style=flat-square&logo=wazuh&logoColor=white)](https://wazuh.com)

</div>

---

## What Is Lighthouse?

Per-flow ML alone cannot catch every attack — a single SYN flood packet is statistically
identical whether it comes from an attacker or a benign client; the signal lives in the
*aggregate rate*, not any one flow (Sommer & Paxson, IEEE S&P 2010). Lighthouse therefore
runs the **three-layer hybrid** the research endorses, with each layer covering the others'
blind spots:

```
 Layer 1 — Rate aggregator    → volumetric truth (SYN flood, port scan, UDP flood)  [O(1), drift-proof]
 Layer 2 — ML flow models     → structured / app-layer attacks (CIC-IDS-2017)       [XGBoost + LightGBM]
 Layer 3 — Suricata signatures → known IOCs / thresholded floods                     [community-id join]
 +        Wazuh host alerts     → host-side context (auth brute force, FIM, process)  [src_ip correlation]
```

Zeek is the **primary sensor** (rich, reliable flow records); Suricata runs alongside it
purely for signatures, joined back to Zeek flows by **community-id**. Every event is risk-
scored 0–100, explained in natural language by an LLM, and — above the auto-block threshold —
actioned automatically (IP block / host isolation), surfaced live on a Terminal Noir React
dashboard over WebSocket.

```
 Network Traffic                                              SOC Analyst
 ┌──────────┐   conn.log    ┌─────────────┐   WebSocket    ┌───────────────┐
 │   ZEEK   │ ─────────────▶│ Lighthouse  │ ─────────────▶ │               │
 │ (primary)│               │  Backend    │                │  ██ CRITICAL  │
 └──────────┘               │             │    REST API    │  ▓▓ SUSPICIOUS│
 ┌──────────┐   eve.json    │ Rate Aggr.  │ ◀──────────── │  ░░ Auto-blk  │
 │ Suricata │ ─────────────▶│ CIC ML      │               │               │
 │  (sigs)  │  community-id │ Risk Score  │  Block / ISO  │  [Block IP]   │
 └──────────┘               │ LLM Expl.   │ ─────────────▶│  [Isolate]    │
 ┌──────────┐   host alert  │ SQLite DB   │               │  [Dismiss]    │
 │  Wazuh   │ ─────────────▶│             │               │               │
 └──────────┘   src_ip corr └─────────────┘               └───────────────┘
```

---

## Dashboard Preview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ 🛡 LIGHTHOUSE  SOC    47 Total Today    12 Critical ●   23 Suspicious ●     │
│                       5 Auto-Blocked 🛡                              ● LIVE │
├──────────┬────────────────────┬──────────────────────────────┬───────┬──────┤
│ CRITICAL │ DDoS               │ High-rate SYN flood from     │ Conf. │      │
│  2s ago  │ 192.168.10.156→web │ 192.168.10.156 targeting port│ ████░ │ AUTO │
│          │                    │ 80 with 4,800 packets/s.     │  97%  │BLOCK │
├──────────┼────────────────────┼──────────────────────────────┼───────┼──────┤
│ CRITICAL │ PortScan           │ SYN-only flows targeting     │ Conf. │[Blk] │
│  9s ago  │ 10.0.0.199 → web   │ 1,200 sequential ports.     │ ████░ │[Iso] │
│          │                    │ Zero bytes returned.         │  91%  │[Dis] │
├──────────┼────────────────────┼──────────────────────────────┼───────┼──────┤
│SUSPICIOUS│ Brute Force [Wazuh]│ 5712 — 47 failed SSH logins  │ Conf. │[Blk] │
│  16s ago │ 192.168.10.156→:22 │ on agent 001. Host+network   │ ███░░ │[Iso] │
│          │                    │ correlated (+10 risk).       │  78%  │[Dis] │
├──────────┼────────────────────┼──────────────────────────────┼───────┼──────┤
│SUSPICIOUS│ DoS                │ Sustained high-byte flows    │ Conf. │[Blk] │
│  23s ago │ 185.220.101.5→:80  │ (420KB avg). Hulk variant.  │ ██░░░ │[Iso] │
│          │                    │                              │  65%  │[Dis] │
└──────────┴────────────────────┴──────────────────────────────┴───────┴──────┘
```

**Alert Detail Drawer** (opens on click):
```
                                    ┌──────────────────────────────────┐
                                    │ [CRITICAL]  [Blocked]        ✕   │
                                    │ DDoS                             │
                                    ├──────────────────────────────────┤
                                    │ ALERT METADATA                   │
                                    │ ID          a1b2c3d4e5f6         │
                                    │ Timestamp   2026-06-14 14:23:01  │
                                    │ Source IP   192.168.10.156       │
                                    │ Rule Level  13                   │
                                    │ Community   1:wCb3qT4k...         │
                                    ├──────────────────────────────────┤
                                    │ DETECTION LAYERS                 │
                                    │ Rate aggr.  ██████████████  DDoS │
                                    │ CIC ML      ████████████░░  97%  │
                                    │ Suricata    LIGHTHOUSE SYN Flood │
                                    ├──────────────────────────────────┤
                                    │ RISK (NIST 800-30)               │
                                    │ Likelihood × Impact × Temporal   │
                                    │ Overall     ████████████░░  92   │
                                    ├──────────────────────────────────┤
                                    │ AI ANALYSIS                      │
                                    │ │ High-rate SYN flood from       │
                                    │ │ 192.168.10.156 targeting :80   │
                                    │ │ at 4,800 pkts/s. Rate layer +  │
                                    │ │ Suricata signature agree.      │
                                    ├──────────────────────────────────┤
                                    │ [  Block IP  ] [Isolate] [Dismiss]│
                                    └──────────────────────────────────┘
```

---

## ML Model Performance

The ML layer is a **two-stage CIC-IDS-2017 pipeline** trained on **~2.8M labeled flows**
(2,830,743 rows across the Monday–Friday captures):

- **Stage 1** — XGBoost binary classifier (BENIGN vs Attack), `scale_pos_weight` for imbalance.
- **Stage 2** — LightGBM 6-class attack-family classifier, SMOTE on minority families.

Critically, both stages train on features computed by the **same shared module**
([`detection/flow_features.py`](detection/flow_features.py)) that the live Zeek/Suricata
bridge uses at serving time — structurally eliminating training-serving skew. Only features
a live flow record can faithfully reproduce are included (no fabricated inter-arrival times),
which is what raised live detection from near-zero to the numbers below.

### CIC-IDS-2017 — macro F1: **0.992** (test) / **0.992** (disjoint validation)

Results on the held-out 20% test set. Benign false-positive rate: **0.46%** (208 / 45,462).

| Attack Family | Recall | Notes |
|---------------|:------:|-------|
| PortScan | **99.97%** | Near-perfect |
| DDoS | **99.8%** | Near-perfect |
| DoS | **99.6%** | Near-perfect |
| Brute Force | **99.3%** | Excellent |
| BENIGN | **99.5%** | 0.46% false-positive rate |
| Bot | 89.1% | Good (limited training samples) |
| Web Attack | 14.0% | **Known limitation — see below** |

**Stage 1 ROC (Binary — BENIGN vs Attack):**

![CIC ROC Stage 1](reports/model_evaluation/cic_roc_stage1.png)

**Stage 2 ROC (Attack families only — no BENIGN):**

![CIC ROC Stage 2](reports/model_evaluation/cic_roc_stage2.png)

**Confusion Matrix:**

![CIC Confusion Matrix](reports/model_evaluation/cic_confusion_matrix.png)

**Classification Report:**

![CIC Classification Report](reports/model_evaluation/cic_classification_report.png)

**Cross-Validation Metrics (SMOTE + StandardScaler fitted inside each fold — no leakage):**

![CIC Validation Metrics](reports/model_evaluation/cic_validation_metrics.png)

---

### The Web Attack Problem (honest limitation)

Web attacks (SQLi / XSS / brute-force) are the one family the flow-only model misses —
**14% recall** in the main pipeline. SHAP feature-discovery over all 69 CIC columns showed
why: the dominant Web-Attack discriminators are **inter-arrival-time features** (Fwd IAT Max,
Flow IAT Std, …) that Suricata/Zeek flow records **cannot faithfully reproduce live**. Putting
them in the model would reintroduce training-serving skew — the exact bug this rewrite fixed.

A dedicated, separately-trained Web Attack model
([`data/models/cic2017_webattack_v3_http.joblib`](data/models/)) was evaluated to push this
further:

| Variant | Recall | Precision | F1 | AUC | Benign FP | Live-servable? |
|---|:---:|:---:|:---:|:---:|:---:|:---:|
| Flow-only (main pipeline) | 14.0% | — | — | — | — | ✅ |
| **Flow + HTTP (Zeek-servable)** | **96.8%** | 27.9% | 0.43 | 0.976 | 5.7% | ✅ |
| Flow + HTTP + IAT (offline ceiling) | 98.0% | 97.8% | 0.979 | 0.9997 | 0.05% | ❌ (IAT not reproducible) |

The flow+HTTP model **recalls 96.8%** of web attacks but at **27.9% precision / 5.7% benign
false-positive rate** — it ranks well (AUC 0.976) but over-flags benign HTTP. The offline
"ceiling" row proves the separating signal lives in IAT features that aren't live-servable.
This is documented rather than hidden: web-attack coverage is a deliberate, evidence-backed
trade-off, and the layered design leans on Suricata web signatures to cover the gap.

![Web Attack Confusion Matrix (v3-http)](reports/model_evaluation/cic_v3_webattack_confusion_matrix.png)

---

### A Note on UNSW-NB15 (explored and dropped)

An earlier version of Lighthouse ran a **second model on UNSW-NB15** as a co-equal detector,
on the theory that it would cover exploit/backdoor families CIC lacks. It was **dropped** after
evaluation: once features were restricted to the Suricata/Zeek-reproducible set (no skew), its
live-reproducible recall on the families that mattered was too low to justify the added
complexity, false positives, and second consensus gate. The research-correct tool for the gaps
UNSW was meant to fill (volumetric floods, known IOCs) turned out to be the **rate aggregator +
Suricata signatures**, not a second supervised model. The system is now **CIC-only** for ML,
which is simpler, faster, and easier to keep skew-free. The UNSW evaluation artifacts remain in
[`reports/model_evaluation/`](reports/model_evaluation/) for the record.

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                          NETWORK TRAFFIC                            │
└───────────┬───────────────────────────────┬────────────────────────┘
            │                               │
            ▼                               ▼
   ┌──────────────────┐            ┌──────────────────┐
   │      ZEEK        │            │    SURICATA      │  Signatures only
   │ (primary sensor) │            │  (IDS sensor)    │  (thresholded floods,
   │  conn.log + http │            │   eve.json       │   IOC rules)
   └────────┬─────────┘            └────────┬─────────┘
            │  rich flow records            │  alert events
            ▼                               │
   ┌──────────────────┐                     │
   │   ZEEK BRIDGE    │  18 CIC features    │  joined by
   │  (detection/)    │  per flow, batched  │  community-id
   │                  │  inference          │
   └────────┬─────────┘                     │
            │                               │
            ▼                               ▼
   ┌──────────────────────────────────────────────────┐
   │            THREE-LAYER DETECTION                  │
   │                                                   │
   │  Layer 1  Rate aggregator   O(1), bounded memory  │
   │           → DDoS / PortScan / DoS (volumetric)    │
   │  Layer 2  CIC ML (18 feat)  XGBoost + LightGBM    │
   │           → structured / app-layer families       │
   │  Layer 3  Suricata signature → known IOCs / floods│
   │                                                   │
   │  Initiates if  rate OR cic OR signature  fires    │
   └────────────┬──────────────────────────────────────┘
                │                  ▲
                │                  │ host+network correlation (+10)
                │       ┌──────────┴─────────┐
                │       │   WAZUH POLLER     │  auth brute force, FIM,
                │       │  (host alerts)     │  process exec — by src_ip
                │       └────────────────────┘
                ▼
   ┌──────────────────────────────┐
   │   RISK SCORER (NIST 800-30)  │  Likelihood × Impact × Temporal
   │   ML 0.50 + Rule 0.30        │  → 0–100 score
   │   + Intel 0.20               │
   └────────────┬─────────────────┘
                ▼
   ┌──────────────────────────────┐
   │      DECISION ENGINE         │
   │  < 25:   log (silent)        │
   │  25-60:  alert               │
   │  61-80:  review              │
   │  81-100: AUTO BLOCK          │
   └────────────┬─────────────────┘
                ▼
   ┌──────────────────────────────┐
   │       LLM ASSISTANT          │  Groq / Ollama / OpenAI
   │  1-2 sentence explanation    │  Falls back to rule-based
   └────────────┬─────────────────┘
                ▼
   ┌──────────────────────────────┐
   │      FASTAPI BACKEND         │  REST + WebSocket
   │      + SQLite (WAL mode)     │  bounded store, indexed
   │      + SOAR engine           │  iptables / Wazuh API
   └────────────┬─────────────────┘
                │  WebSocket /ws/alerts
                ▼
   ┌──────────────────────────────┐
   │    REACT DASHBOARD           │  Terminal Noir aesthetic
   │    (SOC analyst view)        │  Real-time alert feed
   │    http://localhost:5173     │  Block / Isolate / Dismiss
   └──────────────────────────────┘
```

---

## Risk Scoring (NIST SP 800-30)

Each event's 0–100 score follows the NIST adversarial risk model
([`pipeline/risk_scorer.py`](pipeline/risk_scorer.py)):

```
Risk = Likelihood × Impact × Temporal

Likelihood = ML_conf × 0.50  +  Rule_conf × 0.30  +  Intel_conf × 0.20
             (anomaly signal)   (Wazuh/Suricata)     (AbuseIPDB reputation)

Impact     = asset criticality × attack-severity multiplier
Temporal   = recency / sustained-activity amplification

Correlation bonus: +10 when a Wazuh host alert and a network alert share src_ip
                   (host+network agreement is the documented hybrid advantage).
```

Decision routing ([`pipeline/decision_engine.py`](pipeline/decision_engine.py)):

| Action | Risk score | Behaviour |
|--------|:----------:|-----------|
| `auto_block` | ≥ 81 | Critical — IP blocked via iptables / Wazuh, surfaced as CRITICAL |
| `review` | ≥ 61 | Suspicious — surfaced for analyst review |
| `alert` | ≥ 25 | Low severity — surfaced, no action |
| `log` | < 25 | Dropped from dashboard (silent) |

Rate thresholds are **empirically calibrated** against captured baseline + attack traffic
(see [`docs/calibration_baseline.md`](docs/calibration_baseline.md) and
[`scripts/calibrate_thresholds.py`](scripts/calibrate_thresholds.py)) — placed above the
benign 99th percentile and below the observed attack level, a measured margin rather than a
guess.

---

## Quick Start

### Option A — Live sensor (Zeek primary + Suricata signatures)

```bash
# On the sensor host, with Zeek + Suricata running:
export LIGHTHOUSE_ZEEK=1
export ZEEK_CONN_PATH=/zeek/logs/current/conn.log
export EVE_JSON_PATH=/var/log/suricata/eve.json
export LIGHTHOUSE_DEV=0

uvicorn backend.main:app --host 0.0.0.0 --port 8000

# Frontend
cd frontend && npm install && npm run dev   # http://localhost:5173
```

### Option B — Local Dev (mock alerts, no sensor needed)

```powershell
python -m venv .venv && .venv\Scripts\activate
pip install -r requirements.txt

# Terminal 1 — backend (mock alert mode)
$env:LIGHTHOUSE_DEV = "1"
uvicorn backend.main:app --reload --port 8000

# Terminal 2 — frontend
cd frontend && npm install && npm run dev   # http://localhost:5173
```

### Option C — Synthetic Attack Injection (real ML, no sensor)

```powershell
# Terminal 1 — backend watching a log file
$env:LIGHTHOUSE_DEV = "0"
$env:EVE_JSON_PATH  = "logs/test_eve.json"
uvicorn backend.main:app --reload --port 8000

# Terminal 2 — inject synthetic Suricata flow records
python tests/simulate_suricata_attack.py --attack ddos       --count 50  --eve logs/test_eve.json
python tests/simulate_suricata_attack.py --attack portscan   --count 30  --eve logs/test_eve.json
python tests/simulate_suricata_attack.py --attack bruteforce --count 20  --eve logs/test_eve.json

# Terminal 3 — frontend
cd frontend && npm run dev
```

Available attack profiles: `ddos`, `ddos-https`, `dos`, `portscan`, `bruteforce`, `bot`

---

## Live Deployment & Attack Testing

Lighthouse has been deployed and validated across **multiple physical machines** with real
attack tooling (not just synthetic injection):

| Role | Host | Function |
|------|------|----------|
| Lighthouse + Zeek + Suricata | VM1 (bridged) | Detection pipeline + sensors |
| Monitored endpoint | Ubuntu | Wazuh agent (ID 001) |
| Attacker | Kali | nmap / hping3 / hydra |

Validated end-to-end:

```bash
# PortScan → Critical
sudo nmap -sS -p 1-1024 --min-rate 5000 <VM1_IP>

# SYN flood → DDoS, auto-block (rate layer + Suricata signature agree)
sudo hping3 -S -p 80 --flood <VM1_IP>

# SSH brute force → Wazuh host alert (rule 5712), host+network correlation
hydra -l root -P wordlist ssh://<VM1_IP>
```

Confirmed live: real DDoS floods produce **Critical** alerts auto-blocked at **risk 92, 100%
confidence**; Wazuh brute-force alerts surface with `agent_id` populated and elevate the
correlated risk when the same source IP also trips the network layer.

---

## API Reference

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/alerts` | Recent alerts (threat_level DESC) |
| `GET` | `/api/alerts/search` | Filter by src_ip, attack_type, threat_level, status, since, auto_blocked |
| `GET` | `/api/stats` | `{total_today, critical, suspicious, auto_blocked}` |
| `POST` | `/api/alerts/{id}/block` | Block source IP via iptables / Wazuh |
| `POST` | `/api/alerts/{id}/isolate` | Isolate Wazuh agent |
| `POST` | `/api/alerts/{id}/dismiss` | Mark alert dismissed |
| `WS` | `/ws/alerts` | Real-time alert stream |
| `GET` | `/health` | `{status, db_alerts}` |

```bash
curl http://localhost:8000/api/stats
curl "http://localhost:8000/api/alerts/search?attack_type=DDoS&threat_level=2"
curl "http://localhost:8000/api/alerts/search?auto_blocked=true&limit=50"
curl -X POST http://localhost:8000/api/alerts/a1b2c3d4/block
```

---

## Key Files

| File | Purpose |
|------|---------|
| `detection/flow_features.py` | **Shared feature module** — single source of truth, kills training-serving skew |
| `detection/zeek_bridge.py` | conn.log → 18 CIC features → ML → DetectionEvent (primary sensor) |
| `detection/rate_aggregator.py` | O(1) volumetric detector (DDoS / PortScan / DoS) |
| `detection/wazuh_alerts.py` | Wazuh host-alert normalization + host-seen tracking |
| `backend/main.py` | FastAPI — REST, WebSocket, ingest loop, Wazuh poller, batched inference |
| `backend/store.py` | Bounded in-memory store + SQLite persistence |
| `backend/soar.py` | Block / isolate / unblock (iptables exec + Wazuh API) |
| `backend/llm_assistant.py` | LLM explanations — Groq, Ollama, OpenAI, fallback |
| `pipeline/risk_scorer.py` | NIST 800-30 risk formula 0–100 + host/network correlation bonus |
| `pipeline/decision_engine.py` | Route by score: log / alert / review / auto_block |
| `scripts/retrain_cic_smote.py` | Retrain the CIC-IDS-2017 two-stage pipeline |
| `scripts/calibrate_thresholds.py` | Empirically calibrate rate thresholds from captures |
| `scripts/generate_model_reports.py` | Regenerate model performance graphs |
| `infra/zeek/local.zeek` | Zeek site policy (JSON conn.log + http/ssl/dns + community-id) |

---

## Environment Variables

Copy `.env.example` to `.env`:

| Variable | Default | Description |
|---------|---------|-------------|
| `LIGHTHOUSE_DEV` | `1` | `1` = mock alerts, `0` = real pipeline |
| `LIGHTHOUSE_ZEEK` | `0` | `1` = Zeek-primary mode (reads conn.log) |
| `ZEEK_CONN_PATH` | `/zeek/logs/current/conn.log` | Zeek conn.log path |
| `EVE_JSON_PATH` | — | Suricata eve.json path (signatures) |
| `SOAR_DRY_RUN` | `1` | `1` = log only, `0` = real iptables/Wazuh (root required) |
| `CIC_MODEL_PATH` | `data/models/cic2017_pipeline_smote.joblib` | CIC model |
| `LLM_PROVIDER` | `ollama_cloud` | `groq` \| `openai` \| `ollama_cloud` \| `ollama` |
| `LLM_API_KEY` | — | API key for chosen LLM provider |
| `LLM_MODEL` | `gemma3:4b` | Model name |
| `ABUSEIPDB_API_KEY` | — | Free key from abuseipdb.com |
| `REDIS_HOST` | `localhost` | Redis host |
| `WAZUH_HOST` | `localhost` | Wazuh manager host |

---

## Retraining & Calibration

```powershell
.venv\Scripts\activate

python scripts/retrain_cic_smote.py          # retrain CIC 2017 two-stage pipeline
python scripts/generate_model_reports.py     # regenerate performance graphs

# Calibrate rate thresholds from captured traffic (run BEFORE production sign-off)
python scripts/calibrate_thresholds.py --baseline baseline_eve.json --attack attack_eve.json
```

Training applies `scale_pos_weight` (Stage 1) and SMOTE (Stage 2) — never both on the same
stage — with the early-stopping validation set carved from **real** (pre-SMOTE) data. Models
save to `data/models/` (gitignored — do not commit binaries).

---

## Attack Simulation

> Run only against systems you own or have explicit written permission to test.

```bash
sudo hping3 -S -p 80 --flood <target>          # DDoS / SYN flood
sudo hping3 -S --scan 1-1024 <target>          # PortScan
sudo nmap -sS -p 1-1024 --min-rate 5000 <target>   # PortScan
hydra -l root -P wordlist ssh://<target> -t 4  # SSH brute force (→ Wazuh)
```

---


<div align="center">

AI-SOC capstone project

</div>
