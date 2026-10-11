# CanSat Ground Station GUI 🚀

Real-time telemetry monitoring interface for CanSats and On-Board Computers (OBCs). The application visualizes flight telemetry, including altitude, speed, 3-axis acceleration, 3-axis gyroscope readings, barometric pressure, temperature, battery level, elapsed flight time, and free-fall status.

The project supports Linux, macOS, and Windows 11 using Python 3.10+, `uv`, PyQtGraph, and Qt through PyQt6/PyQt5-compatible imports.

![Ground Station Interface](https://i.imgur.com/zDY3DnY.gif)

## Features

- **Real-time telemetry visualization** for flight and sensor data.
- **Serial communication** with CanSat/OBC hardware through `pyserial`.
- **Dummy simulation mode** for trying the GUI without connected hardware.
- **Flight-phase simulation** covering powered ascent, ballistic coast, descent, parachute descent, and landing.
- **CSV data logging** to `flight_data.csv` using the GUI's **Start storage** and **Stop storage** controls.
- **Cross-platform serial-port discovery** for Linux, macOS, and Windows.
- **Telemetry frame validation** to skip incomplete, non-numeric, or non-finite sensor frames.
- **Automated tests and CI** as described in the repository's test and workflow configuration.

## Requirements

- Python **3.10 or newer**
- [`uv`](https://docs.astral.sh/uv/)
- Dependencies declared by the project (including Qt/PyQtGraph, `pyserial`, and NumPy)

### Install `uv`

**macOS / Linux**
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

**Windows (PowerShell)**
```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

## Installation and Usage

### 1. Clone the repository

```bash
git clone https://github.com/darods/CanSat-Ground-station.git
cd CanSat-Ground-station
```

### 2. Run in dummy mode

Dummy mode generates simulated telemetry and does not require a connected sensor or serial device.

```bash
uv run main.py --dummy
```

### 3. Connect to real hardware

Use the serial device name for your operating system. Replace the examples with the port assigned to your board.

**Linux**
```bash
uv run main.py --port /dev/ttyUSB0 --baud 9600
```

**macOS**
```bash
uv run main.py --port /dev/tty.usbmodem14101 --baud 9600
```

**Windows 11 (PowerShell)**
```powershell
uv run main.py --port COM3 --baud 9600
```

The application uses a default baud rate of **9600**. The serial device must transmit telemetry frames in the format described below. If a serial connection cannot be opened, the current communication implementation falls back to dummy mode.

## Command-Line Options

```text
-h, --help           Show the help message and exit
--port PORT          Serial port device (e.g. /dev/ttyUSB0, COM3)
--baud BAUD          Serial baud rate (default: 9600)
--dummy              Force simulated telemetry mode
--interval INTERVAL  GUI telemetry polling interval in milliseconds (default: 50)
```

The application enforces a minimum polling interval of **10 ms**. The polling interval controls how often the GUI asks for a telemetry frame; it does not guarantee that the hardware itself samples or transmits at that rate.

## Telemetry Frame Format

The application expects comma-separated telemetry frames. The first 11 fields are required and must be numeric and finite. A 12th battery field is optional.

| Index | Field | Description | Unit / Values |
|---:|---|---|---|
| `0` | Timestamp | Elapsed flight time | milliseconds |
| `1` | Altitude | Relative altitude | meters |
| `2` | Free fall | Free-fall indicator | `0` = No, `1` = Yes |
| `3` | Temperature | Temperature | °C |
| `4` | Pressure | Barometric pressure | hPa |
| `5` | Gyro pitch | Pitch angular rate | °/s |
| `6` | Gyro roll | Roll angular rate | °/s |
| `7` | Gyro yaw | Yaw angular rate | °/s |
| `8` | Accel X | Acceleration on X axis | m/s² |
| `9` | Accel Y | Acceleration on Y axis | m/s² |
| `10` | Accel Z | Acceleration on Z axis | m/s² |
| `11` | Battery | Battery charge level (optional) | % |

Example frame with the optional battery field:

```text
1250,42.7,0,18.6,1008.4,0.12,-0.34,0.08,0.15,-0.09,9.72,96.5
```

Send each frame as one comma-separated line terminated by a newline. Hardware should use consistent units and field order. Battery display data is available only when the sender supplies the optional 12th field; dummy mode generates it.

## Dummy Flight Simulation

The dummy simulator provides a synthetic flight profile so the GUI can be exercised without hardware. Its modeled phases include:

1. Motor burn / powered ascent
2. Ballistic coast toward apogee
3. Ballistic descent with a terminal-speed cap
4. Parachute descent
5. Landing

It generates simulated altitude, pressure, temperature, gyroscope, acceleration, and battery values. These values are for demonstration and software testing; they are not a validated flight-dynamics model and must not be used as real flight predictions.

## Data Logging

1. Start the application.
2. Click **Start storage** to begin appending telemetry rows to `flight_data.csv`.
3. Click **Stop storage** to stop logging.

The CSV contains the telemetry fields received by the application, followed by a local date/time recording timestamp. The output file is written relative to the application's current working directory unless another path is configured in code.

## Testing

Run the test suite from the project root:

```bash
uv run pytest
```

Run a specific test module:

```bash
uv run pytest tests/test_communication.py
```

Run tests matching a keyword:

```bash
uv run pytest -k "battery or free_fall"
```

The shared test configuration sets Qt to offscreen mode so compatible GUI tests can run without a physical display. See `docs/TESTING.md` if that file is present in your checkout for test organization and conventions.

## CI and Code Quality

The project documentation describes GitHub Actions workflows for checks such as linting, formatting, tests, dependency auditing, and static security analysis. Check the repository's `.github/workflows/` directory for the workflows currently enabled in your checkout.

## Technology Stack

- **Language:** Python 3.10+
- **Environment/package management:** [`uv`](https://docs.astral.sh/uv/)
- **GUI and plotting:** Qt with PyQtGraph; the code imports Qt through `pyqtgraph.Qt`
- **Serial communication:** [`pyserial`](https://pyserial.readthedocs.io/)
- **Numerical/data support:** NumPy (listed by the project documentation)

## Troubleshooting

- **No serial port detected:** Check the USB cable, board power, drivers, and the device name shown by your operating system.
- **Permission denied on Linux:** Check your distribution's serial-device permissions and group configuration.
- **No telemetry displayed:** Confirm the baud rate, newline-terminated CSV format, field order, and that the first 11 fields are numeric and finite.
- **Application starts in dummy mode:** Check the console output; the current communication implementation falls back to dummy mode if no port is available or the selected port cannot be opened.
- **Missing dependencies:** Run the project through `uv` from the repository root and confirm the project's dependency configuration is present.

## License

This project identifies its license as the **MIT License**. See the [`LICENSE`](LICENSE) file for the complete terms.

## Attribution

The original project is credited to **Daniel Alejandro Rodríguez Suárez**, leader of the ATL research seedbed associated with the LIDER Research Group at Universidad Distrital Francisco José de Caldas. Preserve the original attribution and license notices when redistributing or modifying the project.
