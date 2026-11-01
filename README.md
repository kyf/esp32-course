# esp32-course

ESP32-S3 Rust + `esp-hal` + `no_std` 教学项目，优先使用 macOS 和 esp32sim 进行可复现学习。

- 课程文档：[`book/src/SUMMARY.md`](book/src/SUMMARY.md)
- 第一课已验证源码：[`source/gpio-blink-rust`](source/gpio-blink-rust/)
- 第二课源码草案：[`source/gpio-input-polling`](source/gpio-input-polling/)
- 完整课程大纲：[`book/src/roadmap.md`](book/src/roadmap.md)
- esp32sim 的 PADDING 修复与 PR 步骤：[`book/src/appendix-esp32sim-pr.md`](book/src/appendix-esp32sim-pr.md)

## 在本地预览课程

```bash
cargo install mdbook --locked
cd book
mdbook serve --open
```

第 01 课已通过 VCD 验证 GPIO4 每 500ms 翻转；第 02 课仍处于准备阶段。
