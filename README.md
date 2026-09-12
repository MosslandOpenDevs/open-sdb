# Open Software Defined Building (Open SDB)

<!-- opendevs-badges:start -->
[![License: MIT](https://img.shields.io/badge/License-MIT-64748b?style=flat)](LICENSE)
<!-- opendevs-badges:end -->

**English** | [한국어](README.ko.md)

> **"Defining the Next-Generation Operating System for Buildings."**
> 
> Moving beyond static Digital Twins to fully autonomous **Physical AI**.
> A research initiative to integrate specialized legacy domains into a unified Software Defined Building architecture.

[![Status: Incubation](https://img.shields.io/badge/Status-Incubation_&_Research_Notes-blue.svg?style=flat)](https://github.com/MosslandOpenDevs/open-sdb)

## 📖 Introduction
**Open Software Defined Building (Open SDB)** is an open-source research initiative by **Mossland**. Our goal is to establish the foundation for **Software Defined Buildings (SDB)**, a paradigm where building operations are driven by intelligent software architecture.

We believe that the ultimate goal of a Smart Building is **Physical AI**—a state where AI intelligently manages and controls the physical environment in real-time. To achieve this, we are researching ways to seamlessly integrate the deeply evolved software ecosystems of design, construction, and simulation into a unified control loop.

## 🔭 Vision & Mission

### Vision
**Realizing Autonomous Building Operations through Integration.**
We aim to create a cohesive environment where the diverse technologies of the building industry converge, enabling buildings to perceive, think, and act autonomously.

### Mission
**Unifying Legacy Expertise for Closed-Loop Control.**
Our mission is to bridge the gaps between CAD, BIM, and BEM (Building Energy Modeling). We seek to integrate these specialized legacy systems into a **Closed-Loop Control System**, ensuring that design data, simulation results, and physical controls influence each other in real-time.

## 🧩 The Challenge: Connecting Specialized Domains

The architecture, engineering, and construction (AEC) industry has made tremendous advancements in specific domains. However, these specialized software systems often operate independently:

* **Design (CAD):** The standard for precise architectural drafting.
* **Construction (BIM):** A rich repository of building information and geometry.
* **Simulation (BEM):** Highly specialized tools for energy and physics analysis.
* **Operation (BMS):** Systems managing daily building functions.

While each field has reached a high level of maturity, the lack of real-time integration prevents the realization of a true **Closed-Loop System**, where feedback from operations instantly informs simulation and control without manual intervention.

## 💡 The Solution: SDB (Software Defined Building)

We propose **SDB** as a unifying architecture. Instead of replacing existing technologies, we are researching a new software layer that can orchestrate these legacy systems for Physical AI:

1.  **Seamless Integration:** Researching methods to directly parse and utilize data from BIM and Simulation tools without data loss or manual conversion steps.
2.  **Real-time Interoperability:** Enabling instantaneous data flow between static models (BIM) and dynamic engines (Simulation/Control).
3.  **Physical AI & Closed-Loop:** Constructing a cycle where AI plans (via Simulation), acts (via Control), and observes (via Sensors) by leveraging the combined power of existing domain software.

## 📈 Industry Momentum (as of July 2026)

The industry is converging on the direction this project researches:

* **Autonomous building control is shipping.** PassiveLogic announced Level 3 autonomy for generalized building control (July 2026), Johnson Controls acquired Nantum AI to bring autonomous HVAC control into OpenBlue (April 2026), and Siemens now frames its portfolio around "the journey from smart to human-centric autonomous buildings" (Light + Building 2026).
* **Digital twins are becoming operating systems.** NVIDIA's Omniverse DSX blueprint (GA March 2026) uses facility-scale digital twins first to simulate and then to operate AI factories, and Autodesk is positioning Tandem explicitly for "Physical AI" (2026).
* **Open standards keep maturing.** IFC 4.3 is an ISO standard (ISO 16739-1:2024), an IFC 4.4 proposal and the experimental web-native "IFC X" are underway, IDS 1.0 enables machine-readable data requirements, and ASHRAE published Standard 231-2026 (CDL) — a machine-readable language for building control sequences.

These developments validate the SDB thesis — and make an open, vendor-neutral integration layer more relevant, not less.

## 📖 Prior Art & Related Work

"Software-defined buildings" and "building operating systems" are established research lineages. Open SDB builds on them rather than defining the field:

* **BOSS: Building Operating System Services** (Dawson-Haggerty et al., [USENIX NSDI 2013](https://www.usenix.org/conference/nsdi13/technical-sessions/presentation/dawson-haggerty)) and UC Berkeley's NSF-funded *Software Defined Buildings* program (2012–2017) introduced the building-OS abstraction.
* **A Reference Architecture for Smart and Software-Defined Buildings** (Mazzara et al., IEEE SMARTCOMP 2019; [arXiv:1902.09464](https://arxiv.org/abs/1902.09464)) proposed a four-layer reference architecture.
* **Proposal of a Scalable Building Operating System Architecture** (Owaki et al., [IEEE COMPSAC 2024](https://ieeexplore.ieee.org/document/10633523)) explored scalable building-OS architecture and data models toward software-defined buildings.
* **OpenBOS** (Paul et al., LBNL, [*Science and Technology for the Built Environment*, 2025](https://doi.org/10.1080/23744731.2024.2444819)) is the closest working system: a grid-responsive, semantics-driven supervisory-control platform integrating VOLTTRON, BACnet, ASHRAE 223P, BuildingMOTIF, and BOPTEST, validated on one real and one simulated building.

**Where Open SDB aims to contribute:** the handoff *between* design-time BIM data and these operational platforms — verified extraction from proprietary authoring tools, identifier provenance across systems, and automated validation gates. These are precisely the open problems OpenBOS's authors list themselves: control parameters missing from semantic models, control logic living outside the model, and commissioning before deployment.

## 📂 Repository Structure

This repository currently hosts **Research Documents**. **Proof of Concept (PoC) Code** will be added as the project moves into the prototyping phase.

```bash
open-sdb/
├── docs/
│   └── research/       # Published research reports (English / Korean)
├── LICENSE
├── README.md           # This document
└── README.ko.md        # Korean version (한국어)

```

As research matures into prototyping, we plan to expand the tree with `docs/integration/` (strategies for integrating IFC, IDF, gbXML, etc.), `docs/architecture/` (SDB Core architecture definitions), and `src/` (adapters for legacy formats, the SDB orchestration core, and AI agent prototypes for closed-loop control).

## 📚 Featured Research

Detailed research documents regarding our core technologies and frameworks.

* **Automated BIM Extraction Framework** — *published January 2026, technology landscape updated July 2026*
    * A study on fully automated extraction of BIM data from proprietary authoring tools (Autodesk Revit) into open formats for the SDB environment — covering headless automation engines (APS Design Automation, local batch processing), a composite glTF + JSON + IFC data-federation strategy keyed by GUID, and automated validation with IDS.
    * 📄 [Read in English](docs/research/automated-bim-extraction-framework.md)
    * 📄 [한국어 문서 읽기](docs/research/automated-bim-extraction-framework.ko.md)

## 🚀 Roadmap

* [ ] **Phase 1: Research & Analysis** *(in progress)* - Analyzing legacy formats and defining integration protocols.
    * [x] Automated BIM extraction framework research (Jan 2026 · updated Jul 2026)
    * [ ] BEM (EnergyPlus/IDF, gbXML) integration strategy
    * [ ] SDB Core architecture definition
* [ ] **Phase 2: Core Prototyping** - Developing the SDB Core for linking BIM data with simulation engines.
* [ ] **Phase 3: AI Orchestration** - Implementing Physical AI agents that operate the closed-loop system.

## 🤝 Contribution

This project explores new possibilities in building technology. Our working method is **AI-assisted, domain-reviewed, and reproducible**: we leverage AI and multi-agent systems to accelerate research, while every claim and artifact is expected to be reviewable and verifiable by domain experts. We welcome contributions from architects, engineers, and developers who are passionate about integrating diverse building technologies into a unified intelligent system.

## 📜 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
