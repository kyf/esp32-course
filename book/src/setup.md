# 准备：macOS 开发环境

第 01 课实际使用：macOS、ESP32-S3 Xtensa、Rust `no_std`、`esp-hal ~1.2.2`、`espflash 4.7.0`、`esptool 5.6.0`、`esp32sim 0.3.0`。仓库的 `rust-toolchain.toml` 固定了生成工程时的特定工具链，但实际构建命令使用本地安装的 `+esp` 工具链。先以源码和本机输出检查版本，不盲目切换。

## 工具链检查

```bash
xcode-select --install
brew --version
rustup --version
rustup toolchain list
cargo +esp --version
espflash --version
esptool version
```

若 `+esp` 缺失，先参照 [esp-rs/espup](https://github.com/esp-rs/espup) 的当前安装说明配置 Xtensa 工具链；不要假定普通 stable Rust 能直接构建 Xtensa `no_std`。安装工具链后，按照 espup 给出的命令加载环境变量。

```bash
git clone https://github.com/kyf/esp32-course.git
cd esp32-course/source/gpio-blink-rust
cargo +esp build --release
```

生成的 ELF 通常位于 `target/xtensa-esp32s3-none-elf/release/gpio-blink-rust`。

## 生成 BIN 与检查

```bash
ELF=target/xtensa-esp32s3-none-elf/release/gpio-blink-rust
espflash save-image --chip esp32s3 "$ELF" "$ELF.bin"
esptool --chip esp32s3 image-info "$ELF.bin"
```

## 获取模拟器及 ROM

将 `esp32sim` 克隆在**课程仓库外部**，版本以 `v0.3.0` 为研究基线；其镜像 PADDING 问题需要应用附录修复或使用已修复的版本。

```bash
git clone https://github.com/mikroverk/esp32sim.git
cd esp32sim
git checkout v0.3.0
cargo build --release
# 在此版本仓库中确认 tools/fetch-rom-elfs.sh 存在后：
sh tools/fetch-rom-elfs.sh roms
ls -lh roms/esp32s3_rev0_rom.elf
```

模拟器、ROM、固件路径不要混淆。`run.sh` 当前包含原作者机器的**绝对路径**，不具备跨机器可移植性；建议按第 01 课所给的变量化命令运行。

## mdBook

```bash
cargo install mdbook --locked
cd esp32-course/book
mdbook serve --open
# 或 mdbook build
```

书籍输入为 `book/src/`，输出配置为 `book/dist/`（应忽略生成目录）。
