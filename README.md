# Audio Recorder with Auto-Silence Detection and Input Monitoring

Professional audio recorder with manual hotkey control, automatic silence-based recording, and input monitoring for capturing audio from Traktor Audio 6 (or any audio interface).

## Features

- **Input Monitoring (ARM)**: Monitor your input in real-time through your speakers/headphones before recording
- **Silence Trimming**: Automatically remove silence from the beginning and/or end of recordings
- **Manual Recording**: Start/stop with F9/F10 hotkeys
- **Auto-Record Mode**: Automatically starts recording when audio is detected after silence, stops when silence returns
- **Real-time Level Meters**: Visual feedback of input levels for left and right channels
- **Professional Format**: Records WAV files at 44.1kHz, 24-bit stereo
- **Configurable Silence Detection**: Adjustable threshold (dB) and duration
- **Auto-naming**: Sequential file numbering with timestamps
- **Device Selection**: Choose separate input and output devices
- **RX11 Integration**: Optionally auto-open recordings in iZotope RX11

## Installation


1. Install Python 3.8 or higher
2. Install dependencies:
```bash
pip install -r audio_recorder_requirements.txt
```

**Note**: The `keyboard` module requires administrator/root privileges for global hotkeys on some systems. If hotkeys don't work, run the script with elevated permissions.

### Monitoring Latency

When using ARM (input monitoring), you may experience a small delay (latency) between your input and what you hear. This is normal and depends on:
- Your audio interface drivers (ASIO drivers typically have lowest latency)
- System performance
- Buffer size (blocksize in the code - currently 2048 samples ≈ 46ms at 44.1kHz)

For lower latency:
- Use ASIO drivers if available (Windows)
- Close other applications
- If you experience crackling, the latency is too low for your system

**Recording latency:** The recorded audio is perfectly in sync - only the monitoring has latency.

## Usage

### Basic Operation

1. Run the recorder:
```bash
python audio_recorder_armed.py
```

2. Select your input device (Traktor Audio 6 Input B should be auto-selected if connected)
3. Select your output device (for monitoring - can be Traktor Output or your system speakers)
4. Choose output folder and file naming preferences
5. Use ARM feature and recording modes:

### ARM (Input Monitoring)

The ARM feature lets you monitor (hear) your input in real-time before recording - essential for checking levels and making sure everything sounds right.

**To enable monitoring:**
1. Check "🎧 ARM (Enable Input Monitoring)" 
2. Adjust monitor volume (slider)
3. You'll now hear your input through the selected output device
4. Status shows "ARMED - Monitoring Input"

**Note:** You can be armed without recording (to check levels), or record without being armed (if you don't need to hear the input).

### Manual Mode (Default)
- Press **F9** to start recording
- Press **F10** to stop recording
- Files are saved with sequential numbering + timestamp

### Auto-Record Mode
1. Enable "Auto-Record" checkbox
2. Adjust silence threshold (default: -40 dB)
3. Adjust silence duration (default: 2 seconds)
4. The recorder will:
   - Wait for audio above threshold
   - Start recording automatically
   - Stop when silence duration is reached
   - Perfect for recording vinyl, cassettes, or multi-track sessions

### Silence Trimming

The recorder can automatically remove silence from your recordings before saving:

**Trim from Beginning**: Removes all silence before the first sound
- Useful when you hit record before the audio starts
- Eliminates pre-roll silence in auto-record mode

**Trim from End**: Removes all silence after the last sound
- Cleans up recordings that captured silence after the music stopped
- Perfect for auto-record mode

**Trim Threshold**: The dB level below which audio is considered "silence"
- Default: -50 dB (quieter than the auto-record threshold)
- Lower values (e.g., -60 dB) = more aggressive trimming
- Higher values (e.g., -40 dB) = only trim very quiet sections

**Example**: If you record a song with 2 seconds of silence before it starts and 3 seconds after it ends, enabling both trim options will save just the song itself.

**Note**: The console will print how much silence was trimmed, e.g., "Trimmed 4.23s of silence (original: 184.50s, final: 180.27s)"

### Settings Explained

- **Silence Threshold**: Audio level (in dB) below which is considered silence
  - Lower values (e.g., -50 dB) = more sensitive (detects quieter sounds)
  - Higher values (e.g., -30 dB) = less sensitive (only louder sounds trigger)
  
- **Silence Duration**: How many seconds of continuous silence before stopping
  - Useful for avoiding premature stops during quiet passages
  - Typical: 2-3 seconds for music with pauses

- **Min Recording Duration**: Prevents very short recordings from being saved

## RX11 Integration Options

### Option 1: Auto-Open in RX11 (Recommended)
Modify the `save_recording` method to automatically open files in RX11:

```python
# After saving the file, add:
import subprocess
rx11_path = r"C:\Program Files\iZotope\RX 11\RX 11 Audio Editor.exe"
subprocess.Popen([rx11_path, self.current_filename])
```

### Option 2: Batch Processing with RX11
If you have specific RX11 processing chains you want to apply:

1. Save all recordings first
2. In RX11, use Batch Processor:
   - File → Batch Processor
   - Add your recording folder
   - Apply your processing chain (noise reduction, EQ, etc.)
   - Process all files at once

### Option 3: RX11 Command Line (Advanced)
RX11 has limited command-line support. Check documentation for:
```bash
# Example (syntax may vary):
"C:\Program Files\iZotope\RX 11\RX 11 Audio Editor.exe" -process "preset.rxp" "input.wav"
```

## Workflow Example: Vinyl Ripping

1. Connect turntable to Traktor Audio 6 Input B
2. Select Traktor Input B Left/Right as input device
3. Select your speakers/headphones as output device
4. **Enable ARM to hear the turntable** 
5. Adjust monitor volume to comfortable level
6. Enable Auto-Record mode
7. Set silence threshold to -45 dB
8. Set silence duration to 3 seconds
9. Optionally enable "Auto-open in RX11"
10. Start your vinyl record
11. Recorder automatically creates separate files for each track
12. Listen in real-time as it records
13. Files open in RX11 automatically for:
   - Click/pop removal
   - De-noising
   - Leveling
   - Export to MP3/FLAC

## Troubleshooting

### "No audio device selected"
- Click "Refresh Devices"
- Ensure Traktor Audio 6 is connected and recognized by your OS
- Check that drivers are installed

### Hotkeys not working
- Run script as administrator (Windows) or with sudo (Linux/Mac)
- Or use the GUI buttons instead

### Audio clipping/distortion
- Check input levels in Traktor/system settings
- Ensure levels stay in the green/yellow range (not red)
- If clipping in monitors, reduce monitor volume

### Can't hear monitoring when armed
- Check output device is correct
- Verify output device isn't muted in system settings
- Increase monitor volume slider
- Make sure ARM checkbox is enabled (status should show "ARMED")

### Crackling/dropouts when monitoring
- Latency is too low for your system
- Close other applications
- Consider using ASIO drivers (Windows)
- In code, increase blocksize from 2048 to 4096 or 8192

### Auto-record starts/stops erratically
- Adjust silence threshold (try -45 dB or lower)
- Increase silence duration to 3-4 seconds
- Check for background noise/hum

## File Format

All recordings are saved as:
- **Format**: WAV (PCM)
- **Sample Rate**: 44,100 Hz
- **Bit Depth**: 24-bit
- **Channels**: Stereo (2)

This format provides excellent quality for archival and is compatible with all audio editing software.

## Tips

1. **Use ARM to check levels first**: Enable monitoring before recording to verify everything sounds right
2. **Monitor volume**: Keep monitor volume at 50-70% to avoid feedback if using speakers
3. **Test your levels**: Record a short test before long sessions
4. **Watch the meters**: Green is good, yellow is okay, red means clipping
5. **Use auto-mode for multi-track**: Perfect for albums or tapes
6. **Manual mode for single takes**: When you know exactly when to start/stop
7. **Save settings persist**: Your preferences are saved between sessions
8. **Headphones recommended**: Use headphones for monitoring to avoid feedback

## Integration with Video Converter

After recording, use the MP3-to-YouTube converter to create videos:
1. Convert WAV to MP3 if needed (or use WAV directly)
2. Choose a static image or video
3. Create YouTube-ready video file

## License

Free to use and modify for personal and commercial projects.
