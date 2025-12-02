# ICP10111 - STM32 HAL库 C++ 驱动

> [Click here for English docs](README.md)

这是一个针对 ICP10111 压力与温度传感器的轻量级 C++ 驱动，面向 STM32Cube HAL。

## 特性

- 初始化（ID 校验）、软复位与 OTP 校准数据读取
- 阻塞测量辅助函数与数据换算工具
- 可选的 CRC8 校验
- 提供获取压力（Pa）、温度（°C）和海拔高度（m）的辅助方法

## 依赖

- STM32Cube HAL（启用 `HAL_I2C_MODULE_ENABLED`，即在CubeMX中打开一个 I2C 外设）
- 将 `icp10111.cpp` 与 `icp10111.h` 添加到工程

## 接线说明

- 连接 SDA -> MCU SDA（I2C）
- 连接 SCL -> MCU SCL（I2C）
- 电源与接地按传感器模块/数据手册连接（通常使用 3.3V，具体请参考你的模块说明）
- 检查是否已有外部上拉电阻，推荐添加

## 快速示例（STM32 / HAL）

此例程演示了 ICP10111 初始化、测量、数据转换、高度计算等流程。使用的HI2C1外设，并添加了一个USART1 用于输出调试信息。

```cpp
#include <cstdio>
#include "icp10111.h"

char dbg_buf[128];

uint16_t raw_temp = 0;
uint32_t raw_pressure = 0;
float pressure_hPa = 0.0f;
float temp = 0.0f;
float height = 0.0f;
uint16_t elapsed_ms = 0;
ICP10111::ICP10111_Status status;

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
			pressure_hPa = icp10111.getPressure();
			temp = icp10111.getTemperature();

			sprintf(dbg_buf, "elapsed_ms: %d. Pressure: %.2f Pa, Temperature: %.2f C\n", elapsed_ms, pressure_hPa, temp);
			HAL_UART_Transmit(&huart1, (uint8_t*)dbg_buf, strlen(dbg_buf), 100);

			height = icp10111.getAltitudeWithTemp();
			sprintf(dbg_buf, "Height: %.2f m\n", height);
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

## API 概览

- `ICP10111(...)` : 构造函数
- `begin()` : 初始化（ID 校验、复位、读取 OTP）
- `reset()` : 软复位
- `measure()` : 阻塞式测量启动并等待完成
- `convertData()` : 使用 OTP 校准数据将原始值转换为物理量
- `getRawPressure()` / `getRawTemperature()` : 读取原始值
- `getPressure()` / `getTemperature()` / `getAltitude()` / `getAltitudeWithTemp()` : 读取换算后的值，注意这四个函数均需要先调用 `convertData()`

详细接口请参考 `icp10111.h`。

## 参考

- ICP10111 数据手册（请参考传感器数据手册以获取电气规范和工作条件）

#### WilliTourt willitourt@foxmail.com. 欢迎PR和ISSUE建议 :)
