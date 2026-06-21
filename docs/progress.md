# 项目进展

## 阶段 1 - 硬件调查 ✅

- [x] 确认 SoC = SM8735 (骁龙 8s Gen 4) 代号 "sun"
- [x] 确认一加项目代号 = "erhai"（洱海）
- [x] 拉取 Android 内核源码（ACK 6.6.89）
- [x] 拉取 Vendor 设备树源码
- [x] 分析 CPU 拓扑（6 中核 + 2 大核）

## 阶段 2 - 主线设备树 ✅

- [x] 创建 `sm8735.dtsi` — SoC 级设备树（355 行，基础 CPU/GIC/Timer/PSCI/UART）
- [x] 创建 `sm8735-oneplus-turbo6.dts` — 设备级设备树
- [x] 复制 DTS 到 linux-6.13 源码树并添加 Makefile 条目
- [x] DTC 编译通过（生成 6.3KB DTB）
- [ ] 添加 UFS 支持
- [ ] 添加 USB 支持
- [ ] 添加 TLMM/GPIO 完整配置
- [ ] 添加 PMIC/RPMH
- [ ] 添加更多时钟控制器 (GCC, DISPCC, GPUCC 等)
- [ ] 添加 RPMH 资源控制器 (RSC)

## 阶段 3 - 内核 🔄（阻塞中）

- [x] 选择主线内核版本：Linux 6.13 (2025-01)
- [x] 创建 ARM64 defconfig
- [x] 创建自定义配置 fragment（config/kernel.config）
- [x] 创建 GitHub Actions 云端编译工作流
- [ ] ⚠️ **解决 SM8735 驱动缺失问题** → 阻塞项
- [ ] 首次云端编译并验证 Image
- [ ] 验证 DTB 正确加载

> **阻塞原因**: Linux 6.13 不含 SM8735 (qcom,sun-*) 驱动。
> DTS 中使用的兼容字符串（qcom,sun-gcc, qcom,sun-tlmm, qcom,sun-pdc）
> 在 6.13 源码中不存在。需要从 ACK 6.6 移植驱动或使用支持 SM8735 的新内核。

## 阶段 4 - 引导

- [ ] 制作 initramfs
- [ ] 打包 boot.img
- [ ] 串口调试
- [ ] 首次启动测试

## 阶段 5 - 外设

- [ ] 显示驱动
- [ ] 触摸屏
- [ ] WiFi/蓝牙
- [ ] 音频
- [ ] 摄像头
- [ ] 传感器
- [ ] 电池/充电

---

## 已知问题

### 1. SM8735 驱动未上游化
- `qcom,sun-gcc` — 无匹配驱动（SM8650 的 GCC 寄存器地址不同）
- `qcom,sun-tlmm` — 无匹配驱动
- `qcom,sun-pdc` — 无匹配驱动（fallback `qcom,pdc` 可能工作）
- 解决方案：从 ACK 6.6.89 移植相关驱动，或使用新内核

### 2. DTS 中的占位值
- 内存配置使用 768MB 占位值，实际设备为 12GB/16GB
- GCC clock index 使用 `0` 作为占位，需确认真实值
- 缺少多个关键外设节点

### 3. 环境限制
- 本地 ARM64 Debian 13 容器，资源有限（24G 磁盘，2.4G 可用内存）
- 大型编译通过 GitHub Actions 云端完成
- 无本地 GPU 加速，调试依赖串口或 SSH
