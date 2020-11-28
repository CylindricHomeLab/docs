---
title: Marlin Notes
taxonomy:
    category:
        - printing
    tag:
        - printing
        - marlin
---

## Report Settings

```gcode
M503
```

## Config Backup

### Before

This is the `M503` pre-extruder replacement config:

```text
  G21    ; Units in mm (mm)
Filament settings: Disabled
  M200 D1.75
  M200 D0
Steps per unit:
 M92 X80.00 Y80.00 Z400.00 E389.20
Maximum feedrates (units/s):
  M203 X200.00 Y200.00 Z8.00 E60.00
Maximum Acceleration (units/s2):
  M201 X1000.00 Y1000.00 Z200.00 E5000.00
Acceleration (units/s2): P<print_accel> R<retract_accel> T<travel_accel>
  M204 P1250.00 R1250.00 T1250.00
Advanced: B<min_segment_time_us> S<min_feedrate> T<min_travel_feedrate> J<junc_dev>
  M205 B20000.00 S0.00 T0.00 J0.01
Home offset:
  M206 X0.00 Y0.00 Z0.00
Mesh Bed Leveling:
  M420 S0 Z0.00
  G29 S3 X1 Y1 Z0.16000
  G29 S3 X2 Y1 Z0.14000
  G29 S3 X3 Y1 Z0.10000
  G29 S3 X4 Y1 Z0.08000
  G29 S3 X5 Y1 Z0.04000
  G29 S3 X1 Y2 Z0.12000
  G29 S3 X2 Y2 Z0.10000
  G29 S3 X3 Y2 Z0.12000
  G29 S3 X4 Y2 Z0.08000
  G29 S3 X5 Y2 Z0.06000
  G29 S3 X1 Y3 Z0.04000
  G29 S3 X2 Y3 Z0.06000
  G29 S3 X3 Y3 Z0.08000
  G29 S3 X4 Y3 Z0.04000
  G29 S3 X5 Y3 Z0.04000
  G29 S3 X1 Y4 Z0.02000
  G29 S3 X2 Y4 Z0.06000
  G29 S3 X3 Y4 Z0.08000
  G29 S3 X4 Y4 Z0.06000
  G29 S3 X5 Y4 Z0.08000
  G29 S3 X1 Y5 Z0.06000
  G29 S3 X2 Y5 Z0.12000
  G29 S3 X3 Y5 Z0.10000
  G29 S3 X4 Y5 Z0.14000
  G29 S3 X5 Y5 Z0.14000
Endstop adjustment:
  M666 Z0.00
PID settings:
  M301 P15.05 I0.91 D62.57
  M304 P165.37 I31.54 D216.76
Linear Advance:
  M900 K0.00
Filament load/unload lengths:
  M603 L540.00 U490.00
```

### Upgrade

1. `M502` to factory reset
1. `M500` to save that to EEPROM
1. `M503` to see current config
1. reapply any settings you want to keep
1. `M500` to save new config

```text
Send: M503
  G21    ; Units in mm (mm)
Recv: 
; Filament settings: Disabled
  M200 D1.75
  M200 D0
; Steps per unit:
 M92 X80.00 Y80.00 Z400.00 E96.12
; Maximum feedrates (units/s):
  M203 X500.00 Y500.00 Z6.00 E60.00
; Maximum Acceleration (units/s2):
  M201 X3000.00 Y2000.00 Z60.00 E10000.00
; Acceleration (units/s2): P<print_accel> R<retract_accel> T<travel_accel>
  M204 P1500.00 R3000.00 T3000.00
; Advanced: B<min_segment_time_us> S<min_feedrate> T<min_travel_feedrate> J<junc_dev>
  M205 B20000.00 S0.00 T0.00 J0.01
; Home offset:
  M206 X0.00 Y0.00 Z0.00
; Mesh Bed Leveling:
  M420 S0 Z0.00
; Endstop adjustment:
  M666 Z0.00
; PID settings:
  M301 P15.94 I1.17 D54.19
  M304 P165.37 I31.54 D216.76
; Linear Advance:
  M900 K0.00
; Filament load/unload lengths:
  M603 L530.00 U500.00
Recv: ok
```
