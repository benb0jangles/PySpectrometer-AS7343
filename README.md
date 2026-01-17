# PySpectrometer-AS7343

Real-time spectral analysis application for the AS7343 18-channel spectral sensor, inspired by [PySpectrometer2](https://github.com/leswright1977/PySpectrometer2) by Les Wright.

![Spectrum Display](media/screenshot.png)

## Overview

PySpectrometer-AS7343 transforms the AS7343 discrete spectral sensor into a visual spectrometer with:

- **Smooth interpolated spectrum** with rainbow gradient fill
- **Real-time waterfall display** showing spectral changes over time
- **Peak detection and labeling** with adjustable sensitivity
- **Savitzky-Golay filtering** for noise reduction
- **Calculated metrics**: Lux, PPFD, CCT, Blue%
- **PNG and CSV export** for data analysis
- **OpenCV-based GUI** with keyboard controls

## Hardware Requirements

- **AS7343** 18-channel spectral sensor
- **ESP32-C3** development board (or compatible)
- **USB cable** for serial communication
- Computer running Python 3.7+

## Software Dependencies

```bash
pip install opencv-python numpy scipy pyserial
```

## Quick Start

### 1. Upload Firmware

Upload the AS7343 firmware to your ESP32-C3:
```bash
# Open in Arduino IDE:
# ../as7343_18ch_gain/as7343_18ch_gain_esp32c3/as7343_18ch_gain_esp32c3.ino
# Board: ESP32C3 Dev Module
# Upload
```

### 2. Run PySpectrometer-AS7343

```bash
cd PySpectrometer-AS7343
python PySpectrometer-AS7343.py
```

With options:
```bash
# Specify serial port
python PySpectrometer-AS7343.py --port /dev/ttyUSB0

# Start fullscreen
python PySpectrometer-AS7343.py --fullscreen

# Disable waterfall display
python PySpectrometer-AS7343.py --no-waterfall
```

## Keyboard Controls

| Key | Function |
|-----|----------|
| **Display** |
| `q` | Quit application |
| `f` | Toggle fullscreen |
| `w` | Toggle waterfall display |
| `m` | Toggle measurement cursor |
| **Data** |
| `s` | Save spectrum (PNG + CSV) |
| `h` | Hold/freeze peaks |
| **Sensor** |
| `+`/`-` | Increase/decrease gain |
| `Space` | Toggle LED (reflectance mode) |
| **Processing** |
| `o`/`l` | Increase/decrease smoothing |
| `i`/`k` | Increase/decrease peak width |
| `u`/`j` | Increase/decrease peak threshold |

## Display Components

### Header Bar
Shows current status:
- Gain setting
- LED status
- Hold peaks indicator
- Smoothing level
- Peak threshold
- Calculated metrics (Lux, PPFD, CCT, Blue%, NIR, Clear)

### Spectrum Bar
Horizontal color bar showing raw spectrum intensity mapped to wavelength colors.

### Main Graph
- **Rainbow gradient fill** under the spectrum curve
- **Black outline** showing interpolated spectrum
- **Data point markers** (circles) at actual sensor wavelengths
- **Peak labels** showing detected peak wavelengths
- **Wavelength graticule** with 10nm minor / 50nm major gridlines
- **Measurement cursor** (when enabled) showing wavelength at mouse position

### Waterfall Display
Time-stacked spectra showing how the spectrum changes over time. Newest data appears at top.

## AS7343 Channel Mapping

The AS7343 provides 8 spectral channels (plus Clear, NIR, and duplicates):

| Channel | Wavelength | Color |
|---------|------------|-------|
| F1 | 405nm | Violet |
| F2 | 425nm | Deep Violet |
| FZ | 450nm | Blue |
| F3 | 475nm | Cyan |
| F4 | 515nm | Green |
| FY | 555nm | Yellow-Green |
| F5 | 550nm | Yellow-Green |
| FXL | 600nm | Orange |

Additional channels:
- **Clear**: Broadband visible light
- **NIR**: 855nm near-infrared
- **F4L, FYL, F5L, FXLL**: Left-side duplicates
- **FD, FDL**: Flicker detection

## Calculated Metrics

### Lux (Illuminance)
Calculated from the Clear channel. Requires calibration with a reference lux meter.

### PPFD (Photosynthetic Photon Flux Density)
- If PAR calibrated: Uses spectral channels with calibration factor
- If only Lux calibrated: Estimated as `Lux / 65` (typical for LED grow lights)

### CCT (Correlated Color Temperature)
Estimated from the ratio of blue (FZ-450nm) to orange (FXL-600nm) channels.

### Blue Percentage
Ratio of blue channels (F1-F4) to total spectral intensity (F1-FXL).

## Output Files

When you press `s` to save, three files are created:

1. **spectrum-YYYYMMDD-HHMMSS.png** - Screenshot of the display
2. **spectrum-YYYYMMDD-HHMMSS.csv** - Interpolated spectrum data (wavelength, intensity)
3. **channels-YYYYMMDD-HHMMSS.csv** - Raw AS7343 channel values

## Calibration

Calibration data is stored in `as7343_calibration.json` and persists across sessions.

### Lux Calibration
1. Position AS7343 and reference lux meter side-by-side
2. Note the reference lux reading
3. Edit `as7343_calibration.json`:
```json
{
  "lux": {
    "7": 0.5  // gain_index: factor
  }
}
```
Factor = Reference_Lux / Clear_Channel_Value

### PAR Calibration
Similar process using a quantum sensor for reference.

## Comparison: PySpectrometer2 vs PySpectrometer-AS7343

| Feature | PySpectrometer2 | PySpectrometer-AS7343 |
|---------|----------------|----------------------|
| Sensor | Camera + diffraction grating | AS7343 spectral sensor |
| Resolution | 800 continuous points | 8 discrete wavelengths |
| Wavelength Range | ~380-780nm (calibration dependent) | 405-600nm (fixed) |
| Hardware Cost | $50-300 (grating + camera) | $20-50 (sensor + MCU) |
| Setup | Optical alignment required | Plug and play |
| GUI Framework | OpenCV | OpenCV |
| Waterfall | Yes | Yes |
| Peak Detection | Yes | Yes |
| Savitzky-Golay | Yes | Yes |

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                  PySpectrometer-AS7343               │
├─────────────────────────────────────────────────────┤
│                                                      │
│  ┌──────────┐    Serial    ┌──────────────────┐    │
│  │ AS7343   │◄────────────►│ ESP32-C3         │    │
│  │ Sensor   │   115200     │ (Firmware)       │    │
│  └──────────┘   baud       └────────┬─────────┘    │
│                                      │ USB          │
│                                      ▼              │
│  ┌──────────────────────────────────────────────┐  │
│  │              Python Application               │  │
│  ├──────────────────────────────────────────────┤  │
│  │  ┌─────────────┐  ┌─────────────────────┐   │  │
│  │  │ Serial      │  │ specFunctions.py    │   │  │
│  │  │ Reader      │  │ - wavelength_to_rgb │   │  │
│  │  │ (Thread)    │  │ - savitzky_golay    │   │  │
│  │  └──────┬──────┘  │ - find_peaks        │   │  │
│  │         │         │ - interpolate       │   │  │
│  │         ▼         └─────────────────────┘   │  │
│  │  ┌─────────────────────────────────────┐    │  │
│  │  │         Data Processing             │    │  │
│  │  │  - Interpolation (cubic spline)     │    │  │
│  │  │  - Smoothing (Savitzky-Golay)       │    │  │
│  │  │  - Peak detection                   │    │  │
│  │  │  - Metrics calculation              │    │  │
│  │  └──────────────┬──────────────────────┘    │  │
│  │                 │                           │  │
│  │                 ▼                           │  │
│  │  ┌─────────────────────────────────────┐    │  │
│  │  │         OpenCV Rendering            │    │  │
│  │  │  - Header (status, metrics)         │    │  │
│  │  │  - Spectrum bar (color intensity)   │    │  │
│  │  │  - Main graph (rainbow fill)        │    │  │
│  │  │  - Waterfall (time history)         │    │  │
│  │  └─────────────────────────────────────┘    │  │
│  └──────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
```

## File Structure

```
PySpectrometer-AS7343/
├── PySpectrometer-AS7343.py  # Main application
├── specFunctions.py          # Utility functions
├── README.md                 # This file
├── as7343_calibration.json   # Calibration data (auto-created)
└── media/                    # Screenshots (optional)
```

## Troubleshooting

### No serial ports found
- Check USB cable connection
- Verify ESP32-C3 drivers are installed
- On Linux, you may need: `sudo usermod -a -G dialout $USER`

### No data displayed
- Ensure firmware is uploaded and running
- Check that the correct port is selected
- Verify baud rate is 115200

### Noisy spectrum
- Increase smoothing with `o` key
- Adjust gain with `+`/`-` keys
- Ensure stable lighting conditions

### Peaks not detected
- Lower threshold with `j` key
- Adjust peak width with `k`/`i` keys

## Credits

- **PySpectrometer2** by Les Wright - Original inspiration and algorithm reference
- **AS7343 Datasheet** by ams-OSRAM - Channel specifications
- **SparkFun** - AS7343 Arduino library

## License

MIT License - See LICENSE file for details.
