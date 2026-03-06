# Re:fill - Keynes (Robust Article Assistant)

[![Version](https://img.shields.io/badge/version-2.1.1-blue.svg)](https://github.com/yourusername/refill-keynes)
[![Platform](https://img.shields.io/badge/platform-Tampermonkey-green.svg)]()
[![API](https://img.shields.io/badge/AI-Gemini_1.5_Flash-orange.svg)]()

**Re:fill Keynes**는 The Economist, WSJ, FT, NYT 등 주요 경제/시사 매체의 복잡한 웹 환경에서 완벽하게 동작하는 클라이언트 사이드 기반 AI 독서 보조 UserScript입니다. 

방대한 텍스트의 경제 기사를 심도 있게 분석하기 위해 고안되었으며, DOM 렌더링이 잦은 SPA(Single Page Application) 환경에서도 안정적으로 챗봇 UI를 유지(Docking)합니다.

## ✨ Core Features

* **Zero-Server Local RAG (Retrieval-Augmented Generation)**
    * 서버를 거치지 않고 브라우저단에서 기사 본문을 Chunking하고, 사용자 질문 토큰(TF-IDF 기반)과 스코어링하여 관련 문단만 추출합니다. 
    * 이를 통해 API 토큰 소모를 최소화하고 응답 속도를 극대화하며, 불필요한 데이터 외부 유출을 방지합니다.
* **3-Tier Fallback Text Extraction**
    * `JSON-LD` 파싱 -> `Readability.js` -> `DOM Density` 분석으로 이어지는 3단계 추출 파이프라인을 구축하여, 페이월(Paywall)이나 복잡한 레이아웃 뒤에 숨겨진 본문을 정확히 긁어옵니다.
* **Shadow DOM & Aggressive Docking**
    * 기존 웹페이지의 CSS와 완벽히 격리되는 Shadow DOM으로 UI를 구축했습니다.
    * 무의미한 측면 광고 영역(Ad-rail)을 식별하여 숨기고, 해당 영역에 챗봇 UI를 도킹시킵니다.
* **SPA-Safe & Light-weight Observer**
    * `history.pushState` 후킹 및 경량화된 `MutationObserver`를 통해, 호스트 페이지의 DOM이 날아가거나 라우팅이 변경되는 상황에서도 챗봇 상태와 UI가 증발하지 않고 끈질기게 생존합니다. 무한 루프(Recursion) 이슈를 구조적으로 차단했습니다.

## ⚙️ How it works

1.  **Inject & Detect**: 스크립트 로드 시 페이지가 기사 형식인지 판별하고, 측면 광고 패널(Dock Target)을 탐색합니다.
2.  **Extract**: 3-Tier 엔진을 통해 기사의 메타데이터(Title, URL)와 본문을 추출 및 정제합니다.
3.  **Chat (Local RAG)**: 사용자가 질문을 입력하면, 브라우저 내부에서 기사와 질문의 연관성을 스코어링하여 프롬프트를 동적 생성한 뒤 `GM_xmlhttpRequest`를 통해 CORS 제약 없이 Gemini API를 직접 호출합니다.

## 🚀 Installation

1.  브라우저에 [Tampermonkey](https://www.tampermonkey.net/) 확장 프로그램을 설치합니다.
2.  [이곳(스크립트 Raw 링크)](#)을 클릭하여 스크립트를 설치합니다.
3.  지원되는 뉴스 사이트(ex. `economist.com/*`)에 접속합니다.
4.  우측 상단의 🔑 버튼을 눌러 본인의 **Gemini API Key**를 등록합니다. (키는 브라우저 로컬 환경에만 안전하게 저장됩니다.)

## 🛠 Configurations (UI 상단 메뉴)

* `🔑` **API Key 설정**: Gemini API 키 등록 및 수정
* `↻` **본문 재추출**: 강제로 DOM을 다시 스캔하여 기사 텍스트 업데이트
* `📌` **도킹 모드 전환**: 광고 칸에 UI를 임베드(Dock)할지, 화면 우측에 띄울지(Floating) 토글
* `🧹` **대화 초기화**: 현재 컨텍스트를 비우고 대화 히스토리 리셋

## ⚠️ Disclaimer & Privacy

* 본 스크립트는 등록된 API Key와 질문/프롬프트 데이터를 오직 `generativelanguage.googleapis.com`과의 직접 통신에만 사용하며, 중간 서버를 거치지 않습니다.
* API 응답 품질과 처리량은 사용자가 직접 발급한 Gemini API의 할당량(Quota)을 따릅니다.
