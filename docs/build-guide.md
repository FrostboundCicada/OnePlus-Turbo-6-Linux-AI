# 构建指南

## 前提条件

你需要一个 Linux 系统（建议 Ubuntu 22.04+ 或 Debian 12+），以及：

### 1. 安装交叉编译工具链

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
    bc \
    cpio \
    kmod \
    android-sdk-libsparse-utils \
    qemu-user-static \
    debootstrap

# 验证
aarch64-linux-gnu-gcc --version
```

### 2. 下载主线 Linux 内核

```bash
# 建议使用最新的稳定主线版本
cd ~
git clone --depth 1 https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
cd linux
```

## 编译内核

### 1. 应用设备树

将我们的设备树文件复制到内核源码树中：

```bash
# 方法1：直接替换(recommended for development)
cp ~/OnePlus-Turbo-6-Linux-AI/dts/sm8735.dtsi arch/arm64/boot/dts/qcom/
cp ~/OnePlus-Turbo-6-Linux-AI/dts/sm8735-oneplus-turbo6.dts arch/arm64/boot/dts/qcom/
```

### 2. 配置内核

```bash
# 创建初始配置
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- defconfig

# 启用高通 SoC 支持
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- menuconfig
```

在 menuconfig 中启用：
```
Platform selection → 
    [*] Qualcomm Technologies, Inc. SoCs
    
Device Drivers →
    [*] Serial device support →
        [*] ARM AMBA PL011 serial port support
        [*] Qualcomm GENI serial driver
    [*] GPIO Support →
        [*] Qualcomm GPIO controller (TLMM)
    [*] Pin controllers →
        [*] Qualcomm pin controller
    [*] Clock Driver →
        [*] Qualcomm clock controller
    [*] Mailbox Hardware Support →
        [*] Qualcomm IPCC Mailbox
```

### 3. 编译

```bash
# 添加我们的 DTS 到 Makefile
echo "dtb-\$(CONFIG_ARCH_QCOM) += sm8735-oneplus-turbo6.dtb" >> arch/arm64/boot/dts/qcom/Makefile

# 编译设备和内核
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- dtbs -j$(nproc)
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- Image.gz -j$(nproc)
```

如果编译成功，你会看到：
- `arch/arm64/boot/Image.gz` — 压缩内核
- `arch/arm64/boot/dts/qcom/sm8735-oneplus-turbo6.dtb` — 设备树

## 制作 Boot Image

### 1. 准备 initramfs

```bash
# 使用 debootstrap 创建最小 rootfs
mkdir -p rootfs
sudo debootstrap --arch=arm64 --foreign bullseye rootfs

# 或者使用 busybox 创建最小 initramfs
mkdir -p initramfs
cd initramfs
cat > init << 'EOF'
#!/bin/sh
/bin/sh
EOF
chmod +x init
```

### 2. 打包 boot.img

```bash
# 安装 mkbootimg
sudo apt install -y mkbootimg

# 创建 boot.img
mkbootimg \
    --kernel arch/arm64/boot/Image.gz \
    --dtb arch/arm64/boot/dts/qcom/sm8735-oneplus-turbo6.dtb \
    --ramdisk <initramfs.cpio.gz> \
    --cmdline "console=ttyMSM0,115200n8 earlycon root=/dev/mmcblk0pXX" \
    --base 0x80000000 \
    --pagesize 4096 \
    --header_version 4 \
    -o boot.img
```

## 刷机测试

### 前提条件

1. Bootloader 已解锁
2. 安装 fastboot 工具

```bash
# 进入 fastboot 模式（关机后：按住音量上 + 音量下 + 电源键，然后连接 USB）

# 刷入内核
fastboot flash boot boot.img

# 或者只是引导测试（不写入）
fastboot boot boot.img
```

## 串口调试

一加 Turbo 6 的调试串口引脚：

| 信号 | 焊盘位置 |
|------|---------|
| TX | 待确认 |
| RX | 待确认 |
| GND | 待确认 |

需要拆机并焊接串口线到主板的调试焊点。波特率：115200 8N1。

## 参考链接

- [主线 Linux 内核](https://kernel.org)
- [Android Boot Image Header](https://source.android.com/docs/core/architecture/bootloader/boot-image-header)
- [高通 Boot Architecture](https://www.qualcomm.com)
