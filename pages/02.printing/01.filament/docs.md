---
title: Filament
taxonomy:
    category:
        - printing
    tag:
        - printing
        - anycubic
---

Print notes for filament

===

# Filament

Filament                 |
------------------------ |
Filaprint Apple Green
Filaprint Leaf Green
Filaprint Light Grey
Filaprint Red
Filaprint White
Filaprint Chocolatey Brown
Filaprint Bronze Gold
Clear t-glaze

# Stock FW

```
start
echo:V1.1.3
 1.1.0-RC8

echo: Last Updated: 2016-12-06 12:00 | Author: (Jolly, xxxxxxxx.CO.)
Compiled: Oct 24 2018
echo: Free Memory: 3109  PlannerBufferBytes: 1168
echo:V27 stored settings retrieved (398 bytes)
echo:Steps per unit:
echo:  M92 X80.00 Y80.00 Z400.00 E384.00
echo:Maximum feedrates (mm/s):
echo:  M203 X500.00 Y500.00 Z6.00 E60.00
echo:Maximum Acceleration (mm/s2):
echo:  M201 X3000 Y2000 Z60 E10000
echo:Accelerations: P=printing, R=retract and T=travel
echo:  M204 P3000.00 R3000.00 T3000.00
echo:Advanced variables: S=Min feedrate (mm/s), T=Min travel feedrate (mm/s), B=minimum segment time (ms), X=maximum XY jerk (mm/s),  Z=maximum Z jerk (mm/s),  E=maximum E jerk (mm/s)
echo:  M205 S0.00 T0.00 B20000 X10.00 Y10.00 Z0.40 E5.00
echo:Home offset (mm)
echo:  M206 X0.00 Y0.00 Z0.00
echo:Z2 Endstop adjustment (mm):
echo:  M666 Z0.00
echo:Material heatup parameters:
echo:  M145 S0 H180 B70 F0
  M145 S1 H240 B110 F0
echo:PID settings:
echo:  M301 P9.02 I0.31 D42.76
echo:Filament settings: Disabled
echo:  M200 D1.75
echo:  M200 D0
echo:SD card ok
echo:SD card ok
```

```
M503
echo:Steps per unit:
echo:  M92 X80.00 Y80.00 Z400.00 E384.00
echo:Maximum feedrates (mm/s):
echo:  M203 X500.00 Y500.00 Z6.00 E60.00
echo:Maximum Acceleration (mm/s2):
echo:  M201 X3000 Y2000 Z60 E10000
echo:Accelerations: P=printing, R=retract and T=travel
echo:  M204 P3000.00 R3000.00 T3000.00
echo:Advanced variables: S=Min feedrate (mm/s), T=Min travel feedrate (mm/s), B=minimum segment time (ms), X=maximum XY jerk (mm/s),  Z=maximum Z jerk (mm/s),  E=maximum E jerk (mm/s)
echo:  M205 S0.00 T0.00 B20000 X10.00 Y10.00 Z0.40 E5.00
echo:Home offset (mm)
echo:  M206 X0.00 Y0.00 Z0.00
echo:Z2 Endstop adjustment (mm):
echo:  M666 Z0.00
echo:Material heatup parameters:
echo:  M145 S0 H180 B70 F0
  M145 S1 H240 B110 F0
echo:PID settings:
echo:  M301 P9.02 I0.31 D42.76
echo:Filament settings: Disabled
echo:  M200 D1.75
echo:  M200 D0
ok
```

## Neew FW

```
M502
M500
M92 E384
M203 E30
M204 R1500.00
M500


M503
echo:  G21    ; (mm)

echo:Filament settings: Disabled
echo:  M200 D1.75
echo:  M200 D0
echo:Steps per unit:
echo:  M92 X80.00 Y80.00 Z400.00 E384.00
echo:Maximum feedrates (units/s):
echo:  M203 X500.00 Y500.00 Z6.00 E30.00
echo:Maximum Acceleration (units/s2):
echo:  M201 X3000 Y2000 Z60 E10000
echo:Acceleration (units/s2): P<print_accel> R<retract_accel> T<travel_accel>
echo:  M204 P1500.00 R1500.00 T3000.00
echo:Advanced: Q<min_segment_time_us> S<min_feedrate> T<min_travel_feedrate> X<max_x_jerk> Y<max_y_jerk> Z<max_z_jerk> E<max_e_jerk>
echo:  M205 Q20000 S0.00 T0.00 X10.00 Y10.00 Z0.40 E5.00
echo:Home offset:
echo:  M206 X0.00 Y0.00 Z0.00
echo:Mesh Bed Leveling:
echo:  M420 S0 Z0.00
echo:Endstop adjustment:
echo:  M666 Z0.00
echo:PID settings:
echo:  M301 P15.94 I1.17 D54.19
echo:  M304 P251.78 I49.57 D319.73
echo:Linear Advance:
echo:  M900 K0.00
echo:Filament load/unload lengths:
echo:  M603 L538.00 U555.00
ok
```


## Bed Vis
```
M155 S30 
G29 T
M155 S3
```


## Start GCode
```
M501
M420 S1
```

