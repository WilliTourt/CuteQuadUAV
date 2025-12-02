# PID 控制器库

> [Click here for English docs](README.md)

一个功能全面的 C++ PID 控制器库，支持多种算法变体和高级功能。

## 特性

- **多种 PID 模式**: 支持标准、基于测量、加权 PID 算法，以及死区处理、输出限制和积分钳位等功能
- **灵活配置**: 可配置的比例、积分和微分模式
- **易于使用**: 简单的 API 和全面的文档

## 快速开始

#### 在项目中包含头文件

```cpp
#include "pid.h"
```

### 基本用法

```cpp
// 创建带有增益的 PID 控制器
PIDCtrller pid(1.0f, 0.1f, 0.05f);

// 设置目标值
pid.setTarget(100.0f);

// 使用测量值和限制计算输出
float output = pid.calc(current_measurement, 255.0f, 0.0f);
```

## API 文档

### 构造函数

```cpp
PIDCtrller(float kp, float ki, float kd)
```

使用指定的增益创建新的 PID 控制器。

- `kp`: 比例增益
- `ki`: 积分增益  
- `kd`: 微分增益

### 核心方法

#### setParams(float kp, float ki, float kd)
在运行时更新 PID 增益。

#### setTarget(float target)
设置期望的目标值。

#### setSampleTime(float sample_time)
设置计算之间的时间间隔（秒）。

#### calc(float curr_measurement, float upperLimit, float lowerLimit)
基于当前测量值计算 PID 输出。

- `curr_measurement`: 当前测量值
- `upperLimit`: 最大输出限制
- `lowerLimit`: 最小输出限制
- 返回值: 计算得到的输出（在限制范围内钳位）

#### reset()
重置控制器状态（清除积分和先前值）。

### 模式配置

#### 比例模式

```cpp
// 标准比例: P = Kp * 误差
pid.setProportionalMode(PIDCtrller::ProportionalMode_t::Standard);

// 基于测量的比例: P = -Kp * 测量值  
pid.setProportionalMode(PIDCtrller::ProportionalMode_t::OnMeasurement);

// 加权比例: P = Kp*(b*目标值 - 测量值)
pid.setProportionalMode(PIDCtrller::ProportionalMode_t::Weighted, 0.8f);
```

#### 积分模式

```cpp
// 标准积分: I = Ki * ∫误差 dt
pid.setIntegralMode(PIDCtrller::IntegralMode_t::Standard);

// 钳位积分: I = Ki * min(∫误差 dt, 限制值)
pid.setIntegralMode(PIDCtrller::IntegralMode_t::Clamped, 50.0f);

// 条件积分: 仅当 |误差| < 阈值时累积积分
pid.setIntegralMode(PIDCtrller::IntegralMode_t::Conditional, 10.0f);
```

#### 微分模式

```cpp
// 标准微分: D = Kd * d(误差)/dt
pid.setDerivativeMode(PIDCtrller::DerivativeMode_t::Standard);

// 基于测量的微分: D = -Kd * d(测量值)/dt
pid.setDerivativeMode(PIDCtrller::DerivativeMode_t::OnMeasurement);

// 加权微分: D = Kd*(c*d(目标值)/dt - d(测量值)/dt)
pid.setDerivativeMode(PIDCtrller::DerivativeMode_t::Weighted, 0.5f);
```

#### 死区配置

```cpp
// 设置死区 - 在此范围内的误差被视为零
pid.setDeadzone(2.0f);
```

### 获取方法

```cpp
float lastOutput = pid.getLastOutput();  // 上次计算的输出
float error = pid.getError();           // 当前误差
float integral = pid.getIntegral();     // 当前积分值
```

## 使用示例

### 示例 1: 基本温度控制

```cpp
#include "pid.h"

// 创建用于温度控制的 PID 控制器
PIDCtrller tempController(2.0f, 0.05f, 1.0f);
tempController.setTarget(75.0f);  // 目标温度: 75°C
tempController.setSampleTime(1.0f);  // 每秒更新一次

// 在控制循环中
float currentTemp = readTemperatureSensor();
float heaterPower = tempController.calc(currentTemp, 100.0f, 0.0f);
setHeaterPower(heaterPower);
```

### 示例 2: 带高级功能的电机速度控制

```cpp
#include "pid.h"

// 创建用于电机速度控制的 PID 控制器
PIDCtrller motorController(0.5f, 0.02f, 0.1f);

// 配置高级功能
motorController.setProportionalMode(PIDCtrller::ProportionalMode_t::Weighted, 0.9f);
motorController.setIntegralMode(PIDCtrller::IntegralMode_t::Clamped, 100.0f);
motorController.setDeadzone(5.0f);  // 忽略小的速度误差
motorController.setTarget(1500.0f);  // 目标转速: 1500 RPM

// 在控制循环中
float currentRPM = readEncoder();
float pwmDuty = motorController.calc(currentRPM, 1.0f, 0.0f);
setMotorPWM(pwmDuty);
```

### 示例 3: 带条件积分的位置控制

```cpp
#include "pid.h"

// 创建用于精确定位的 PID 控制器
PIDCtrller positionController(1.2f, 0.01f, 0.2f);

// 使用条件积分防止大误差时的积分饱和
positionController.setIntegralMode(PIDCtrller::IntegralMode_t::Conditional, 5.0f);
positionController.setTarget(45.0f);  // 目标位置: 45 度

// 在控制循环中  
float currentPos = readPotentiometer();
float motorVoltage = positionController.calc(currentPos, 12.0f, -12.0f);
setMotorVoltage(motorVoltage);
```

## 默认参数

库提供合理的默认值：

```cpp
#define PID_DEFAULT_SAMPLE_TIME 1.0f           // 1 秒
#define PID_DEFAULT_PROP_WEIGHT 1.0f           // 全权重
#define PID_DEFAULT_CLAMP_INT_LIMIT 200.0f     // 积分限制
#define PID_DEFAULT_COND_INT_ERR_THRESHOLD 20.0f // 误差阈值
#define PID_DEFAULT_DERIVATIVE_WEIGHT 1.0f     // 全权重
#define PID_DEFAULT_DEADZONE 0.0f              // 无死区
```

## 算法细节

### 标准 PID 公式
```
u(t) = Kp*e(t) + Ki*∫e(t)dt + Kd*de(t)/dt
     = P + I + D
```

其中：
- `u(t)`: 输出
- `e(t) = 目标值 - 测量值`: 误差
- `de(t)/dt = (e(t) - e(t-1))/dt`: 误差导数
- `∫e(t)dt`: 误差积分
- `dt`: 采样时间

### 替代公式

有关所有支持模式的详细数学公式，请参阅头文件。

## 注意事项

- **采样时间**: 必须为正数。如果 <= 0，则使用默认值
- **输出限制**: 如果 `lowerLimit > upperLimit`，它们会自动交换
- **死区**: 使用绝对值，死区内的误差被视为零
- **线程安全**: 此实现不是线程安全的

## 作者

WilliTourt - willitourt@foxmail.com

*本库按 GPLv3 许可证提供，欢迎任何反馈和贡献 :)*
