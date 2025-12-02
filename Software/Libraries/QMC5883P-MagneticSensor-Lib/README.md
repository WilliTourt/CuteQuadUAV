# QMC5883P Magnetic Sensor STM32 HAL C++ Driver

> [点击此处跳转到中文文档](README-zh-CN.md)

Lightweight C++ driver for the QMC5883P 3-axis magnetometer, designed for STM32 projects using the STM32Cube HAL I2C interface.

## Features
- Simple C++ class interface for initialization, configuration and reading magnetometer data
- Supports multiple output data rates and measurement ranges
- Built-in calibration offsets and scale factors (configurable in `qmc5883p.h`)
- Designed for STM32Cube HAL (`HAL_I2C_MODULE_ENABLED` required)

## Requirements
- STM32Cube HAL with I2C enabled
- A working `I2C_HandleTypeDef` passed into the driver

## Wiring
Connect the QMC5883P I2C pins to your MCU I2C peripheral. Typical connections:

- VCC -> 3.3V
- GND -> GND
- SCL -> MCU I2C SCL
- SDA -> MCU I2C SDA

The driver uses I2C address `0x2C` (see `qmc5883p.h`).

## Quick Usage
Example using STM32 HAL:

```cpp
// extern I2C_HandleTypeDef hi2c1; // your I2C handle
#include "qmc5883p.h"

QMC5883P mag(&hi2c1, QMC5883P::QMC5883P_Mode::CONTINUOUS,
			  QMC5883P::QMC5883P_Spd::ODR_200HZ,
			  QMC5883P::QMC5883P_Rng::RNG_2G);

if (mag.begin() != QMC5883P::QMC5883P_Status::OK) {
	// handle initialization error
}

// In your main loop:
if (mag.update() == QMC5883P::QMC5883P_Status::OK) {
	float mx = mag.getX();
	float my = mag.getY();
	float mz = mag.getZ();
	// mx/my/mz are calibrated magnetic field values (units: Gauss)
}
```

Notes:
- Call `begin()` once after power-up to reset and configure the sensor.
- Call `update()` to read the latest measurement; it will wait until data ready or timeout.

## Calibration
Default calibration offsets and scale factors are defined in `qmc5883p.h` as `MAG_X_OFFSET`, `MAG_X_SCALE`, etc. Adjust these values if you perform your own calibration for improved accuracy.

## Registers & Constants
Useful definitions (see `qmc5883p.h`):
- I2C address: `QMC5883P_ADDR`
- Chip ID: `QMC5883P_CHIP_ID`
- Data registers: `QMC5883P_REG_XOUT_L`, `QMC5883P_REG_XOUT_H`, ...
- Control registers and bitfields for mode, ODR and range

## Troubleshooting
- If `begin()` returns `ERROR_ID`, check wiring and confirm the chip ID read matches `QMC5883P_CHIP_ID`.
- Ensure the I2C peripheral is initialized and no bus conflicts exist.

## Contributing
Contributions, issue reports and PRs are welcome.

WilliTourt willitourt@foxmail.com