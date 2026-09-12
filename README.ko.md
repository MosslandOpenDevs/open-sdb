# Open Software Defined Building (Open SDB)

<!-- opendevs-badges:start -->
[![License: MIT](https://img.shields.io/badge/License-MIT-64748b?style=flat)](LICENSE)
<!-- opendevs-badges:end -->

[English](README.md) | **한국어**

> **"차세대 빌딩 운영 시스템(OS)을 정의하다."**
> 
> 정적인 디지털 트윈을 넘어, 스스로 판단하고 제어하는 **Physical AI**를 향해.
> 각 분야에서 발전해 온 전문 기술을 통합하여 Software Defined Building(SDB) 아키텍처를 연구합니다.

[![Status: Incubation](https://img.shields.io/badge/Status-Incubation_&_Research_Notes-blue.svg?style=flat)](https://github.com/MosslandOpenDevs/open-sdb)

## 📖 소개 (Introduction)
**Open Software Defined Building (Open SDB)**은 **Mossland**가 주도하는 오픈소스 리서치 프로젝트입니다. 우리는 지능형 소프트웨어 아키텍처가 건물 운영의 핵심이 되는 **Software Defined Building (SDB)** 의 기반을 연구하고 구축하고자 합니다.

스마트 빌딩의 궁극적인 목표는 단순한 관제(Monitoring)가 아닌, AI가 현실 세계를 지능적으로 제어하는 **Physical AI**의 실현입니다. 이를 위해 우리는 설계, 시공, 시뮬레이션 각 분야에서 깊이 있게 발전해 온 소프트웨어 생태계를 하나의 유기적인 제어 루프로 통합하는 방법을 연구합니다.

## 🔭 비전 및 미션 (Vision & Mission)

### Vision
**통합을 통한 자율 운영 빌딩(Autonomous Building)의 실현**
건축 산업의 다양한 기술 요소들이 서로 단절되지 않고 융합되는 환경을 조성하여, 건물이 스스로 인지하고 판단하며 행동하는 자율 운영 환경을 구현합니다.

### Mission
**전문가 영역의 유기적 통합과 폐쇄 루프(Closed-Loop) 구축**
CAD, BIM, BEM(에너지 모델링) 등 각 영역에서 고도화된 레거시 시스템들을 연결합니다. 설계 데이터와 시뮬레이션 결과, 그리고 실제 운영 제어가 실시간으로 상호작용하는 **폐쇄 루프 제어 시스템(Closed-Loop Control System)** 을 완성하는 것이 우리의 미션입니다.

## 🧩 과제: 전문화된 영역의 연결 (The Challenge)

AEC(건축, 엔지니어링, 건설) 산업은 각 도메인별로 눈부신 기술 발전을 이뤘습니다. 하지만 고도로 전문화된 소프트웨어들은 종종 독립적으로 작동합니다.

* **설계(CAD):** 정밀한 건축 도면 작성을 위한 표준.
* **구축(BIM):** 건물의 상세 정보와 기하학적 데이터를 담은 저장소.
* **시뮬레이션(BEM):** 에너지 흐름과 물리적 현상을 해석하는 고도화된 도구.
* **운영(BMS):** 건물의 설비와 기능을 관리하는 시스템.

각 분야는 이미 훌륭한 완성도를 갖추고 있지만, 이들 사이의 실시간 통합 부재는 운영 데이터가 즉시 시뮬레이션과 제어에 반영되는 진정한 의미의 **폐쇄 루프(Closed-Loop)** 구현을 어렵게 만들고 있습니다.

## 💡 해결책: SDB (Software Defined Building)

우리는 **SDB**를 통해 이들을 통합하는 아키텍처를 제안합니다. 기존 기술을 대체하는 것이 아니라, 레거시 시스템들의 가치를 Physical AI 관점에서 엮어내는 새로운 소프트웨어 계층을 연구합니다:

1.  **Seamless Integration:** BIM 및 시뮬레이션 툴의 데이터를 수작업 변환이나 데이터 손실 없이 직접 파싱하고 활용하는 방법 연구.
2.  **Real-time Interoperability:** 정적인 모델(BIM)과 동적인 엔진(시뮬레이션/제어) 간의 실시간 데이터 흐름 구현.
3.  **Physical AI & Closed-Loop:** 검증된 기존 도메인 소프트웨어들의 역량을 결합하여, AI가 계획(시뮬레이션), 행동(제어), 관찰(센서)하는 순환 구조 완성.

## 📈 산업 동향 (2026년 7월 기준)

업계는 이 프로젝트가 연구하는 방향으로 수렴하고 있습니다:

* **자율 빌딩 제어가 실제 제품으로 출시되고 있습니다.** PassiveLogic은 범용 빌딩 제어의 Level 3 자율성을 발표했고(2026년 7월), Johnson Controls는 OpenBlue에 자율 HVAC 제어를 통합하기 위해 Nantum AI를 인수했으며(2026년 4월), Siemens는 "스마트 빌딩에서 인간 중심 자율 빌딩으로의 여정"을 포트폴리오 전면에 내세웠습니다(Light + Building 2026).
* **디지털 트윈이 운영 체제가 되어가고 있습니다.** NVIDIA의 Omniverse DSX 블루프린트(2026년 3월 GA)는 시설 규모의 디지털 트윈으로 먼저 시뮬레이션한 뒤 AI 팩토리를 운영하는 방식을 제시했고, Autodesk는 Tandem을 명시적으로 "Physical AI"용으로 포지셔닝하고 있습니다(2026년).
* **오픈 표준은 계속 성숙하고 있습니다.** IFC 4.3은 ISO 표준(ISO 16739-1:2024)이 되었고, IFC 4.4 제안과 실험적인 웹 네이티브 "IFC X"가 진행 중이며, IDS 1.0은 기계 판독 가능한 데이터 요구사항 정의를 가능하게 합니다. ASHRAE는 빌딩 제어 시퀀스를 기계가 읽을 수 있게 기술하는 Standard 231-2026(CDL)을 발행했습니다.

이러한 흐름은 SDB 논제를 검증해 주는 동시에, 개방적이고 벤더 중립적인 통합 계층의 필요성을 더욱 부각시킵니다.

## 📖 선행연구 (Prior Art & Related Work)

"Software-Defined Building"과 "빌딩 운영 체제(Building OS)"는 이미 확립된 연구 계보입니다. Open SDB는 이 분야를 새로 정의하는 것이 아니라 그 위에서 출발합니다:

* **BOSS: Building Operating System Services** (Dawson-Haggerty et al., [USENIX NSDI 2013](https://www.usenix.org/conference/nsdi13/technical-sessions/presentation/dawson-haggerty))와 UC 버클리의 NSF 지원 *Software Defined Buildings* 프로그램(2012–2017)이 빌딩 OS 추상화를 처음 제시했습니다.
* **A Reference Architecture for Smart and Software-Defined Buildings** (Mazzara et al., IEEE SMARTCOMP 2019; [arXiv:1902.09464](https://arxiv.org/abs/1902.09464))는 4계층 참조 아키텍처를 제안했습니다.
* **Proposal of a Scalable Building Operating System Architecture** (Owaki et al., [IEEE COMPSAC 2024](https://ieeexplore.ieee.org/document/10633523))는 Software-Defined Building을 향한 확장 가능한 빌딩 OS 아키텍처와 데이터 모델을 탐구했습니다.
* **OpenBOS** (Paul et al., LBNL, [*Science and Technology for the Built Environment*, 2025](https://doi.org/10.1080/23744731.2024.2444819))는 가장 가까운 동작하는 시스템입니다: VOLTTRON, BACnet, ASHRAE 223P, BuildingMOTIF, BOPTEST를 통합한 그리드 반응형 의미 기반 상위 제어 플랫폼으로, 실제 건물 1개와 시뮬레이션 건물 1개에서 검증되었습니다.

**Open SDB가 기여하려는 지점:** 설계 단계 BIM 데이터와 이들 운영 플랫폼 *사이*의 인계 — 독점 저작 도구로부터의 검증된 추출, 시스템 간 식별자 출처 계보(provenance), 자동 검증 게이트입니다. 이는 OpenBOS 저자들 스스로 열거한 미해결 과제 — 의미모델에 없는 제어 파라미터, 모델 밖에 존재하는 제어 로직, 배포 전 커미셔닝 — 와 정확히 맞닿아 있습니다.

## 📂 리포지토리 구조

이 리포지토리는 현재 **리서치 문서**를 중심으로 운영되며, 프로젝트가 프로토타이핑 단계로 진입하면 **PoC(개념 증명) 코드**가 추가될 예정입니다.

```bash
open-sdb/
├── docs/
│   └── research/       # 발행된 리서치 보고서 (영어 / 한국어)
├── LICENSE
├── README.md           # 영문 문서 (English)
└── README.ko.md        # 본 문서

```

연구가 프로토타이핑으로 이어지면 `docs/integration/`(IFC, IDF, gbXML 등 이기종 포맷 통합 전략), `docs/architecture/`(SDB 코어 아키텍처 정의), `src/`(레거시 포맷 어댑터, SDB 오케스트레이션 코어, 폐쇄 루프 제어용 AI 에이전트 프로토타입)로 확장할 계획입니다.

## 📚 주요 연구 (Featured Research)

핵심 기술과 프레임워크에 대한 상세 리서치 문서입니다.

* **자동화된 BIM 추출 프레임워크 (Automated BIM Extraction Framework)** — *2026년 1월 발행, 2026년 7월 기술 동향 업데이트*
    * 독점 저작 도구(Autodesk Revit)의 BIM 데이터를 SDB 환경을 위한 오픈 포맷으로 완전 자동 추출하기 위한 연구입니다. 헤드리스 자동화 엔진(APS Design Automation, 로컬 배치 처리), GUID 기반 glTF + JSON + IFC 복합 데이터 연합 전략, IDS 기반 자동 검증을 다룹니다.
    * 📄 [한국어 문서 읽기](docs/research/automated-bim-extraction-framework.ko.md)
    * 📄 [Read in English](docs/research/automated-bim-extraction-framework.md)

## 🚀 로드맵 (Roadmap)

* [ ] **Phase 1: Research & Analysis** *(진행 중)* - 레거시 포맷 분석 및 통합 프로토콜 정의
    * [x] 자동화된 BIM 추출 프레임워크 연구 (2026년 1월 · 2026년 7월 업데이트)
    * [ ] BEM(EnergyPlus/IDF, gbXML) 통합 전략
    * [ ] SDB 코어 아키텍처 정의
* [ ] **Phase 2: Core Prototyping** - BIM 데이터와 시뮬레이션 엔진을 연결하는 SDB Core 개발
* [ ] **Phase 3: AI Orchestration** - 통합된 시스템 위에서 폐쇄 루프를 제어하는 Physical AI 구현

## 🤝 기여하기 (Contribution)

이 프로젝트는 건축 기술의 새로운 가능성을 탐구합니다. 우리의 작업 방식은 **AI 지원(AI-assisted) · 도메인 전문가 검토(domain-reviewed) · 재현 가능(reproducible)** 을 원칙으로 합니다. AI와 멀티 에이전트 기술로 연구를 가속하되, 모든 주장과 산출물은 도메인 전문가가 검토하고 검증할 수 있어야 합니다. 다양한 분야의 전문 기술을 하나의 지능형 시스템으로 통합하는 데 열정을 가진 아키텍트, 엔지니어, 개발자분들의 참여를 환영합니다.

## 📜 라이선스 (License)

이 프로젝트는 MIT 라이선스를 따릅니다. 자세한 내용은 [LICENSE](LICENSE) 파일을 참고하세요.
