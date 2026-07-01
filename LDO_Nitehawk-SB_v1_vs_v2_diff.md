# LDO Nitehawk-SB v1 vs v2 Diff Report

## MCU

| | v1.0 | v2.0 |
|---|---|---|
| Chip | RP2040 | STM32G0B1 |
| Pin notation | `gpio#` | STM32 port (`PA#`, `PB#`, etc.) |
| Serial ID format | `usb-1a86_USB2.0-Serial-if00-port0` | `usb-Klipper_stm32g0b1xx_XXXX-if00` |

---

## Pin Mapping Changes (`config/mcu_definitions/toolhead/`)

### Changed pins

| Logical alias | v1 (RP2040) | v2 (STM32G0B1) |
|---|---|---|
| `MCU_EMOT_EN` | gpio25 | PC14 |
| `MCU_EMOT_STEP` | gpio23 | PB8 |
| `MCU_EMOT_DIR` | gpio24 | PB9 |
| `MCU_EMOT_UART` | gpio0 | PB7 |
| `MCU_EMOT_TX` | gpio1 | PB6 |
| `MCU_ENDSTOP_X` | gpio13 | PB0 |
| `MCU_ENDSTOP_Y` | gpio12 | PB1 |
| `MCU_HV_ENDSTOP` | gpio10 | PC15 |
| `MCU_FAN0` | gpio5 | PD0 |
| `MCU_FAN1` | gpio6 | PA15 |
| `MCU_PWM0` | gpio16 | PD1 |
| `MCU_PWM1` | gpio17 | PD2 |
| `MCU_TEMP` | gpio29 | PB12 |
| `MCU_HEAT` | gpio9 | PA7 |
| `MCU_HEAT_CHAMBER` | gpio28 | PB2 |
| `MCU_RGB` | gpio7 | PD3 |
| `MCU_ACTIVITY_LED` | gpio8 | PC6 |
| `MCU_ADXL_CS` | gpio21 | PB10 |
| `MCU_ADXL_SCK` | gpio18 | PA5 |
| `MCU_ADXL_MOSI` | gpio20 | PA2 |
| `MCU_ADXL_MISO` | gpio19 | PA6 |

### Removed in v2

| Alias | v1 pin | Reason |
|---|---|---|
| `MCU_NH_TEMP` | gpio26 | PCB temperature sensor not present on v2 hardware |
| `MCU_FS` | gpio3 | Defined in v1 MCU definition but never aliased in the user template; not carried forward to v2 |
| `MCU_SU` | gpio2 | Same as above |
| `MCU_ST_UART_TX` | gpio1 | Duplicate alias for `MCU_EMOT_TX`; dropped in v2 |
| `MCU_ST_UART_RX` | gpio0 | Duplicate alias for `MCU_EMOT_UART`; dropped in v2 |

### Added in v2

| Alias | v2 pin | Purpose |
|---|---|---|
| `MCU_I2C_SCL` | PB3 | I2C bus (USB hub / eddy current probe support) |
| `MCU_I2C_SDA` | PB4 | I2C bus |

---

## User Template Changes (`user_templates/mcu_defaults/toolhead/`)

### Board alias differences

| Alias | v1 | v2 |
|---|---|---|
| `TX_PIN` | `MCU_ST_UART_TX` | `MCU_EMOT_TX` (same physical pin, different alias name) |
| `NH_MCU_TEMP` | present | **removed** (no hardware support) |

### Klipper config differences

| Section | v1 | v2 |
|---|---|---|
| `[fan]` tachometer | commented out | commented out |
| `[heater_fan]` tachometer | commented out | commented out |
| `[temperature_sensor nh_temp]` | present (CMFB103F3950FANT, pullup 2200) | **removed** |
| PCB temp monitoring | built-in via `NH_MCU_TEMP` | must use `config/hardware/temperature_sensors/toolhead_mcu_temp.cfg` instead |
| Chamber thermistor | no caveats | no caveats |

---

## Summary

v2 is a hardware revision from RP2040 to STM32G0B1. All logical pin aliases are preserved so existing klippain configs referencing `toolhead:E_STEP`, `toolhead:PART_FAN`, etc. are unaffected. The notable functional differences are:

- **PCB thermistor gone** — `nh_temp` sensor section must be removed or replaced.
- **I2C added** — enables eddy current probes (e.g. Cartographer) and USB hub without extra adapters.
- **Fan tachs on by default** — v2 hardware reliably supports them; v1 had them commented out.
- **`MCU_ST_UART_TX` / `MCU_ST_UART_RX` dropped** — v1 had duplicate serial aliases alongside the motor-driver names; v2 only keeps the motor-driver names (`MCU_EMOT_TX` / `MCU_EMOT_UART`).
