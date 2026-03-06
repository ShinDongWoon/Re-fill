<div align="center">

# 📰 Re:fill Keynes
**Robust Article Assistant for Dynamic News Sites**

[![Version](https://img.shields.io/badge/version-2.1.1-blue.svg)](https://github.com/yourusername/refill-keynes)
[![Platform](https://img.shields.io/badge/platform-Tampermonkey-green.svg)]()
[![API](https://img.shields.io/badge/AI-Gemini_3.0_Flash-orange.svg)]()

<br/>

브라우저에서 현재 읽고 있는 기사 본문을 기준으로 질문에 답하는 **Tampermonkey 사용자 스크립트**입니다.<br/>
The Economist, WSJ, FT, NYT 등 레이아웃 변화가 심한 동적 뉴스 사이트에서,<br/>
**정확한 본문 추출**과 강건한 **UI 생존성**을 보장하는 데 집중합니다.

</div>

<br/>

## 💡 이 프로젝트가 푸는 문제

본 프로젝트에서의 핵심 문제의식은 다음 2가지로부터 시작하였습니다. 

1. **휘발되는 UI:** 뉴스 사이트는 SPA 전환, 지연 로딩, 우측 rail 교체, 광고 영역 재구성이 잦아 주입한 UI가 쉽게 사라집니다.
2. **불완전한 문맥:** 기사 본문이 초기에 불완전하게 렌더링되는 경우가 많아, 너무 빨리 추출하면 문맥이 비거나 끊깁니다.

이 프로젝트는 **본문 추출 정확도**와 **페이지 라이프사이클 안정성**을 최우선순위로 두고, 진행되었습니다. 기본 모델은 별도의 서버 없이 브라우저에서 직접 호출되는 `gemini-3-flash-preview`를 사용합니다.

---

## ✨ 핵심 기능

* **📡 3단계 본문 추출 (Fallback System)**
  * JSON-LD `articleBody` → Mozilla Readability → 텍스트 밀도(Density) 기반 fallback
* **🎯 기사 페이지 정밀 판별**
  * `isProbablyReaderable` / JSON-LD 기사 메타데이터 / `main/article` 문단 수 휴리스틱
* **🛡️ SPA-safe 재초기화 및 생존**
  * `pushState`, `replaceState`, `popstate` 감지
  * host 제거 감지용 lightweight `MutationObserver` 및 debounce 기반 재초기화
* **🧠 Local RAG-lite**
  * 문단 단위 분리 및 질문 토큰 매칭 (점수화)
  * 앞/뒤 문단 보강 및 전송 문맥 길이 제한으로 효율성 극대화
* **UI/UX: Shadow DOM & 스마트 도킹**
  * 호스트 페이지 CSS와 격리된 Shadow DOM 렌더링 (상태줄, 메시지, 입력창, 최소화 제공)
  * 우측 rail 후보를 탐색해 도킹하고, 실패 시 우측 플로팅 카드로 안전하게 Fallback
* **⚙️ 견고한 API 파이프라인**
  * role alternation 정리, system instruction 분리
  * timeout, in-flight 복구, 에러 노출 및 본문 지연 로딩 시 backoff 기반 추출 재시도

---

## 🚀 설치 및 사용 방법

### 1. 지원 대상 도메인
해당 도메인에 매치되더라도, 실제 UI는 **기사 페이지로 판단될 때만** 표시됩니다.
> `economist.com/*` | `wsj.com/*` | `ft.com/*` | `nytimes.com/*`

### 2. 설치 방법
추가 빌드 단계는 없습니다. (`Readability.js`는 userscript 메타데이터의 `@require`로 로드합니다.)
1. 브라우저에 [Tampermonkey](https://www.tampermonkey.net/)를 설치합니다.
2. 새 userscript를 생성하여 이 저장소의 스크립트 내용을 붙여넣고 저장합니다.
3. 지원 사이트의 기사 페이지에 접속합니다.
4. 우측 위젯의 `🔑` 버튼 또는 Tampermonkey 메뉴에서 **Gemini API Key**를 입력합니다.

### 3. 사용 가이드
1. 기사 페이지 접속 시 스크립트가 페이지를 판별하고 본문을 추출합니다 (상태줄 표시).
2. 하단 입력창에 질문을 넣고 전송합니다.
3. **UI 컨트롤 버튼**:
   * `🔑` API Key 설정 | `↻` 본문 재추출 | `📌` 도킹/플로팅 전환 | `🧹` 대화 초기화 | `—` 최소화

---

## 🔍 심층 동작 방식 (How it works)

<details>
<summary><b>1) 페이지 판별 및 본문 추출</b> (클릭하여 펼치기)</summary>

* **판별:** `isProbablyReaderable`, JSON-LD(`NewsArticle`, `Article`, `Report`), 문단 수 휴리스틱을 조합해 기사 페이지 여부를 판단합니다. 기사가 아니면 UI를 제거합니다.
* **추출:** JSON-LD → Readability(DOM clone) → Density fallback 순으로 시도합니다. 본문이 너무 짧으면 지연 로딩을 의심해 재시도합니다.
* **상태 갱신:** URL, 제목, 발췌, 전체 텍스트, 문단 배열, 소스, fingerprint를 갱신합니다. fingerprint가 바뀌면 새 기사 문맥으로 간주합니다.
</details>

<details>
<summary><b>2) Local RAG-lite 및 Gemini 요청</b> (클릭하여 펼치기)</summary>

* **문맥 구성:** 질문을 토큰화하여 겹치는 토큰이 많은 문단에 점수를 줍니다. 상위 문단 주변과 문서 앞/뒤 문단을 포함하되, `maxContextChars`를 넘지 않게 잘라냅니다. (임베딩 검색이 아닌 가벼운 lexical matching)
* **API 요청:** system instruction을 분리하고, 대화 이력(`user` → `model`) 순서를 강제합니다. 현재 질문과 기사 문맥은 마지막 `user` 메시지에 합쳐 전송합니다.
</details>

<details>
<summary><b>3) UI 라이프사이클 및 성능 고려</b> (클릭하여 펼치기)</summary>

* **UI 복구:** 라우팅이나 DOM 교체 발생 시 `history` API hook, `popstate` listener, 경량 `MutationObserver`를 통해 복구합니다.
* **성능 관리:** 판별/추출 로직은 Observer 내부에서 직접 돌리지 않고 `requestIdleCallback`이나 timeout으로 미루며, geometry 갱신은 `ResizeObserver`와 debounce를 사용합니다.
</details>

---

## 🛠 주요 설정 값

아래 값은 스크립트 코드 상의 기본값(`DEFAULTS`)입니다.

| 카테고리 | 설정 키 | 기본값 | 설명 |
| :--- | :--- | :--- | :--- |
| **UI** | `topOffsetPx` / `bottomMarginPx` | `80` / `16` | 상/하단 여백 및 오프셋 |
| | `floatRightPx` / `floatWidthPx` | `16` / `380` | 플로팅 우측 여백 및 폭 |
| | `dockMode` / `hideDockTargetIfAd` | `true` / `true` | 우측 rail 도킹 시도 및 광고 숨김 여부 |
| **RAG** | `minUsefulChars` | `1200` | 로드 완료로 판단할 최소 본문 길이 |
| | `maxContextChars` | `14000` | API에 전송할 최대 컨텍스트 길이 |
| | `ragTopK` / `ragWindow` | `10` / `1` | 상위 관련 문단 개수 및 이웃 문단 범위 |
| | `includeHeadTailParas` | `2` | 문서 앞/뒤 필수 포함 문단 수 |
| **API** | `temperature` / `maxOutputTokens` | `0.2` / `2000` | 생성 다양성 및 최대 출력 토큰 |

---

## ⚖️ 응답 정책 및 보안

### 응답 정책 (기사 근거 우선)
본 모델은 일반 목적 챗봇이 아닌 **기사 근거 우선 Q&A 도구**입니다.
* 기본 응답 언어는 한국어입니다.
* 가능한 한 기사 본문을 근거로 답하며, 기사 문맥을 벗어난 질문일 경우 반드시 아래 문구를 먼저 출력한 뒤 외부 지식을 사용합니다.
  > *"말씀하신 질문은 기사 내용을 벗어납니다. 외부 지식을 사용해 답변드리겠습니다."*

### 보안 및 개인정보 (Privacy)
* 중간 서버나 백엔드 프록시가 **전혀 없습니다.** 브라우저에서 API로 직접 통신합니다.
* API Key는 Tampermonkey 로컬 저장소에 저장됩니다. (단, 비밀관리 솔루션 수준의 보안은 아니므로 고가치 운영 키 사용은 권장하지 않습니다.)

### 제약 사항 (Non-Goals)
* Paywall을 우회하지 않으며, 브라우저에 렌더링된 텍스트만 처리합니다.
* Lexical matching 기반이므로 질문 표현이 완전히 다르면 관련 문단을 놓칠 수 있습니다.
* 우측 rail 도킹은 휴리스틱이므로, 대대적인 사이트 레이아웃 개편에 민감할 수 있습니다.

---

## 🧑‍💻 개발자 가이드: 수정할 때 주의할 점

안정적인 라이프사이클 유지를 위해 다음 규칙을 준수하세요.

1. **디바운스 활용:** `MutationObserver` 안에서 무거운 판별/추출 로직을 직접 돌리지 마세요. (복구 트리거로만 사용)
2. **재귀 호출 주의:** UI helper 안에서 `ensureUI()` 재진입을 조심하세요. (특히 최소화 상태나 geometry 갱신)
3. **상태 원복:** `STATE.inFlight`는 반드시 `finally`에서 원복하여 버튼 영구 disable을 방지하세요.
4. **대화 이력 유지:** Gemini payload는 `user`와 `model`의 교차 순서를 엄격히 맞춰야 합니다.
5. **Fallback 허용:** 레이아웃 도킹은 실패할 수 있습니다. 예외 처리하지 말고 자연스럽게 플로팅 카드로 내려오게 두세요.

<br/>

> **Troubleshooting 팁:** UI가 보이지 않으면 기사 페이지 판별에 실패했거나 로딩 지연일 수 있습니다. 모델 답변이 이상하다면 `↻` 버튼으로 본문을 다시 추출해 보세요.
