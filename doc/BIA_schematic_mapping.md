# AD5940_BIA electrode-name translation guide (EVAL-AD5940BIOZ)

This document explains **how the electrode labels on the schematic** (for example `F+`, `S-`) map to the **AD5940 pin/net names** (`CE0`, `AIN1`, `AIN2`, `AIN3`) and then to the **firmware symbols** used in `examples/AD5940_BIA/BodyImpedance.c`.

---

## 1) Physical lead set used on your setup

Per your note, the connected USB ECG lead colors are:

- **Red -> F+**
- **Blue -> S-**
- **Black -> F-**

So in this setup, **S+ is not connected**.

---

## 2) Translation dictionary: physical/electrode names -> code names

| Physical lead color | Human/electrode label | Net name seen in schematic text | AD5940 pin/net label | Name used in BIA code | Used with your lead set? |
|---|---|---|---|---|---|
| Red | `F+` | `USB_F+` | `CE0` | `SWD_CE0`, `SWP_CE0` | **Yes** |
| Black | `F-` | `USB_F-` | `AIN1` | `SWN_AIN1`, `SWT_AIN1` | **Yes** |
| Blue | `S-` | `USB_S-` | `AIN2` | `ADCMUXN_AIN2` | **Yes** |
| *(not connected)* | `S+` | `USB_S+` | `AIN3` | `ADCMUXP_AIN3` | **No (unused physically)** |

> Short version for your setup: **Force path is active (`CE0`/`AIN1`)**, but the **differential sense pair is incomplete** because `AIN3 (S+)` is not physically connected.

---

## 3) What the BIA example configures in firmware

In `AppBIASeqMeasureGen`, the first measurement section configures:

- `Dswitch = SWD_CE0`
- `Pswitch = SWP_CE0`
- `Nswitch = SWN_AIN1`
- `Tswitch = SWT_AIN1 | SWT_TRTIA`

Then it performs a second ADC capture using:

- `ADCMuxP = ADCMUXP_AIN3`
- `ADCMuxN = ADCMUXN_AIN2`

So the code schedules two capture contexts:

1. **HSTIA-based impedance capture (active in your setup)** via `CE0` + `AIN1`.
2. **Voltage differential capture across AIN3-AIN2 (configured, but only partially driven in your setup)** because `AIN2(S-)` is connected while `AIN3(S+)` is unconnected.

---

## 4) Which captures are actually happening with your 3 leads

### Active / meaningful captures

- **Capture #1 (HSTIA path): meaningful**
  - Uses `CE0` drive and `AIN1` return (`F+` red and `F-` black both connected).

### Configured in code but not fully valid with current wiring

- **Capture #2 (ADC diff AIN3-AIN2): not fully meaningful with current lead set**
  - `AIN2` (`S-`, blue) is connected.
  - `AIN3` (`S+`) is not connected.
  - Result: the firmware still performs this conversion, but it is not a proper `S+ - S-` differential sense measurement.

### Unused in this setup

- `S+` / `USB_S+` / `AIN3` is **physically unused**.
- `SE0` exists on the board but is not selected in this BIA switch path.

---

## 5) What was verified from schematic extraction

From `EVAL-AD5940BIOZ.pdf` text extraction, these labels are present:

- Electrode markers: `F-`, `S-`, `S+`, `F+`
- USB electrode nets: `USB_F+`, `USB_F-`, `USB_S+`, `USB_S-`
- AD5940 nets/pins: `CE0`, `SE0`, `RE0`, `DE0`, `AIN1`, `AIN2`, `AIN3`
- Breakout/test entries: `Instance I129 ... LOCATION AIN2`, `Instance I130 ... LOCATION AIN3`

This confirms `AIN2/AIN3` are routed nets available on the board design.
