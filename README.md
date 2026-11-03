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

 ## 3. PLC Code and Control Logic

The PLC program was developed in the CODESYS V3.5 environment. The control and mode-switching tasks are implemented by a main program block (`PLC_PRG`), a custom scaling function block (`FC_Scale`), a custom-written PID controller block (`FB_PID_MAN`), and the built-in CODESYS library `PID` function block.

---

### Variable Declarations (`VAR`)

The structure of the input/output, internal, and parameter variables used in the program, based on the code:

```pascal
PROGRAM PLC_PRG
VAR
    // Physical outputs (valve position commands 0-10V)
    rFillValveCmd      : REAL;
    rDischargeValveCmd : REAL;
    
    // Mode selector register
    iControlMode       : INT;
    
    // Raw and scaled analog signals
    rLevel_Raw         : REAL;
    rLevel_PV          : REAL;
    rPot_Raw           : REAL;
    rSetpoint_SP       : REAL;
    rScaledPot         : REAL;
    rFlowMeterRaw      : REAL;
    rFlowMeterScaled   : REAL;
    
    // Control and enable bit signals
    bAutoMode          : BOOL := TRUE;
    bEnableFFFB        : BOOL;
    bHMIControl        : BOOL := FALSE;
    rHMISetPOint       : REAL;
    bHMI_FillBtn       : BOOL;
    bHMI_DischargeBtn  : BOOL;
    
    // PID_1 parameters (Hand PID / HMI PID)
    rKp                : REAL := 1.0;
    rTi                : REAL := 5.0;
    rTd                : REAL;
    rTs                : REAL := 0.02;
    
    // PID_2 and Feedforward parameters (FB+FF)
    rFKp               : REAL := 0.7;
    rFTi               : REAL := 15.0;
    rFTd               : REAL := 0;
    rFKff              : REAL := 1.2;
    rKDisc             : REAL := 2;
    
    // Error values and deadband
    rError             : REAL;
    rDeadband          : REAL := 1.5;
    
    // Control outputs and intermediate calculation variables
    rPID_Out           : REAL;
    rFBPID_Out         : REAL;
    rFFProduct         : REAL;
    rNoLimitOut        : REAL;
    rLimitedOut        : REAL;
    
    // Function block instances
    FB_PID_MAN_1       : FB_PID_MAN;
    FB_PID_MAN_2       : PID;
END_VAR
```

---

### Mode Selection Logic (Networks 1–4)

Switching between modes is determined by the value of the integer variable `iControlMode`, based on which the program automatically selects the appropriate setpoint and control branch:

- **Network 1 (`iControlMode = 0`):** The `EQ` block compares the value of `iControlMode` with 0. If they match, the `MOVE` block copies the scaled potentiometer value (`rScaledPot`) to the active setpoint (`rSetpoint_SP`).
- **Network 2 (`iControlMode = 1`):** When the value of `iControlMode` is 1 (HMI Manual), the normally closed (N.C.) logic switches off the `bAutoMode` enable bit (`FALSE`), thereby disabling the automatic PID controllers.
- **Network 3 (`iControlMode = 2`):** When the condition `iControlMode = 2` is met, the `MOVE` block writes the digital value entered on the HMI (`rHMISetPOint`) to the active setpoint (`rSetpoint_SP`).
- **Network 4 (`iControlMode = 3`):** When `iControlMode = 3` is set, the output coil activates the `bEnableFFFB` bit (`TRUE`), which enables the combined Feedback + Feedforward control loop.

### Custom PID Function Block (`FB_PID_MAN`)

In modes 1 (Hand PID) and 3 (HMI PID), instead of a library element, I used a self-developed, discrete-time PID function block (`FB_PID_MAN`) written in Structured Text (ST). The block includes output limiting and integral windup protection (Clamping Anti-Windup).

#### ST Source Code of the Function Block

```pascal
FUNCTION_BLOCK FB_PID_MAN
VAR_INPUT
    PV_k   : REAL; // Measured process value (Level PV)
    SP_k   : REAL; // Setpoint
    Kp     : REAL; // Proportional gain
    Ti     : REAL; // Integral time [s]
    Td     : REAL; // Derivative time [s]
    Ts     : REAL; // Sampling time [s]
    u_min  : REAL; // Output lower limit (-10.0V / 0.0V)
    u_max  : REAL; // Output upper limit (10.0V)
END_VAR
VAR
    bFirstScan : BOOL := TRUE;
    e_k        : REAL;
    P_k        : REAL;
    I_kTemp    : REAL;
    I_k_1      : REAL;
    Dk         : REAL;
    PV_k_1     : REAL;
    u_raw      : REAL;
END_VAR
VAR_OUTPUT
    u_k        : REAL; // Limited control signal
END_VAR

// 1. Error calculation and Proportional (P) term
e_k := SP_k - PV_k;
P_k := Kp * e_k;

// First-scan initialization (avoiding derivative spike at startup)
IF bFirstScan THEN
    PV_k_1 := PV_k;
    bFirstScan := FALSE;
END_IF;

// 2. Integral (I) term calculation (Discrete Euler method)
IF Ti > 0.0 THEN
    I_kTemp := I_k_1 + ((Kp / Ti) * e_k * Ts);
ELSE
    I_kTemp := 0.0;
END_IF;

// 3. Derivative (D) term calculation based on the measured value (PV)
IF Td > 0.0 AND Ts > 0.0 THEN
    Dk := -Kp * Td * (PV_k - PV_k_1) / Ts;
ELSE
    Dk := 0.0;
END_IF;

// 4. Summation and output saturation / Anti-Windup handling
u_raw := P_k + I_kTemp + Dk;

IF u_raw >= u_max THEN
    u_k := u_max;
ELSIF u_raw <= u_min THEN
    u_k := u_min;
ELSE
    u_k := u_raw;
    I_k_1 := I_kTemp; // Update integral only when the output is not saturated
END_IF;

PV_k_1 := PV_k;
```

#### Main Properties of the Algorithm

- **Derivative on PV:** The derivative term is calculated not from the error signal ($e_k$) but from the change of the measured value ($(PV_k - PV_{k-1})$). This prevents the so-called derivative kick that occurs when the setpoint changes suddenly.
- **Conditional integration (Clamping Anti-Windup):** If the raw output (`u_raw`) exceeds the configured $[u_{min}, u_{max}]$ limits, the internal state of the integral (`I_k_1`) is not updated. This prevents integral windup when the actuator saturates, and the significant overshoot resulting from it.

---

### Detailed Implementation of the Modes

#### 1. Hand PID Mode (`iControlMode = 0`)
<img width="904" height="747" alt="{703A11B6-3EC2-4F42-AA3D-C69E5754C7AC}" src="https://github.com/user-attachments/assets/d735c619-fc21-49e1-98ca-dcf60d6cf137" />
<img width="1183" height="801" alt="{5BF4796C-DA99-4C39-A239-573CF8D13D15}" src="https://github.com/user-attachments/assets/9990edd0-e625-4045-9809-e8ab561a36c6" />
<img width="481" height="373" alt="{82ABEC6C-FF17-41FE-9271-5E0D4FF72B8F}" src="https://github.com/user-attachments/assets/d0113354-bdac-4363-a1c4-b30bbc2f3117" />


- **Operating principle:** The water level setpoint is provided by the physical potentiometer. The raw analog input (`rPot_Raw`) is converted by the `FC_Scale` block to the $0\text{--}300\text{ cm}$ physical range (`rScaledPot`), which is copied into the `rSetpoint_SP` variable in Network 1.
- **Control logic (Networks 5, 6, 7):**
  * The `bAutoMode = TRUE` signal enables the operation of the custom `FB_PID_MAN_1` controller block.
  * Based on the controller's $u_k$ output (`rPID_Out`), the program decides how to move the valves:
    * **If `rPID_Out > 0`:** The fill valve receives an open command (`rFillValveCmd = rPID_Out`), and the discharge valve remains closed (`rDischargeValveCmd = 0`).
    * **If `rPID_Out < 0`:** The discharge valve opens after the sign inversion (`rDischargeValveCmd = rPID_Out * -1`), and the fill valve closes (`rFillValveCmd = 0`).
    * **If the error signal is within the deadband ($|rError| < rDeadband$):** Both valve commands are set to 0 to avoid unnecessary valve actuation.

#### Analog Signal Interfacing and Scaling (`FC_Scale`)

The conversion of the raw $0\text{--}10\text{ V}$ analog input signals — such as the tank water level (`rLevel_Raw`), the physical potentiometer (`rPot_Raw`), and the discharge-side flow meter (`rFlowMeterRaw`) — into standard physical units ($0\text{--}300\text{ cm}$, flow rate) is performed by the custom-written `FC_Scale` function block using linear interpolation:

$$\text{rScaledOut} = \frac{\text{rIn} - \text{rInMin}}{\text{rInMax} - \text{rInMin}} \cdot (\text{rOutMax} - \text{rOutMin}) + \text{rOutMin}$$

##### Block Interface Declaration

```pascal
FUNCTION FC_Scale : REAL
VAR_INPUT
    rIn      : REAL; // Raw analog input (e.g. rPot_Raw: 0.0 - 10.0 V)
    rInMin   : REAL; // Input scale minimum (0.0 V)
    rInMax   : REAL; // Input scale maximum (10.0 V)
    rOutMin  : REAL; // Physical range minimum (0.0 cm)
    rOutMax  : REAL; // Physical range maximum (300.0 cm)
END_VAR
VAR
    rInDiff       : REAL;
    rInSpan       : REAL;
    rOutSpan      : REAL;
    rRatio        : REAL;
    rScaledOffset : REAL;
END_VAR
VAR_OUTPUT
    rScaledOut    : REAL; // Scaled physical value (rScaledPot)
END_VAR
```
<img width="707" height="613" alt="{5D6B2EB1-0D85-46C6-8CB6-936CC67DFDBD}" src="https://github.com/user-attachments/assets/96414f7e-cbfc-4332-ab0a-0a2a293e104e" />
