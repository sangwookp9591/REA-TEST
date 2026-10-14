# Aside.app 리버스 엔지니어링 분석 계획

- 대상: /Applications/Aside.app — at.studio.AsideBrowser 1.0.928.1 (Contents/Frameworks/Aside Framework.framework 포함)
- 도구: REA MCP (rea-agents@4.0.1), Orca orchestration Run `run_2635c3047f62`
- 시작: 2026-10-14

## 배치 (model-routing)

| 단계 | 담당 | 모델/effort | 산출물 |
| --- | --- | --- | --- |
| A. 정적 분석 (기술·모듈·리소스·저장소) | Codex | gpt-6-luna / high | reports/A-static.md |
| B. 검색 기능 추적 (정적+런타임) | Codex | gpt-6.1-sol / medium | reports/B-search.md, reports/B-artifacts/ |
| C. 독립 검증 (사실/추정 재분류) | Claude | opus / high | reports/C-verification.md |
| D. 최종 보고서 | Claude | sonnet / high | reports/REPORT.md |

## 진행 로그

- A 최초 dispatch `ctx_a1635ac8ebe1` 실패 (agent-trust-workspace) → release 후 `ctx_915591cfec64` 로 재시도
- B dispatch `ctx_34b35ae8d301` 정상 시작

## Task A spec
```
[Task A: Aside.app 정적 분석 — REA MCP 사용 필수]
Target: /Applications/Aside.app (at.studio.AsideBrowser 1.0.928.1, 읽기 전용). Contents/MacOS/Aside, Contents/Frameworks/Aside Framework.framework 및 기타 Frameworks/Helpers, Resources(*.pak, Assets.car, *.lproj, at.studio.AsideBrowser.manifest 등), Info.plist, 서명/entitlements, embedded.provisionprofile, Library/.
Change: REA MCP 도구(inspect_artifact, inspect_macho, inspect_plist, inspect_signature, inspect_asset_catalog, list_strings/search_strings, get_objc_classes, analyze_swift_types, analyze_web_bundle 등)로 분석하고 보고서를 작성한다:
 1) 사용 기술 스택 (Chromium/Electron/Swift/ObjC 여부, 버전 근거, 서드파티 라이브러리: Sparkle, Sentry 등)
 2) 주요 모듈/번들 구성 (메인 실행파일, 프레임워크, helper 앱, XPC, 확장, 내장 웹 리소스) 과 각 역할
 3) 리소스 구성 (pak, 로컬라이제이션, asset catalog, 내장 HTML/JS)
 4) 데이터 저장 관련 코드/문자열 (SQLite/LevelDB/IndexedDB/Keychain/UserDefaults/App Group/~/Library/Application Support 경로, 프로필 디렉터리명 등) — 실제 ~/Library 하위 Aside 관련 디렉터리 존재 여부도 ls로 확인(파일 내용 중 개인정보는 열람하지 말고 파일명/스키마 수준만)
 5) 검색 기능 관련 정적 단서(검색엔진 URL 템플릿, omnibox/search/suggest 문자열, 관련 클래스/함수 주소) — Task B가 이어받을 수 있게 목록화
Constraints: 앱을 실행하지 않는다(실행은 Task B 담당). 앱 번들을 수정하지 않는다. 모든 주장마다 [사실]/[추정] 태그와 근거(파일 경로, 오프셋/주소, 심볼·함수명, 문자열, REA 도구명)를 붙인다.
Ownership: 쓰기는 /Users/iron/Project/psw/rea-test/reports/A-static.md 단일 파일만.
Observable acceptance: reports/A-static.md 에 위 1~5 섹션이 모두 있고 각 항목에 태그+근거가 있다. worker_done 에 --report-path 로 지정.
```

## Task B spec
```
[Task B: Aside.app 검색 기능 동작 추적 — REA MCP 사용 필수, 정적+런타임]
Target: /Applications/Aside.app (at.studio.AsideBrowser 1.0.928.1, Chromium 계열 브라우저로 추정, Contents/Frameworks/Aside Framework.framework).
Change: 사용자가 검색을 할 때(주소창/검색 UI/사이드 패널 등 앱에 존재하는 검색 진입점) 입력 → 처리 → 요청/결과 표시까지의 흐름을 추적해 보고서를 작성한다.
 1) 정적: REA(search_strings, list_strings, get_objc_classes, analyze_swift_types, search_procedures, xrefs, procedure_pseudo_code, trace_feature/trace_application_feature, build_call_path, analyze_web_bundle 등)로 검색 관련 문자열·URL 템플릿·클래스·함수·호출 경로를 찾는다. 내장 웹 UI(pak/HTML/JS)에 검색 로직이 있으면 그것도 추적.
 2) 런타임: 필요하면 앱을 실행해 관찰한다(REA capture_native_ui_scenario / observe_native_ui / capture_process_scenario / list_javascript_runtime_targets 등, 또는 open -a). 검색어는 무해한 "test" 만 사용. 로그인·결제·설정 변경·데이터 삭제 금지. 관찰 후 앱 종료.
 3) 검색 결과 저장 여부(히스토리 DB, 검색 기록) 단서가 있으면 파일명/스키마 수준으로만 확인(개인 기록 내용 열람 금지).
Constraints: 앱 번들 수정 금지. 모든 주장마다 [사실](직접 관찰/디스어셈블리/문자열로 확인) 또는 [추정](간접 추론) 태그와 근거(파일 경로, 주소, 함수/클래스명, 문자열, 캡처 결과, REA 도구명)를 붙인다. 흐름은 단계별 시퀀스로 정리.
Ownership: 쓰기는 /Users/iron/Project/psw/rea-test/reports/B-search.md 와 reports/B-artifacts/ (스크린샷·캡처) 만.
Observable acceptance: reports/B-search.md 에 진입점, 처리 경로(함수/모듈), 네트워크 요청 대상, 결과 표시, 저장 여부, 미확인 사항 섹션이 있고 각 항목에 태그+근거. worker_done 에 --report-path 로 지정.
```
