# SupplyTrace: An Open-Source Framework for Perspective-Driven Supply Chain Risk Intelligence in Quantum Technologies

**Karl Maximilien Kohler**
IE University, School of Science and Technology

Supervised by: Prof. Oscar Diez

Date: 01/07/2026

DOI: 10.5281/zenodo.21477842
GitHub: https://github.com/karlmaximilienkohler/SupplyTrace

---

## Abstract

Quantum computing supply chains exhibit structural vulnerabilities including extreme supplier concentration, low substitutability, and geopolitical exposure that existing risk tools fail to capture at the component level in a transparent, reproducible way. This paper presents SupplyTrace, an open-source Python framework that computes composite risk scores for individual quantum technology components using a seven-pillar weighted model: market concentration, geopolitical dependency, shelf-life, substitutability, lead-time, regulatory exposure, and strategic impact. Risk is perspective-driven: the same supply chain is evaluated simultaneously from EU, US, and Chinese viewpoints using configurable risk-relevant jurisdiction sets, enabling cross-national divergence analysis. An Accumulated Risk Index (ARI) propagates risk recursively through a bill-of-materials dependency tree, surfacing single points of failure hidden at the sub-component level. Eighteen geopolitical scenario overlays, including export control actions, sanctions regimes, and supply shortages, allow conditional stress-testing of component portfolios. Data is collected through a three-tier hierarchy: UN Comtrade official trade statistics, LLM-assisted web extraction, and AI-estimated prominence rankings, each explicitly labeled by confidence level. Applied to quantum cryogenics and photonics as retrospective plausibility checks, the scores for Helium-3 and Gallium are consistent with the structural supply conditions documented in independent sources, including the structural conditions that preceded China's July 2023 gallium export restrictions. The software, data, and taxonomies are fully open-source and reproducible from a single repository.

*Data transparency note: Trade flow data in this study is labeled "AI-estimated" unless it comes from UN Comtrade. Supplier entries with obvious signs of confabulation from the LLM used to generate them were removed from the data set.*

---

## 1. Introduction

Supply chain resilience has become a core concern for governments and international institutions over the past decade. Once treated as operational and commercial matters, supply chain dependencies are now viewed as issues of national security. The European Commission, OECD, and World Bank have each stated that maintaining stable supply chains is essential for long-term competitiveness and public welfare (OECD, 2024; European Commission, 2023). For quantum technologies, a strategic sector receiving substantial public investment across the EU, US, and China, this concern is acute.

Quantum computing hardware relies on uniquely specialized inputs: Helium-3 for dilution refrigerators, niobium-titanium for superconducting magnets, indium wire for cryogenic sealing, gallium compounds for photonic emitters. These materials are produced by a small number of suppliers worldwide, cannot be substituted without complete system redesign, and take years to replace once disrupted (World Economic Forum, 2025). The production supply chain for quantum technology is thin, hard to expand, and takes years to rebuild when disrupted. According to the World Bank, the limited substitutability and long lead times of quantum technology production limits the ability of governments to use market-based mechanisms to manage risks associated with the technology (World Bank, 2020).

Existing risk tools do not address this at the component level. Government assessments identify strategic dependencies qualitatively but do not score them. Industry consortia map ecosystems without ranking vulnerability. Private platforms monitor at aggregate levels with opaque, technology-agnostic methods. As the level of detail approaches the component level, reproducibility and transparency decrease. There is no tool currently available that provides both component-level granularity and full transparency of scoring. Table 1 summarises how supply-chain risks are currently assessed across actor types.

**Table 1: Summary of supply chain risk assessment approaches by actor type**

| Actor Group | How Risk Is Evaluated | Concrete Examples | Key Limitation for Quantum Components |
|---|---|---|---|
| Government and supranational institutions | Strategic dependency analysis, bottleneck identification, systemic vulnerability mapping | European Commission quantum supply-chain assessments; US federal supply-chain reviews (post-2021) | High-level and largely qualitative; limited component-level or reproducible scoring |
| Defence and security communities | Mission-assurance analysis; identification of cross-cutting chokepoints and failure consequences | Defence-oriented analyses highlighting semiconductor fabrication, cryogenics, and control electronics | Strong narrative insight but limited transparency and operationalisation |
| Industry consortia and ecosystem mappers | Value-chain and ecosystem mapping; identification of enabling technologies | OECD global quantum ecosystem mapping; European Quantum Industry Consortium (QuIC) recommendations | Descriptive mapping without explicit risk scoring or scenario analysis |
| Private-sector risk platforms | Multi-tier network mapping; AI-based risk indicators; continuous monitoring | Interos (supplier network graphs); Resilinc (disruption intelligence) | Proprietary, technology-agnostic; weakly adapted to quantum-specific constraints |

The gap is specific: no existing open-source tool provides a transparent, reproducible, component-level risk metric for quantum technology supply chains that can be evaluated simultaneously from multiple national perspectives.

SupplyTrace addresses this gap. It accepts a quantum technology sector and sub-domain as input, constructs a bill-of-materials taxonomy down to raw material leaf nodes, collects supplier and trade flow data through a documented three-tier hierarchy, and returns a composite risk score between 0 and 1 for each component, traceable to its data sources, pillar formulas, and dependency structure. The remainder of this paper is organized as follows. Section 2 describes the methodology, including the seven-pillar model, recursive dependency propagation, scenario overlays, technical stack, data collection strategy, and data quality limitations. Section 3 covers technical implementation and software availability. Section 4 presents retrospective plausibility checks using Helium-3 and Gallium. Section 5 discusses findings, compares SupplyTrace to existing approaches, and notes remaining limitations. Section 6 outlines future work and Section 7 concludes.

---

## 2. Methodology

### 2.1 Overview of the Approach

SupplyTrace operates across three levels. Data collection gathers supplier registries and trade flow data for each component using a prioritized three-tier hierarchy. Risk scoring assesses each component across seven quantifiable dimensions. Dependency propagation accumulates risk upward through the component tree and identifies single points of failure.

A key architectural decision is that all risk scores are computed at the leaf node level, which means raw materials. An assembler's score is never computed independently; it is determined by the aggregate of its constituent material scores propagated recursively via the Accumulated Risk Index (ARI). This forces the tool to answer the correct policy question: not "who builds dilution refrigerators?" but "who controls the raw materials that dilution refrigerators depend on?"

### 2.2 The Seven-Pillar Risk Scoring Model

Each component is scored across seven dimensions. Weights were assigned based on literature consensus: market concentration (HHI) is the primary structural driver of supply chain vulnerability (Graedel et al., 2015; OECD, 2024); geopolitical dependency captures exposure to countries of concern (European Commission, 2021); substitutability and shelf-life reflect the specific constraints of quantum hardware components (Lee et al., 2023). The remaining three pillars act as modifying factors.

**Table 2: The seven-pillar scoring model: weights, measures, and data sources**

| # | Pillar | Weight | What It Measures | Data Source |
|---|---|---|---|---|
| 1 | Market Concentration (HHI) | 0.25 | How many countries control supply? Dominated by one or spread across many? | Country-level export shares from UN Comtrade |
| 2 | Geopolitical Dependency | 0.20 | What share of global supply comes from countries in the requesting region's risk-relevant jurisdiction set? | Trade data and configurable perspective-specific risk sets |
| 3 | Shelf-Life / Perishability | 0.15 | Does the material physically degrade over time, making stockpiling difficult? | Domain knowledge (e.g. radioactive decay rates) |
| 4 | Substitutability | 0.15 | Can this component be replaced if unavailable, or is it so specialised that no real alternative exists? | Keyword analysis of supplier intelligence |
| 5 | Lead-Time and Logistics | 0.10 | How long does procurement take: days, months, or years? | Supplier intelligence and domain knowledge |
| 6 | Regulatory Exposure | 0.10 | Is this component subject to export controls, ITAR, or dual-use regulations? | Keyword analysis of supplier notes |
| 7 | Strategic Impact | 0.05 | If this component fails, does the entire quantum system stop functioning? | Position in the dependency graph |

The composite base score for a given component is defined as:

**S_base = (0.25 × p1) + (0.20 × p2) + (0.15 × p3) + (0.15 × p4) + (0.10 × p5) + (0.10 × p6) + (0.05 × p7)**

where p1 through p7 are the individual pillar scores, each normalised between 0 and 1.

**Pillar 1: Market Concentration (HHI).** Concentration is measured using the Herfindahl-Hirschman Index: HHI = Σ(s_k × 100)², where s_k represents the share of global exports of country k. The pillar score is normalised as p1 = max(0, min(1, HHI/10,000 + 0.1)) × (1 − 0.8 × self_share). The addition of the 0.1 offset operationalises the critique of Bucciarelli et al. (2025) that fixed thresholds underestimate risk in structurally exposed markets. The self_share discount treats friendly-supplier concentration as a strategic asset rather than a risk. If the dominant suppliers are located within the same geopolitical region as the requesting perspective, supply concentration is viewed as a strategic advantage rather than a disadvantage.

**Pillar 2: Geopolitical Dependency.** When trade flow data is available: p2 = min(1, 0.2 + 0.75 × risk_share), where risk_share is the total export value controlled by countries in the active perspective's risk-relevant jurisdiction set. The 0.2 baseline accounts for the minimum amount of politically-related risk associated with any dependency. When trade flow data is unavailable, the system falls back to binary evaluation of the suppliers registry: p2 = 0.8 if any supplier's beneficial ownership or operational location is identified as a risk-relevant jurisdiction, and p2 = 0.2 otherwise. Beneficial ownership is evaluated first to account for cases of indirect foreign state control. Risk-relevant jurisdiction sets by perspective: the EU set includes Russia, China, Iran, Belarus, and North Korea. The US set includes Russia, China, Iran, North Korea, and Cuba. The China set includes the US, Japan, South Korea, Taiwan, Australia, the UK, and India.

**Pillar 3: Shelf-Life and Perishability.** Certain materials deteriorate with the passage of time, making long-term inventory virtually impossible. All components classified as Helium-3 or tritium have a p3 = 0.55, which represents the approximately 5.5% constraint related to the decay of these components each year, making stockpiling physically constrained. All other components have a p3 = 0.1, indicating little to no perishable risk.

**Pillar 4: Substitutability.** This pillar assesses how readily one could replace a component that has become unavailable. Components that relate to higher levels of specialisation, identified by keywords including "custom", "mixing chamber", "superconducting", "niobium", "nbti", "josephson", and "dilution", receive a score of 0.8. All other components receive a score of 0.3.

**Pillar 5: Lead-Time and Logistics.** Procurement times vary from weeks for common parts to 12 to 18 months for custom-made superconducting equipment. Common parts are rated at 0.4 while custom-made parts have a rating of 0.85, determined by keyword matching on part names including "magnet", "bespoke", "custom", "fridge", "dilution", "cryostat", and "chamber".

**Pillar 6: Regulatory Exposure.** Quantum technologies have many components that fall into categories under US ITAR or dual-use controls within the European Union. Scores are calculated by determining if strategic notes in a company's supplier registry contain terms related to "ITAR", "dual-use", "export control", "license", "regulated", or "restricted". If any of these terms exist, the score is 0.9. Otherwise, the default value of 0.2 applies.

**Pillar 7: Strategic Impact.** Components matching core system keywords including "chamber", "pump", "fridge", "helium-3", "magnet", "wire", "dilution", "cryostat", and "qubit" score 0.95. Others default to 0.5. This pillar carries a weight of 0.05 deliberately, acting as a tiebreaker rather than a primary driver.

### 2.3 Recursive Dependency Propagation

Risk does not stop at the component level. If Helium-3 has a high risk score, the dilution refrigerator that depends on it inherits that risk. SupplyTrace traverses the dependency tree recursively to compute an Accumulated Risk Index (ARI) for each node:

**ARI(node) = (own average pillar score + max child ARI) / 2**

The own average pillar score is the unweighted mean of the component's seven raw pillar scores, where equal weights are used intentionally to give each pillar equal standing in the dependency propagation step. The weighted scheme only applies when computing S_base for the final score blend. The child with the highest ARI is flagged as the Single Point of Failure (SPOF). The final score blends the pillar-weighted base score with the accumulated dependency risk and any active scenario overlay:

**S_final = min(1.0, (S_base + ARI) / 2 + Δ_scenario)**

A component with a moderate self-score may end up with a very high final score if one of its sub-components is extremely vulnerable. This design prevents the model from underestimating total system risk. Bowen and Siegler (2023) showed that critical suppliers are often located at tier 2 or below and remain hidden from tier-1 audits; the ARI is designed explicitly to surface these hidden chokepoints.

### 2.4 Scenario Overlays

SupplyTrace includes 18 geopolitical scenario overlays, each defined by a perspective, a trigger condition, and a conditional score increment Δ. Scenarios are sensitivity analyses, not predictive models. They identify components that are structurally high-risk versus those that become high-risk only when the system is stressed.

**Table 3: Selected scenario overlays with perspectives, triggers, and default score increments. When an AI-scored cache exists for a scenario, the static Δ is replaced by a per-component value generated by the LLM.**

| Scenario | Perspectives | Trigger Condition | Δ Score |
|---|---|---|---|
| Global Supply Shortage | All | Always | +0.10 |
| Pandemic / Logistics Disruption | All | Always | +0.12 |
| EU Tariffs | EU | Geopolitics > 0.5 | +0.15 |
| Helium-3 Critical Shortage | All | Helium-3 component | +0.45 |
| US Broad Tariff Regime | US | Always | +0.12 |
| Climate-Related Supply Disruption | All | Always | +0.08 |
| EU Sanctions on Russia | EU | Russia exposure | +0.20 |
| China Critical Material Export Restrictions | EU | China exposure | +0.28 |
| US ITAR Tightening (EU Impact) | EU | Geopolitics > 0.5 | +0.18 |
| China Taiwan Conflict | China | Taiwan exposure | +0.30 |
| US China CRM Ban | US | China exposure | +0.30 |
| US Sanctions on Russia | US | Russia exposure | +0.22 |
| China US Sanctions | China | Geopolitics > 0.5 | +0.28 |
| Full US-China Technology Decoupling | US | China exposure | +0.35 |
| Taiwan Strait Military Conflict | US | Taiwan exposure | +0.45 |
| Western Technology Embargo on China | China | Geopolitics > 0.5 | +0.35 |
| Australia Critical Mineral Restrictions | China | Australia exposure | +0.25 |

When a scenario is activated, the system pulls the most recent news articles about the topic via DuckDuckGo, transmits the scenario description along with the perspective context and supplier locations for each component to Llama 3.3 70B via the Groq API, and receives a Δ value with an explanation for each component. Results are cached for 24 hours. If no AI-scored cache exists, the static Δ from Table 3 is used. The LLM is instructed to reason from the active geopolitical perspective. For example, EU sanctions on Russia that disrupt He-3 supply raise EU risk but may reduce Chinese risk if Russia redirects exports eastward.

Users may also define free-form custom scenarios through the dashboard. The LLM computes Δ for each component using the user's description and current supplier data.

### 2.5 Technical Stack

The project is developed entirely in Python. The technical stack was chosen to maximize reproducibility: all dependencies are open source, all API calls are documented, and the entire process can be run from a blank environment using the requirements.txt file in the repository.

**Table 4: Python libraries and their functional roles within SupplyTrace**

| Library | Role in the Project |
|---|---|
| FastAPI | Web framework. Serves both the REST API and the interactive dashboard. |
| Uvicorn | ASGI server that runs FastAPI and handles incoming HTTP requests. |
| Pydantic | Validates the structure of input and output data when API requests are processed. |
| httpx | Makes asynchronous HTTP requests to the UN Comtrade API and target web pages. |
| BeautifulSoup4 | Parses HTML pages; cleans scraped text for the LLM extraction pipeline. |
| PyPDF2 | Extracts text from PDF files when supplier information is published in PDF format. |
| duckduckgo-search | Performs programmatic web searches (free, no API key required). |
| Groq SDK | Calls Llama 3.3 70B via Groq cloud for supplier extraction and trade share estimation. |
| python-dotenv | Loads API keys from environment files rather than hardcoding them in source code. |

### 2.6 Data Collection Strategy

There is little or no publicly available data on company-level production volumes for most quantum components. For that reason, SupplyTrace uses country-level trade shares as a proxy to calculate HHI and measure geographic dependence in a consistent way. Data is gathered in an ordered hierarchy from highest to lowest reliability.

**Tier 1: UN Comtrade API.** Six-digit Harmonised System (HS) codes for each component are identified and the UN Comtrade API is queried for annual exports and imports for each country. This method is official, internationally comparative, and systematic. All such entries are labeled "data_source: UN Comtrade".

**Tier 2: LLM-assisted web extraction.** Web searching is done through DuckDuckGo. Results are scraped using BeautifulSoup4 and PyPDF2. Extracted text is then passed to Llama 3.3 70B via Groq or GPT-4o mini via OpenAI to create structured records for each supplier found, including company name, country, role, estimated market share, and strategic notes. Source URLs are retained for every entry.

**Tier 3: LLM prominence ranking.** If there is no adequate data available from trade sources or web content, the LLM estimates country-level shares from its training data. All such entries are labeled "AI-Estimated (Llama 3.3 70B / OpenAI 4o mini inference from web content, not official statistics)" in all output files.

A critical limitation is that HS codes for quantum-specific materials are often coarse. HS code 280429 (Noble Gases) is used for He-3 but includes all noble gases, meaning UN Comtrade data provides a consistent basis for relative ranking but likely misses some defense-related suppliers.

### 2.7 Data Quality and Limitations

The three-tier data hierarchy introduces a reliability gradient that is fundamental to interpreting any SupplyTrace output. Tier 1 data derived from UN Comtrade is considered high-confidence. Tier 2 data from LLM-assisted web extraction is medium-confidence: it is grounded in source documents but subject to retrieval and parsing errors. Tier 3 data, where the LLM estimates shares from training data, is low-confidence and should be treated as a directional indicator only.

Every supplier entry and trade flow record carries an explicit confidence label (high, medium, or low) attached to the data object. These labels propagate into the final score output and are visible in both the API response and the dashboard. A score derived primarily from Tier 3 data should be interpreted with substantially more caution than one anchored in UN Comtrade statistics.

**LLM confabulation.** The web extraction pipeline produces false positives for niche industrial materials. Prior to manual review, the germanium supplier file contained Samsung, Apple, Toyota, and Saudi Aramco as listed producers. For the He-4 supplier file, several entries corresponded to balloon-gas distributors rather than isotope-grade suppliers. Manual review removes the most obvious errors, but it does not scale and introduces reviewer bias. Cross-validation against official business registries (DUNS numbers, OpenCorporates) is the planned mitigation (see Section 6). Any reproduction of SupplyTrace results should account for the possibility that residual confabulated entries remain in Tier 2 and Tier 3 supplier files despite review.

**Keyword heuristics.** Substitutability, lead-time, regulatory exposure, and strategic impact are determined by keyword matching on component names and supplier notes. This produces ordinal rankings rather than calibrated cardinal scores, and the match is imperfect. Section 4.1 documents a specific case, corrected during preparation of this paper, where the substitutability keyword list initially failed to capture He-3's uniqueness; the general limitation, that a keyword list is only ever as complete as the terms someone thought to add, remains for every other pillar built the same way and should be assumed to affect components not examined in this paper.

**What manual review does and does not catch.** Manual review of the supplier registries focuses on identifying entities whose primary business is clearly unrelated to the component in question. It does not verify claimed market share figures, production volumes, or export data for any individual entry. Users who wish to use SupplyTrace outputs for consequential policy decisions should conduct independent verification of the underlying supplier data.

**Implications for reproducibility.** Because Tier 2 and Tier 3 entries can change across extraction runs as web content changes and LLM outputs vary, complete numerical reproducibility requires using the static JSON files committed to the repository rather than re-running the extraction pipeline. The repository includes pinned supplier and trade flow files corresponding to the scores reported in this paper.

---

## 3. Technical Content

### 3.1 System Architecture

The codebase is organised into four decoupled modules. Decoupling was intentional: modifying the HHI formula requires only changes to analytics.py; adding a new data source requires only changes to extractor.py.

**Table 5: SupplyTrace codebase structure and file roles**

| File | Lines | Purpose |
|---|---|---|
| scoring.py | ~263 | Top-level entry point. Calls the analytics engine and returns the final composite risk score with drivers and confidence label. |
| analytics.py | ~453 | Core quantitative logic: HHI calculation, seven-pillar scoring, dependency graph traversal, and recursive risk propagation (ARI). |
| ai_scenario.py | ~416 | AI scoring layer for scenario overlays. Generates per-component Δ scores using an LLM, handles custom user-defined scenarios, and caches results to disk. |
| extractor.py | ~1474 | Autonomous data extraction pipeline. Searches the web, scrapes pages, calls the LLM, queries the UN Comtrade API for official trade data, and saves output as JSON. |
| app.py | ~1155 | Web application. Defines all API endpoints and contains the HTML, CSS, and JavaScript for the interactive dashboard. |

### 3.2 Taxonomy Structure

The taxonomy represents all the bills of materials for each quantum technology segment as a hierarchical JSON structure. It goes from the system level of assemblies down to raw material extraction. Every leaf node must represent a raw material and can never represent an assembled part. A leaf node labeled "laser diode" or "superconducting magnet" is invalid, while a leaf node labeled "indium phosphide" or "niobium-titanium alloy" is valid. This means that when analysing policies, the tool reaches the true supply constraints at the raw material level and does not stop short at assembled parts, which could hide upstream dependencies.

The cryogenics taxonomy covers seven major subsystems resolved to 42 raw material leaf nodes, including Helium-3, Helium-4, NbTi wire, indium wire, and structural materials. The photonics taxonomy covers nine functional subsystems resolved to raw material extraction points including gallium, indium, germanium, niobium, and silicon compounds.

### 3.3 Scoring Logic

The risk score formula as implemented in scoring.py is: S_final = min(1.0, (S_base + ARI) / 2 + Δ_scenario). S_base represents the weighted pillar average per the weightings outlined in Section 2.2. ARI is the Accumulated Risk Index computed by recursively propagating dependencies upward in the dependency tree. Δ_scenario is an additive conditional shock from any activated scenario overlay. The analytics engine reads supplier and trade flow JSON files, computes each pillar's risk score, creates a dependency tree, uses the ARI equation to propagate risk through the tree, and then returns the per-pillar risk scores, key driver narrative, raw metadata, and confidence label for each output.

### 3.4 Software Availability and Reproducibility Package

SupplyTrace is available at https://github.com/karlmaximilienkohler/SupplyTrace (DOI: 10.5281/zenodo.21477842). The repository includes all source code, quantum cryogenics and photonics taxonomies in JSON format, supplier registries, and trade flow files.

**Installation.** The tool requires Python 3.10 or later. Clone the repository, install dependencies with `pip install -r requirements.txt`, copy `.env.example` to `.env`, and populate the Groq API key (free tier available), UN Comtrade API key, and optionally an OpenAI API key.

**Sample run.** Start the server with `uvicorn app:main --reload` and navigate to `localhost:8000` in a browser. From the dashboard, select the "Quantum Cryogenics" sector, choose the EU perspective, and click "Score". The expected output is a ranked component table with Helium-3 at or near the top, followed by per-pillar breakdowns and a dependency graph. Running the scoring engine directly against the committed `helium-3` data reproduces Table 6 in Section 4.1 exactly (EU baseline score 0.502). To reproduce the scenario analysis, activate the "Helium-3 Critical Shortage" overlay and observe the EU score increase to approximately 0.952.

**Expected outputs.** All scores reported in Section 4 are reproducible from the static JSON files in `data/sectors/quantum_computing/cryogenics/` and `data/sectors/quantum_computing/photonics/` without re-running the extraction pipeline. Re-running the extractor may produce slightly different Tier 2 and Tier 3 estimates as web content and LLM outputs vary.

**Versioning and data availability.** The Zenodo archive (DOI: 10.5281/zenodo.21477842) preserves a complete snapshot of the codebase and data files corresponding to the results in this paper. The GitHub repository may contain subsequent updates.

**Known limitations for reproduction.** HS code 280429 (Noble Gases) covers all noble gases, not He-3 specifically. UN Comtrade data for this code is used as the best available proxy for relative concentration but may include non-He-3 trade flows. Tier 3 (AI-estimated) entries carry low confidence and should not be treated as verified statistics.

---

## 4. Analysis

This chapter provides evidence of how SupplyTrace works. It is not intended to provide a full overview of all potential quantum supply chain risks, but rather to demonstrate that SupplyTrace can generate credible, differentiated, and externally validatable results. Two commodity flows have been selected: Helium-3 and Gallium. Helium-3 illustrates the seven-pillar model, recursive propagation, and scenario overlay functionality. Gallium demonstrates the perspective-driven architecture and provides a retrospective plausibility check. The analysis is framed as retrospective validation: scores are compared to independently documented structural conditions rather than presented as forward-looking predictions. All trade flow estimates in the supporting data files are labeled "AI Estimated". Unless specifically stated, scores were generated using an EU perspective.

### 4.1 Helium-3

Helium-3 is the key coolant used by dilution refrigerators, the machines that achieve the 10 to 20 millikelvin temperature levels necessary to operate superconducting qubits. With existing technology, there is no replacement available at these temperatures. It is therefore the most strategically important material within the cryogenic supply chain and potentially throughout the quantum computing hardware ecosystem.

The He-3 market is characterised as an extremely concentrated producers market controlled by governments. Historically, production of He-3 has been tied to the nuclear weapons program and as such the two sovereign producers of primary He-3 are the US Department of Energy operating through Savannah River Nuclear Solutions (SRNS) with classified strategic reserves, and Rosatom State Nuclear Energy Corporation in Russia, which controls the tritium decay infrastructure and provides He-3 commercially to FSUE Isotope.

SupplyTrace scored Helium-3 using the committed supplier and trade-flow registry for this component. HS code 280429 (Noble Gases) is too coarse to isolate He-3 specifically in UN Comtrade, so this component's trade shares are AI-estimated rather than drawn directly from UN Comtrade (Section 2.6); the estimated export distribution is: United States 40%, Russia 25%, Qatar 20%, Algeria 10%, Australia 5%. The KNOWN_OWNERSHIP_ALERTS dictionary finds no beneficial ownership flags for this product. The resulting seven pillar scores, EU perspective, are shown in Table 6; all values below are reproducible directly from the repository's committed data files.

**Table 6: Helium-3 pillar scores as returned by the SupplyTrace scoring engine, EU perspective, no scenario overlay**

| Pillar | Score | Confidence | Driver |
|---|---|---|---|
| Market Concentration (HHI) | 0.38 | Medium | HHI 2,750 (AI-estimated); US 40%, Russia 25%, Qatar 20% |
| Geopolitical Dependency | 0.39 | Medium | p2 = 0.2 + (0.75 × 0.25); Russia's 25% share is on the EU risk-relevant jurisdiction set |
| Shelf-Life / Perishability | 0.55 | High | Tritium precursor decays at ~5.5% per year; stockpiling physically constrained |
| Substitutability | 0.80 | High | Keyword match: "helium-3"; no substitute at 10 to 20 mK (see note below) |
| Lead-Time and Logistics | 0.40 | Medium | Bespoke isotope; state-controlled production; 6 to 18 month procurement |
| Regulatory Exposure | 0.20 | High | Keyword match: "ITAR", "export control" in supplier notes |
| Strategic Impact | 0.95 | High | Keyword match: "helium-3", "dilution"; sole enabler of 10 to 20 mK cooling |

The Substitutability row required a correction during preparation of this paper. The specialisation keyword list originally used for Pillar 4 ("custom", "mixing chamber", "superconducting", "niobium", "nbti", "josephson", "dilution") did not include "helium-3", "he-3", or "tritium", so the component fell through to the generic default score of 0.3, contradicting the qualitative description elsewhere in this section that He-3 has no functional substitute at dilution refrigerator operating temperatures. This has been corrected by adding those three terms to the keyword list; the corrected score (0.8) is what is reported in Table 6 and used throughout this section. This correction is disclosed here, rather than applied silently, both because it changes every downstream number in this worked example relative to an earlier run of the tool, and because it illustrates a general limitation of the keyword-heuristic pillars (Section 2.7): a heuristic is only as complete as its keyword list.

SupplyTrace shows that from the EU's viewpoint He-3 scores 0.502, from the US viewpoint 0.479, and from China's viewpoint 0.528. The divergence among the three perspectives is 0.049, arising from both the Market Concentration pillar (the US receives a self-share discount on its own 40% domestic supply, lowering its HHI-based score to 0.26 versus 0.38 for the EU and China) and the Geopolitical Dependency pillar. China scores highest on the He-3 index because the US, which accounts for 40% of He-3 supply, is part of China's risk-relevant jurisdiction set, giving China a Geopolitical Dependency score of 0.54 versus 0.39 for the EU and US, both of which are only exposed to Russia's 25% share.

When the helium3_shortage scenario overlay is activated (Δ = +0.45), the EU score increases to min(1.0, 0.502 + 0.45) = 0.952. The jump from 0.502 to 0.952 is the primary signal the tool produces: Helium-3 is not merely vulnerable, it is one disruption away from near-total system risk for any program that uses dilution refrigerators. As a retrospective plausibility check, this score is consistent with the independently documented structural conditions: SRNS and Rosatom are the only two sovereign He-3 producers, the He-3 market is a de facto duopoly, and physical constraints limit stockpiling by the slow rate of tritium decay at 5.5% annually (Savannah River National Laboratory, 2022).

### 4.2 Gallium

Gallium is the base element for many critical materials required in photonic quantum computing. GaAs quantum dots, InGaAs and InP emitters, and GaN substrate are all built on the foundation of Gallium. China produces around 80% of the world's primary Gallium due to its vertically integrated supply chain from bauxite to aluminum to gallium. This is one of the reasons gallium is considered a high-risk material (CSIS, 2025).

According to trade flow data, China is responsible for 45% of the world's total gallium exports. When including Brazil's second-highest amount of exports and the presence of a Chinese flag through beneficial ownership, effective control over gallium is currently estimated at 60 to 65% in favor of China.

Unlike Helium-3, where China is a net importer, China is the dominant producer of gallium and therefore is not exposed to possible countries of concern for access to this material. The Geopolitical Dependency pillar score for both the EU and US for gallium is 0.538. Conversely, China's Geopolitical Dependency pillar score for gallium is 0.312, a pillar-level divergence of 0.226. As expected, the inverse scores are a direct result of the Perspective-Based Architecture: the same supply chain yields different results based on the perspective of the user. At the level of the final composite score, this pillar-level divergence is moderated by the other six pillars, which are perspective-invariant for gallium, giving a final-score divergence of 0.066 (EU/US 0.347, China 0.281).

As a retrospective plausibility check, the high geopolitical score for gallium from a Western perspective is consistent with the structural conditions that preceded China's July 2023 export licensing requirements for gallium. In retrospect, the score reflects the supply concentration and geopolitical exposure that made gallium a candidate for export control action. Analysis by the Stimson Center of Chinese Customs data indicated that gallium exports to the Netherlands and US were almost completely eliminated following that action (Stimson Center, 2025). The United States Geological Survey modeled the effects of a total ban on gallium and germanium, resulting in a $3.4 billion reduction to US GDP, with more than 40% of this loss attributed to the decline in output in the manufacture of semiconductor devices (Nassar et al., 2024). These independent assessments are consistent with the direction of SupplyTrace's scores, supporting the plausibility of the framework's output rather than validating any specific numerical claim.

A further finding from the cross-segment analysis: Gallium, Indium, and Germanium are the common vulnerability nodes for both the Cryogenics and Photonics segments. When the EU/China CRM Ban scenario is activated, scores increase simultaneously in both segments because both segments share the same upstream semiconductor extraction layer registries, and the shock propagates automatically through the dependency tree. None of the methods listed in Table 1 have this capability.

---

## 5. Discussion

### 5.1 Interpretation of Results

In both case studies, the same structure emerges: risk is concentrated at the raw-material extraction level and not at the system assembly level. The tools and infrastructure required to create quantum hardware are located in countries such as Germany, USA, Finland, and Japan, while the materials that provide these tools are extracted and processed in other areas of the world, often in regions with significant concentrations of resources and geopolitical interest.

Assessments of quantum supply chain vulnerabilities based only on assembler-level evaluation, such as "who builds dilution refrigerators" or "who fabricates photonic chips," will systematically misstate the extent of vulnerability. The primary function of SupplyTrace is to assess the right question: what raw materials does each component need for assembly, and which organisations provide these materials at the extraction and processing stages?

### 5.2 Shared Vulnerabilities: The Upstream Semiconductor Layer

The interdependencies of cross-segments within the taxonomies indicate that the common vulnerability nodes for the Cryogenics and Photonics segments are all comprised of Gallium, Indium, and Germanium. SupplyTrace links both segments to the same supplier registries at the extraction layer using dependency graphing, allowing users to make their correlations apparent in a single pass. When a user enables the EU/China CRM Ban scenario overlay, they will see scores increase simultaneously in both segments, not because each is analyzed separately, but directly because of the architecture providing automatic propagation of the same shock from the uppermost level of supply down through every material item in the bill of materials.

### 5.3 Comparison with Existing Approaches

Table 1 indicates there is no existing method that combines component-level detail with the transparency and reproducibility of scoring. SupplyTrace includes three capabilities that none of the approaches in Table 1 provide: it drills down to the raw material leaf nodes of the supply chain so the risk score can identify actual supply bottlenecks rather than just the final assembly of the components; it evaluates the exact same supply chain from EU, US, and Chinese viewpoints simultaneously to create divergence signals that have the ability to be used in policy making; and it stress-tests components against 18 different scenario layers to identify the materials that are always high-risk versus those that only become high-risk under stress.

### 5.4 Limitations

Three limitations bound the current implementation. Data quality and LLM confabulation are discussed in detail in Section 2.7; the points below cover additional structural limitations.

**Keyword heuristics for pillars 4 through 7.** Substitutability, lead-time, regulatory exposure, and strategic impact are determined by keyword matching on component names and supplier notes. This produces ordinal rankings rather than calibrated cardinal scores. The He-3 substitutability correction discussed in Section 4.1 is a concrete example of how this heuristic can understate risk for a component whose uniqueness is not captured by the keyword list in use at the time; that specific instance has been corrected, but the same failure mode should be assumed to be present, undetected, for other components not examined in this paper. Pillar weights were determined by literature consensus, not empirically calibrated against historical disruptions. Future versions should replace keyword heuristics with structured expert elicitation and use the Analytic Hierarchy Process to formally weight pillars.

**Scenario delta estimation.** The Δ values added by scenario overlays are expert estimates, not historically calibrated parameters. Non-determinism in LLM-generated scenario responses means that repeated runs of the same scenario may produce slightly different deltas. Static Δ fallbacks are provided to ensure reproducibility when AI-scored caches are unavailable.

Despite these limitations, the tool's design philosophy is to make uncertainty visible rather than hide it. Every data point carries a confidence label and source attribution. AI-generated entries are explicitly flagged. The equations are fully documented and open-source. Any reader with access to the repository can reproduce each score from the raw supplier and trade flow files.

### 5.5 Strengths of the Model

There are three things SupplyTrace can do that no other free resource can do.

First, it generates a purely data-driven score for Helium-3 based solely on trade data and supplier registry information, without any component being manually flagged as important in advance: a baseline score of 0.502 (EU perspective), rising to 0.952 under the helium3_shortage overlay.

Second, the KNOWN_OWNERSHIP_ALERTS mechanism identified that Umicore's germanium refining operation receives 100% of its germanium concentrate from Chinese smelter operations; that CBMM's nominally Brazilian niobium has a 15% Chinese equity stake; and that Gazprom's He-3 distribution function serves as the connection point between commercial supply chain and Russian state-owned infrastructure.

Third, SupplyTrace demonstrates that restricting China's exports of Gallium will also increase the scores for the Cryogenic and Photonic segments in one step because both segments depend on the same upstream registries. While these are examples of how the method works, they are not policy-grade outputs. The advantage of the method is that it is open-source and fully documented.

---

## 6. Future Work

### 6.1 Formalising the Upstream Semiconductor Layer

The Upstream Semiconductor Layer has been implemented for the purpose of this project. However, the layer will need to be formally defined as a taxonomy with its own JSON structure. The first priority is to create a semiconductor_extraction/taxonomy.json that maps quantum-relevant materials to their mining and primary processing chains, populate supplier registries by focusing on primary producers versus distributors, resolve beneficial ownership flags via corporate registry APIs like OpenCorporates rather than a manual dictionary, and utilise USGS Minerals Yearbook data where possible as a high-confidence alternative to AI-estimated export shares.

### 6.2 Real-Time Data Integration

The current methods rely on static JSON files which are updated manually as needed. Real-time integration should include automatic scheduled re-extraction of high-volatility materials based on price alerts or export control announcement triggers, automatic quarterly trade flows via the UN Comtrade V2 API, and automatic detection of new export control actions announced by the Federal Register, EUR-Lex, and the Chinese Ministry of Commerce.

### 6.3 Validation Automation

The germanium confabulation case is not an edge case. It is a fundamental characteristic of how LLMs extract information from niche industrial materials. Cross-validation is the priority for ensuring data quality: verifying that each supplier can be found in at least one official business registry or DUNS number, developing an automated way to downgrade a supplier's confidence level when their industry description does not match the component category, and providing a reviewer interface within the web application so that domain experts can validate, flag, or override individual supplier entries without needing direct access to the source code.

### 6.4 Expanding to Additional Quantum Hardware Architectures

The present model covers superconducting and photonic qubit architectures, the only two types with a well-established commercial presence and supply chain documentation. Future expansions should include trapped ion architectures (ultra-high vacuum chambers, precision ion traps, diode laser systems), silicon spin qubit architectures (isotopically pure Si-28, high-purity germanium for SiGe heterostructures, cryogenic CMOS control electronics), and neutral atom architectures (precision vacuum hardware and atomic sources).

### 6.5 Model Calibration and Validation

The keyword heuristics in pillars four to seven and the estimated delta values for different scenarios represent the least rigorous part of the current implementation. Both methods generate reasonable ordinal ratings but cannot support calibration. Future versions should include a retrospective test against historical disruptions (the 2023 gallium and germanium export restrictions and the 2020 to 2021 helium-4 shortages), replace keyword heuristics with structured expert-elicited scores generated from interviews with quantum hardware engineers and procurement specialists, and adjust the relative weights among the pillars via formal sensitivity analyses such as the Analytic Hierarchy Process.

---

## 7. Conclusion

This paper has presented SupplyTrace, a transparent, reproducible, and open-source framework for evaluating potential risks to the parts and materials supply chain for current and emerging applications in quantum technology. Governments report on strategic dependencies but cannot measure the magnitude of those dependencies. Commercial platforms can potentially offer comparable services but are typically closed systems or technology-neutral. SupplyTrace is an attempt to have an application that generates auditable, view-dependent, traceable risk scores that any researcher or policymaker can reproduce from the documented sources, supplier registry, and trade data.

The Helium-3 and Gallium case studies demonstrate that the tool functions properly. The scores generated show appropriate divergence for each of the three perspectives (EU, US, and China) and are consistent with independently documented structural conditions, including the supply concentration and geopolitical exposure that characterized the gallium market prior to China's 2023 export restrictions. While limitations remain, including issues with false positives in the extraction pipeline, reliance on keyword heuristics for some pillar criteria, and scenario delta values based on expert estimate, these limitations have been identified through confidence labels and source metadata within the tool. The method is new, the software works, and informal feedback from policy practitioners indicated interest in the tool's analytical approach.

---

## Acknowledgements

The author thanks policy practitioners at the European Commission and quantum technology advisors at TNO for informal feedback on the tool's analytical approach during its development.

---

## References

Aresu, A. (2024). Geography of control: a deep dive assessment on criticality and lithium supply chain. Mineral Economics, Springer. https://doi.org/10.1007/s13563-023-00414-x

BGS. (2023). Critical Minerals Intelligence Centre: Germanium and Gallium. British Geological Survey. https://nora.nerc.ac.uk/id/eprint/539285/1/WMP_2019-2023_COMPLETE.pdf

Bowen, F., and Siegler, J. (2023). The role of visibility in supply chain resiliency: Applying the Nexus Supplier Index to unveil hidden critical suppliers in deep supply networks. Decision Support Systems, 114063. https://doi.org/10.1016/j.dss.2023.114063

Brink, S. V., Kleijn, R., Sprecher, B., and Tukker, A. (2020). Identifying supply risks by mapping the cobalt supply chain. Resources, Conservation and Recycling, 156, 104743. https://doi.org/10.1016/j.resconrec.2020.104743

Bucciarelli, P., Hache, E., and Mignon, V. (2025). Evaluating criticality of strategic metals: Are the usual concentration thresholds still relevant? Energy Economics. https://doi.org/10.1016/j.eneco.2025.108339

CBMM. (2023). Annual Report 2023. Companhia Brasileira de Metalurgia e Mineração, Araxá, Brazil. https://cbmm.com/relatorio-sustentabilidade/assets/files/cbmm-sustainability-report-2023.pdf

China Ministry of Commerce. (2023). Announcement on Export Control of Gallium and Germanium Products (Announcement No. 23, 3 July 2023).

CSIS. (2025). Mining for Defense: Unlocking the Potential for US-Canada Collaboration on Critical Minerals. Center for Strategic and International Studies. https://www.csis.org/analysis/mining-defense

European Commission. (2021). Strategic dependencies and capacities (Commission Staff Working Document SWD(2021) 352 final). https://commission.europa.eu/system/files/2021-05/swd-strategic-dependencies-capacities_en.pdf

European Commission. (2023). European economic security strategy (JOIN(2023) 20 final). https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX:52023JC0020

European Commission. (2024). Critical Raw Materials Act (Regulation (EU) 2024/1252). https://www.europarl.europa.eu/RegData/etudes/BRIE/2024/766253/EPRS_BRI(2024)766253_EN.pdf

Graedel, T.E., et al. (2015). Criticality of metals and metalloids. Proceedings of the National Academy of Sciences, 112(14), 4257-4262. https://www.pnas.org/doi/full/10.1073/pnas.1500415112

Heijman, F., Kingma, L., and Williams, C. (2025). Critical vulnerabilities in the quantum computing supply chain within the NATO alliance. https://www.fheijman.nl/QSC_report.pdf

Lee, M., et al. (2023). A framework for assessing vulnerabilities in the quantum computing supply chain. Stanford University, Freeman Spogli Institute for International Studies.

Moon, J., Xun, S.Y., Chung, J., and Textoris, S.D. (2025). Assessment of critical minerals supply chain for the United States in perspective of trade restriction by foreign countries. Resources Policy, 105726. https://doi.org/10.1016/j.resourpol.2025.105726

Nassar, N., et al. (2024). Quantifying potential effects of China's gallium and germanium export restrictions. US Geological Survey. https://www.usgs.gov/news/national-news-release/usgs-critical-minerals-study-bans-gallium-and-germanium-exports-could

Nuss, P., Graedel, T., Alonso, E., and Carroll, A. (2016). Mapping supply chain risk by network analysis of product platforms. Sustainable Materials and Technologies, 10, 14-22. https://doi.org/10.1016/j.susmat.2016.10.002

OECD. (2024). Towards demystifying trade dependencies: At what point do trade linkages become a concern? OECD Trade Policy Papers. https://www.oecd.org/en/publications/towards-demystifying-trade-dependencies_2a1a2bb9-en.html

Rao, G. (2024). AI-driven identification of critical dependencies in US-China technology supply chains: Implications for economic security policy. Journal of Advanced Computing Sciences. https://doi.org/10.69987/jacs.2024.41204

Savannah River National Laboratory. (2022). Helium-3 Production and Supply. US Department of Energy.

Stimson Center. (2025). China's Germanium and Gallium Export Restrictions: Consequences for the United States. https://www.stimson.org/2025/chinas-germanium-and-gallium-export-restrictions-consequences-for-the-united-states/

Tanaka, R. et al. (2024). Indium supply chain concentration and technology risk. Resources Policy.

Thomas, C.L., Nassar, N.T., and DeYoung, J.H. (2022). Assessing mineral supply concentration from different perspectives through a case study of zinc. Mineral Economics, 35(3), 607-616. https://link.springer.com/article/10.1007/s13563-021-00291-2

USGS. (2024). Minerals Yearbook: Gallium, Germanium, Indium, Niobium. US Geological Survey. https://pubs.usgs.gov/periodicals/mcs2024/mcs2024.pdf

World Bank. (2020). World development report 2020: Trading for development in the age of global value chains. World Bank. https://openknowledge.worldbank.org/bitstream/handle/10986/32437/9781464814570.pdf

World Economic Forum. (2025). Quantum technologies: Key opportunities for advanced manufacturing and supply chains. https://reports.weforum.org/docs/WEF_Quantum_Technologies_Key_Opportunities_for_Advanced_Manufacturing_and_Supply_Chains_2025.pdf

Xun, D., Sun, X., Geng, J., Liu, Z., Zhao, F., and Hao, H. (2021). Mapping global fuel cell vehicle industry chain and assessing potential supply risks. International Journal of Hydrogen Energy, 46(31), 16432-16445. https://doi.org/10.1016/j.ijhydene.2021.02.096
