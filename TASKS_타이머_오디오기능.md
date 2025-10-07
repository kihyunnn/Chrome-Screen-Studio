# 타이머 기반 자동 종료 & 오디오 녹화 기능 세부 작업 목록

상위 문서: `PRD_화면녹화도구.md`
대상 코드: 단일 HTML 파일(현행 구조 유지, 점진적 리팩터 고려)
우선순위: 타이머 → 오디오(시스템/탭) → 추가 UX 다듬기

---
## 0. 공통 사전 준비
- [v] (공통) 기존 전역 변수 목록 재확인 및 중복/이름 충돌 위험 표 작성
- [ ] (공통) `startRecording`, `stopRecording` 함수 내부 훅 포인트 주석 추가 (// EXT: autoStop, // EXT: audio)
- [ ] (공통) 새 기능 토글 시나리오 플래그 정의 (autoStopEnabled, audioEnabled)
- [ ] (공통) 에러 메시지 표준 함수 `notify(type, message)` 초안 작성 (console wrapper)

---
## 1. 타이머 기반 자동 녹화 종료 (고정 시간 방식)
### 1.1 UI 작업
- [v] 설정 패널 섹션 삽입 위치 결정 (품질/FPS 아래)
- [v ] 체크박스: "자동 종료" (`id=autoStopEnabled`)
- [v] 시간 입력: 분(`id=autoStopMinutes`), 초(`id=autoStopSeconds`), 기본 10:00
- [v ] 유효성 안내 텍스트(최대 180분, 음수 불가)
- [v] 남은 시간 표시 영역(선택) `id=autoStopRemain` (녹화 중 활성)

### 1.2 상태 & 로직
- [v ] 상태 필드: `autoStopTimeoutId` 추가
- [v ] util 함수: `calcAutoStopMs(m, s)` (입력 검증 + clamping)
- [v ] schedule 함수: `scheduleAutoStop(ms)` 구현 (기존 예약 있으면 clear)
- [v] clear 함수: `clearAutoStop()` 구현
- [v] 통합: `startRecording` 끝부분에서 autoStopEnabled 체크 후 `scheduleAutoStop`
- [v] 통합: `stopRecording`에서 `clearAutoStop()` 호출
- [v] 남은 시간 tick: 기존 녹화 타이머 interval 내부에서 (if autoStop) 남은 ms 계산 후 표시

### 1.3 예외 & 에러 처리
- [v ] 입력 NaN/빈값 → 기본값 10:00 재설정 + 안내
- [v ] 0:00 입력 → 기능 비활성 처리 및 경고
- [v ] 180분 초과 → 180분으로 강제 및 안내

### 1.4 UX 마감
- [v ] 자동 종료 예약 시 Start 버튼 hover 툴팁: "자동 종료 예정: MM:SS"
- [v ] 종료 시 알림: `notify('info', '자동 종료 시간이 도달하여 녹화를 중지했습니다.')`
- [v ] 남은 시간 60초 이하 색상 강조 (orange → 10초 이하 red)

### 1.5 테스트
- [ ] 단위: `calcAutoStopMs` 경계 (음수, 0, 180분 초과)
- [ ] 수동: 5초 설정 후 정상 종료 확인
- [ ] 수동: 수동 Stop 시 타임아웃 해제 여부 확인 (중복 종료 방지)
- [ ] 회귀: autoStop 꺼진 상태에서 기존 흐름 영향 없는지

---
## 2. 오디오 녹화(시스템/탭 오디오)
### 2.1 UI 작업
- [v] 체크박스: "오디오 포함" (`id=recordAudio`)
- [v] 도움말 아이콘: "브라우저 공유 대화상자에서 오디오 공유를 선택해야 합니다." 툴팁

### 2.2 스트림 획득 로직
- [ ] 함수 분리: `async createDisplayStream({ audio })`
- [ ] 기존 startRecording 내 `getDisplayMedia` 호출을 함수 사용하도록 대체
- [ ] 오디오 불가 시(트랙 0) 사용자 안내 + 체크박스 해제 처리

### 2.3 MediaRecorder 구성
- [ ] 옵션 객체 추출: `const recorderOptions = { mimeType: 'video/webm' /* 확장 가능 */ }`
- [ ] 브라우저 지원 여부 try/catch (mimeType 지원 안 되면 fallback null)
- [ ] start 시 chunk push, stop 시 blob 생성 기존 로직 재사용

### 2.4 파일 네이밍 개선
- [ ] 오디오 포함 여부 suffix: `_av` vs `_v`

### 2.5 에러 & 예외
- [ ] 권한 거부: `notify('error','화면 또는 오디오 권한이 거부되었습니다.')`
- [ ] 오디오 트랙 미획득 → 비디오만 녹화 fallback + 사용자 안내

### 2.6 테스트
- [ ] 시스템 오디오 ON 상태에서 정상 녹음 확인
- [ ] 오디오 OFF → 기존과 동일 파일 크기/메타 확인
- [ ] 오디오 ON인데 트랙 미획득 시 fallback 동작 확인

---
## 3. 공통 코드 정리 (선택: 타이머/오디오 뒤)
- [ ] 전역 → `AppState` 객체 1차 도입 (stream, recorder, timers)
- [ ] util: `formatTime(sec)` 재사용 위치 통일
- [ ] `init()` 함수에서 모든 DOM 캐시 (query 반복 제거)
- [ ] 이벤트 리스너 add 위치 묶기

---
## 4. 리스크 & 대응
| 리스크 | 설명 | 대응 |
|--------|------|------|
| 오디오 미지원 | 일부 브라우저/OS 시스템 오디오 불가 | 체크박스 자동 비활성 + tooltip |
| 긴 녹화 메모리 | chunks 배열 증가 | 즉시 flush(Blob URL 기록 후 clear) |
| 타이머 중복 | 재시작 시 clear 누락 | `clearAutoStop()` 일관 호출 |
| OCR 이후 확장 | CPU 점유 증가 | 모듈식 enable 플래그 유지 |

---
## 5. 구현 순서 제안 (실행 플로우)
1. (1.1~1.2) 타이머 UI + 로직 기본
2. (1.3~1.4) 예외/UX 마감 + 테스트
3. (2.1~2.2) 오디오 옵션 단일(시스템/탭) 녹음
4. (2.3~2.4) MediaRecorder 옵션/네이밍 정리
5. (2.5~2.6) 예외 + 테스트
6. (3) 공통 리팩터 (필요 범위만)

---
## 6. 산출물 정의
| 항목 | 설명 |
|------|------|
| 수정된 HTML | 추가 UI + script 변경 |
| 기능 플래그 | autoStopEnabled, recordAudio |
| 신규 함수 | `calcAutoStopMs`, `scheduleAutoStop`, `clearAutoStop`, `createDisplayStream` |
| 로그/알림 | `notify(type,msg)` 단순 구현 |
| 테스트 체크리스트 | TASKS 문서 내 유지 |

---
## 7. 완료 조건 (Definition of Done)
| 기능 | DoD |
|------|-----|
| 자동 종료 | 입력 검증 + 예약/취소 + 로그 + 남은 시간 표시 선택적 |
| 오디오 녹음 | 오디오 ON/OFF 토글, 오디오 트랙 존재 여부 안내, 파일명 suffix 구분 |
| 회귀 | 기존 스크린샷/연속 캡처 기능 정상 |
| 문서 | PRD 반영 + TASKS 문서 최신화 |

---
## 8. 후속 (차기 스프린트 후보)
- 설정 LocalStorage 저장 (last used quality/FPS/audio/autoStop)
- Toast 컴포넌트 경량 구현 (fade-out)
- OCR 기능 플래그 & 성능 측정 지표 수집 (grabFrame 평균 ms)
- 비디오 비트레이트 커스텀 (MediaRecorder `videoBitsPerSecond`)

