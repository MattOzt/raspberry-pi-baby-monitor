# Raspberry Pi Baby Monitor

A multifunctional embedded baby monitor system built on Raspberry Pi OS using Python. Developed as part of an Operating Systems module at university.

## Features

- **Real-time cry detection** — USB microphone + RMS analysis + reference sample similarity matching
- **Automated email alerts** — Gmail SMTP notifications with cooldown logic to prevent spam
- **Live CCTV streaming** — Flask web server + Picamera2 MJPEG stream accessible from any browser
- **Lullaby playback** — DSP-generated audio (sine wave + white noise) triggered automatically on cry detection
- **Windows GUI controller** — Remote Tkinter GUI to start/stop modules via SSH (Paramiko)
- **Systemd service** — Auto-starts on boot, restarts on failure

## System Architecture

```
cry_detection.py   ← main process: audio capture, RMS analysis, event triggers
lullaby.py         ← subprocess: DSP audio generation + playback
cctv_stream.py     ← Flask server: Picamera2 MJPEG stream
remote_gui.py      ← Windows GUI: SSH remote control via Paramiko
```

## Tech Stack

- **Language:** Python 3
- **Hardware:** Raspberry Pi 4B, USB microphone, Raspberry Pi Camera Module
- **Libraries:** sounddevice, NumPy, SciPy, Flask, Picamera2, OpenCV, Paramiko, Tkinter
- **OS:** Raspberry Pi OS (64-bit), systemd

## Setup

### On the Raspberry Pi

1. Install system dependencies:
   ```bash
   sudo apt update && sudo apt upgrade -y
   sudo apt install python3 python3-pip python3-venv libatlas-base-dev python3-picamera2 alsa-utils ffmpeg
   ```

2. Create and activate a virtual environment:
   ```bash
   python3 -m venv venv
   source venv/bin/activate
   ```

3. Install Python dependencies:
   ```bash
   pip install -r requirements.txt
   ```

4. Set up environment variables:
   ```bash
   cp .env.example .env
   # Fill in your email credentials and Pi details
   ```

5. Add cry reference samples to a `cry_samples/` folder (WAV format).

### Running the modules

```bash
# Cry detection (main module)
python3 cry_detection.py

# CCTV stream (access at http://<pi-ip>:5000)
python3 cctv_stream.py

# Lullaby playback
python3 lullaby.py
```

### Autostart with systemd

```bash
sudo cp baby-monitor.service /etc/systemd/system/
sudo systemctl enable baby-monitor
sudo systemctl start baby-monitor
```

### Windows remote GUI

Run `remote_gui.py` on a Windows machine on the same network. Set `PI_HOST`, `PI_USER`, and `PI_PASSWORD` in your `.env`.

## Performance (Raspberry Pi 4B )

| Scenario | CPU Usage | Memory |
|----------|-----------|--------|
| Idle | 1–3% | ~480 MB |
| Cry detection only | 12–18% | +50–60 MB |
| CCTV streaming only | 30–45% | — |
| Combined (typical) | 50–65% | ~650–750 MB |
| Peak (all modules) | 75–85% | — |

Thermal readings stayed well below the 80°C throttle point throughout testing.
