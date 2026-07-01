# SupplyTrace

**Quantum Technology Supply Chain Risk Intelligence**

SupplyTrace is an open-source framework for mapping and scoring supply chain risk in quantum computing technologies. It computes a composite risk score (0–1) for individual quantum hardware components using a seven-pillar weighted methodology, with perspective-driven geopolitical analysis (EU / US / China), recursive dependency propagation, and scenario stress-testing.

Developed as part of the AI for Policy initiative at IE University. Reviewed by policy officers at the European Commission and TNO.

---

## What it does

- **Seven-pillar risk scoring** — market concentration (HHI), geopolitical dependency, shelf-life, substitutability, lead-time, regulatory exposure, and strategic impact
- **Perspective-driven** — the same supply chain scored from EU, US, and Chinese viewpoints using configurable adversary sets
- **Recursive dependency propagation** — risk accumulates upward through a bill-of-materials tree via the Accumulated Risk Index (ARI), surfacing single points of failure
- **Scenario overlays** — 18 geopolitical scenarios (sanctions, export controls, supply shortages) plus user-defined custom scenarios via LLM
- **Three-tier data pipeline** — UN Comtrade official trade data → LLM-assisted web extraction → AI-estimated prominence rankings, each labeled by confidence level
- **Beneficial ownership tracking** — surfaces hidden foreign state control that nominal country-of-origin data conceals
- **Interactive dashboard** — risk table, geospatial map, dependency graph, perspective comparison, scenario selector

---

## Installation

**Requirements:** Python 3.10+

```bash
git clone https://github.com/karlmaximilienkohler/SupplyTrace.git
cd SupplyTrace
pip install -r requirements.txt
cp .env.example .env
```

Edit `.env` and add your API keys (see below).

---

## API Keys

SupplyTrace requires the following API keys in your `.env` file:

| Key | Required | Free tier | Purpose |
|-----|----------|-----------|---------|
| `GROQ_API_KEY` | Yes | Yes | Llama 3.3 70B for extraction and scenario scoring |
| `COMTRADE_API_KEY` | Recommended | Yes (limited) | UN Comtrade official trade data |
| `OPENAI_API_KEY` | Optional | No | GPT-4o mini fallback for extraction |

- Groq: [console.groq.com](https://console.groq.com)
- UN Comtrade: [comtradeplus.un.org](https://comtradeplus.un.org)

---

## Quickstart

```bash
uvicorn src.app:app --reload --port 8000
```

Open your browser at `http://localhost:8000`.

1. Click **New Analysis**
2. Select sector: `Quantum Computing`, segment: `Cryogenics`, perspective: `European Union`
3. Click **Generate Taxonomy** — the AI proposes a full bill-of-materials down to raw materials
4. Review and confirm the taxonomy
5. Click **Run Extraction** — the pipeline collects supplier data for each component
6. View results in the **Dashboard** — risk scores, pillar breakdown, supplier cards, scenario overlays

Pre-extracted data for **Cryogenics**, **Photonics**, and **Semiconductors** is included in the repository. You can view results immediately without running extraction.

---

## Repository Structure

```
SupplyTrace/
├── src/
│   ├── app.py          # FastAPI web application and REST API
│   ├── scoring.py      # Composite risk score computation
│   ├── analytics.py    # HHI, pillar scoring, ARI dependency propagation
│   ├── ai_scenario.py  # LLM scenario overlay scoring and caching
│   └── extractor.py    # Data extraction pipeline: Comtrade, web, LLM
├── data/
│   └── sectors/
│       └── quantum_computing/
│           ├── cryogenics/     # 42 components: suppliers, trade flows, taxonomy
│           ├── photonics/      # 40 components: suppliers, trade flows, taxonomy
│           └── semiconductors/ # Upstream extraction layer: Ga, In, Ge, Nb, Si
├── requirements.txt
├── .env.example
└── README.md
```

---

## Methodology

### Seven-Pillar Scoring Model

| # | Pillar | Weight | Measures |
|---|--------|--------|----------|
| 1 | Market Concentration (HHI) | 0.25 | How many countries control supply? |
| 2 | Geopolitical Dependency | 0.20 | What share comes from adversarial nations? |
| 3 | Shelf-Life / Perishability | 0.15 | Does the material degrade, preventing stockpiling? |
| 4 | Substitutability | 0.15 | Can this component be replaced if unavailable? |
| 5 | Lead-Time & Logistics | 0.10 | How long does procurement take? |
| 6 | Regulatory Exposure | 0.10 | Is this subject to ITAR or dual-use controls? |
| 7 | Strategic Impact | 0.05 | Does failure halt the entire quantum system? |

**Base score:** `S_base = (0.25×p1) + (0.20×p2) + (0.15×p3) + (0.15×p4) + (0.10×p5) + (0.10×p6) + (0.05×p7)`

**Dependency propagation:** `ARI(node) = (own_avg_pillar_score + max_child_ARI) / 2`

**Final score:** `S_final = min(1.0, (S_base + ARI) / 2 + Δ_scenario)`

### Perspective-Driven Geopolitics

Risk is not absolute — it depends on the analyzing nation's geopolitical framing. Adversary sets:

- **EU:** Russia, China, Iran, Belarus, North Korea
- **US:** Russia, China, Iran, North Korea, Cuba
- **China:** US, Japan, South Korea, Taiwan, Australia, UK, India

The HHI pillar applies a self-share discount: supply concentration among friendly nations is treated as a strategic asset, not a risk.

### Data Confidence

Every supplier entry carries a `data_confidence` field (high / medium / low) and a `share_source` field documenting whether market share comes from UN Comtrade, web-extracted industry reports, or LLM estimation. AI-generated entries are explicitly labeled `"data_source": "AI-Estimated"`. Supplier entries with signs of confabulation were removed manually before inclusion.

---

## Case Studies

### Helium-3
He-3 is the coolant enabling 10–20 millikelvin temperatures in dilution refrigerators — the sole operating environment for superconducting qubits. No substitute exists. The market is a state-controlled duopoly (US DOE and Rosatom). SupplyTrace baseline score (EU perspective): **0.394**. Under the `helium3_shortage` scenario overlay: **0.844** — one disruption away from complete system collapse.

### Gallium
China produces ~80% of primary gallium globally. SupplyTrace geopolitical score for gallium (EU/US perspective): **0.538**; China perspective: **0.312** — a divergence of 0.130, the largest of any material in this study. This correctly identified the structural exposure before China imposed export licensing requirements on gallium in July 2023.

---

## Citation

If you use SupplyTrace in your research, please cite:

```
Kohler, K.M. (2026). SupplyTrace: Mapping and Scoring Supply Chain Risk in Quantum
Computing Technologies. IE University.
GitHub: https://github.com/karlmaximilienkohler/SupplyTrace
```

A journal paper describing the methodology is forthcoming.

---

## License

MIT License. See `LICENSE`.

---

## Acknowledgements

Supervised by Prof. Oscar Diez (IE University). Methodology and results reviewed by Dr. Paula Kleij and Dr. Laurent Olislager (European Commission) and Julian Rabbie, Sr. Strategy Advisor for Quantum Technologies (TNO).
