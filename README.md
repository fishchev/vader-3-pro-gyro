Internet said that Flydigi reports only 2 gyroscope axis outside of Switch emulation mode (e.g. [reWASD manual says so](https://help.rewasd.com/how-to-remap/supported-devices.html#flydigi)), but I've found no substantial proof (_why would they make it just two?_) and decided to take a closer look. 
What I've found out is that at least with Vader 3 Pro data for all 6 axis of IMU is being reported.

- [Gyro Demo](https://fishchev.github.io/vader-3-pro-gyro/) -- it's simple, so only gyro is used / expect quick drift
- [HID input report analyzer](https://fishchev.github.io/vader-3-pro-gyro/hid-message-analyser-tool.html) - tool I've made to help me figure out message format, allows to select byte and plot it's value over time; could be used to measure real Gyro/Accel/etc refresh rate (not just message rate, but how ofter actual new values are reported); based on [webhid-explorer](https://github.com/nondebug/webhid-explorer). 

Update rate is 1000Hz when using dongle, 500Hz when wired (yes)\
Gyroscope updates at exactly 100Hz and accelerometer updates in ~50-53Hz range\
Accelerometer seems to self-calibrate to 1G corresponding to 4096 value whenever gamepad is left untouched for few seconds on hard surface; not sure whether it's my unit, but Y axis behaves strangely in comparasion.

Vader 3 Pro DInput HID report format
|bytes|format|range|comment|
|-----|------|--------------|-------|
| 0-1 | | | seems to be completely static
| 2 | | | first bit starts flipping at constant rate when aeromouse is enabled
| 3 | int8 | [-512, 512] | gyro yaw 8 least significant bits 
| 4 | | | first 4 bits: gyro pitch LSB, last 4 bits: gyro yaw MSB; combine with bytes 3 and 5 for 12-bit signed integer value
| 5 | int8 | [-512, 512] | gyro pitch 8 most significant bits 
| 6 | flags | | 00, M4, M3, M2, M1, Z, C
| 7 | flags | | 0000, HOME, 00, aeromouse/screenshot button
| 8 | flags | | X, SELECT, B, A, LEFT, DOWN, RIGHT, UP 
| 9 | flags | | RS, LS, RT, LT, RB, LB, START, Y
| 10-11 | int16le | [-32768, 32767] | accelerometer, X-axis, 4096 is 1G
| 12-13 | int16le | [-32768, 32767] | accelerometer, Y-axis, -4800 (?) is about 1G in normal oriantation, 3800 triggers facing floor
| 14-15 | int16le | [-32768, 32767] | accelerometer, Z-axis, 4096 is 1G
| 16 | int8 | | right stick X-axis
| 17 and 19 | int16le | [-512, 512] | gyro yaw (yeah.. not continuous bytes for some reason) 
| 18 | int8 | | right stick Y-axis
| 20 | int8 | | left stick X-axis
| 21 | int8 | | left stick Y-axis
| 22 | uint8 | | left trigger analog
| 23 | uint8 | | right trigger analog
| 24 | flags | | last two bits are flags for triggers in digital mode, reported only if triggers are in discrete/short press mode
| 25 | int16le | [-512, 512] | gyro pitch
| 27 | | | seems to be completely static
| 28 | int16le | [-32768, 32767] | gyro roll 