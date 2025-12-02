# QMC5883P 磁强计 STM32 HAL C++ 驱动库

> [Click here for English docs](README.md)

这是一个面向 STM32（使用 STM32Cube HAL I2C）的轻量级 QMC5883P 三轴磁力计 C++ 驱动库。

## 功能
- 提供简洁的 C++ 类接口用于初始化、配置与读取磁力计数据
- 支持多种输出数据率（ODR）与测量量程
- 内置标定偏移与缩放系数（可在 `qmc5883p.h` 中调整）
- 依赖 STM32Cube HAL（需要启用 `HAL_I2C_MODULE_ENABLED`）

## 先决条件
- 已启用的 STM32Cube HAL I2C
- 一个初始化好的 `I2C_HandleTypeDef` 实例

## 接线（示意）
将 QMC5883P 的 I2C 引脚接到 MCU 的 I2C 外设：

- VCC -> 3.3V
- GND -> GND
- SCL -> MCU I2C SCL
- SDA -> MCU I2C SDA

驱动默认使用 I2C 地址 `0x2C`（详见 `qmc5883p.h`）。

## 快速上手
STM32 HAL 下的示例：

```cpp
// extern I2C_HandleTypeDef hi2c1; // 在工程中定义并初始化
#include "qmc5883p.h"

QMC5883P mag(&hi2c1, QMC5883P::QMC5883P_Mode::CONTINUOUS,
			  QMC5883P::QMC5883P_Spd::ODR_200HZ,
			  QMC5883P::QMC5883P_Rng::RNG_2G);

if (mag.begin() != QMC5883P::QMC5883P_Status::OK) {
	// 处理初始化错误
}

// 主循环中读取数据：
if (mag.update() == QMC5883P::QMC5883P_Status::OK) {
	float mx = mag.getX();
	float my = mag.getY();
	float mz = mag.getZ();
	// mx/my/mz 为已标定的磁场值（单位：高斯，Gauss）
}
```

备注：
- 上电后调用一次 `begin()` 完成复位与配置。
- 使用 `update()` 读取新数据；函数在超时前会等待数据就绪。

## 标定
默认的偏移与缩放系数在 `qmc5883p.h` 中以宏形式定义（例如 `MAG_X_OFFSET`、`MAG_X_SCALE` 等）。如需更高精度，请自行标定并替换这些宏值。

## 寄存器与常量
主要定义位于 `qmc5883p.h`：
- I2C 地址：`QMC5883P_ADDR`
- 芯片 ID：`QMC5883P_CHIP_ID`
- 数据寄存器：`QMC5883P_REG_XOUT_L`、`QMC5883P_REG_XOUT_H` 等
- 控制寄存器：模式、ODR、量程相关位域

## 故障排查
- 如果 `begin()` 返回 `ERROR_ID`，请检查接线并确认芯片 ID 是否匹配 `QMC5883P_CHIP_ID`。
- 确保 I2C 外设已正确初始化且总线没有冲突。

## 贡献
欢迎提交 issue、PR 与改进建议。

WilliTourt willitourt@foxmail.com