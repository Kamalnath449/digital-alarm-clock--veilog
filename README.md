Digital Alarm Clock (Verilog HDL) – Code Explanation

Overview

This project implements a 24-hour Digital Alarm Clock using Verilog HDL. The clock can:

- Display Hours, Minutes, and Seconds
- Load a custom time
- Set an alarm time
- Enable or disable the alarm
- Stop the alarm manually
- Generate a 1-second clock from a faster system clock

---

Module Declaration

module Aclock(

Defines the top-level module named "Aclock".

Inputs

input reset,
input clk,

- "reset" : Resets the clock and alarm settings.
- "clk" : Main system clock.

input [1:0] H_in1,
input [3:0] H_in0,
input [3:0] M_in1,
input [3:0] M_in0,

These inputs are used to load the current time or alarm time.

Example:

- 23 hours → "H_in1 = 2", "H_in0 = 3"
- 45 minutes → "M_in1 = 4", "M_in0 = 5"

input LD_time,
input LD_alarm,
input STOP_al,
input AL_ON,

Control signals:

- "LD_time" : Load current time.
- "LD_alarm" : Load alarm time.
- "STOP_al" : Stop active alarm.
- "AL_ON" : Enable alarm functionality.

Outputs

output reg Alarm,

Alarm status output.

output [1:0] H_out1,
output [3:0] H_out0,
output [3:0] M_out1,
output [3:0] M_out0,
output [3:0] S_out1,
output [3:0] S_out0

Display outputs:

Signal| Description
H_out1| Hour tens digit
H_out0| Hour units digit
M_out1| Minute tens digit
M_out0| Minute units digit
S_out1| Second tens digit
S_out0| Second units digit

---

Internal Registers

reg clk_1s;

Generated 1-second clock.

reg [3:0] tmp_1s;

Counter used to create the slower clock.

reg [5:0] tmp_hour, tmp_minute, tmp_second;

Stores current time values.

Time Range

- Hours: 0–23
- Minutes: 0–59
- Seconds: 0–59

---

Alarm Registers

reg [1:0] c_hour1,a_hour1;
reg [3:0] c_hour0,a_hour0;
reg [3:0] c_min1,a_min1;
reg [3:0] c_min0,a_min0;
reg [3:0] c_sec1,a_sec1;
reg [3:0] c_sec0,a_sec0;

Where:

- "c_*" = Current clock digits
- "a_*" = Alarm digits

---

Function: mod_10

function [3:0] mod_10;
 input [5:0] number;

Returns the tens digit of a number between 0 and 59.

Examples:

Input| Output
58| 5
43| 4
29| 2
15| 1
07| 0

This function is used during binary-to-BCD conversion.

---

Main Timekeeping Block

always @(posedge clk_1s or posedge reset)

Executes every second.

Reset Operation

When "reset" is asserted:

- Alarm time is cleared.
- Clock time is loaded from inputs.
- Seconds are reset to zero.

tmp_hour <= H_in1*10 + H_in0;
tmp_minute <= M_in1*10 + M_in0;
tmp_second <= 0;

Example:

Input = 23:45

Stored values:

tmp_hour   = 23
tmp_minute = 45
tmp_second = 0

---

Alarm Loading

if(LD_alarm)

Stores the alarm time.

a_hour1 <= H_in1;
a_hour0 <= H_in0;
a_min1 <= M_in1;
a_min0 <= M_in0;

Example:

Alarm = 07:30

Stored as:

a_hour1 = 0
a_hour0 = 7
a_min1  = 3
a_min0  = 0

---

Time Loading

if(LD_time)

Loads a new current time.

tmp_hour <= H_in1*10 + H_in0;
tmp_minute <= M_in1*10 + M_in0;
tmp_second <= 0;

---

Clock Counting Logic

tmp_second <= tmp_second + 1;

Increments seconds every clock tick.

Second Overflow

if(tmp_second >= 59)

Actions:

tmp_second <= 0;
tmp_minute <= tmp_minute + 1;

---

Minute Overflow

if(tmp_minute >= 59)

Actions:

tmp_minute <= 0;
tmp_hour <= tmp_hour + 1;

---

Hour Overflow

if(tmp_hour >= 24)

Actions:

tmp_hour <= 0;

Clock rolls over from:

23:59:59
   ↓
00:00:00

---

Clock Divider

always @(posedge clk or posedge reset)

Converts the fast system clock into a slower clock.

tmp_1s <= tmp_1s + 1;

Counter increments every system clock cycle.

if(tmp_1s <= 5)

Output remains LOW.

else if(tmp_1s >= 10)

Output becomes HIGH and counter restarts.

Result:

System Clock
      ↓
Clock Divider
      ↓
clk_1s

For simulation purposes, this behaves as a slower clock.

---

Binary-to-BCD Conversion

always @(*)

Continuously converts internal binary values into display digits.

Hour Tens Digit

if(tmp_hour >= 20)
    c_hour1 = 2;
else if(tmp_hour >= 10)
    c_hour1 = 1;
else
    c_hour1 = 0;

Examples:

Hour| c_hour1
23| 2
15| 1
08| 0

---

Hour Units Digit

c_hour0 = tmp_hour - c_hour1*10;

Example:

23 → 23 - 20 = 3

---

Minute Conversion

c_min1 = mod_10(tmp_minute);
c_min0 = tmp_minute - c_min1*10;

Example:

45 → 4 and 5

---

Second Conversion

c_sec1 = mod_10(tmp_second);
c_sec0 = tmp_second - c_sec1*10;

Example:

58 → 5 and 8

---

Alarm Comparator

always @(posedge clk_1s or posedge reset)

Checks the alarm condition every second.

Alarm Match

if({a_hour1,a_hour0,a_min1,a_min0}
==
{c_hour1,c_hour0,c_min1,c_min0})

Compares:

Alarm Time
     ==
Current Time

Example:

Alarm   = 07:30
Current = 07:30

Match detected.

---

Alarm Enable

if(AL_ON)
    Alarm <= 1;

Alarm activates only when enabled.

---

Alarm Stop

if(STOP_al)
    Alarm <= 0;

Stops the alarm manually.

---

Output Assignments

assign H_out1 = c_hour1;
assign H_out0 = c_hour0;
assign M_out1 = c_min1;
assign M_out0 = c_min0;
assign S_out1 = c_sec1;
assign S_out0 = c_sec0;

Connects internal BCD values to external display outputs.

---

Working Flow

System Clock
      │
      ▼
Clock Divider
      │
      ▼
1-Second Clock
      │
      ▼
Time Counter
(HH:MM:SS)
      │
      ▼
BCD Converter
      │
      ▼
Display Outputs

Alarm Time
      │
      ▼
Comparator
      │
      ▼
Alarm Output

Conclusion

This Verilog design demonstrates several important digital design concepts:

- Clock Division
- Sequential Logic
- Time Counters
- Binary-to-BCD Conversion
- Alarm Time Storage
- Comparator Logic
- Alarm Control System

It is an excellent beginner-to-intermediate FPGA/Verilog project for learning digital system design and verification.
