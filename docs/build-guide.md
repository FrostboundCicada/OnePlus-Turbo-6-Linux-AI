# 构建指南

## 概述

这是 OnePlus Turbo 6 (SM8735) 的主线 Linux 内核移植构建指南。

⚠️ **重要：Linux 6.13 主线内核不含 SM8735 (qcom,sun-*) 驱动支持。**
DTS 中使用的专用兼容字符串（qcom,sun-gcc, qcom,sun-tlmm, qcom,sun-pdc）
在 6.13 源码中不存在匹配驱动。需要先从 ACK 6.6.89 移植驱动，
或使用支持 SM8735 的更新内核版本。

**由于本地资源有限（ARM64 Debian 容器，2.4G 可用内存），
推荐通过 GitHub Actions 云端编译。**

---

## 方法一：GitHub Actions 云端编译（推荐）

### 1. 推送更新到 GitHub

```bash
cd ~/oneplus-turbo6/kernel
git add .
git commit -m "Update DTS/config/build workflows"
git push origin main
```

### 2. 触发编译

推送会自动触发 GitHub Actions 工作流：
- `.github/workflows/build-kernel.yml`

也可以在 GitHub 仓库的 **Actions** 页面手动运行（`workflow_dispatch`），
可选择编译不同内核版本（如 `v6.13`, `v7.0`, `v7.1` 等）。

### 3. 获取编译产物

编译完成后，在 Actions 页面 → 对应运行 → **Artifacts** 下载：
- `kernel-image` — 内核 Image / Image.gz
- `device-tree-blobs` — 设备树 DTB 文件
- `boot-image` — 完整的 boot.img（仅 workflow_dispatch）

---

## 方法二：本地编译（如果有足够资源）

### 前提条件

需要 Linux 系统（建议 Ubuntu 22.04+ 或 Debian 12+）和交叉编译工具链：

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

### 1. 应用设备树

```bash
cp ~/OnePlus-Turbo-6-Linux-AI/dts/sm8735.dtsi arch/arm64/boot/dts/qcom/
cp ~/OnePlus-Turbo-6-Linux-AI/dts/sm8735-oneplus-turbo6.dts arch/arm64/boot/dts/qcom/
```

### 2. 配置内核

```bash
# 从 defconfig 开始
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- defconfig

# 或使用我们的自定义配置
cp config/kernel.config linux/.config
```

### 3. 编译

```bash
# 仅编译 DTB（轻量，几秒）
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- dtbs

# 完整编译内核（重，需要多核 + 大内存）
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- Image -j$(nproc)
```

### 4. 检查编译产物

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
echo "OnePlus Turbo 6 - Mainline Linux"
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

1. **SM8735 驱动缺失**: Linux 6.13 不含 SM8735 驱动，需要从 ACK 6.6 移植
2. **串口调试**: 需要拆机焊接串口线到主板调试焊点
3. **内存配置**: 当前 DTS 使用占位值，需根据实际硬件调整
4. **本地资源**: 完整编译需要 8GB+ 内存 + 多核 CPU，建议用 GitHub Actions
