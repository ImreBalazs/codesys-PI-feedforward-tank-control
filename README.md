[🇭🇺 Magyar nyelvű leírásért kattints ide](./README_HU.md)

## 1. Project Overview

In this project, I implemented the full automation and optimal control architecture for an asymmetrical flow industrial water tank level control system in a **CODESYS V3.5** environment. Tank filling is managed by a fill valve, while discharge is handled by a gravity-drain valve. The discharge side has a larger capacity than the fill side, and the discharge flow rate varies proportionally with the height of the water column (water pressure) in the tank.

The system's objective is to accurately track the specified water level setpoint with the actual tank level, as well as to immediately compensate for sudden load changes (disturbances) occurring on the discharge side.

---

### Key Features and Properties

- **Four Operating Modes (Manual and Automatic Control):** The system can be flexibly switched from both the HMI interface and physical inputs:
  * **Hand PID (`iControlMode = 0`):** The water level setpoint is provided by a physical potentiometer ($0\text{--}10\text{ V} \rightarrow 0\text{--}300\text{ cm}$), with control executed by the PID loop.
  * **HMI Manual (`iControlMode = 1`):** Direct manual control of the fill and discharge valves from the HMI (switching between $0\text{ V}$ and $10\text{ V}$ states).
  * **HMI PID (`iControlMode = 2`):** Automatic level control based on a digital setpoint specified via the HMI.
  * **HMI FB+FF (`iControlMode = 3`):** Advanced Feedback + Feedforward (PI + Feedforward) control, where the discharge valve acts as an input disturbance, and the fill valve compensates automatically.

### Implemented System Components

- **CODESYS V3.5 PLC Control Program:** Development of the entire control architecture, signal scaling, interfaces, and mode selection logic.
- **Ignition HMI (Mobile / Android Compatible):** A modern, responsive industrial user interface for fully monitoring and controlling the system remotely, even from a smartphone or tablet.
- **Factory I/O Digital Twin:** A real-time 3D simulation of the tank system to model physical processes and perform live testing of control algorithms.

## 2. HMI User Interface (Ignition HMI)

System monitoring and control are provided by a mobile- and tablet-compatible responsive HMI built on Ignition. The operator interface is divided into two main views: **MainView** and **TrendView**.

---

### MainView
<img width="787" height="773" alt="{9FB7FA17-B848-4164-A4E1-689B645A708F}" src="https://github.com/user-attachments/assets/b3deec7b-cdbd-48d0-9835-23ce842dab4a" />

MainView is used for real-time process visualization, switching between operating modes, setting setpoints, and manual control.

#### Navigation
- **Chart button (top left):** Navigates to the TrendView (trend graph) display.

#### Process Indicators and Visual Feedback
- **Animated tank graphic:** Visually displays the tank level in percentage ($0\text{--}100\%$).
- **Fill Valve indicator:** Shows the current opening position of the fill valve as a percentage ($0\text{--}100\%$).
- **Discharge Valve indicator:** Shows the current opening position of the discharge valve as a percentage ($0\text{--}100\%$).
- **Discharge Flow Rate indicator:** Displays the instantaneous flow rate measured by the discharge flow meter ($m^3/s$).
- **Set Point display:** Displays the currently active water level setpoint in engineering units ($cm$).
- **Actual Level display:** Displays the measured water level ($PV$) in the tank in engineering units ($cm$).

#### Mode Selection Buttons
The currently active mode is highlighted in green by the HMI:
- **Hand PID:** Selects automatic PID control with manual setpoint adjustment via potentiometer (`iControlMode = 0`).
- **HMI Manual:** Switches to direct valve operation manual mode (`iControlMode = 1`).
- **HMI PID:** Activates automatic PID control operating with an HMI setpoint (`iControlMode = 2`).
- **HMI FB+FF:** Activates advanced Feedback + Feedforward (PI + Feedforward) disturbance compensation control (`iControlMode = 3`).

#### Control Elements and Input Fields
- **Fill button (left):** Opens and closes the fill valve in HMI Manual mode ($0\text{ V} / 10\text{ V}$).
- **Discharge button (left):** Opens and closes the discharge valve in HMI Manual mode ($0\text{ V} / 10\text{ V}$).
- **Set Point Input (bottom right):** Numerical input field for specifying the desired water level setpoint ($0\text{--}300\text{ cm}$) in HMI PID and HMI FB+FF modes.

---

### TrendView
<img width="783" height="704" alt="{D4CED08C-BE83-43E8-88B4-1FE098CE39F5}" src="https://github.com/user-attachments/assets/866a7348-85f8-44f7-8614-0d40237f0050" />

TrendView is used for real-time process dynamics monitoring and evaluating the settling response of the control loop.

#### Navigation
- **Back button (top center):** Navigates back to the MainView.

#### Trend Display and Data Analysis
- **Time axis range:** The displayed time range can be adjusted using the timeframe selector above the chart (e.g., `Last 1 minutes`).
- **Green curve (`rlevel_pv`):** Measured actual water level in the tank over time.
- **Orange curve (`rsetpoint_sp`):** Configured water level setpoint over time.
- **Pen Table (Bottom Data Table):** Displays detailed statistical data for registered signals:
  * **Current Value:** Instantaneous value.
  * **Minimum:** Minimum value recorded within the selected time window.
  * **Maximum:** Maximum value recorded within the selected time window.
  * **Average:** Average value over the selected time window.
