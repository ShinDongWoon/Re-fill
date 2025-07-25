# Re:fill - The Economist AI Assistant (Prototype)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

> 웹페이지의 버려진 공간을 지적 탐구를 위한 '동반자'로 전환합니다.

**Re:fill**은 뉴스 기사 페이지의 광고 차단으로 생긴 여백(Whitespace)에 컨텍스트 인식 AI 챗봇을 삽입하는 브라우저 확장 프로그램의 프로토타입입니다. 이 프로토타입은 The Economist 웹사이트를 대상으로 구현되었습니다.

![Project Screenshot](https://i.imgur.com/G4hD7tV.jpg)
*(프로젝트 동작 예시 스크린샷)*

## 🚀 주요 기능 (Features)

* **동적 UI 삽입:** The Economist 기사 페이지의 오른쪽 광고 영역을 자동으로 탐지하여 채팅 UI를 삽입합니다.
* **컨텍스트 자동 분석:** 페이지에 접속하면 현재 기사의 전체 내용을 자동으로 분석하여 대화의 기반으로 삼습니다.
* **컨텍스트 기반 Q&A:** 기사 내용에 기반한 질문에 대해 답변합니다.
    * **AI 페르소나:** "엄밀한 경제학자"의 관점에서 The Economist 기사를 분석하고 답변하도록 설정되어 있습니다.
* **대화 연속성:** 이전 대화 내용을 기억하여 후속 질문에 답변할 수 있습니다.
* **실시간 사이트 변화 대응:** 웹사이트 스크립트가 UI를 숨기려는 시도를 `MutationObserver`를 통해 감지하고 무력화하여 안정적으로 표시됩니다.

## 🛠️ 기술 스택 (Technology Stack)

* **실행 환경:** [Tampermonkey](https://www.tampermonkey.net/) (브라우저 유저스크립트)
* **언어:** JavaScript (ES6+)
* **핵심 AI 모델:** Google Gemini 1.5 Flash
* **API 통신:** `GM_xmlhttpRequest`

## ⚙️ 설치 및 실행 방법 (Setup)

이 프로토타입은 Tampermonkey 스크립트로 작성되었으므로, 실행을 위해서는 브라우저에 Tampermonkey 확장 프로그램이 설치되어 있어야 합니다.

1.  **Tampermonkey 설치:** 사용하시는 브라우저에 맞는 [Tampermonkey](https://www.tampermonkey.net/)를 설치합니다.
2.  **새 스크립트 추가:** Tampermonkey 대시보드에서 '새 스크립트 만들기'를 선택합니다.
3.  **코드 복사:** 이 저장소에 있는 `refill-economist-demo.js` 파일의 전체 코드를 복사하여 편집기에 붙여넣습니다.
4.  **API 키 설정:** 스크립트 상단의 `GEMINI_API_KEY` 변수에 본인의 Google Gemini API 키를 입력합니다.
    ```javascript
    const GEMINI_API_KEY = 'YOUR_API_KEY_HERE';
    ```
5.  **저장:** 스크립트를 저장합니다 (`Ctrl+S`).
6.  **실행 확인:** [The Economist](https://www.economist.com/) 웹사이트의 기사 페이지를 방문하면 오른쪽에 Re:fill 채팅창이 나타납니다.

## 📜 라이선스 (License)

이 프로젝트는 [MIT 라이선스](https://opensource.org/licenses/MIT)를 따릅니다. 자세한 내용은 `LICENSE` 파일을 참고하십시오.

---

Copyright (c) 2025 [ShinDongWoon]
