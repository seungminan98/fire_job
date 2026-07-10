# fire_job 프로젝트 지침

## 이모지 금지 (필수)
- **이모지(픽토그램)를 절대 사용하지 않는다.** UI(HTML/JS 문자열), README 등 문서, 커밋 메시지,
  alert/confirm 문구 등 이 저장소의 모든 산출물에 적용된다.
- 유니코드 중 플랫폼에 따라 컬러 이모지로 렌더링되는 문자도 금지한다
  (예: U+26A0 경고 삼각형, U+2B07 아래 화살표, U+2714 체크 — 문서에는 코드포인트로만 언급).
- 아이콘이 필요하면 **인라인 SVG**로 만든다:
  - `viewBox="0 0 16 16"`, `fill="none"`, `stroke="currentColor"`, `stroke-width="1.5"` 기본
  - 버튼 안 아이콘은 13~14px, 기존 index.html 버튼 SVG 스타일을 따른다
  - JS 문자열에서 반복 사용하는 아이콘은 `ICO_*` 상수로 정의해 재사용
- `<option>`·`optgroup label`·`alert()` 등 마크업이 불가능한 곳에는 기호 없이 일반 텍스트만 쓴다
  (예: "주의:" 접두어).
- 순수 텍스트 기호는 허용: 화살표 `→`, 닫기 `✕`, 마일스톤 `◆`, 편집 `✎`, 드래그 핸들 `⠿`.

## 설계 원칙
- **HTML 파일 하나로 완결** — 외부 라이브러리·CDN·서버·DB 없음 (LZ-String은 인라인).
- 데이터는 localStorage 자동 저장, 공유는 URL 해시(LZ-String 압축)·CSV·JSON 백업.
- 새 기능은 반드시 Playwright(Chromium `/opt/pw-browsers/chromium`)로 실제 동작을 검증한 뒤 커밋한다.

## 파일 구성
| 파일 | 역할 | localStorage 키 |
|------|------|-----------------|
| `index.html` | 전체 공사 일정 · M/M Capacity (7단계, 간트 편집 드래그) | `fireSchedule.v2` |
| `hookup.html` | 공사별 소방 훅업 공정표 (사용자 정의 공종 + 머터리얼 셰이딩) | `fireHookup.v1` |
| `bridge.html` | PPT/이미지를 Claude API로 읽어 훅업에 밀어넣는 브리지 | 동일 (`fireHookup.v1`) |
| `process.html` | 업무 표준 매뉴얼 (단계·조건 분기 → 플로우차트 자동 렌더) | `fireProcess.v1` |
| `calc.html` | 법규 계산기 (건폐율·용적률, 스프링클러 살수장애 시각화 판정) | `fireCalc.v1` |

- 전 페이지 공통: 접속 암호 게이트(SHA-256 해시 비교, 소스에 평문 금지).
- hookup의 공종은 사용자 정의 그룹(`p.groups`, 머터리얼 컬러) — 작업은 `gid`로 참조하며
  같은 공종 안에서 순서대로 700→600→500… 셰이드로 옅어진다.
- hookup 작업은 1단계 하위 작업(`parentId`)을 가질 수 있다(Epic=공종 > Task > Sub-Task).
  접기 상태는 `p.collapsed`(작업)·`p.gcol`(공종)에 저장.
- 간트 헤더는 년→월→주→일 계층 병합, 최소 단위 토글(월/주/일) 공통 패턴을 유지한다.
- process의 단계는 재귀적 세부단계(`step.children`, 깊이 제한 없음)를 가진다 — 폴더처럼
  `navPath`(id 경로)로 드릴다운하고 편집·플로우차트는 `currentSteps()`(현재 레벨)만 다룬다.
  next/분기 연결은 같은 레벨 안에서만, 매뉴얼 본문은 전체 트리를 1→1.1→1.1.1로 렌더한다.
- process 단계의 `par`(바로 위 단계와 동시 진행)는 `calcNums()`로 4-1/4-2 번호가 되고
  플로우차트에서 나란히 배치된다(기본 다음 = 다음 그룹 전체). `deps`(선행 단계 id, 같은 레벨)는
  파란 점선 화살표(#4a7ab5)로, 흐름 화살표와 겹치면 생략 — 본문에는 "선행:"으로 항상 표기.
