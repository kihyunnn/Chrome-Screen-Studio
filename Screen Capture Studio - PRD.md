# Screen Capture Studio - PRD (Product Requirements Document)

## 1. 문서 목적
현재 단일 `HTML` 파일로 동작하는 화면 녹화/스크린샷 웹 도구의 기능 명세와 향후 확장 계획을 정의한다. 자동 시간 종료, 오디오 녹화 등 핵심 기능이 구현 완료되었으며, 추가 기능은 선택적으로 진행한다.

## 2. 현재 버전 요약 (v1.1 Complete)
| 구분 | 내용 |
|------|------|
| 아키텍처 | Pure HTML + Inline CSS + Vanilla JS (단일 파일) |
| 주요 API | `getDisplayMedia`, `MediaRecorder`, `Canvas` |
| 코덱/포맷 | WebM (VP8/VP9) |
| 구현 완료 | 화면 녹화, 스크린샷 1회/연속 캡처, 품질(4K/FHD/HD)/FPS 설정, 자동 시간 종료(최대 180분), 시스템/탭 오디오 녹음, 기록 목록·다운로드, 미리보기/모달 확대 |
| 미구현 | OCR 기반 타이머 감지, 설정 LocalStorage 저장, Toast 알림, 다국어 지원 |
| 대상 브라우저 | Chromium 기반 (Chrome 94+ / Edge 94+) 권장 |

### 2.1 현재 UI 구조
- 좌측: 설정 패널 (녹화 품질, FPS, 연속 캡처 여부/간격, 스크린샷 포맷 등)
- 우측: 상태 표시 (녹화 여부, 타이머), 실시간 미리보기, 최근 스크린샷, 기록 리스트
- 모달: 스크린샷 확대 표시

### 2.2 상태 관리 (전역 변수)
- `mediaRecorder`, `recordedChunks`, `stream`, `continuousStream`
- 카운터: `recordingCount`, `screenshotCount`
- 타이머: `timerInterval`, `continuousInterval`, `autoStopTimeoutId`
- 기능 플래그: `autoStopEnabled`, `audioEnabled`
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

## 3. 구현 완료 기능 상세

### 3.1 자동 시간 기반 녹화 종료 ✅
| 항목 | 내용 |
|------|------|
| 목적 | 사용자가 미리 설정한 분/초 경과 후 자동 `stopRecording()` |
| 입력 | (체크박스) 자동 종료 활성화, (숫자) 분, 초 |
| 기본값 | 비활성 / 10분 0초 |
| 한계 | 최대 180분 제한, 0분 0초 입력 시 비활성 처리 |
| 구현 | `calcAutoStopMs()` 검증, `scheduleAutoStop()` 예약, `clearAutoStop()` 해제 |
| UI | 설정 패널 섹션, 남은 시간 실시간 표시, 60초 이하 orange / 10초 이하 red 강조 |
| 에러 처리 | NaN/음수 → 기본값, 180분 초과 → 180분 clamp + 알림 |

### 3.2 오디오 녹화 (시스템/탭 오디오) ✅
| 항목 | 내용 |
|------|------|
| 목적 | 강의 영상 등 오디오 포함 녹화 |
| 입력 | (체크박스) 오디오 포함 |
| API | `navigator.mediaDevices.getDisplayMedia({ audio: true })` |
| 제약 | 브라우저/OS별 시스템 오디오 지원 상이 (Chrome/Edge Windows: 완전 지원) |
| 구현 | `createDisplayStream({ audio })` 함수로 스트림 생성, 오디오 트랙 체크 |
| 파일명 | 오디오 포함 시 `_av` suffix, 없으면 `_v` |
| 에러 처리 | 오디오 트랙 미획득 → 비디오만 녹화 fallback + 사용자 안내 |
| 참고 | 마이크 믹싱 기능 제외 (시스템/탭 오디오만 지원) |

---

## 4. 향후 확장 계획 (Optional)

### 4.1 우선순위 중 (v1.2 후보)
- **LocalStorage 설정 저장**: 마지막 사용한 품질/FPS/오디오/자동종료 설정 복원
- **Toast 알림**: alert 대체, 부드러운 fade-in/out 메시지

### 4.2 우선순위 하 (v1.3 이후)
- **OCR 기반 화면 타이머 감지**: tesseract.js 활용, 실험적 기능
- **다국어 지원 (i18n)**: 영어/한국어 전환
- **비디오 비트레이트 커스텀**: MediaRecorder 옵션 확장
- **다크/라이트 테마 토글**: 사용자 선택 가능

---

## 5. 제거된 기능
- **마이크 믹싱**: 시스템 오디오와 마이크 동시 녹음 → 복잡도 및 echo 문제로 제외
---

## 6. UX / UI 설계 지침
| 항목 | 지침 |
|------|------|
| 시각 상태 | 녹화 중: LIVE 배지 + 빨간 점 애니메이션 + 타이머 / 자동 종료 활성 시 남은 시간 실시간 표시 |
| 버튼 상태 | 녹화 중: Start 비활성, Stop 활성 / 대기 중: Start 활성, Stop 비활성 |
| 색상 강조 | 남은 시간 60초 이하 orange, 10초 이하 red |
| 오류 메시지 | 현재 alert 사용, 추후 Toast 스타일 고려 |
| 접근성 | 버튼에 `aria-label` 추가 권장, 포커스 outline 유지 |
| 반응형 | 1024px 이하 고려 필요 (현재 고정 레이아웃) |

---

---

## 7. 기술 설계

### 7.1 구조 (단일 파일)
```text
<html>
  <head>
    <style>/* Dark Theme CSS */</style>
  </head>
  <body>
    <!-- Settings Panel: 품질/FPS/오디오/자동종료 설정 -->
    <!-- Result Panel: 상태/타이머/미리보기/기록 리스트 -->
    <!-- Modal: 이미지 확대 -->
    <script>
      // 전역 변수: mediaRecorder, stream, timers, flags
      // 핵심 함수: startRecording(), stopRecording(), takeScreenshot()
      // 자동 종료: calcAutoStopMs(), scheduleAutoStop(), clearAutoStop()
      // 오디오: createDisplayStream({ audio })
    </script>
  </body>
</html>
```

### 7.2 구현 완료 함수
| 함수 | 역할 |
|------|------|
| `calcAutoStopMs(m, s)` | 입력 검증 (0~180분 clamping, NaN 처리) |
| `scheduleAutoStop(ms)` | setTimeout 예약, 기존 예약 clear |
| `clearAutoStop()` | 타이머 해제 |
| `createDisplayStream({ audio })` | getDisplayMedia 호출, 오디오 옵션 포함 |
| `formatTime(sec)` | MM:SS 포맷 변환 |
| `addToList(type, url, name, time, detail)` | 기록 리스트 추가 (video/_av, screenshot) |

### 7.3 미구현 함수 (향후)
- `mixAudioStreams()`: 마이크 + 시스템 오디오 믹싱 (제외됨)
- `ocrLoop()`: OCR 기반 타이머 감지 (선택적)
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
---

## 8. 에러 처리
| 상황 | 처리 방식 |
|------|----------|
| 화면 공유 권한 거부 | alert: "화면 녹화를 시작할 수 없습니다" |
| 오디오 트랙 미획득 | 비디오만 녹화 진행, 콘솔 로그 |
| 자동 종료 입력 오류 | NaN/음수 → 기본값, 180분 초과 → clamp + alert |
| 녹화 중 스트림 종료 | 자동 stopRecording() 트리거 |

---

## 9. 성능 고려사항
| 항목 | 현재 상태 | 향후 개선 |
|------|-----------|----------|
| 메모리 | recordedChunks 배열 누적 | 장시간 녹화 시 주기적 flush 검토 |
| 캔버스 재사용 | 매번 생성 | 전역 캔버스 재사용 (연속 캡처) |
| DOM 쿼리 | 이벤트마다 getElementById | 초기화 시 캐싱 |

---

## 10. 테스트 전략
---

## 10. 테스트 전략
| 구분 | 시나리오 |
|------|----------|
| 기본 기능 | (1) 녹화 시작/중지 (2) 스크린샷 (3) 연속 캡처 |
| 자동 종료 | 5초 설정 후 정상 종료, 수동 중지 시 타이머 해제 확인 |
| 오디오 | 오디오 ON/OFF 각각 녹화, 트랙 미획득 시 fallback 확인 |
| 입력 검증 | 자동 종료 0분 0초, 180분 초과, NaN 입력 처리 |
| 회귀 | 새 기능 추가 후 기존 스크린샷/연속 캡처 영향 없음 |

---

## 11. 보안 & 프라이버시
| 항목 | 내용 |
|------|------|
| 권한 | 화면 공유 + 오디오 (브라우저 프롬프트) |
| 데이터 처리 | 100% 로컬 처리, 외부 서버 전송 없음 |
| 외부 의존성 | 없음 (OCR은 선택 기능, 현재 미구현) |
| DRM 콘텐츠 | 보호된 콘텐츠는 검은 화면으로 표시될 수 있음 |

---

## 12. 릴리즈 계획
| 버전 | 상태 | 주요 내용 |
|------|------|-----------|
| v1.0 | ✅ 완료 | 기본 녹화/스크린샷 |
| v1.1 | ✅ 완료 | 자동 종료 + 시스템 오디오 |
| v1.2 | 📋 계획 | LocalStorage 설정 저장, Toast 알림 |
| v1.3 | 📋 계획 | OCR 타이머 감지 (실험적) |

---

## 13. 용어 정의
| 용어 | 정의 |
|------|------|
| 기록(record) | 녹화된 비디오 또는 캡처된 이미지 |
| 연속 캡처 | 주기적 자동 스크린샷 촬영 모드 |
| 자동 종료 | 설정 시간 경과 시 자동 녹화 중지 |
| 시스템 오디오 | 탭 또는 전체 시스템 사운드 (브라우저/OS 의존) |

---

## 14. 오픈 이슈 / 후속 과제
| ID | 제목 | 우선순위 | 상태 |
|----|------|---------|------|
| #1 | LocalStorage 설정 저장 | 중 | 미착수 |
| #2 | Toast 알림 (alert 대체) | 중 | 미착수 |
| #3 | 비디오 비트레이트 커스텀 | 하 | 미착수 |
| #4 | 다국어 지원 (i18n) | 하 | 미착수 |
| #5 | OCR 화면 타이머 감지 | 하 | 미착수 |

---

## 15. 변경 로그 (Changelog)
| 날짜 | 버전 | 변경 내용 |
|------|------|-----------|
| 2025-10-07 | 1.0 | 최초 작성 (Baseline + 확장 계획) |
| 2025-10-08 | 1.1 | v1.1 기능 완료 반영 (자동 종료, 오디오), 영문 제목 변경, 마이크 믹싱 제외 확정 |
