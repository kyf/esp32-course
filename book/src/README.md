# ESP32-S3 Rust 嵌入式开发实战

本书采用 **Rust + `esp-hal` + `#![no_std]`**，面向 ESP32-S3，先在 macOS 构建和模拟，再逐步迁移到真实开发板。每节课固定包含：目标 → 环境 → 原理 → 实现 → 构建 → 模拟/硬件验证 → 故障排查 → 验收标准 → 复盘。

- 课程：`book/src/`
- 对应示例：`source/`
- 第 01 课源码：[`source/gpio-blink-rust`](../../source/gpio-blink-rust/)
- 第 02 课草案：[`source/gpio-input-polling`](../../source/gpio-input-polling/)

> 状态说明：第 01 课已在 esp32sim 验证 GPIO4 波形；第 02 课是待构建、待仿真/硬件验证的教学草案，不能视为已经完成的实验。
