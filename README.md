#PySpectrometer-AS7343

Real-time spectral analysis application for the AS7343 18-channel spectral sensor, inspired by [PySpectrometer2](https://github.com/leswright1977/PySpectrometer2) by Les Wright.

![Spectrum Display](https://github.com/benb0jangles/PySpectrometer-AS7343/blob/main/img/spectrum-20260117-143753.png)
Test using Sansi 15w full spectrum bulb.
![Spectrum Bulb Test](https://github.com/benb0jangles/PySpectrometer-AS7343/blob/main/img/Screenshot%202026-01-17%20at%2014.30.32.png)

Video Demo here >>> [Youtube Demo](https://youtu.be/eOc9nWSASqQ)


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
# as7343_18ch_gain_esp32c3.ino
# Board: Seeed_Xiao_ESP32C3 Dev Module
# Upload
```

### 2. Run PySpectrometer-AS7343

```bash
cd PySpectrometer-AS7343
python3 PySpectrometer-AS7343.py
```

With options:
```bash
# Specify serial port
python3 PySpectrometer-AS7343.py --port /dev/ttyUSB0

# Start fullscreen
python3 PySpectrometer-AS7343.py --fullscreen

# Disable waterfall display
python3 PySpectrometer-AS7343.py --no-waterfall
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

PySpectrometer-AS7343 uses all 12 spectral channels of the AS7343, displayed in wavelength order:

| # | Key | Channel | Wavelength | Color |
|---|-----|---------|------------|-------|
| 1 | `1` | F1 | 405nm | Violet |
| 2 | `2` | F2 | 425nm | Deep Violet |
| 3 | `3` | FZ | 450nm | Blue |
| 4 | `4` | F3 | 475nm | Cyan |
| 5 | `5` | F4 | 515nm | Green |
| 6 | `6` | F5 | 550nm | Yellow-Green |
| 7 | `7` | FY | 555nm | Yellow-Green |
| 8 | `8` | FXL | 600nm | Orange |
| 9 | `9` | F6 | 620nm | Orange-Red |
| 10 | `a` | F7 | 670nm | Red |
| 11 | `b` | F8 | 730nm | Deep Red |
| 12 | `n` | NIR | 855nm | Near-Infrared |

The display range extends from 380nm to 905nm to show the full channel bandwidth coverage. The spectrum curve tapers naturally to zero at both edges.

Additional channels (not used for this project):
- **Clear**: Broadband visible light
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
| Hardware Cost | $50-300 (grating + camera) | $13-20 (sensor + MCU) |
| Setup | Optical alignment required | Plug and play |
| GUI Framework | OpenCV | OpenCV |
| Waterfall | Yes | Yes |
| Peak Detection | Yes | Yes |
| Savitzky-Golay | Yes | Yes |

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

## Author

- Benb0jangles (2026)

## License

MIT License - See LICENSE file for details.
