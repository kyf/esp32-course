# 第 02 课实验草案：GPIO 输入轮询

**尚未编译或仿真验证。**

从相邻的 `../gpio-blink-rust` 复制项目骨架并将包/二进制名称改为 `gpio-input-polling`。将 `src/main.rs` 中的程序内容移入复制工程的 `src/bin/main.rs`。保留第一课的 Xtensa / build-std / bootloader 设置。

硬件预期接线：GPIO5 经按钮连接 GND，使用内部上拉；GPIO4 接指示设备（需限流电阻/合适驱动）。按钮按下时 GPIO4 高，释放时低。

建议验收：先 `cargo +esp build --release`，再保存 BIN，并在真实硬件上测量或先确认模拟器支持如何注入 GPIO5 输入。下一步加入 20ms 软件去抖。

详见 `book/src/lesson-02-input.md`。
