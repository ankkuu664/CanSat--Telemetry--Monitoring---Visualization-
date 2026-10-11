# CanSat Ground Station

## Overview

I worked on this project to build a ground-station interface for monitoring CanSat telemetry in real time. The application displays incoming flight and sensor data in a graphical interface, making it easier to observe how the system behaves during a simulated flight or when connected to a CanSat/on-board computer.

The dashboard can display altitude, temperature, barometric pressure, gyroscope readings, acceleration on three axes, battery level, elapsed flight time, and free-fall status.

## What the Project Can Do

- Display telemetry values and plots in a live GUI.
- Receive data from a connected device over a serial port.
- Run in simulation mode when hardware is not available.
- Simulate different flight stages, from powered ascent through descent and landing.
- Save received telemetry to a CSV file for later review.
- Detect available serial ports across Linux, macOS, and Windows.
- Ignore telemetry frames that are incomplete or contain invalid numeric values.

![Ground Station Interface](https://i.imgur.com/zDY3DnY.gif)

## Tools and Technologies

- **Python 3.10+**
- **PyQtGraph and Qt** for the graphical interface and plots
- **pyserial** for serial communication
- **NumPy** for numerical data support
- **uv** for managing the Python environment and running the project

## Getting Started

### Requirements

Install Python 3.10 or newer and [`uv`](https://docs.astral.sh/uv/). The project dependencies are defined in the repository.

### 1. Download the project

```bash
git clone https://github.com/darods/CanSat-Ground-station.git
cd CanSat-Ground-station
```

### 2. Run without hardware

Use dummy mode to try the interface with generated telemetry:

```bash
uv run main.py --dummy
```

This mode is useful for checking the dashboard without connecting a board or sensor.

### 3. Connect to a device

Connect your CanSat or on-board computer, identify the serial port assigned by your operating system, and run the application with that port.

**Linux**
```bash
uv run main.py --port /dev/ttyUSB0 --baud 9600
```

**macOS**
```bash
uv run main.py --port /dev/tty.usbmodem14101 --baud 9600
```

**Windows PowerShell**
```powershell
uv run main.py --port COM3 --baud 9600
```

The default baud rate is `9600`. Replace the example port with the one assigned to your device.

## Command-Line Options

| Option | Purpose |
| --- | --- |
| `-h`, `--help` | Show available options |
| `--port PORT` | Choose the serial port |
| `--baud BAUD` | Set the serial baud rate (default: `9600`) |
| `--dummy` | Use simulated telemetry |
| `--interval INTERVAL` | Set the GUI polling interval in milliseconds (default: `50`) |

The polling interval has a minimum of 10 ms. It controls how often the interface checks for a telemetry frame; it does not guarantee the hardware's sampling or transmission rate.

## Telemetry Data Format

The application expects each telemetry frame as a comma-separated line ending with a newline. The first 11 values are required, and a 12th battery value is optional. Required values must be numeric and finite.

| Position | Field | Meaning | Unit / Values |
| ---: | --- | --- | --- |
| 0 | Timestamp | Elapsed flight time | milliseconds |
| 1 | Altitude | Relative altitude | meters |
| 2 | Free fall | Free-fall indicator | `0` = No, `1` = Yes |
| 3 | Temperature | Temperature reading | °C |
| 4 | Pressure | Barometric pressure | hPa |
| 5 | Gyro pitch | Pitch angular rate | °/s |
| 6 | Gyro roll | Roll angular rate | °/s |
| 7 | Gyro yaw | Yaw angular rate | °/s |
| 8 | Accel X | Acceleration on X axis | m/s² |
| 9 | Accel Y | Acceleration on Y axis | m/s² |
| 10 | Accel Z | Acceleration on Z axis | m/s² |
| 11 | Battery | Battery level (optional) | % |

Example frame:

```text
1250,42.7,0,18.6,1008.4,0.12,-0.34,0.08,0.15,-0.09,9.72,96.5
```

Make sure your device sends values in the same order and uses consistent units. Battery information appears only when the sender includes the optional 12th value.

## Flight Simulation

Dummy mode generates example telemetry for these stages:

1. Powered ascent
2. Ballistic coast toward apogee
3. Descent
4. Parachute descent
5. Landing

The generated values help test the interface and data flow. They are simulated values, not validated flight predictions, so they should not be used to make real flight decisions.

## Saving Telemetry

1. Start the application.
2. Click **Start storage** to begin recording.
3. Click **Stop storage** when you want to finish.

The data is saved to `flight_data.csv` in the application's current working directory. The CSV contains the telemetry fields and a local date/time recording timestamp.

## Running Tests

From the project directory, run:

```bash
uv run pytest
```

To run one test module:

```bash
uv run pytest tests/test_communication.py
```

To run tests matching a keyword:

```bash
uv run pytest -k "battery or free_fall"
```

## Troubleshooting

- **No serial port appears:** Check the USB cable, board power, drivers, and the port name shown by your operating system.
- **Permission denied on Linux:** Check access permissions for the serial device.
- **No telemetry appears:** Verify the baud rate, newline at the end of each frame, field order, and numeric values.
- **The app switches to dummy mode:** Check the terminal output and confirm that the selected serial port is available and can be opened.
- **Dependencies are missing:** Run the commands from the project root and check that the dependency configuration is present.

## Possible Improvements

Some useful directions for further development include adding telemetry export options, improving the layout for different screen sizes, and testing the interface with real hardware across different flight conditions.
