# HC-SR04 Sensor driver in micropython

Micropython driver for the well-known untrasonic sensor [HC-SR04](https://www.mpja.com/download/hc-sr04_ultrasonic_module_user_guidejohn.pdf)

The driver has been tested on ***Wemos D1 mini PRO*** and ***ESP8266 12e***, but It should work on whatever other micropython board, 
if anyone find problems in other boards, please open an issue and we'll see.

## Motivation

Using the more accurate method `machine.time_pulse_us()` that is compliant with "standard" micropython, there is no code for specific boards.

The `distance_mm()` don't use floating point operations, for environments where there is no
floating point capabilities.

## Breaking changes:

This fork is an extension of the original driver which now take into account the room temperature, check the author's work. The new constructor is:

```python
from hcsr04 import HCSR04

sensor = HCSR04(trigger_pin=16, echo_pin=0, echo_timeout_us=500*2*30, air_temp=20)
```

This is a patch for micropython version 1.9.x and 1.10 for the method `machine.time_pulse_us()` which doesn't raise an OSError with ETIMEDOUT anymore, but it has a few [more](http://docs.micropython.org/en/latest/library/machine.html?highlight=machine%20time_pulse_us) return conditions.
In this patch no timeouts exceptions are raised, a mathematical **inf** approach is preferred, so error management is left to the user.



## Examples of use:

### How to get the distance

The `distance_cm()` method returns a `float` with the distance measured by the sensor.

```python
from hcsr04 import HCSR04

sensor = HCSR04(trigger_pin=16, echo_pin=0)

distance = sensor.distance_cm()

print('Distance:', distance, 'cm')
```

There is another method, `distance_mm()`, that returns the distance in milimeters (`int` type) and **no floating point is used**, designed 
for environments that doesn't support floating point operations.

```python
distance = sensor.distance_mm()

print('Distance:', distance, 'mm')
```
The default timeout is based on the sensor limit (4m), but we can also define a different timeout, 
passing the new value in microseconds to the constructor.

```python
from hcsr04 import HCSR04

sensor = HCSR04(trigger_pin=16, echo_pin=0, echo_timeout_us=1000000)

distance = sensor.distance_cm()

print('Distance:', distance, 'cm')
```

### Error management

When the driver reaches the timeout while is listening the echo pin, a **inf** value is returned. The error management is handled by the user, for example: 

```python
from hcsr04 import HCSR04
from math import isinf

sensor = HCSR04(trigger_pin=16, echo_pin=0)

distance = sensor.distance_cm()
if isinf(distance):
	print("Too far...")
else:
	print('Distance:', distance, 'cm')

```