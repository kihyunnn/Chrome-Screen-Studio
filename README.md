# Screen Capture Studio

A lightweight, browser-based screen recording and screenshot tool built with pure HTML, CSS, and JavaScript. No server required, no data transmission — everything runs locally in your browser.

## ✨ Features

### 🎥 Screen Recording
- **High-Quality Recording**: Support for 4K (3840×2160), Full HD (1920×1080), and HD (1280×720)
- **Frame Rate Control**: Choose between 60, 30, or 24 FPS
- **System Audio Capture**: Record screen with tab/system audio (browser-dependent)
- **Auto-Stop Timer**: Set automatic recording duration (up to 180 minutes)
- **Live Preview**: Real-time preview while recording with LIVE indicator
- **WebM Format**: Outputs in WebM format (VP8/VP9 codec)

### 📸 Screenshot Capture
- **Multiple Formats**: PNG, JPEG, or WebP
- **Single Capture**: Take instant screenshots
- **Continuous Mode**: Automatic interval-based screenshots (customizable interval in seconds)
- **Image Preview**: View captured images with zoom functionality

### 📋 Recording Management
- **History List**: All recordings and screenshots saved in chronological order
- **Quick Download**: One-click download for each capture
- **Preview & Playback**: Review videos and images before downloading
- **Timestamp Tracking**: Each item tagged with capture date/time and duration

## 🚀 Quick Start

### Option 1: Local File
1. Download `화면 캡처 및 녹화 프로그램.html`
2. Open the file in Chrome or Edge browser
3. Click "녹화 시작" (Start Recording) and grant screen sharing permission
4. Your recording will be saved locally

### Option 2: Deploy to Server
Since this is a static HTML file, you can deploy it anywhere:

**Using Docker + Nginx:**
```bash
# Create Dockerfile
FROM nginx:alpine
COPY 화면\ 캡처\ 및\ 녹화\ 프로그램.html /usr/share/nginx/html/index.html
EXPOSE 80

# Build and run
docker build -t screen-capture-studio .
docker run -d -p 8080:80 screen-capture-studio
```

**Using Python HTTP Server:**
```bash
python -m http.server 8000
```

Then access via `http://localhost:8000`

## 🎮 Usage

### Recording a Video
1. Select desired quality and frame rate
2. Enable "오디오 포함" (Include Audio) if you want system audio
3. (Optional) Enable "자동 종료" (Auto-Stop) and set duration
4. Click "녹화 시작" (Start Recording)
5. Choose screen/tab to share in browser dialog
6. Click "녹화 중지" (Stop Recording) when done
7. Download from the recording history list

### Taking Screenshots
1. Select image format (PNG/JPEG/WebP)
2. Click "스크린샷" (Screenshot)
3. Select screen/window to capture
4. Image appears in preview and history list

### Continuous Screenshots
1. Check "연속 스크린샷 모드" (Continuous Mode)
2. Set capture interval in seconds
3. Click "연속 캡처 시작" (Start Continuous)
4. Each screenshot auto-saves to history
5. Click "연속 캡처 중지" (Stop Continuous) to end

## 🛠️ Technical Details

### Browser Compatibility
- **Recommended**: Chrome 94+, Edge 94+ (full feature support)
- **Limited**: Firefox (may lack system audio capture)
- **Not Supported**: Safari, mobile browsers (missing `getDisplayMedia` API)

### APIs Used
- `navigator.mediaDevices.getDisplayMedia()` - Screen capture
- `MediaRecorder` - Video encoding
- `Canvas API` - Screenshot rendering
- `Blob` & `URL.createObjectURL()` - File generation

### File Formats
| Type | Format | Details |
|------|--------|---------|
| Video | WebM | VP8/VP9 codec, variable bitrate |
| Screenshot | PNG/JPEG/WebP | User-selectable |

### Privacy & Security
- ✅ **100% Local Processing**: No data sent to any server
- ✅ **No Tracking**: No analytics or external scripts
- ✅ **User Permissions**: Browser prompts for screen/audio access
- ⚠️ **DRM Content**: May show black screen for protected content

## 📐 Architecture

```
Single HTML File
├── Inline CSS (Dark Theme)
├── Vanilla JavaScript
│   ├── MediaRecorder Management
│   ├── Timer & Auto-Stop Logic
│   ├── Screenshot Canvas Processing
│   └── UI State Management
└── No External Dependencies
```

## ⚙️ Configuration

### Auto-Stop Timer
- Max duration: 180 minutes
- Default: 10 minutes
- Remaining time display during recording

### Video Quality Presets
| Quality | Resolution | Bitrate |
|---------|-----------|---------|
| 4K | 3840×2160 | 20 Mbps |
| Full HD | 1920×1080 | 8 Mbps |
| HD | 1280×720 | 5 Mbps |

## 🐛 Known Limitations

1. **Audio Capture**: System audio availability depends on OS/browser support
   - Windows Chrome/Edge: ✅ System audio
   - macOS: ⚠️ Limited to tab audio
   - Linux: Varies by distribution

2. **File Format**: Direct MP4 export not supported
   - WebM is native output
   - Convert to MP4 using FFmpeg if needed:
     ```bash
     ffmpeg -i recording.webm -c:v libx264 -c:a aac output.mp4
     ```

3. **Memory**: Long recordings accumulate chunks in RAM
   - Monitor browser memory usage
   - Recommended max: 60 minutes per session

4. **Browser Volume**: If Chrome is muted in system mixer, audio won't be captured

## 🔮 Roadmap

- [ ] LocalStorage settings persistence
- [ ] Toast notifications (replace alerts)
- [ ] OCR-based screen timer detection (experimental)
- [ ] Video bitrate customization
- [ ] Multi-language support (i18n)

## 📄 Documentation

- `PRD_화면녹화도구.md` - Product Requirements Document (Korean)
- `TASKS_타이머_오디오기능.md` - Implementation Task List (Korean)

## 🤝 Contributing

This is a personal project but feedback is welcome! Open issues or PRs if you find bugs or have suggestions.

## 📜 License

MIT License - Feel free to use and modify

## 🙏 Acknowledgments

Built with:
- Chrome DevTools Media APIs
- Do Hyeon Font (Google Fonts)
- Pure vanilla JavaScript (no frameworks!)

---

**Note**: This tool respects user privacy by design. All processing happens in your browser, and no data ever leaves your device.
