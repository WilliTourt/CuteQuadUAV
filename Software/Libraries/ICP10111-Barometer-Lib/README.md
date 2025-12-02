
# ICP10111 - STM32 HAL C++ Driver

> [点击此处跳转到中文文档](README-zh-CN.md)

This is a lightweight C++ driver for the ICP10111 pressure and temperature sensor, intended for use with STM32Cube HAL.

## Features

- Initialization (ID check), soft reset and OTP calibration read
- Blocking measurement helper and data conversion utilities
- Optional CRC8 validation
- Helpers to get pressure (Pa), temperature (°C) and altitude (m)

## Requirements

- STM32Cube HAL (enable `HAL_I2C_MODULE_ENABLED` and open an I2C peripheral in CubeMX)
- Add `icp10111.cpp` and `icp10111.h` to your project

## Wiring

- Connect SDA -> MCU SDA (I2C)
- Connect SCL -> MCU SCL (I2C)
- Connect ground and VCC according to your sensor module / datasheet (commonly 3.3V)
- Add pull-ups on SDA/SCL if required

## Quick Example (STM32 / HAL)

The following example demonstrates initialization, measurement, data conversion and altitude calculation. It assumes `hi2c1` is configured and `huart1` is used for debug output.

```cpp
#include <cstdio>
#include "icp10111.h"

char dbg_buf[128];

uint16_t raw_temp = 0;
uint32_t raw_pressure = 0;
float pressure_Pa = 0.0f;
float temp_C = 0.0f;
float height_m = 0.0f;
uint16_t elapsed_ms = 0;
ICP10111::ICP10111_Status status;

// Use an appropriate measurement mode
ICP10111 icp10111(&hi2c1, ICP10111::ICP10111_MeasurementMode::ULTRA_LOW_NOISE);

int cpp_main() {
    sprintf(dbg_buf, "ICP10111 Init in 3 seconds...\n");
    HAL_UART_Transmit(&huart1, (uint8_t*)dbg_buf, strlen(dbg_buf), 100);
    HAL_Delay(3000);

    status = icp10111.begin();
    if (status == ICP10111::ICP10111_Status::OK) {
        sprintf(dbg_buf, "ICP10111 Init Complete.\n");
        HAL_UART_Transmit(&huart1, (uint8_t*)dbg_buf, strlen(dbg_buf), 100);
    } else {
        switch (status) {
            case ICP10111::ICP10111_Status::ERROR:  sprintf(dbg_buf, "ICP10111 Init Failed. Status: ERROR\n"); break;
            case ICP10111::ICP10111_Status::ERROR_ID: sprintf(dbg_buf, "ICP10111 Init Failed. Status: ERROR_ID\n"); break;
            case ICP10111::ICP10111_Status::ERROR_I2C: sprintf(dbg_buf, "ICP10111 Init Failed. Status: ERROR_I2C\n"); break;
            case ICP10111::ICP10111_Status::ERROR_CRC: sprintf(dbg_buf, "ICP10111 Init Failed. Status: ERROR_CRC\n"); break;
            default: sprintf(dbg_buf, "ICP10111 Init Failed.\n"); break;
        }
        HAL_UART_Transmit(&huart1, (uint8_t*)dbg_buf, strlen(dbg_buf), 100);
        while (1);
    }

    while (1) {
        status = icp10111.measure();
        if (status == ICP10111::ICP10111_Status::OK) {
            sprintf(dbg_buf, "Start Measure.\n");
            HAL_UART_Transmit(&huart1, (uint8_t*)dbg_buf, strlen(dbg_buf), 100);
        } else if (status == ICP10111::ICP10111_Status::BUSY) {
            HAL_Delay(1);
            elapsed_ms += 1;
        } else if (status == ICP10111::ICP10111_Status::DRDY) {
            raw_pressure = icp10111.getRawPressure();
            raw_temp = icp10111.getRawTemperature();
            icp10111.convertData();
            pressure_Pa = icp10111.getPressure();
            temp_C = icp10111.getTemperature();

            sprintf(dbg_buf, "elapsed_ms: %d. Pressure: %.2f Pa, Temperature: %.2f C\n", elapsed_ms, pressure_Pa, temp_C);
            HAL_UART_Transmit(&huart1, (uint8_t*)dbg_buf, strlen(dbg_buf), 100);

            height_m = icp10111.getAltitudeWithTemp();
            sprintf(dbg_buf, "Height: %.2f m\n", height_m);
            HAL_UART_Transmit(&huart1, (uint8_t*)dbg_buf, strlen(dbg_buf), 100);
            elapsed_ms = 0;
        } else {
            sprintf(dbg_buf, "ICP10111 Measure Failed.\n");
            HAL_UART_Transmit(&huart1, (uint8_t*)dbg_buf, strlen(dbg_buf), 100);
            while (1);
        }
    }
}
```

## API Summary

- `ICP10111(...)` : Constructor
- `begin()` : Initialize (ID check, reset, read OTP)
- `reset()` : Soft reset
- `measure()` : Blocking measurement start and wait
- `convertData()` : Convert raw values using OTP calibration
- `getRawPressure()` / `getRawTemperature()` : Read raw sensor values
- `getPressure()` / `getTemperature()` / `getAltitude()` / `getAltitudeWithTemp()` : Read converted values (call `convertData()` first)

See `icp10111.h` for full API details.

## References

- ICP10111 datasheet (refer to the datasheet for electrical specs and operating conditions)

---

WilliTourt <willitourt@foxmail.com>. Contributions and issues are welcomed :)

