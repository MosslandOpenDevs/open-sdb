# 자동화된 BIM 데이터 추출 및 디지털 트윈 상호 운용성을 위한 포괄적 프레임워크 연구 보고서

> **발행:** 2026년 1월 · **기술 동향 재검증:** 2026년 7월
> 본 보고서가 인용한 모든 도구·서비스·표준을 2026년 7월에 1차 출처 기준으로 재검증했습니다. 핵심 결론과 권고안은 여전히 유효하며, 변경된 내용은 문서 말미의 [부록 B: 기술 동향 업데이트](#부록-b-기술-동향-업데이트-2026년-7월)를 참고하세요.

## 📋 요약 (Executive Summary)

건축, 엔지니어링, 건설(AEC) 산업은 디지털 트윈과 시설 유지관리(FM) 시스템의 통합에 있어 근본적인 기술적 장벽에 직면해 있습니다. 그 중심에는 업계 표준 저작 도구인 Autodesk Revit의 `.rvt` 파일 형식이 가진 독점적이고 폐쇄적인 특성이 존재합니다. 본 보고서는 Revit 파일을 **설계 단계 형상·파라미터의 '진실 공급원(Single Source of Truth)'** 으로 설정하고, 설계부터 유지보수 단계까지 수작업 없는 완전 자동화된 데이터 파이프라인을 구축하기 위한 기술적 방법론을 심층 분석합니다. (운영 시스템 — BMS, CMMS, 시계열 저장소, 제어 정책 — 은 각자의 데이터 권위(authority)를 유지하며, 파이프라인이 보장해야 할 것은 이들 사이의 추적 가능한 출처 계보(provenance)입니다.)

연구 결과, Revit 엔진 없이 순수 오픈 소스 라이브러리만으로 `.rvt` 파일의 파라메트릭 형상과 메타데이터를 완전한 무결성으로 '읽는(Read)' 것이 검증된 라이브러리는 확인되지 않았으며, 이를 시도할 경우 데이터 손실 위험이 매우 높음이 밝혀졌습니다.

따라서 본 보고서는 **'헤드리스(Headless)' 자동화 엔진** 을 중개자로 사용하여 Revit 데이터를 해방시키고, 이를 다중 오픈 포맷(Multi-Format Composite) 전략으로 변환하여 디지털 트윈에 통합하는 하이브리드 아키텍처를 제안합니다.

특히 단일 포맷(예: IFC 또는 glTF) 변환 시 발생하는 기하학적 형상 변형이나 속성 정보 누락 문제를 해결하기 위해, 형상용 `glTF/Fragments`, 의미론적 데이터용 `JSON/SQL`, 그리고 개방형 기록 아카이빙용 `IFC`를 병렬로 추출하고 **GUID(Global Unique Identifier)** 를 통해 재결합하는 '복합 데이터 연합(Composite Data Federation)' 모델을 제시합니다.

---

## 1. 독점적 .rvt 포맷의 구조적 난제와 접근 한계

### 1.1. OLE 복합 파일 구조와 바이너리 난독화

Revit의 `.rvt` 파일은 단순한 문서 파일이 아니라, Microsoft의 **OLE(Object Linking and Embedding)** 복합 파일 기술을 기반으로 한 복잡한 구조적 스토리지 데이터베이스입니다. 이 파일 내부에는 형상 정보, 매개변수, 관계 데이터가 독점적인 스키마로 저장되어 있으며, 이 스키마는 Autodesk의 소프트웨어 릴리스 주기에 맞춰 매년 변경됩니다.

오픈 소스 커뮤니티의 연구에 따르면, Revit API를 통하지 않고 디스크 상의 `.rvt` 파일을 직접 바이너리 레벨에서 파싱하려는 시도는 지속적으로 있어왔으나, 완벽한 호환성을 제공하는 오픈 소스 라이브러리는 현재 존재하지 않습니다. 일부 파이썬 기반의 라이브러리(예: `revit-extractor`)가 존재하지만, 이는 독립적인 파서가 아니라 설치된 Revit 애플리케이션을 제어하는 래퍼(Wrapper)에 불과합니다. 즉, 파일을 '읽기' 위해서는 반드시 Revit 라이선스와 설치된 소프트웨어가 필요하다는 근본적인 제약이 따릅니다.

### 1.2. 파라메트릭 엔진의 필요성

Revit 데이터 추출의 핵심적인 어려움은 데이터가 정적인 형상(Static Mesh)이 아닌, **구속조건과 관계성(Parametric Constraints)** 으로 정의되어 있다는 점입니다. 예를 들어, Revit 내부의 '벽(Wall)'은 3D 좌표의 집합이 아니라 *"레벨 1에서 시작하여 3,000mm 높이를 가지며, 특정 패밀리 유형을 따르는 선형 객체"* 로 정의됩니다.

이러한 데이터를 디지털 트윈에서 시각화 가능한 형상으로 변환하기 위해서는, 이러한 구속조건을 계산하여 최종적인 3D 메쉬를 생성할 수 있는 '솔버(Solver)' 엔진이 필요합니다. 현재 이 역할을 수행할 수 있는 것은 Autodesk의 Revit 엔진과, 이를 리버스 엔지니어링한 ODA(Open Design Alliance)의 **BimRv SDK** 뿐입니다. ODA BimRv는 상용 라이선스이므로, 순수 오픈 소스 전략을 추구하는 경우 선택지에서 제외되거나 비용 구조 검토가 필요합니다.

### 1.3. '읽기'에서 '자동화된 내보내기'로의 패러다임 전환

따라서, 오픈 소스로 Revit 파일을 직접 읽으려는 시도는 데이터 무결성을 보장할 수 없습니다. 대신, 본 보고서는 **"Revit 엔진을 자동화하여 오픈 포맷으로 데이터를 내보내는 파이프라인"** 을 구축하는 것을 현실적이고 강력한 대안으로 정의합니다. 이는 사용자의 개입 없이 서버 단에서 수행되는 '헤드리스(Headless)' 자동화를 통해 달성될 수 있습니다.

---

## 2. 데이터 추출 자동화를 위한 헤드리스(Headless) 아키텍처

수작업 없는 디지털 트윈 업데이트를 위해서는 Revit의 GUI(그래픽 사용자 인터페이스)를 거치지 않고 백그라운드에서 실행되는 자동화 프로세스가 필수적입니다. 이를 위한 두 가지 주요 기술적 경로를 분석합니다.

### 2.1. Autodesk Platform Services (APS) Design Automation

과거 Forge로 불리던 APS의 **Design Automation API for Revit (DA4R)** 은 클라우드 기반의 헤드리스 Revit 엔진을 제공하는 유일한 공식 솔루션입니다. 이 서비스는 사용자가 로컬 장비에 Revit을 설치하지 않고도, 클라우드 상에서 Revit 애드인(Plugin)을 실행하여 데이터를 처리할 수 있게 합니다.

#### 2.1.1. 기술적 워크플로우 및 구현

APS Design Automation을 활용한 데이터 추출 파이프라인은 다음과 같이 구성됩니다:

1. **AppBundle 개발:** 개발자는 `IExternalDBApplication` 인터페이스를 구현한 C# 플러그인을 작성합니다. 이 인터페이스는 UI 관련 코드를 제외하고 Revit의 DB 레벨 API에만 접근하도록 설계되어 있어, 서버 환경에서의 안정성을 보장합니다.
2. **Activity 정의:** 특정 Revit 엔진 버전(예: `Autodesk.Revit+2024`)과 실행할 AppBundle, 그리고 입출력 파라미터를 정의한 'Activity'를 APS에 등록합니다.
3. **WorkItem 실행:** 사용자가 BIM 파일을 클라우드 스토리지(BIM 360, ACC, S3 등)에 업로드하면, 웹훅(Webhook)이 이를 감지하여 'WorkItem'을 트리거합니다. WorkItem은 입력 파일의 URL과 추출된 데이터가 저장될 출력 URL을 포함합니다.

#### 2.1.2. 비용 및 확장성 분석

APS는 사용량 기반(Token) 과금 모델을 따르며, 대규모 병렬 처리가 가능합니다. 수천 개의 모델을 동시에 업데이트해야 하는 엔터프라이즈급 디지털 트윈 환경에서 가장 안정적이고 확장 가능한 옵션입니다. 또한, Autodesk Construction Cloud(ACC)와의 네이티브 통합을 통해 파일 버전이 변경될 때마다 자동으로 추출 로직을 수행하는 이벤트 드리븐(Event-Driven) 아키텍처를 쉽게 구현할 수 있습니다.

### 2.2. 로컬 배치 자동화 (RevitCoreConsole & Revit Batch Processor)

클라우드 비용이나 데이터 주권(Data Sovereignty) 문제로 인해 온프레미스 환경을 선호하는 경우, 로컬 서버에서 자동화를 구현해야 합니다. Revit은 공식적으로 완전한 CLI 모드를 지원하지 않지만, `RevitCoreConsole`이나 자동화 래퍼 도구를 통해 유사한 환경을 구축할 수 있습니다.

#### 2.2.1. Revit Batch Processor (RBP)

Revit Batch Processor는 오픈 소스 도구로, Revit을 백그라운드에서 실행하고 사전에 정의된 Python 또는 Dynamo 스크립트를 순차적으로 실행하는 기능을 제공합니다.

* **작동 원리:** RBP는 작업 대기열(Queue)을 관리하며, 각 파일에 대해 Revit 프로세스를 띄우고, 스크립트를 주입하여 실행한 뒤, 프로세스를 종료합니다. 이 과정에서 발생하는 팝업이나 경고창을 자동으로 처리하여 프로세스가 중단되는 것을 방지합니다.
* **한계:** Revit 라이선스가 설치된 윈도우 환경이 필요하며, 병렬 처리를 위해서는 다수의 가상 머신(VM)과 라이선스가 필요하여 하드웨어 리소스 관리가 복잡해질 수 있습니다.

#### 2.2.2. pyRevit CLI

pyRevit은 강력한 오픈 소스 애드인 개발 프레임워크로, `pyrevit run` 명령어를 통해 CLI 환경에서 특정 모델에 대한 Python 스크립트를 실행할 수 있는 기능을 제공합니다. 이를 Windows Task Scheduler나 Jenkins와 결합하면, 야간에 서버의 모든 모델을 스캔하고 변경된 파일에서 데이터를 추출하는 자동화 서버를 구축할 수 있습니다.

### 📊 표 1. 자동화 엔진 비교 분석

| 기능적 요소 | APS Design Automation (Cloud) | Revit Batch Processor (Local) | pyRevit CLI (Local) |
| --- | --- | --- | --- |
| **UI 의존성** | 완전한 Headless (UI 없음) | GUI 자동 제어 (화면 필요) | GUI 자동 제어 (화면 필요) |
| **인프라 요구사항** | 없음 (SaaS) | 고사양 워크스테이션/서버 | 고사양 워크스테이션/서버 |
| **라이선스** | Flex Token (종량제) | Revit 라이선스 (고정비) | Revit 라이선스 (고정비) |
| **안정성** | 최상 (샌드박스 환경) | 중 (크래시 발생 가능성) | 중 (스크립트 의존) |
| **처리 속도** | 대규모 병렬 처리 가능 | 하드웨어 성능에 종속 | 하드웨어 성능에 종속 |
| **주요 용도** | 실시간 디지털 트윈 동기화 | 야간 일괄 처리, 아카이빙 | 비정기적 데이터 추출 |

---

## 3. 정보 손실 없는 데이터 확보를 위한 다중 오픈 포맷 전략

Revit 데이터를 단일 포맷으로 변환할 때 발생하는 정보 손실(Data Entropy)은 디지털 트윈 구축의 가장 큰 위험 요소입니다. 예를 들어, 시각화에 최적화된 포맷은 메타데이터를 누락하고, 데이터 중심 포맷은 형상을 단순화하는 경향이 있습니다. 이를 해결하기 위해 본 보고서는 **'복합 데이터 연합(Composite Data Federation)'** 전략을 제안합니다.

### 3.1. 단일 포맷 변환의 딜레마

* **IFC (Industry Foundation Classes):** BIM 데이터의 국제 표준(IFC 4.3, ISO 16739-1:2024로 발행)으로, 의미론적 정보(속성, 관계) 보존에 강점이 있습니다. 그러나 형상 변환 과정에서 BREP(Boundary Representation)과 Mesh 간의 변환 오류가 발생할 수 있으며, 웹 브라우저에서 직접 렌더링하기에는 파일 구조가 무겁고 복잡합니다. 또한, Revit의 내장 IFC 익스포터 설정에 따라 일부 파라미터가 누락될 수 있습니다.
* **glTF (Graphics Library Transmission Format):** '3D의 JPEG'라 불리는 웹 표준 포맷으로, 시각화 성능이 뛰어나고 파일 크기가 작습니다. 그러나 glTF는 기본적으로 형상과 재질 정보에 집중되어 있어, BIM의 복잡한 계층 구조, 패밀리 유형, 비형상 속성(예: 열관류율, 제조사 정보)을 담기에는 표준 스펙의 한계가 있습니다.

### 3.2. 복합 포맷 전략: 시각, 데이터, 원본의 분리 및 재결합

정보 누락을 원천 차단하기 위해, 하나의 Revit 모델을 용도별로 특화된 세 가지 포맷으로 분리하여 추출하고, 이를 디지털 트윈 플랫폼에서 **GUID(Global Unique Identifier)** 를 키(Key)로 사용하여 실시간으로 결합하는 방식을 권장합니다.

#### 구성 요소 A: 시각적 트윈 (Visual Twin) - `glTF` / `Fragments`

* **목적:** 고성능 웹 시각화, 사용자 인터랙션, 공간 인지.
* **기술:** IfcOpenShell의 `IfcConvert` 도구를 사용하여 IFC를 glTF로 변환하거나, That Open Company의 Components 라이브러리를 사용하여 Fragments 포맷으로 변환합니다.
* **핵심 요구사항:** 변환 시 형상 데이터에 반드시 원본 Revit 객체의 **GUID** 가 실려야 합니다 — `extras`/`userData` 필드에 넣거나, 노드 이름에 포함시키는 방식(IfcConvert가 GUID를 보존하는 방식)을 사용합니다. 이 GUID는 시각 정보와 의미 정보를 연결하는 핵심 고리가 됩니다.
* **최적화:** Draco 압축 알고리즘을 적용하여 형상 데이터의 용량을 절감하면서도 시각적 정밀도를 유지합니다. 40~60% 수준의 절감이 일반적으로 보고되지만 실제 비율은 모델에 따라 달라지므로 프로젝트별 벤치마크로 확인해야 합니다.

#### 구성 요소 B: 의미론적 트윈 (Semantic Twin) - `JSON` / `SQL`

* **목적:** 데이터 쿼리, 분석, 유지보수 이력 관리, ERP 연동.
* **기술:** Revit API(자동화 스크립트) 또는 IfcOpenShell-Python을 사용하여 모든 객체의 속성(Parameter) 정보를 추출합니다.
* **구조:** 데이터는 객체의 GUID를 Primary Key로 하는 관계형 데이터베이스(RDBMS)나 문서형 데이터베이스(NoSQL)에 저장됩니다.
* **예시:** `{"GUID": "1a2b...", "Type": "Wall", "FireRating": "2hr", "InstallDate": "2023-10-01"}`


* **이점:** 형상 파일(glTF)을 다시 로드하지 않고도 속성 데이터만 가볍게 업데이트하거나 검색할 수 있어 시스템 성능이 극대화됩니다.

#### 구성 요소 C: 표준 트윈 (Canonical Twin) - `IFC`

* **목적:** 개방형 포맷 기록 스냅샷, 타 시스템과의 상호 운용성, 원본 데이터 백업. (IFC 스냅샷이 법적 효력을 갖는 원본인지는 계약·관할·CDE 정책에 따라 결정됩니다. 자동으로 법적 원본이 되는 것이 아니라 개방형 교환/기록 스냅샷으로 취급해야 합니다.)
* **기술:** Revit의 네이티브 IFC 내보내기 기능을 사용하되, 정보 누락을 방지하기 위해 사용자 정의 속성 세트(User Defined Property Sets) 매핑 파일을 적용하여 모든 Revit 파라미터가 IFC 속성으로 변환되도록 설정합니다.
* **검증:** `IfcTester`를 사용하여 추출된 IFC 파일이 프로젝트의 정보 요구사항(IDS)을 충족하는지 자동 검증합니다.

---

## 4. 오픈 소스 인터페이스 및 라이브러리 상세 분석

제안된 복합 전략을 구현하기 위해 활용 가능한 오픈 소스 도구들을 심층 분석합니다.

### 4.1. IfcOpenShell: 기하학 및 데이터 처리의 핵심 엔진

IfcOpenShell은 IFC 파일을 처리하기 위한 가장 강력한 오픈 소스 라이브러리입니다. Revit에서 1차적으로 IFC로 추출된 데이터를 가공하여 시각화 모델(glTF)과 데이터 모델(JSON)을 생성하는 후처리 엔진으로 활용됩니다.

* **IfcConvert:** CLI 도구로, IFC 형상을 glTF, OBJ, DAE 등으로 변환합니다. GUID 연결성은 노드 네이밍을 통해 보존됩니다. 기본적으로 glTF 노드 이름에 각 요소의 GlobalId가 (`product-<확장된 uuid>-<컨텍스트>` 형식으로) 포함되며, `--use-element-guids` 옵션을 쓰면 22자 원본 IFC GlobalId가 노드 이름으로 사용됩니다. (참고: `--include attribute GlobalId <id>`는 메타데이터와 무관한 옵션으로, *어떤* 요소를 변환할지 선택하는 필터입니다.) 또한, 테셀레이션(Tessellation) 정밀도를 조절하여 곡면의 품질과 파일 크기 간의 균형을 맞출 수 있습니다.
* **IfcOpenShell-Python:** 파이썬 바인딩을 통해 IFC 파일 내부의 깊은 계층 구조를 탐색하고, 사용자 정의 속성을 추출하거나 데이터를 검증하는 스크립트를 작성할 수 있습니다. 이는 "의미론적 트윈"을 구축하는 데 핵심적인 역할을 합니다.

### 4.2. Speckle: 객체 기반의 실시간 데이터 교환

Speckle은 파일 기반 교환의 한계를 넘어, 데이터를 객체(Object) 단위로 스트리밍하는 오픈 소스 플랫폼입니다.

* **작동 방식:** Speckle Connector(Revit 플러그인)는 Revit 객체를 Speckle의 중립적인 JSON 스키마로 직렬화(Serialization)하여 Speckle Server로 전송합니다. 이 과정에서 형상과 속성이 모두 보존됩니다.
* **자동화 통합:** Speckle Connector는 APS Design Automation 환경에서도 실행 가능하도록 설계될 수 있어, 클라우드 상에서 Revit 파일이 업데이트되는 즉시 Speckle 데이터베이스를 동기화하는 파이프라인 구축이 가능합니다.
* **강점:** 별도의 파일 변환 과정 없이 API를 통해 디지털 트윈 웹 애플리케이션에서 직접 데이터를 쿼리하고 3D로 시각화할 수 있는 뷰어(Speckle Viewer)를 제공합니다.

### 4.3. That Open Company (구 IFC.js): 웹 네이티브 BIM

That Open Company는 웹 브라우저에서 BIM 데이터를 고성능으로 처리하기 위한 자바스크립트 라이브러리 모음입니다.

* **Fragments 기술:** 대용량 BIM 모델을 웹에서 효율적으로 렌더링하기 위해 고안된 'Fragments' 포맷을 사용합니다. 이는 중복되는 형상(예: 반복되는 기둥)을 인스턴싱하여 메모리 사용량을 획기적으로 줄입니다.
* **활용:** 추출된 IFC 데이터를 웹 애플리케이션에서 로드할 때, Three.js 기반의 컴포넌트들을 사용하여 고성능 뷰어를 구축할 수 있습니다.

---

## 5. 데이터 무결성 보장 및 자동 검증 프로세스

자동화된 시스템에서 가장 중요한 것은 데이터의 신뢰성입니다. 사람이 개입하지 않으므로, 시스템이 스스로 데이터의 품질을 검증하고 오류를 차단해야 합니다.

### 5.1. IDS (Information Delivery Specification) 기반 검증

buildingSMART의 IDS 표준을 활용하여 기계 판독 가능한 데이터 요구사항을 정의합니다.

* **검증 로직:** 예를 들어, *"모든 펌프(IfcPump)는 '유지보수' 속성 세트에 '설치일자' 속성을 반드시 포함해야 한다"* 는 규칙을 XML 형식의 IDS 파일로 작성합니다.
* **자동화:** 데이터 추출 파이프라인의 마지막 단계에서 `IfcTester` 도구를 실행하여 생성된 IFC 파일이 IDS 규칙을 준수하는지 검사합니다. 검증 실패 시, 해당 모델의 디지털 트윈 반영을 차단하고 설계 팀에 자동으로 수정 요청 알림을 발송합니다.

### 5.2. 기하학적 정합성 검증 (Geometric Validation)

Revit 원본과 변환된 glTF/IFC 모델 간의 형상 차이를 감지하기 위해 바운딩 박스(Bounding Box) 비교나 체적(Volume) 비교 알고리즘을 적용합니다.

* **구현:** 자동화 스크립트에서 Revit API로 전체 객체의 체적 합계를 계산하고, 변환 후 IfcOpenShell로 다시 체적을 계산하여 오차 범위를 확인합니다. 오차가 임계값(예: 0.1%)을 초과하면 변환 실패로 간주합니다.
* **한계:** 총 체적 비교는 대략적인 스모크 테스트일 뿐입니다. 상쇄되는 오류(누락된 요소를 과대 계산된 요소가 상쇄하는 경우)는 총합에서 드러나지 않습니다. 프로덕션 파이프라인에서는 요소 단위 검증 — 요소 개수와 누락/추가 객체, 배치(placement), 바운딩 박스, 면적·체적, 단위, 속성 커버리지 — 을 병행해야 합니다.

---

## 6. 결론 및 제언: 완전 자동화된 파이프라인 구축 로드맵

본 연구는 Revit 파일의 폐쇄성을 극복하고 디지털 트윈을 위한 지속 가능한 데이터 파이프라인을 구축하기 위해 다음과 같은 로드맵을 제안합니다.

1. **엔진의 분리:** 오픈 소스 라이브러리로 .rvt를 직접 읽으려는 시도를 멈추고, APS Design Automation이나 로컬 자동화 서버를 도입하여 데이터 추출의 '실행 엔진'을 확보하십시오. 이는 데이터 무결성을 보장하는 유일한 방법입니다.
2. **데이터의 연합:** 단일 파일 변환 대신, `glTF`(형상), `JSON`(데이터), **`IFC`(아카이브)** 의 3중 구조로 데이터를 추출하십시오. 이들은 GUID를 통해 디지털 트윈 플랫폼에서 하나로 통합되어야 합니다.
3. **개방형 표준 채택:** 데이터 교환의 매개체로 Speckle이나 IFC와 같은 개방형 표준을 사용하여 특정 벤더에 대한 종속성(Vendor Lock-in)을 제거하십시오.
4. **자동 검증 체계:** `IfcTester`와 IDS를 활용한 자동 품질 검사 게이트(Gate)를 파이프라인에 배치하여, 검증되지 않은 데이터가 유지보수 시스템으로 유입되는 것을 원천 봉쇄하십시오.

이러한 접근 방식은 초기 구축 난이도가 높을 수 있으나, 장기적으로 설계 변경 사항이 수작업 없이 즉시 운영 단계로 반영되는 진정한 의미의 '살아있는 디지털 트윈(Living Digital Twin)'을 실현하는 가장 견고한 토대가 될 것입니다.

---

## 부록 A: 기술 구현 참조

### 참조 1: IfcOpenShell을 이용한 속성 추출 및 JSON 변환 (Python)

IFC 파일에서 속성 데이터를 추출하여 의미론적 트윈(JSON)을 생성하는 코드 예시입니다.

```python
import ifcopenshell
import ifcopenshell.util.element
import json

def extract_properties_to_json(ifc_file_path, output_path):
    # IFC 파일 로드
    model = ifcopenshell.open(ifc_file_path)
    data_registry = {}

    # IfcBuildingElement가 아니라 IfcElement를 순회해야 합니다:
    # IfcBuildingElement(IFC4x3에서 IfcBuiltElement로 개명)는 건축 요소만
    # 포함하고, FM/디지털 트윈 추출기가 반드시 다뤄야 할 MEP 설비
    # (IfcDistributionElement: 펌프, 팬, 센서 등)를 전부 놓칩니다
    for element in model.by_type("IfcElement"):
        # 피처 요소(개구부, 보이드)와 가상 경계 요소는 제외
        if element.is_a("IfcFeatureElement") or element.is_a("IfcVirtualElement"):
            continue

        # get_psets()는 element.IsDefinedBy를 수동 순회하는 대신
        # 속성 세트와 수량 세트를 모두 해석하며, 타입에서 상속되는
        # 속성 세트까지 포함합니다 (should_inherit=True)
        psets = ifcopenshell.util.element.get_psets(element)

        data_registry[element.GlobalId] = {
            "type": element.is_a(),
            "name": element.Name,
            "properties": psets,
        }

    # JSON 파일로 저장
    with open(output_path, 'w', encoding='utf-8') as f:
        json.dump(data_registry, f, ensure_ascii=False, indent=4, default=str)

# 실행
extract_properties_to_json("model.ifc", "semantic_twin.json")

```

*(프로덕션 추출기는 단위, 재료, 시스템/포트, 공간 포함 관계까지 추가로 수집해야 합니다 — `ifcopenshell.util.unit`, `ifcopenshell.util.system`, `ifcopenshell.util.element.get_container` 참조.)*

### 참조 2: 도구별 기능 비교 요약

| 도구 (Tool) | 라이선스 | Revit 직접 읽기 | 형상 처리 (Geometry) | 데이터 처리 (Data) | 주요 역할 |
| --- | --- | --- | --- | --- | --- |
| **IfcOpenShell** | LGPL-3.0 | 불가 (IFC 필요) | 높음 (C++ 커널) | 높음 (Python 스크립팅) | IFC 변환, 분석, 검증 |
| **Speckle** | Apache 2.0 (코어) | 플러그인 필요 | 높음 (객체 스트리밍) | 높음 (API 쿼리) | 데이터 교환 허브, 웹 뷰어 |
| **APS DA4R** | 상용 (유료) | 가능 (네이티브) | 최상 (엔진 사용) | 최상 (API 접근) | 자동화 실행 엔진 (헤드리스) |
| **Revit Batch Processor** | GPL-3.0 | 불가 (GUI 제어) | 해당 없음 | 해당 없음 | 로컬 자동화 오케스트레이션 |
| **That Open Company** | MIT | 불가 (IFC 필요) | 높음 (Fragments) | 중 (웹 기반) | 웹 기반 BIM 애플리케이션 구축 |

---

## 부록 B: 기술 동향 업데이트 (2026년 7월)

*본 보고서는 2026년 1월에 발행되었습니다. 2026년 7월, 보고서가 인용한 모든 도구·서비스·표준을 1차 출처(공식 릴리스 노트, 벤더 공지, 표준화 기구 발표, 패키지 레지스트리) 기준으로 재검증했습니다. 핵심 아키텍처 — 헤드리스 추출 엔진, GUID 기반 복합 데이터 연합, 자동 검증 게이트 — 는 여전히 유효합니다. 본 부록은 주변 기술 지형에서 무엇이 바뀌었고, 그것이 보고서의 권고안에 어떤 영향을 주는지 기록합니다.*

### B.1. Autodesk Platform Services: 명칭 변경, 엔진 수명 주기, 과금 체계

* **"Design Automation API"가 "Automation API"로 개명되었습니다.** 2025년 6월 30일부로 Autodesk는 Design Automation API를 **Automation API**로 개명했으며, DA4R은 **Revit Automation API**라는 이름으로 제공됩니다. 모든 v3 엔드포인트·워크플로우·기존 코드는 그대로 유지되어 마이그레이션이 필요 없습니다. 여전히 유일한 공식 클라우드 헤드리스 Revit 엔진이며, 2.1.1절의 `IExternalDBApplication` 변환 가이드도 공식 튜토리얼 기준으로 유효합니다.
* **데이터 추출에 더 이상 항상 엔진이 필요하지는 않습니다.** 읽기 중심 추출을 위해 Autodesk는 엔진 없이 쓸 수 있는 공식 경로를 여럿 제공합니다: **AEC Data Model API**(Revit 2024+ 클라우드 모델의 요소/속성을 GraphQL로 조회, 2024년 6월 GA, 세밀 형상 접근은 2025년 9월부터 퍼블릭 베타), 오래 제공되어 온 **Model Derivative API**(계층·속성·형상 파생물, IFC/OBJ 출력 포함), **Data Exchange API**(부분 데이터 교환, GraphQL API는 아직 베타). 엔진 기반 Automation API는 임의의 Revit API 애드인 코드를 실행할 수 있는 유일한 경로로 남아 있으며 — 쓰기 워크플로우, 임의 연산/내보내기, Autodesk 클라우드 밖의 `.rvt` 파일 처리에는 여전히 필수입니다. 따라서 2장은 "엔진 자동화가 유일한 선택지"가 아니라 "요구사항에 따라 추출 경로를 선택"하는 구도로 읽어야 합니다.
* **Revit 2027이 2026년 4월 7일 출시되었습니다.** SDK가 .NET 10으로 이동해 모든 애드인의 재빌드가 필요하며, 64비트 Windows 11에서만 구동됩니다. 또한 **Autodesk Assistant**(대화형 AI, 테크 프리뷰)와 공식 **Revit MCP Server**(테크 프리뷰)가 도입되었는데, 후자는 Autodesk가 처음으로 공식 지원하는 '실행 중인 Revit 세션에 대한 AI 에이전트 브리지'입니다.
* **엔진 수명 주기 관리가 필수가 되었습니다.** Automation API는 2026년 4월에 Revit 2027 엔진을 추가했으며, 현재 참조 가능한 엔진은 **Revit 2022–2027**입니다. 2019/2020 엔진은 2025년 9월 29일, 2021 엔진은 2026년 3월 29일에 제거되었고, 2022/2023 엔진은 '4년 지원 + 2년 지원 중단(Deprecation)' 수명 주기 정책에 따라 이미 지원 중단 상태입니다. 특정 엔진 버전을 고정(예: 2.1.1절의 `Autodesk.Revit+2024`)한 파이프라인은 공지된 제거 일정에 맞춰 마이그레이션을 계획해야 합니다.
* **과금 체계가 2025년 12월 8일 변경되었습니다.** APS는 공식적인 2단계(Free/Paid) 비즈니스 모델로 전환했고, Automation API는 4개의 '종량 과금(rated)' API 중 하나가 되었습니다. 기존 클라우드 크레딧은 2025년 말에 폐지되었고 Automation API 요금은 인상되었으며, 결제는 Flex 토큰 선불 또는 월별 후불(Pay-as-You-Go) 방식에 월간 무료 사용 한도가 결합된 구조입니다. 2.1.2절의 비용 분석은 새 요금 기준으로 재산정이 필요합니다.
* **Autodesk Tandem이 명시적으로 "Physical AI"를 표방하기 시작했습니다**(예: 2026년 4월, 공항·스마트 빌딩·제조·물류를 겨냥한 Globant의 "Tandem Digital Twin Solution Provider" 파트너십). 본 보고서가 제시한 디지털 트윈 방향성에 대한 벤더 차원의 검증이라 할 수 있습니다.

### B.2. IfcOpenShell 툴체인

* 현재 안정 버전은 **IfcOpenShell 0.8.5**(2026년 4월 13일, LGPL-3.0-or-later, Python 3.10–3.14)이며, **ifctester도 동일 버전 0.8.5**로 릴리스되어 콘솔·JSON·ODS·HTML·BCF 형식의 검증 리포트를 지원합니다. 2026년 7월까지 0.8.6-alpha 데일리 빌드가 계속 배포되고 있어 프로젝트는 매우 활발히 유지되고 있습니다.
* 완전한 스키마 파싱은 **IFC4x3 Add2(IFC 4.3)** 까지 지원합니다. **IFC5/IFCX는 아직 지원되지 않습니다.**
* BlenderBIM 애드온의 **Bonsai** 개명이 완료되었습니다(안정 버전 0.8.5, 2026년 4월). 현재 공식 문서에서는 옛 이름이 더 이상 쓰이지 않습니다.
* IfcConvert 세부 사항: glTF 출력은 **바이너리 `.glb`(glTF 2.0)** 형식이며, **Draco 압축 플래그는 내장되어 있지 않으므로** 3.2절(구성 요소 A)에서 권장한 Draco 압축은 변환 후 `gltf-pipeline` 등으로 후처리해야 합니다.
* **정정 (2026년 7월):** 4.1절 원문은 `--include attribute GlobalId`를 "메타데이터 연결성을 보장하는" 옵션으로 설명했으나, 공식 사용 문서 기준으로 이는 요소 선택 *필터*이며 출력 메타데이터에는 아무 영향이 없습니다. glTF에서의 GUID 연결성은 실제로는 노드 네이밍에서 나옵니다 — 기본 노드 이름에 GlobalId가 포함되고(`product-<uuid>-<컨텍스트>`), `--use-element-guids`를 쓰면 22자 원본 GlobalId가 노드 이름이 됩니다. 4.1절 본문과 부록 A의 코드 예제(`IfcBuildingElement`만 순회하여 MEP 설비를 모두 놓치고, `ifcopenshell.util.element.get_psets()` 대신 `IsDefinedBy`를 수동 순회하던 문제)를 모두 본문에서 수정했습니다.

### B.3. IFC 및 IDS 표준

* IFC 4.3의 정확한 인용 표기는 **ISO 16739-1:2024**(스키마 4.3.2.0, ISO 발행 2024년 3월, 유럽 표준 EN ISO 16739-1:2024로도 채택)입니다. 3.1절 본문도 이에 맞게 수정했습니다.
* **차세대 IFC(구 "IFC5")가 2026년 3월 포르투(Porto) buildingSMART 국제 서밋에서 "IFC X"로 개명되었습니다.** 서버/API 접근, 증분 데이터 교환, JSON 데이터셋을 지향하는 웹 네이티브 재설계로, 아직 개념 증명(PoC) 단계이며 공식 릴리스가 없습니다 — 즉, 아직 파이프라인의 구현 대상이 아닙니다.
* **IFC 4.4** 공식 프로젝트 제안이 확정되었습니다(터널, 토공, IFC 4.3의 누적 이슈 반영). ISO 표준화는 대략 2028년 완료가 목표로, IFC 4.x 계열의 구체적인 다음 단계입니다.
* **IDS는 여전히 1.0 버전**(2024년 6월)이 최신이며, 1.1/2.0은 계획 단계에 머물러 있고 buildingSMART의 IDS 소프트웨어 인증 프로그램도 아직 없습니다. 5.1절의 권고대로 `IfcTester`가 실질적인 검증 도구입니다. 여기에 더해, buildingSMART의 공식 오픈소스 **IFC Validation Service**(validate.buildingsmart.org, STEP 구문·스키마 적합성·규범 규칙 검사)가 사실상의 표준 적합성 검사기로 자리 잡았으므로, IDS 검증과 병행하는 2차 게이트로 도입할 가치가 있습니다.

### B.4. Speckle

* Speckle의 **"차세대(v3)" 커넥터가 2025년 6월 1일부터 기본이 되었습니다.** 레거시 v2 커넥터는 2026년 1월 1일부로 공식 지원 중단(deprecated) 상태가 되었으며, v2와 v3 커넥터로 발행된 모델은 **상호 호환되지 않습니다.** 신규 파이프라인은 v3 SDK(`speckle-sharp-connectors`, `speckle-sharp-sdk`)를 대상으로 해야 합니다.
* 코어 서버는 여전히 Apache 2.0이지만, **워크스페이스/엔터프라이즈 서버 모듈에는 독점 라이선스(Speckle Enterprise Edition)가 적용**되었습니다. 또한 **Speckle Automate는 2025년 11월 GA가 되었으나 Enterprise 플랜 전용이며 Speckle이 호스팅하는 인프라에서만 실행**됩니다(자체 호스팅 불가). 클라우드 요금제는 Explore(무료) / Team(월 $99) / Enterprise(협의)이며, 오픈소스 서버의 자체 호스팅은 여전히 무료입니다.
* **4.2절에서 제시한 'Speckle Revit 커넥터를 APS Design Automation 안에서 실행'하는 패턴은 폐기된 v2 아키텍처를 전제로 한 것입니다.** v3 커넥터는 데스크톱(WebView/DUI3) 기반이며 공식적인 헤드리스 실행 경로가 없습니다. 커넥터 없이 데이터를 수집하는 Speckle의 현행 공식 답은 **호스티드 Revit 파일 임포터**(2025년 12월)로, Revit 커넥터나 라이선스 없이 `.rvt` 파일을 직접 수집하며, ACC 동기화를 지원하는 Navisworks(`.nwd`/`.nwc`) 임포터(2026년 4월)도 추가되었습니다.
* 회사 동향: 2026년 4월 Suffolk Technologies가 Speckle에 투자했으며(2024년 10월 $12.5M 시리즈 A에 이은 것), 플랫폼은 "AI-ready design intelligence"로 재포지셔닝 중입니다(Speckle Intelligence 프리뷰, Model Validation 베타).

### B.5. That Open Company: Fragments의 오픈 바이너리 포맷 전환

* `@thatopen/components`가 **3.x 라인으로 이동**했습니다(3.1.0: 2025년 7월, 최신 3.4.6: 2026년 5월, MIT). 본 보고서 작성 시점에 최신이던 2.x API 기준 코드는 브레이킹 체인지가 있는 한 개 메이저 버전 뒤처진 상태입니다.
* **Fragments가 언어 중립적인 오픈 바이너리 파일 포맷으로 재작성되었습니다.** Google FlatBuffers 기반의 `.frag` 포맷으로, 전용 MIT 저장소(`ThatOpen/engine_fragment`)에서 개발됩니다. 3.2절/4.3절의 '중복 형상 인스턴싱' 설명은 구세대 Fragments에 해당하며, 현행 Fragments는 직렬화된 파일 아티팩트입니다. 공식 수치로는 약 2GB IFC STEP 파일이 약 80MB `.frag`로 압축되고, Web Worker 아키텍처로 수백만 개 요소를 브라우저에서 60fps로 렌더링한다고 밝히고 있습니다.
* That Open의 권장 프로덕션 플로우는 이제 런타임 IFC 파싱이 아니라, 새 `IfcImporter`(브라우저와 Node.js 모두 지원)를 이용한 **서버 측 1회성 IFC → `.frag` 사전 변환**입니다. 이는 3.2절 구성 요소 A(시각적 트윈)의 사전 처리 파이프라인 설계와 정확히 부합합니다.
* IFC 파싱은 여전히 **web-ifc**가 담당하며, 현재 `ThatOpen/engine_web-ifc`에서 유지되고 있습니다(0.0.77, 2026년 3월, MPL-2.0).

### B.6. 로컬 자동화

* **정정: Revit Batch Processor의 라이선스는 MIT가 아니라 GPL-3.0입니다**(부록 A 참조 2의 비교표를 수정했습니다). RBP는 2026년 2월 안정 버전 **v1.12.1**(2019년 이후 첫 정식 릴리스)을 배포했고 Revit 2015–2026을 지원하지만, 원저자가 공개적으로 유지보수에서 물러나 **커뮤니티 지원 전용** 상태입니다. 프로덕션 파이프라인에서 RBP 의존은 유지보수 리스크로 관리해야 합니다.
* **pyRevit은 6.x 라인으로 이동했습니다**(v6.0.0: 2026년 2월, 로더 전면 재작성 및 Revit 2027/.NET 10 대응; 최신 v6.5.3: 2026년 6월). `pyrevit run` CLI는 활발히 유지되고 있어 2.2.2절의 가이드는 유효합니다.
* 오픈소스 **RevitCoreConsole 래퍼(dosymep)는 휴면 상태**입니다(마지막 릴리스 2022년 10월, 마지막 커밋 2024년 5월). 권장 경로가 아닌 미유지 경로로 취급해야 합니다.
* 새로운 도구 범주가 등장했습니다: **Revit용 MCP 서버.** Autodesk의 공식 Revit MCP Server 테크 프리뷰(Revit 2027, 2026년 4–6월)는 실행 중인 Revit 세션을 stdio로 AI 클라이언트에 노출하며(요소 검색, 파라미터 확인, 일괄 파라미터 편집, 뷰 스냅샷), 2025–2026년에는 오픈소스 커뮤니티 Revit MCP 서버 생태계도 형성되었습니다(변동성이 큼 — 초기 최다 스타 프로젝트는 이미 아카이브됨). 이들은 '실행 중인' Revit 세션을 자동화하는 도구이므로, 2장의 헤드리스 추출 경로를 대체하기보다 보완합니다.

### B.7. 디지털 트윈 및 Physical AI 지형

2026년 들어 '빌딩을 위한 Physical AI' 논제는 비전 단계를 지나 실제 출시 제품으로 이동했습니다:

* **PassiveLogic**이 물리 기반 월드 모델로 구동되는 범용 자율 빌딩 제어의 **Level 3 자율성**을 발표했습니다(2026년 7월 8일). 2026년 하반기에는 Level 4(엣지 학습, 더 긴 예측 지평)를 목표로 하고 있습니다.
* **Johnson Controls가 Nantum AI를 인수**(2026년 4월 27일)하여 OpenBlue에 자율 AI HVAC 제어를 통합하고 있으며, **Siemens**는 Light + Building 2026에서 "스마트 빌딩에서 인간 중심 자율 빌딩으로의 여정"을 전면에 내세웠습니다.
* **NVIDIA Omniverse DSX Blueprint가 GA에 도달**했습니다(GTC, 2026년 3월). 시설 규모의 디지털 트윈으로 건설 전 시뮬레이션을 수행한 뒤, 운영 단계에서는 시설의 '운영 체제'로 모니터링·최적화에 활용하는 청사진으로 — AI 데이터센터에 적용된 사례이긴 하나 — '디지털 트윈 = 실시간 운영 체제' 논지의 가장 강력한 산업 사례입니다.
* 메타데이터 계층에서는 **Brick 1.5.0-rc1**(2026년 6월)이 Brick–RealEstateCore 조화를 심화했으며, **W3C WoT Thing Description 2.0**은 TD 1.1과의 하위 호환성을 명시적으로 보장하지 않는 최초 공개 작업 초안(2025년 11월) 단계에 머물러 있습니다 — 인용 가능한 표준은 여전히 TD 1.1입니다.
* 제어 루프의 표준 계층: **ANSI/ASHRAE 231-2026**(Control Description Language)이 2026년 2월 27일 발행되었습니다 — 기계와 사람이 모두 읽을 수 있는 제어 시퀀스 표현(Modelica 기반 CDL + JSON-LD 기반 Controls eXchange Format). **ASHRAE 223P**(빌딩 자동화용 의미 데이터 모델)는 2026년 7월 현재도 최종 발행 검토 중인 제안 표준(proposed standard)입니다. **BACnet 135-2024**(Protocol Revision 31, 보안 전송은 BACnet/SC)가 현행 필드 프로토콜이며, **BOPTEST 0.9.0**(2025년 11월, Modelica/FMI 기반)이 실건물 배포 전 제어 애플리케이션을 평가하는 표준적인 가상 시험 환경입니다.
* 폐쇄 루프 비전의 가장 가까운 선행연구: **LBNL의 OpenBOS**(Paul et al., *Science and Technology for the Built Environment* 31(3), 2025)는 실제 건물 1개와 시뮬레이션 건물 1개에서 그리드 반응형 의미 기반 상위 제어(supervisory control) 플랫폼(VOLTTRON + BACnet + ASHRAE 223P/BuildingMOTIF + BOPTEST)을 실증하고 양쪽 모두에서 일일 에너지 비용 20% 이상 절감을 보고했습니다. 논문이 스스로 밝힌 한계 — 223P에 제어 파라미터(예: 예열 시간, 외피 열저항) 부재, 저수준 제어 로직이 의미모델 밖에 존재, 애플리케이션 2개·건물 2개만 검증, 배포 전 커미셔닝 필요 — 는 의미 기반 BMS 플랫폼과 본 보고서가 다루는 BIM 기반 파이프라인 사이의 미개척 영역을 정확히 보여줍니다.

### B.8. 식별자 안정성과 Canonical ID 매핑 (동료 검토 노트, 2026년 7월)

3.2절의 GUID 기반 연합 전략은 여전히 타당하지만, "GUID"는 단일 체계가 아니라 서로 다른 여러 식별자 체계로 다뤄야 합니다:

* **Revit `UniqueId` ≠ IFC `GlobalId`.** Revit의 UniqueId는 44–45자 문자열(에피소드 GUID + 요소 ID)입니다. 내보내진 IFC GlobalId는 여기서 *유도*됩니다(에피소드 GUID의 마지막 32비트를 ElementId와 XOR한 뒤 22자로 압축) — 서로 연관되지만 결코 같지 않습니다. Revit 요소와 1:1로 대응하지 않는 엔티티(타입, 관계, 속성 세트, 공간 병합 요소)는 해시 또는 캐시 기반 GUID를 받습니다.
* **내보내진 GUID는 자동으로 안정적이지 않습니다.** Autodesk `revit-ifc` 이슈 트래커에는 룸, 요소 타입, 링크 파일 병합, 그룹에서 내보내기마다 GUID가 재생성되는 사례가 기록되어 있습니다. 완화책: 내보내기 옵션 **"Store the IFC GUID in an element parameter after export"** 를 활성화하면 GUID가 요소 파라미터로 기록되어 이후 내보내기에서 재사용됩니다.
* **뷰어/플랫폼 ID는 또 다른 체계입니다.** APS/Model Derivative의 `dbId`는 모델 버전이나 변환 간에 유지되지 않으며 — Autodesk는 `externalId`(Revit 모델의 경우 IFC GlobalId가 아니라 Revit UniqueId)를 키로 쓸 것을 권장합니다. BACnet `Object_Identifier`(10비트 타입 + 22비트 인스턴스, 단일 디바이스 내에서만 고유)는 어떤 GUID 체계와도 구조적으로 무관합니다.
* **권장사항:** 각 소스 시스템의 고유 식별자를 그대로 유지하고, 명시적인 **출처 계보(provenance)를 갖춘 canonical ID 매핑 테이블** — 소스 시스템, 모델/네트워크 ID, 소스 리비전, 소스 객체 ID, canonical ID, 유효 기간, 콘텐츠 해시 — 로 결합하십시오. 이는 업계의 통용 관행이기도 합니다(예: Azure Digital Twins용 RealEstateCore 온톨로지는 정확히 이 용도의 `externalIds` 맵 속성을 제공합니다).

### B.9. 본 보고서 권고안에 대한 영향

| # | 권고안 (6장) | 2026년 7월 기준 상태 |
| --- | --- | --- |
| 1 | 엔진의 분리 (APS / 로컬 자동화) | **유효.** 서비스 명칭이 "Automation API"로 변경. 엔진 수명 주기 정책과 2025년 12월 과금 개편을 반영해 계획 필요. 로컬은 pyRevit 6.x 우선, RBP는 커뮤니티 지원 전용으로 취급. 읽기 중심 추출은 엔진 없는 공식 API로 충분할 수 있음(B.1 참조). |
| 2 | GUID 기반 `glTF` + `JSON` + `IFC` 데이터 연합 | **유효하나 규율 필요.** 시각화 레그는 바이너리 `.glb` 또는 신형 바이너리 `.frag`(Fragments)를 대상으로 하고, Draco 압축은 변환 후 후처리로 적용. GUID 키잉은 "Store the IFC GUID" 내보내기 옵션과 출처 계보를 갖춘 canonical ID 매핑을 전제로 함(B.8 참조). |
| 3 | 개방형 표준 채택 (IFC, Speckle) | **유효하나 단서 있음.** IFC 4.3(ISO 16739-1:2024)을 대상으로 할 것. IFC X는 아직 구현 대상이 아님. Speckle 파이프라인은 v3 커넥터 필수이며 Automate는 Enterprise 플랜 전용. |
| 4 | 자동 검증 체계 (IDS + IfcTester) | **유효하며 강화됨.** IDS 1.0이 여전히 최신. buildingSMART 공식 Validation Service를 추가 적합성 게이트로 도입 권장. |
