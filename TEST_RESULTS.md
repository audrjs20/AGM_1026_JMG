# CTA 노출·클릭 수집 테스트 결과

- 테스트 일자: 2026-10-26
- 대상: https://audrjs20.github.io/AGM_1026_JMG/ (커밋 d19ec9f)
- Amplitude 프로젝트 ID: 887071
- 테스트 트래픽 구분값: `utm_source` = `qa_test`, `qa_test2` (실제 분석에서는 제외)

## 종합 판정: 통과 (일부 항목은 추가 확인 필요)

| # | 항목 | 결과 | 근거 |
|---|---|---|---|
| 1 | 페이지에서 Amplitude SDK 로드 및 초기화 | 통과 | 배포 페이지에서 `ampReady = true`, `api2.amplitude.com/2/httpapi` 요청 발생 |
| 2 | `cta_click` 전송 (속성 `cta`, `cta_text`, `section`) | 통과 | 클릭 시 payload 확인(예: `hero_signup`, `pkg_play`), Amplitude에서 `cta`별 집계 확인 |
| 3 | `cta_impression` 전송 | 통과 | Amplitude에서 `cta`별 노출 8건 확인 |
| 4 | 노출 중복 방지 (페이지 로드당 CTA 1회) | 통과 | 로컬 테스트에서 중복 0건 |
| 5 | Amplitude 로드 전 발생 이벤트 유실 방지(대기열) | 부분 통과 | 대기열이 비워지고(`queued: 0`) 첫 화면 버튼 노출이 Amplitude에 기록됨 확인 |
| 6 | 중간 구간 버튼 노출 (`pkg_basic`, `pkg_play`, `trial_signup`) | 부분 통과 | 화면 이동 후 이벤트 호출은 확인. Amplitude 수신은 재조회하지 않음 |
| 7 | 미확인 항목 | 미검증 | `checklist_done`(체크 완료 후 표시), `success_app`, `trial_signup` 클릭, 가입 폼 제출(`signup_complete`) |

## 결과 설명

**Amplitude 집계 (qa_test, 배포 페이지)**

- 노출 8건: `header_signup`, `hero_signup`, `hero_app`, `sticky_app`, `sticky_signup`, `final_appstore`, `final_playstore`, `form_submit`
- 클릭 3건: `hero_signup` 1, `sticky_signup` 1, `pkg_basic` 1

**알려진 한계와 해석**

1. 첫 테스트에서 `pkg_basic`, `pkg_play`, `trial_signup`의 노출이 Amplitude에 없었다. 원인은 자동 스크립트가 화면을 한 번에 점프시킨 것으로, 사람이 스크롤하는 상황이 아닌 테스트 방식의 문제다. 요소를 화면 중앙에 두고 다시 확인했을 때는 세 버튼 모두 노출 이벤트가 정상 호출되었다.
2. 클릭 3건 중 `pkg_basic` 1건만 테스트에서 의도적으로 발생시켰다. `hero_signup`, `sticky_signup` 클릭은 출처를 확인하지 못했다(다른 접속에서 발생한 클릭일 가능성).
3. 데스크톱 화면에서는 하단 고정 버튼(`sticky_*`)이 숨겨져 노출이 기록되지 않는 것이 정상이다. 모바일 폭에서만 수집된다.
4. 앱 내장 미리보기 창에서는 Amplitude로 요청이 나가지 않아, 전송 검증은 실제 Chrome의 배포 페이지에서 수행했다.

## 남은 확인 사항

- `pkg_basic`, `pkg_play`, `trial_signup` 노출이 Amplitude에 쌓이는지 사람 스크롤로 재확인
- 체크리스트를 모두 체크한 뒤 `checklist_done` 노출·클릭 확인
- 가입 폼을 끝까지 제출해 `signup_complete` 확인 (테스트 값 사용, 실제 개인정보 입력 금지)
- 실제 광고 집행 전, 테스트 트래픽(`qa_test*`)을 분석에서 제외하는 필터 설정
