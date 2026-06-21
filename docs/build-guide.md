# 构建指南

## 概述

这是 OnePlus Turbo 6 (SM8735) 的主线 Linux 内核移植构建指南。

⚠️ **重要：截至 Linux 7.1.1 (2026-06-19)，主线内核仍不含 SM8735 (qcom,sun-*) 驱动支持。**
DTS 中使用的专用兼容字符串（qcom,sun-gcc, qcom,sun-tlmm, qcom,sun-pdc）
在 7.1.1 源码中不存在匹配驱动。需要从 ACK 6.6.89 移植驱动。

**由于本地资源有限（ARM64 Debian 容器，2.4G 可用内存），
推荐通过 GitHub Actions 云端编译。**

---

## 方法一：GitHub Actions 云端编译（推荐）

### 1. 推送更新到 GitHub

```bash
cd ~/oneplus-turbo6/kernel
git add .
git commit -m "更新 DTS/配置/工作流"
git push origin main
```

### 2. 触发编译

推送会自动触发 GitHub Actions 工作流：
- `.github/workflows/build-kernel.yml`
- 目标内核：Linux 7.1.1

也可以在 GitHub 仓库的 **Actions** 页面手动运行（`workflow_dispatch`），
可选择编译不同的内核版本：
- `v7.1`（最新稳定版，默认）
- `v7.0`
- `v6.18`（LTS）
- `v6.13`（旧版）

### 3. 工作流自动执行的操作

编译工作流会：
1. 下载指定的 Linux 内核源码（缓存加速）
2. 应用自定义 DTS 文件（sm8735.dtsi, sm8735-oneplus-turbo6.dts）
3. **检查目标内核中是否有 SM8735(sun) 驱动支持**
4. **如果未找到 → 自动从 OnePlusOSS ACK 6.6.89 拉取 sun 驱动：**
   - `drivers/clk/qcom/gcc-sun.c` — GCC 时钟控制器
   - `drivers/pinctrl/qcom/pinctrl-sun.c` — TLMM Pin 控制器
   - `include/dt-bindings/clock/qcom,gcc-sun.h` — DT binding 头文件
   - 自动添加 Kconfig/Makefile 条目
5. 配置内核（ARM64 defconfig + 关键 Qualcomm 驱动 + sun 驱动）
6. 编译 DTB
7. 编译内核 Image
8. **创建 boot.img**（含最小 initramfs）
9. 上传编译产物

> ✅ **2026-06-21 验证**: SM8735 sun 驱动已成功从 ACK 6.6.89 拉取并在
> Linux 7.1.1 中编译通过。API 兼容性良好。

### 4. 获取编译产物

编译完成后，在 Actions 页面 → 对应运行 → **Artifacts** 下载：
- `kernel-image` — 内核 Image / Image.gz
- `device-tree-blobs` — 设备树 DTB 文件
- `boot-image` — 完整的 boot.img（所有触发器均生成）

---

## 方法二：本地编译（如果有足够资源）

### 前提条件

需要 Linux 系统（建议 Ubuntu 24.04+）和交叉编译工具链：

```bash
# Debian/Ubuntu
sudo apt update
sudo apt install -y \
    gcc-aarch64-linux-gnu \
    binutils-aarch64-linux-gnu \
    device-tree-compiler \
    flex bison \
    libssl-dev \
    libncurses-dev \
    bc cpio \
    kmod \
    android-sdk-libsparse-utils \
    lz4
```

### 1. 下载内核源码

```bash
cd ~/oneplus-turbo6
git clone --depth 1 --branch v7.1 \
    https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git \
    linux-7.1
```

### 2. 应用设备树

```bash
cp kernel/dts/sm8735.dtsi linux-7.1/arch/arm64/boot/dts/qcom/
cp kernel/dts/sm8735-oneplus-turbo6.dts linux-7.1/arch/arm64/boot/dts/qcom/
```

### 3. 配置内核

```bash
cd linux-7.1
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- defconfig

# 手动启用关键驱动
./scripts/config \
    --enable ARCH_QCOM \
    --enable QCOM_SCM \
    --enable SERIAL_QCOM_GENI \
    --enable SERIAL_QCOM_GENI_CONSOLE \
    --enable QCOM_PDC \
    --enable SM_GCC_8650 \
    --enable PINCTRL_SM8650 \
    --enable INTERCONNECT_QCOM_SM8650
```

### 4. 编译

```bash
# 仅编译 DTB（轻量，几秒）
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- dtbs

# 完整编译内核（重，需要多核 + 大内存）
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- Image -j$(nproc)
```

### 5. 检查编译产物

```bash
ls -la arch/arm64/boot/Image
ls -la arch/arm64/boot/dts/qcom/sm8735-oneplus-turbo6.dtb
```

---

## 制作 Boot Image

```bash
# 安装 mkbootimg
sudo apt install -y mkbootimg

# 创建最小 initramfs
mkdir -p initramfs
cd initramfs
cat > init << 'SCRIPT'
#!/bin/sh
mount -t proc none /proc
mount -t sysfs none /sys
echo "OnePlus Turbo 6 - Mainline Linux 7.1"
exec /bin/sh
SCRIPT
chmod +x init
find . | cpio -H newc -o | gzip > ../initramfs.cpio.gz
cd ..

# 打包 boot.img
mkbootimg \
    --kernel arch/arm64/boot/Image.gz \
    --dtb arch/arm64/boot/dts/qcom/sm8735-oneplus-turbo6.dtb \
    --ramdisk initramfs.cpio.gz \
    --cmdline "console=ttyMSM0,115200n8 earlycon root=/dev/ram0" \
    --base 0x80000000 \
    --pagesize 4096 \
    --header_version 4 \
    -o boot.img
```

---

## 刷机测试

### 前提条件
1. Bootloader 已解锁
2. 安装 fastboot 工具

```bash
# 进入 fastboot 模式（关机：音量上 + 音量下 + 电源键，然后连 USB）

# 引导测试（不写入，推荐）
fastboot boot boot.img

# 刷入
fastboot flash boot boot.img
```

---

## 已知限制

1. ~~**SM8735 驱动缺失（已解决）**~~: ✅ CI 现在自动从 OnePlusOSS ACK 6.6.89 拉取 gcc-sun.c + pinctrl-sun.c 并编译进 Linux 7.1.1
2. **串口调试**: 需要拆机焊接串口线到主板调试焊点
3. **内存配置**: 当前 DTS 使用占位值，需根据实际硬件调整
4. **DTS 外设节点**: UFS、USB、IOMMU 节点已添加但寄存器地址需与真机验证
5. **本地资源**: 完整编译需要 8GB+ 内存 + 多核 CPU，建议用 GitHub Actions
