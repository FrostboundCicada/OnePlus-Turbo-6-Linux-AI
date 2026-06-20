# OnePlus Turbo 6 Linux

> 一加 Turbo 6（骁龙 8s Gen 4 / SM8735）主线 Linux 移植项目

## 项目目标

将主线 Linux 内核移植到一加 Turbo 6 手机，使其能够运行桌面 Linux 发行版（Ubuntu/Debian/Kali 等）。

## 硬件规格

| 项目 | 参数 |
|------|------|
| 处理器 | 高通 SM8735（骁龙 8s Gen 4），代号 **sun** |
| 项目代号 | **erhai**（洱海） |
| CPU | 8 核 ARMv8（6 中核 + 2 大核） |
| 内存 | 12GB/16GB LPDDR5X |
| 存储 | UFS 4.0 |
| Bootloader | 高通 UEFI（ABL） |

## 仓库结构

```
├── dts/             ← 设备树（Device Tree）
│   ├── sm8735.dtsi         ← SoC 级设备树
│   └── sm8735-oneplus-turbo6.dts  ← 设备级设备树
├── kernel/
│   ├── config/              ← 内核配置文件
│   └── patches/             ← 内核补丁
├── boot/                    ← 启动引导
│   ├── initramfs/
│   └── build-bootimg.sh
├── scripts/                 ← 构建脚本
│   └── build-kernel.sh
└── docs/                    ← 文档
    ├── hardware-specs.md
    └── build-guide.md
```

## 进展

正在进行的阶段性工作：

- [x] 硬件规格调查
- [x] Android 内核源码获取（ACK 6.6.89）
- [x] Vendor 设备树提取
- [ ] **主线设备树创建**（当前阶段）
- [ ] 主线内核选择与配置
- [ ] 交叉编译环境搭建
- [ ] 内核编译与 boot.img 制作
- [ ] 基础启动测试
- [ ] 调试输出验证
- [ ] 图形/外设驱动适配

## 技术栈

- Linux 内核（主线，目标版本 ≥ 6.13）
- Device Tree（DTB/DTS）
- ARM64 交叉编译（aarch64-linux-gnu-）
- Android Boot Image（bootimg）
- Ubuntu/Debian RootFS（debootstrap）

## 参考资源

- [OnePlus GitHub](https://github.com/OnePlusOSS)
- [主线 Linux 内核](https://kernel.org)
- [高通设备主线 Linux 支持 - Linaro](https://www.linaro.org)
- [Arch Linux ARM](https://archlinuxarm.org)

---

*由 AI 辅助开发的社区项目*
