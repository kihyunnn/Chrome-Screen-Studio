# 화면 녹화 & 스크린샷 웹 도구 PRD (Product Requirements Document)

## 1. 문서 목적
현재 단일 `HTML` 파일로 동작하는 화면 녹화/스크린샷 웹 도구를 체계적으로 확장하기 위한 기준을 정의한다. 향후 기능 추가(자동 시간 종료, 오디오 녹화, OCR 기반 타이머 감지 등)를 안전하게 진행하고 일관된 UI/코드 품질을 유지한다.

## 2. 현재 버전 요약 (Baseline v1.0)
| 구분 | 내용 |
|------|------|
| 아키텍처 | Pure HTML + Inline CSS + Vanilla JS (단일 파일) |
| 주요 API | `getDisplayMedia`, `MediaRecorder`, `Canvas` |
| 코덱/포맷 | 기본 브라우저 제공 (WebM, VP8/VP9/H.264 가능성) |
| 기능 | 화면 녹화, 스크린샷 1회/연속 캡처, 품질(FHD/4K)/FPS 설정, 기록 목록(다운로드), 미리보기/모달 확대 |
| 미구현 | 오디오 녹음 옵션, 자동 시간 종료, OCR 감지 기반 종료, 설정 저장, 진단/로그 |
| 대상 브라우저 | Chromium 기반 (Chrome / Edge 최신) 우선 |

### 2.1 현재 UI 구조
- 좌측: 설정 패널 (녹화 품질, FPS, 연속 캡처 여부/간격, 스크린샷 포맷 등)
- 우측: 상태 표시 (녹화 여부, 타이머), 실시간 미리보기, 최근 스크린샷, 기록 리스트
- 모달: 스크린샷 확대 표시

### 2.2 상태 관리 (암묵적 전역 변수)
- `mediaRecorder`, `recordedChunks`, `stream`, `continuousStream`
- 카운터: `recordingCount`, `screenshotCount`
- 타이머: `timerInterval`, `continuousInterval`
- UI 상태: 녹화 여부(버튼 enable/disable), LIVE 뱃지, 빈 상태 메시지 표시 여부

### 2.3 주요 흐름
1. Start Recording → `getDisplayMedia` → `MediaRecorder` 시작 → 실시간 프리뷰 표시
2. Stop Recording → `mediaRecorder.stop()` → blob 생성 → 기록 리스트에 추가
3. Screenshot → `drawImage` → `toDataURL` → 기록 추가 & 미리보기 갱신
4. 연속 캡처 → 간격 기반 `setInterval` 로 반복 스크린샷 → 리스트 누적

### 2.4 현재 한계/리스크
| 영역 | 이슈 | 영향 |
|------|------|------|
| 단일 파일 | 규모 증가 시 유지보수 난이도 상승 | 가독성 저하 |
| 전역 변수 | 예측 어려움/사이드이펙트 가능 | 버그 추적 어려움 |
| 오류 처리 | 사용자 피드백 제한 (alert 위주) | UX 저하 |
| 포맷 옵션 | 비디오 코덱/비트레이트 제어 부족 | 품질/용량 제어 어려움 |
| 접근성 | 키보드 포커스/ARIA 미구현 | 접근성 부족 |
| 국제화 | 하드코딩된 한국어 UI | 다국어 확장 어려움 |
| 성능 | 대용량 연속 캡처 누적 메모리 | 탭 느려짐 가능 |

---

## 3. 확장 목표 (Planned v1.1 ~ v1.3)
| 버전 | 범위 | 핵심 가치 |
|------|------|-----------|
| v1.1 | 자동 녹화 종료(고정 시간), 오디오 녹음 옵션 | 핵심 기능 확장 |
| v1.2 | OCR 기반 화면 타이머 감지 종료(선택적) | 학습 영상 자동화 |
| v1.3 | 설정 로컬 저장, 에러/진단 로그, 간단 성능 최적화 | 지속 사용성 향상 |

---

## 4. 신규 기능 상세 요구사항
### 4.1 기능 A: 자동 시간 기반 녹화 종료
| 항목 | 내용 |
|------|------|
| 목적 | 사용자가 미리 설정한 분/초 경과 후 자동 `stopRecording()` |
| 입력 | (체크박스) 자동 종료 활성화, (숫자) 분, 초 |
| 기본값 | 비활성 / 10분 |
| 한계 | 최대 180분 제한 (UX 가이드) |
| 로직 | 녹화 시작 시 `setTimeout` 예약 → 수동 중지 시 `clearTimeout` |
| UI 변화 | 설정 패널에 섹션 추가, 진행 중 남은 시간 표시(선택적) |
| 에러 | 잘못된 값(0 이하, NaN) → 비활성 처리 & 사용자 안내 |

### 4.2 기능 B: 오디오 녹화(시스템 / 탭 / 마이크)
| 항목 | 내용 |
|------|------|
| 목적 | 강의 영상 사운드 포함 녹화 |
| 입력 | (체크박스) 오디오 포함, (선택) 마이크 혼합 여부 |
| API | `navigator.mediaDevices.getDisplayMedia({ audio: true })` + `getUserMedia({ audio: true })` (옵션) |
| 믹싱 | `AudioContext` + `MediaStreamDestination` 으로 트랙 병합 |
| 제약 | 브라우저/OS 별 시스템 오디오 허용 상이, 사용자 권한 필요 |
| 에러 | 오디오 트랙 0 → 안내 메시지 표시 |
| 저장 포맷 | 기존 WebM 그대로 (추후 MSE/Transcode 논외) |

### 4.3 기능 C: OCR 기반 화면 타이머 감지 후 종료 (실험적)
| 항목 | 내용 |
|------|------|
| 목적 | 영상 플레이어 우측하단 표시 시간 도달 시 자동 종료 |
| 입력 | (체크박스) 기능 활성화, 목표시간 문자열 (MM:SS or H:MM:SS), 감지 간격 초, 감지 영역 기준 (%) |
| 라이브러리 | `tesseract.js` CDN |
| 처리 흐름 | (1) 비디오 트랙 frame grab → (2) 지정 영역 crop → (3) OCR → (4) Regex 시간 매칭 → (5) 목표 도달 시 stop |
| 성능 | 2~5초 간격 권장, crop 영역 최소화 |
| 실패 전략 | 5회 연속 미인식 시 로그+UI 안내 (기능 유지) |
| 위험 | CPU 사용량 상승, 오탐/미탐 가능 → 사용자는 보조 기능으로 인식 |

---

## 5. UX / UI 설계 지침
| 항목 | 지침 |
|------|------|
| 시각 상태 | 녹화 중: LIVE 배지 + 빨간 점 + 타이머 / 자동 종료 예약 시 남은 시간 툴팁 |
| 버튼 상태 | 녹화 중에는 Start 비활성, Stop 활성, 타이머/설정 필드 일부 잠금 |
| 오류 메시지 | 토스트 스타일(간단 div fade) 추가 고려 (alert 대체) |
| 접근성 | 버튼에 `aria-label`, 포커스 outline 유지 |
| 반응형 | 1024 이하: 좌/우 패널 수직 스택 |

---

## 6. 기술 설계
### 6.1 구조 (단일 파일 유지 전제)
```text
<html>
  <head>
    <style>/* 모듈화된 섹션 주석 */</style>
    <script src="https://cdn.jsdelivr.net/npm/tesseract.js@4/dist/tesseract.min.js" defer></script> <!-- (필요 시 조건부 로드) -->
  </head>
  <body>
    <!-- Settings Panel -->
    <!-- Result Panel -->
    <!-- Modal -->
    <script>
      // 1) State 캡슐화용 즉시실행 모듈 (IIFE) 또는 namespace 객체
      // 2) 기능별 함수 그룹: recording, screenshot, continuous, autoStop, audio, ocr
      // 3) init() → 이벤트 바인딩 & UI 초기화
    </script>
  </body>
</html>
```

### 6.2 상태 객체 리팩터 (향후)
```js
const AppState = {
  media: { stream: null, recorder: null, chunks: [] },
  counters: { recording: 0, screenshot: 0 },
  timers: { ui: null, continuous: null, autoStop: null, ocr: null, startTime: null },
  flags: { isRecording: false, hasRecordings: false },
};
```

### 6.3 의사코드 (핵심 신규 기능)
#### 자동 종료
```js
function scheduleAutoStop(ms) {
  clearAutoStop();
  AppState.timers.autoStop = setTimeout(() => {
    stopRecording({ reason: 'AUTO_STOP' });
  }, ms);
}
```

#### 오디오 믹싱
```js
async function buildStream({ withSystemAudio, withMic }) {
  const display = await navigator.mediaDevices.getDisplayMedia({ video: {...}, audio: withSystemAudio });
  if (!withMic) return display;
  const mic = await navigator.mediaDevices.getUserMedia({ audio: true });
  const ctx = new AudioContext();
  const dest = ctx.createMediaStreamDestination();
  if (display.getAudioTracks()[0]) ctx.createMediaStreamSource(display).connect(dest);
  ctx.createMediaStreamSource(mic).connect(dest);
  const tracks = [ ...display.getVideoTracks(), ...dest.stream.getAudioTracks() ];
  return new MediaStream(tracks);
}
```

#### OCR 감지 루프
```js
async function ocrLoop() {
  const { stream } = AppState.media;
  if (!stream) return;
  const track = stream.getVideoTracks()[0];
  const imageCapture = new ImageCapture(track);
  const bitmap = await imageCapture.grabFrame();
  cropToCanvas(bitmap, region);
  const { data: { text } } = await Tesseract.recognize(canvas, 'eng');
  if (matchTarget(text)) stopRecording({ reason: 'OCR_TARGET' });
}
```

---

## 7. 에러 처리 & 로깅
| 상황 | 처리 |
|------|------|
| 권한 거부 | UI 배너: "화면 공유 권한이 필요합니다" |
| 오디오 트랙 없음 | 체크박스 해제 + 경고 배너 |
| OCR 라이브러리 미로드 | 기능 비활성 (회색 처리) |
| 녹화 중 예외 | 즉시 `stopRecording` + 로그 + 사용자 알림 |

로깅 전략(경량):
```js
function log(event, payload = {}) {
  console.debug(`[LOG:${event}]`, payload);
}
```

---

## 8. 성능 고려사항
| 항목 | 전략 |
|------|------|
| 메모리 | 오래된 기록 X개 초과 시 LRU 삭제 옵션 (추후) |
| OCR | 영역 crop + 2~5초 주기 + 필요 시 `requestIdleCallback` |
| 캔버스 | 캔버스 재사용 (생성 최소화) |
| Reflow | 클래스 토글 / `documentFragment` 사용 (리스트 추가 시) |

---

## 9. 테스트 전략
| 구분 | 시나리오 |
|------|----------|
| 단위 | 시간 포맷 변환, 자동 종료 ms 계산, 파일명 생성 규칙 |
| 수동 | (1) 녹화 시작/중지 (2) 스크린샷 (3) 연속 캡처 (4) 오디오 포함 녹화 (5) 자동 종료 (6) OCR 종료 |
| 성능 | 10분 연속 캡처 후 UI 반응성 확인 |
| 회귀 | 기존 기능: 녹화/스크린샷 영향 없는지 |

---

## 10. 보안/프라이버시
| 항목 | 내용 |
|------|------|
| 권한 | 최소 권한 (화면 + 선택적 오디오/마이크) |
| 저장 | 모든 데이터는 메모리/브라우저 다운로드(로컬)로만 처리, 서버 전송 없음 |
| 외부 라이브러리 | `tesseract.js` CDN 무결성 옵션 (SRI) 고려 |

---

## 11. 릴리즈 계획
| 단계 | 설명 |
|------|------|
| v1.1 개발 | 자동 종료 + 오디오 옵션 추가, 회귀 테스트 |
| v1.1 릴리즈 | 태그: `v1.1` / PRD 갱신 |
| v1.2 PoC | OCR 기능 플래그(기본 Off) 배포 |
| v1.2 확정 | 정확도 OK 시 기본 On 안내 |
| v1.3 | 설정 저장(LocalStorage), 경량 토스트, 리팩터 일부 |

---

## 12. 용어 정의
| 용어 | 정의 |
|------|------|
| 기록(record) | 녹화된 비디오 또는 캡처된 이미지 하나 |
| 연속 캡처 | 주기적 스크린샷 자동 촬영 기능 |
| OCR 종료 | 화면 타이머 글자를 인식하여 목표 도달 자동 종료 |

---

## 13. 오픈 이슈 / 후속 과제
| ID | 제목 | 설명 | 우선순위 |
|----|------|------|---------|
| #1 | 비디오 비트레이트 제어 | MediaRecorder 옵션 확장 | M |
| #2 | HLS 변환 | 장시간 녹화 스트리밍 저장 | L |
| #3 | 국제화(i18n) | 다국어 UI 구조 도입 | L |
| #4 | 다크/라이트 전환 | 사용자 테마 토글 | M |
| #5 | 리스트 검색/필터 | 파일 유형/기간별 필터 | L |

---

## 14. 승인 & 변경 관리
- 문서 버전: 1.0 (초안)
- 변경 시: 상단 버전 + 변경 내역 섹션 추가

---

## 15. 변경 로그 (Changelog)
| 날짜 | 버전 | 작성자 | 변경 내용 |
|------|------|--------|-----------|
| 2025-10-07 | 1.0 | 작성 | 최초 작성 (Baseline + 확장 계획) |
