# 第 02 课：GPIO 输入、上拉与轮询

**状态：准备中，代码尚未在目标工具链及 esp32sim 实测。** 示例草案：[`source/gpio-input-polling`](../../source/gpio-input-polling/)。

## 目标与承接

延续第 01 课的 ESP32-S3、`no_std`、`esp-hal 1.2.x`、GPIO4 输出。新增 GPIO5 输入，学习数字输入、内部上拉、按键低电平有效、轮询与去抖。实验目标：按下 GPIO5 对应按钮时 GPIO4 为高，否则为低。

> 硬件接线：GPIO5 — 按钮 — GND；软件开启上拉。严禁把 GPIO 引脚直接连接到高于芯片允许的电压。开发板实际引脚复用/冲突需要查具体原理图。

## 准备工作

1. 将第 01 课工程复制为新工程，保留 `.cargo/config.toml`、`.cargo/esp-config.toml`、`rust-toolchain.toml`、`build.rs`、`Cargo.lock` 等配置。
2. 更改 `Cargo.toml` 包名和 bin 名称为 `gpio-input-polling`，将主程序替换为 `source/gpio-input-polling/src/main.rs` 草案；必要时按原工程的 `src/bin/main.rs` 路径放置。
3. 执行 `cargo +esp build --release`；若 API 与当前 lock 版本不匹配，优先查该版本 esp-hal 源码，不复制其他版本的 API。
4. 使用 `espflash save-image` 生成 BIN，使用 `esptool image-info` 核验。
5. 使用支持输入注入的模拟器功能或真实硬件验证。**尚未确认 `esp32sim --board none` 输入 GPIO5 的具体控制接口**，因此不能直接声称本课已通过仿真。

## 核心逻辑

读取 `Input::is_low()`，持续采样；检测到按下时，输出 GPIO4 高电平，否则低电平。初版不做去抖，以便观察机械按键抖动和逻辑行为。扩展任务：设置连续稳定 20ms 才确认按键状态。

## 验证标准

- [ ] 在 `esp-hal 1.2.x` 构建成功。
- [ ] 输入 GPIO5 释放时，GPIO4 低电平。
- [ ] 输入 GPIO5 接地时，GPIO4 高电平。
- [ ] 实测/模拟验证连续切换与去抖版本。
- [ ] 将运行日志、VCD 或硬件测试说明保存为验收证据。

## 易错点与排错

- 悬空输入：未配置上拉时输入值不可预测。
- 高低电平判断反向：按钮接地时为低电平，`is_low()` 才表示按下。
- 参数 API 变化：本项目以 lock 中的 esp-hal 版本为准。
- 把 GPIO 输出波形当输入成功：必须证明 GPIO5 输入变化确实驱动 GPIO4。
- 模拟器外设支持不全：如无法注入 GPIO5，应转用真实开发板，不要伪造波形。

下一节规划：第 03 课定时器与非阻塞状态机。
