# PID Controller Library

> [点击此处跳转到中文文档](README-zh-CN.md)

A comprehensive C++ PID controller library with multiple algorithm variants and advanced features.

## Features

- **Multiple PID Modes**: Support for standard, measurement-based, weighted PID algorithms, and deadzone handling, output limiting, and integral clamping, etc.
- **Flexible Configuration**: Configurable proportional, integral, and derivative modes
- **Easy to Use**: Simple API with comprehensive documentation

## Quick Start

#### Include the header file in your project

```cpp
#include "pid.h"
```

### Basic Usage

```cpp
// Create a PID controller with gains
PIDCtrller pid(1.0f, 0.1f, 0.05f);

// Set the target value
pid.setTarget(100.0f);

// Calculate output with measurement and limits
float output = pid.calc(current_measurement, 255.0f, 0.0f);
```

## API Documentation

### Constructor

```cpp
PIDCtrller(float kp, float ki, float kd)
```

Creates a new PID controller with the specified gains.

- `kp`: Proportional gain
- `ki`: Integral gain  
- `kd`: Derivative gain

### Core Methods

#### setParams(float kp, float ki, float kd)
Update PID gains during runtime.

#### setTarget(float target)
Set the desired target value.

#### setSampleTime(float sample_time)
Set the time between calculations in seconds.

#### calc(float curr_measurement, float upperLimit, float lowerLimit)
Calculate the PID output based on current measurement.

- `curr_measurement`: Current measured value
- `upperLimit`: Maximum output limit
- `lowerLimit`: Minimum output limit
- Returns: Calculated output (clamped between limits)

#### reset()
Reset controller state (clears integrals and previous values).

### Mode Configuration

#### Proportional Modes

```cpp
// Standard proportional: P = Kp * error
pid.setProportionalMode(PIDCtrller::ProportionalMode_t::Standard);

// Proportional on measurement: P = -Kp * measurement  
pid.setProportionalMode(PIDCtrller::ProportionalMode_t::OnMeasurement);

// Weighted proportional: P = Kp*(b*target - measurement)
pid.setProportionalMode(PIDCtrller::ProportionalMode_t::Weighted, 0.8f);
```

#### Integral Modes

```cpp
// Standard integral: I = Ki * ∫error dt
pid.setIntegralMode(PIDCtrller::IntegralMode_t::Standard);

// Clamped integral: I = Ki * min(∫error dt, limit)
pid.setIntegralMode(PIDCtrller::IntegralMode_t::Clamped, 50.0f);

// Conditional integral: I only accumulates when |error| < threshold
pid.setIntegralMode(PIDCtrller::IntegralMode_t::Conditional, 10.0f);
```

#### Derivative Modes

```cpp
// Standard derivative: D = Kd * d(error)/dt
pid.setDerivativeMode(PIDCtrller::DerivativeMode_t::Standard);

// Derivative on measurement: D = -Kd * d(measurement)/dt
pid.setDerivativeMode(PIDCtrller::DerivativeMode_t::OnMeasurement);

// Weighted derivative: D = Kd*(c*d(target)/dt - d(measurement)/dt)
pid.setDerivativeMode(PIDCtrller::DerivativeMode_t::Weighted, 0.5f);
```

#### Deadzone Configuration

```cpp
// Set deadzone - errors within this range are treated as zero
pid.setDeadzone(2.0f);
```

### Getters

```cpp
float lastOutput = pid.getLastOutput();  // Last calculated output
float error = pid.getError();           // Current error
float integral = pid.getIntegral();     // Current integral value
```

## Usage Examples

### Example 1: Basic Temperature Control

```cpp
#include "pid.h"

// Create PID controller for temperature control
PIDCtrller tempController(2.0f, 0.05f, 1.0f);
tempController.setTarget(75.0f);  // Target temperature: 75°C
tempController.setSampleTime(1.0f);  // Update every second

// In control loop
float currentTemp = readTemperatureSensor();
float heaterPower = tempController.calc(currentTemp, 100.0f, 0.0f);
setHeaterPower(heaterPower);
```

### Example 2: Motor Speed Control with Advanced Features

```cpp
#include "pid.h"

// Create PID controller for motor speed
PIDCtrller motorController(0.5f, 0.02f, 0.1f);

// Configure advanced features
motorController.setProportionalMode(PIDCtrller::ProportionalMode_t::Weighted, 0.9f);
motorController.setIntegralMode(PIDCtrller::IntegralMode_t::Clamped, 100.0f);
motorController.setDeadzone(5.0f);  // Ignore small speed errors
motorController.setTarget(1500.0f);  // Target RPM: 1500

// In control loop
float currentRPM = readEncoder();
float pwmDuty = motorController.calc(currentRPM, 1.0f, 0.0f);
setMotorPWM(pwmDuty);
```

### Example 3: Position Control with Conditional Integral

```cpp
#include "pid.h"

// Create PID controller for precise positioning
PIDCtrller positionController(1.2f, 0.01f, 0.2f);

// Use conditional integral to prevent windup during large errors
positionController.setIntegralMode(PIDCtrller::IntegralMode_t::Conditional, 5.0f);
positionController.setTarget(45.0f);  // Target position: 45 degrees

// In control loop  
float currentPos = readPotentiometer();
float motorVoltage = positionController.calc(currentPos, 12.0f, -12.0f);
setMotorVoltage(motorVoltage);
```

## Default Parameters

The library provides sensible defaults:

```cpp
#define PID_DEFAULT_SAMPLE_TIME 1.0f           // 1 second
#define PID_DEFAULT_PROP_WEIGHT 1.0f           // Full weight
#define PID_DEFAULT_CLAMP_INT_LIMIT 200.0f     // Integral limit
#define PID_DEFAULT_COND_INT_ERR_THRESHOLD 20.0f // Error threshold
#define PID_DEFAULT_DERIVATIVE_WEIGHT 1.0f     // Full weight
#define PID_DEFAULT_DEADZONE 0.0f              // No deadzone
```

## Algorithm Details

### Standard PID Formulation
```
u(t) = Kp*e(t) + Ki*∫e(t)dt + Kd*de(t)/dt
     = P + I + D
```

Where:
- `u(t)`: Output
- `e(t) = target - measurement`: Error
- `de(t)/dt = (e(t) - e(t-1))/dt`: Error derivative
- `∫e(t)dt`: Error integral
- `dt`: Sample time

### Alternative Formulations

See the header file for detailed mathematical formulations of all supported modes.

## Notes

- **Sample Time**: Must be positive. If <= 0, default value is used
- **Output Limits**: If `lowerLimit > upperLimit`, they are automatically swapped
- **Deadzone**: Absolute value is used, errors within deadzone are treated as zero
- **Thread Safety**: This implementation is not thread-safe

## Author

WilliTourt - willitourt@foxmail.com

*This library is provided as-is under GPLv3 license. Welcoming any feedback and contributions :)*