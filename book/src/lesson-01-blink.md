# 第 01 课：GPIO4 闪烁与模拟器调试

**状态：已通过 esp32sim 验证。** 源码：[`source/gpio-blink-rust`](../../source/gpio-blink-rust/)。

## 学习目标

1. 搭建 macOS 的 ESP32-S3 Xtensa Rust 工具链。
2. 理解 `#![no_std]`、`#![no_main]`、`esp_hal::init`、GPIO Output 与轮询延时。
3. 产出 ELF、espflash BIN；区分静态镜像检查与运行验证。
4. 使用 esp32sim 加载 ROM、模拟执行、输出 VCD，利用波形验证行为。
5. 掌握按“症状 → 证据 → 根因 → 最小修复 → 复验”排错。

## 源码与工作原理

该项目使用 `esp-hal ~1.2.2`，目标 `xtensa-esp32s3-none-elf`，设置最大 CPU 时钟，将 GPIO4 设为输出、初始低电平，循环调用 `led.toggle()`，再用 `Instant::now()` 忙等待 500 ms。

关键代码（节选；**以仓库源码为准**）：

```rust
let config = esp_hal::Config::default().with_cpu_clock(CpuClock::max());
let peripherals = esp_hal::init(config);
let mut led = Output::new(
    peripherals.GPIO4,
    Level::Low,
    OutputConfig::default(),
);
loop {
    led.toggle();
    let delay_start = Instant::now();
    while delay_start.elapsed() < Duration::from_millis(500) {}
}
```

忙等待适合第一课验证时钟与 GPIO，但会持续占用 CPU。第 03 课再研究非阻塞定时。

## 从源码到镜像

```bash
cd source/gpio-blink-rust
cargo +esp build --release
ELF=target/xtensa-esp32s3-none-elf/release/gpio-blink-rust
espflash save-image --chip esp32s3 "$ELF" "$ELF.bin"
esptool --chip esp32s3 image-info "$ELF.bin"
```

记录：ELF 入口 `0x40378620`，BIN 约 78 KB。镜像包含 DROM/DRAM/IRAM、零地址 PADDING 和 IROM 段。`image-info` 能检查镜像结构，不能证明模拟器兼容。

## 运行模拟器

需使用**已处理零地址 PADDING 的 esp32sim** 和已下载的 ESP32-S3 ROM。示例从模拟器仓库根目录运行：

```bash
COURSE=/absolute/path/to/esp32-course
APP="$COURSE/source/gpio-blink-rust/target/xtensa-esp32s3-none-elf/release/gpio-blink-rust"
./target/release/esp32sim \
  --chip s3 --board none --boot app \
  --app "$APP.bin" --elf "$APP" \
  --rom roms/esp32s3_rev0_rom.elf \
  --max-seconds 5 --vcd gpio.vcd
```

不要把 `--stop-after-exceptions 1` 用作最终验收：正常的 Xtensa Window Overflow 异常也可能触发该停止条件。排障时才用 `--trace` 和异常停止限制。

## 完整排障复盘

| 阶段 | 现象 | 证据与定位 | 处理 |
| --- | --- | --- | --- |
| 镜像加载 | `load: address 0x00000000 not mapped` | BIN 含 `load_addr=0` 的 PADDING 段；模拟器将其当 RAM 加载 | 在 `esp32s3/src/soc.rs` 的 `boot_app` 循环跳过该 padding；整理上游 PR |
| ROM 缺失 | `ill` 于 `0x400003c0` | 固件调用了 ROM 区域的函数，但未提供 ROM ELF | 下载校验 ROM 并用 `--rom` 指定 |
| 下载脚本 | macOS `mkdir: illegal option -- -` | 需要使用 `sh -x` 确认具体执行参数；不要简单推断是 BSD `mkdir` 本身不能运行 `mkdir -p` | 检查目录参数和脚本执行环境；最终成功生成 ROM |
| 首次 ROM 启动 | `cause 513`，`_WindowOverflow4` | 反汇编见 `s32e` 和 `rfwo`；属于 Xtensa 寄存器窗口处理路径 | 去掉 `--stop-after-exceptions 1`，继续执行 |
| 最终运行 | 模拟 5.000 秒，约 12 亿指令、3 次异常、VCD 10 个事件 | CPU 持续运行，波形显示 GPIO4 每 500 ms 翻转 | 验收通过 |

注意：此前对于 `mkdir` 报错的根因**未由跟踪日志最终证实**，不把推测写成确定事实。3 次异常不等于有 3 个未恢复故障，异常类型仍应结合 trace 分析。

## 用实际 VCD 验收

```text
$timescale 1ps $end
$var wire 1 s4 gpio4 $end
#0
xs4
#2377333333
1s4
#502377333333
0s4
#1002377333333
1s4
```

1ps 为时间刻度，因此高电平第一次从 2.377333333ms 到 502.377333333ms，持续 **500ms**；之后每隔 500ms 翻转。完整周期 1s，即 **1Hz**。初始为 `x`（未知电平）后开始输出高低交替。

`gpio.vcd` 已提交在第一课示例中，可以用 VS Code 的 VCD 扩展或其他波形软件检查；无需依赖失效的 GTKWave Homebrew Cask。

## 验收

- [x] 固件生成 ELF/BIN。
- [x] ROM 成功加载；固件从 `Reset` 进入执行。
- [x] esp32sim 执行至 `--max-seconds 5`。
- [x] VCD 包含 `gpio4` 且每 500ms 翻转。
- [ ] 真实 ESP32-S3 开发板验证（未完成）。

## 课后思考

为什么镜像的 PADDING 不应加载到地址 0？为什么不能见到一次 Xtensa Window Overflow 就判定程序崩溃？为什么忙等待在下一阶段需要替换？
