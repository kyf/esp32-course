# 故障排查速查表

| 症状 | 首选检查 | 判断与修复 |
| --- | --- | --- |
| `cargo +esp` 不存在 | `rustup toolchain list` | 安装 esp-rs Xtensa 工具链，加载环境 |
| ELF 正常，BIN 无法加载 | `esptool --chip esp32s3 image-info ...` | 区分实际段与 PADDING |
| `address 0x00000000 not mapped` | `image-info` + 模拟器 `boot_app` | esp32sim v0.3.0 零地址段加载问题 |
| ROM 地址出现非法指令 | 检查 `--rom` 路径、ROM 加载日志 | 确保 S3 对应的 ROM ELF 已加载 |
| `mkdir: illegal option -- -` | `sh -x tools/fetch-rom-elfs.sh roms` | 核实实际传参，不预判根因 |
| `_WindowOverflow4` | 反汇编异常入口、检查停止选项 | 窗口溢出可能可恢复；不要过早停机 |
| `vcd` 0 个事件 | 检查 CPU 是否进入业务循环 | 先解决启动问题 |
| WaveTrace 显示异常或空白 | 检查 `$timescale`、时间轴缩放、实际 VCD 事件 | 本次 WaveTrace 未正确呈现原始文件；Surfer 直接打开同一 VCD 后显示正常，具体 WaveTrace 原因未确定 |
| Surfer 时间轴显示很大的 `ps` 数字 | 查看 VCD 中 `$timescale 1ps $end` | `1000000000000ps=1s`；缩放到 0～5s 即可看到完整方波 |
| VCD 无法查看 | 检查 `$timescale` 和 `$var` | 手工核对时间戳，或使用 Surfer 等查看器交叉验证 |
| `--board none` 看不到 LED 动画 | 检查板卡模型和 GPIO VCD | `none` 不连接外设；可用 `tools/led-vcd-viewer.html` 回放，不等于实时板卡 LED |
| VCD、UART、模拟器日志混淆 | 检查来源：引脚事件 / 固件主动串口输出 / `[emu]` 运行状态 | 不要把 GPIO VCD 当作 `espflash monitor` 串口内容 |
| `run.sh` 在别人机器失败 | `cat source/gpio-blink-rust/run.sh` | 当前脚本含个人绝对路径，改用变量化命令 |

课程默认保留可复现命令、版本、最小日志和验收波形。技术推测与经实测验证的结论应明确分开。
