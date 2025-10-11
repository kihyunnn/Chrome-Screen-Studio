# Chrome Recording Studio

A professional browser-based screen recording tool that runs entirely in your browser without any server or external dependencies.

## ✨ Key Features

### 🎬 Recording Capabilities
- **Screen Recording**: Capture entire screen, specific window, or browser tab
- **Audio Recording**: Simultaneously record system audio and microphone
- **Webcam Recording**: Record from webcam with customizable resolution
- **Auto-split for Long Videos**: Automatically splits recordings over 30 minutes into multiple files (30min + remainder)

### 🎨 Advanced Features
- **Real-time Preview**: Monitor recording in real-time
- **Quality Control**: Adjustable video quality (1-10 Mbps)
- **Format Support**: WebM format with audio extraction capability
- **MP3 Conversion**: Automatic audio-only MP3 conversion (browser-based, up to 10-15 min recommended)
- **Recording History**: Manage all recordings with rename/download features
- **Responsive Design**: Modern dark theme UI with smooth animations

### 💾 Storage & Privacy
- **100% Local Processing**: All data stored in browser (IndexedDB)
- **No Server Required**: Complete privacy - nothing uploaded anywhere
- **Instant Download**: Direct download without compression or upload wait time

## 🚀 Getting Started

### Installation
1. Download or clone this repository
2. Open `chrome_recording_Studio.html` in Chrome/Edge browser
3. Allow camera/microphone/screen permissions when prompted

### Basic Usage
1. **Select Recording Source**
   - Choose from Screen/Window/Tab/Webcam
   - Enable/disable system audio or microphone as needed

2. **Configure Settings**
   - Set video quality (bitrate)
   - Choose webcam resolution if using camera

3. **Start Recording**
   - Click "Start Recording" button
   - Select screen/window to capture in the browser dialog
   - Real-time preview shows what's being recorded

4. **Stop & Save**
   - Click "Stop Recording" when finished
   - Recording automatically saved to list
   - MP3 audio extraction starts automatically (for videos under 15 min)

5. **Manage Recordings**
   - Rename: Click "Edit" to change file name
   - Download Video: Click "Video Download"
   - Download MP3: Click "MP3 Download" (when ready)

## 📋 Technical Details

### Browser Support
- ✅ Chrome 94+
- ✅ Edge 94+
- ⚠️ Firefox (limited support - some features may not work)
- ❌ Safari (not supported)

### Recording Specifications
- **Video Codec**: VP8/VP9 (WebM)
- **Audio Codec**: Opus
- **Auto-split**: 30-minute segments for long recordings
- **MP3 Conversion**: Browser-based using Web Audio API
  - Stable: 5-10 minutes
  - Caution: 10-20 minutes (may fail due to memory)
  - Risky: 20+ minutes (high chance of OOM error)

### Storage
- Uses IndexedDB for persistent storage
- Storage limit depends on browser (typically 50-100GB+)
- Recordings persist until manually deleted

## ⚠️ Important Notes

### MP3 Conversion Limitations
- Conversion happens entirely in browser memory
- **Recommended**: Videos under 10-15 minutes
- Longer videos may cause:
  - High memory usage
  - Browser slowdown/freeze
  - Conversion failure
- For longer videos, consider:
  - Using "Audio (WebM)" direct download instead
  - Converting with desktop FFmpeg afterwards

### Long Recording Considerations
- Auto-split feature activates at 30 minutes
- Each segment saved separately for stability
- Reduces memory pressure during extended recordings

### Performance Tips
- Close unnecessary tabs before long recordings
- Higher bitrate = larger file size but better quality
- Keep browser window active during recording to avoid throttling

## 🛠️ Troubleshooting

### Recording doesn't start
- Check browser permissions for camera/microphone/screen
- Try refreshing the page
- Ensure you're using a supported browser

### MP3 conversion fails
- Check video length (keep under 15 minutes)
- Close other tabs to free memory
- Try converting shorter segments

### Audio not captured
- Ensure "System Audio" is enabled
- On Windows, select "Share audio" in screen picker dialog
- Some apps may block audio capture (DRM-protected content)

### Memory issues
- Use auto-split feature for long recordings
- Download and clear recordings regularly
- Restart browser if performance degrades

## 📄 License
MIT License - Free to use and modify

## 🤝 Contributing
Issues and pull requests are welcome!

## 📧 Contact
For bugs or feature requests, please open an issue on GitHub.

---

**Note**: This tool is designed for legitimate recording purposes. Please respect copyright and privacy laws when recording content.