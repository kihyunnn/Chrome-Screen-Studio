# Chrome Recording Studio - PRD (Product Requirements Document)

## 1. Document Purpose
Defines the feature specifications and future expansion plans for a screen recording web tool operating as a single `HTML` file. Core features including auto-stop timer and audio recording have been completed, with additional features to be implemented selectively.

## 2. Current Version Summary (v1.1 Complete)
| Category | Content |
|----------|---------|
| Architecture | Pure HTML + Inline CSS + Vanilla JS (single file) |
| Main APIs | `getDisplayMedia`, `MediaRecorder` |
| Codec/Format | WebM (VP8/VP9), output extension: `.webm` |
| Implemented | Screen recording, quality settings (4K/FHD/HD), FPS control, auto-stop timer (max 180 minutes), system/tab audio recording, recording history & download, live preview |
| Not Implemented | OCR-based timer detection, LocalStorage settings persistence, Toast notifications, multi-language support |
| Target Browsers | Chromium-based (Chrome 94+ / Edge 94+) recommended |
| File Name | `Chrome_Recording_Studio.html` |

### 2.1 Current UI Structure
- Left: Settings panel (recording quality, FPS, audio options, auto-stop timer settings)
- Right: Status display (recording status, timer), live preview, recording history list
- No screenshot or modal features

### 2.2 State Management (Global Variables)
- **Media**: `mediaRecorder`, `recordedChunks`, `stream`
- **Counters**: `recordingCount`, `hasRecordings`
- **Timers**: `timerInterval`, `autoStopTimeoutId`, `autoStopDeadline`, `startTime`
- **Feature Flags**: `autoStopEnabled`, `audioEnabled`
- **UI State**: Recording status (button enable/disable), LIVE badge, empty state message display

### 2.3 Main Flow
1. Start Recording → `getDisplayMedia` → `MediaRecorder` starts → Live preview display
2. Stop Recording → `mediaRecorder.stop()` → blob generation → Add to recording list
3. Auto-stop → Timer expires → Automatic `stopRecording()` execution

### 2.4 Current Limitations/Risks
| Area | Issue | Impact |
|------|-------|--------|
| Single File | Increasing maintenance difficulty as scale grows | Reduced readability |
| Global Variables | Unpredictable side effects | Difficult bug tracking |
| Error Handling | Limited user feedback (mainly alerts) | Poor UX |
| Format Options | Lack of video codec/bitrate control | Difficult quality/size control |
| Accessibility | Missing keyboard focus/ARIA | Poor accessibility |
| Internationalization | Hardcoded Korean UI | Difficult multi-language expansion |

---

## 3. Implemented Features Details

### 3.1 Auto Time-based Recording Stop ✅
| Item | Content |
|------|---------|
| Purpose | Automatically execute `stopRecording()` after user-set minutes/seconds |
| Input | (Checkbox) Auto-stop activation, (Number) minutes, seconds |
| Default | Inactive / 10 minutes 0 seconds |
| Limits | Max 180 minutes, deactivates if 0m 0s input |
| Implementation | `calcAutoStopMs()` validation, `scheduleAutoStop()` scheduling, `clearAutoStop()` release |
| UI | Settings panel section, real-time remaining time display, orange when ≤60s / red when ≤10s |
| Error Handling | NaN/negative → default value, >180min → clamp to 180min + notification |

### 3.2 Audio Recording (System/Tab Audio) ✅
| Item | Content |
|------|---------|
| Purpose | Record with lecture audio included |
| Input | (Checkbox) Include audio |
| API | `navigator.mediaDevices.getDisplayMedia({ audio: true })` |
| Constraints | System audio support varies by browser/OS (Chrome/Edge Windows: full support) |
| Implementation | `createDisplayStream({ audio })` function creates stream, checks audio tracks |
| Filename | `_av` suffix when audio included, `_v` without |
| Error Handling | Audio track not acquired → video-only recording fallback + user notification |
| Note | Microphone mixing feature excluded (system/tab audio only supported) |

---

## 4. Future Expansion Plans (Optional)

### 4.1 Medium Priority (v1.2 Candidates)
- **LocalStorage Settings Persistence**: Restore last used quality/FPS/audio/auto-stop settings
- **Toast Notifications**: Replace alerts with smooth fade-in/out messages

### 4.2 Low Priority (v1.3 and Beyond)
- **OCR-based Screen Timer Detection**: Using tesseract.js, experimental feature
- **Multi-language Support (i18n)**: English/Korean switching
- **Custom Video Bitrate**: Extend MediaRecorder options
- **Dark/Light Theme Toggle**: User-selectable

---

## 5. Removed Features
- **Screenshot Capture**: Single and continuous screenshot features removed - now pure recording tool
- **Microphone Mixing**: Simultaneous system audio and microphone recording → Excluded due to complexity and echo issues
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

## 7. Technical Design

### 7.1 Structure (Single File)
```text
<html>
  <head>
    <style>/* Dark Theme CSS + Modern Gradient Effects */</style>
    <link> Font Awesome icons
    <link> Noto Sans KR font
  </head>
  <body>
    <!-- Settings Panel: quality/FPS/audio/auto-stop settings -->
    <!-- Result Panel: status/timer/preview/recording list -->
    <script>
      // Global variables: mediaRecorder, stream, timers, flags
      // Core functions: startRecording(), stopRecording()
      // Auto-stop: calcAutoStopMs(), scheduleAutoStop(), clearAutoStop()
      // Audio: createDisplayStream({ audio })
    </script>
  </body>
</html>
```

### 7.2 Implemented Functions
| Function | Role |
|----------|------|
| `calcAutoStopMs(m, s)` | Input validation (0~180min clamping, NaN handling), clamp notification |
| `scheduleAutoStop(ms)` | setTimeout scheduling, clear existing schedule, set deadline |
| `clearAutoStop()` | Release timer, reset deadline |
| `createDisplayStream({ audio })` | Call getDisplayMedia, include audio option, check tracks |
| `formatTime(sec)` | MM:SS format conversion |
| `addToList(type, url, name, time, detail)` | Add to recording list (video/_av/_v) |
| `startRecording()` | Start recording, schedule auto-stop, acquire audio stream |
| `stopRecording()` | Stop recording, release auto-stop, cleanup streams |
| `startTimer()` | Update recording timer UI, display remaining time (≤60s orange, ≤10s red) |

### 7.3 Not Implemented Functions (Future)
- `ocrLoop()`: OCR-based timer detection (optional)

---

## 8. Error Handling
| Situation | Response |
|-----------|----------|
| Screen sharing permission denied | alert: "Cannot start screen recording" |
| Audio track not acquired | Video-only recording proceeds, console log |
| Auto-stop input error | NaN/negative → default, >180min → clamp + alert |
| Stream ends during recording | Auto-trigger stopRecording() |

---

## 9. Performance Considerations
| Item | Current State | Future Improvement |
|------|---------------|-------------------|
| Memory | recordedChunks array accumulation | Consider periodic flush for long recordings |
| Canvas Reuse | N/A (screenshot feature removed) | N/A |
| DOM Queries | getElementById on each event | Cache during initialization |

---

## 10. Testing Strategy
| Category | Scenario |
|----------|----------|
| Basic Features | (1) Start/stop recording |
| Auto-stop | Set 5s then verify normal stop, check timer release on manual stop |
| Audio | Record with audio ON/OFF each, verify fallback when track not acquired |
| Input Validation | Auto-stop 0m 0s, >180min, NaN input handling |
| Regression | No impact on existing recording features after new feature addition |

---

## 11. Security & Privacy
| Item | Content |
|------|---------|
| Permissions | Screen sharing + audio (browser prompts) |
| Data Processing | 100% local processing, no external server transmission |
| External Dependencies | None (OCR is optional feature, currently not implemented) |
| DRM Content | Protected content may display as black screen |

---

## 12. Release Plan
| Version | Status | Main Content |
|---------|--------|--------------|
| v1.0 | ✅ Complete | Basic recording |
| v1.1 | ✅ Complete | Auto-stop + system audio |
| v1.2 | 📋 Planned | LocalStorage settings persistence, Toast notifications |
| v1.3 | 📋 Planned | OCR timer detection (experimental) |

---

## 13. Terminology
| Term | Definition |
|------|------------|
| Recording | Captured video file |
| Auto-stop | Automatic recording termination after set time |
| System Audio | Tab or full system sound (browser/OS dependent) |

---

## 14. Open Issues / Follow-up Tasks
| ID | Title | Priority | Status |
|----|-------|----------|--------|
| #1 | LocalStorage settings persistence | Medium | Not Started |
| #2 | Toast notifications (replace alerts) | Medium | Not Started |
| #3 | Custom video bitrate | Low | Not Started |
| #4 | Multi-language support (i18n) | Low | Not Started |
| #5 | OCR screen timer detection | Low | Not Started |

---

## 15. Changelog
| Date | Version | Changes |
|------|---------|---------|
| 2025-10-07 | 1.0 | Initial creation (Baseline + expansion plan) |
| 2025-10-08 | 1.1 | v1.1 feature completion reflection (auto-stop, audio), English title change, microphone mixing exclusion confirmed, screenshot feature removed, filename changed to Chrome_Recording_Studio.html |

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
