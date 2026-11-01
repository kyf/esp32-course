# 附录：esp32sim v0.3.0 零地址 PADDING 修复与 PR

**问题复现：** 使用 `espflash save-image` 生成 ESP32-S3 APP BIN，`esptool image-info` 显示 `load_addr=0x00000000` 的 PADDING 段。原始 `esp32sim v0.3.0` 在 `esp32s3/src/soc.rs` 的 `boot_app` 中对所有非 flash-mapped 段执行 `SocBus::load_bytes`，因此报 `load: address 0x00000000 not mapped`。

## 最小补丁（根据本课程成功的本地修复）

```diff
--- a/esp32s3/src/soc.rs
+++ b/esp32s3/src/soc.rs
@@
         for s in &img.segments {
+            // ESP image padding is not a loadable RAM segment.
+            if s.load_addr == 0 { continue; }
             let start = app_off + s.file_off as usize;
```

该补丁已通过课程的实际 GPIO Blink 用例验证；但需要补回归测试、检查其他芯片的相同加载路径，并核实上游默认分支是否已有更全面的实现。**不要直接向 v0.3.0 标签提交 PR**，PR base 应选择上游的默认开发分支。

## 推荐提交流程

```bash
# 先在 GitHub 网页 Fork https://github.com/mikroverk/esp32sim 到自己的账号
git clone https://github.com/kyf/esp32sim.git
cd esp32sim
git remote add upstream https://github.com/mikroverk/esp32sim.git
git fetch upstream
git switch -c fix/s3-skip-zero-address-padding upstream/main
# 修改 esp32s3/src/soc.rs 并添加必要测试
cargo fmt --all --check
cargo test
cargo build --release
git diff --check
git add esp32s3/src/soc.rs
git commit -m "fix(esp32s3): skip zero-address app image padding"
git push -u origin fix/s3-skip-zero-address-padding
```

注意：若上游默认分支并非 `main`，应以 GitHub 页面显示的默认分支替换 `upstream/main`。`cargo test` 是否全部可运行取决于本地环境。

PR 标题：`fix(esp32s3): skip zero-address ESP image padding during app boot`

PR 正文建议：

```markdown
## Problem
espflash-generated ESP32-S3 app images can contain a zero-address PADDING segment.
The S3 app boot loader treats it as a loadable RAM segment and errors with:
`load: address 0x00000000 not mapped`.

## Fix
Skip the zero-address padding segment when loading app image segments.

## Reproduction
1. Build a Rust esp-hal 1.2.x GPIO4 blink app targeting xtensa-esp32s3-none-elf.
2. Convert ELF to BIN via espflash save-image --chip esp32s3.
3. Run esp32sim --chip s3 --board none --boot app with the BIN, ELF, and S3 ROM.

## Verification
With the patch, the image boots and runs for 5 emulated seconds.
The VCD records GPIO4 toggling every 500 ms.
Without the patch, app boot fails on address 0x00000000.

## Notes
Observed on esp32sim v0.3.0; please advise if the default branch has a preferred shared image-loader fix.
```

比较 URL：`https://github.com/mikroverk/esp32sim/compare/main...kyf:esp32sim:fix/s3-skip-zero-address-padding`（仅在 fork 与分支创建后使用）。确保 PR 中没有临时 ROM、BIN、构建产物和绝对本机路径。
