# SM8735 (Sun) 内核驱动补丁

## 说明

Linux 主线内核（截至 7.1.1，2026-06-19）尚未包含 SM8735 (Snapdragon 8s Gen 4) 
的专用驱动支持。本目录包含从 ACK 6.6.89 移植的必要驱动补丁。

## ACK 驱动仓库

**来源**: OnePlusOSS 官方开源仓库  
**URL**: https://github.com/OnePlusOSS/android_kernel_oneplus_sm8735  
**分支**: `oneplus/sm8735_b_16.0.0_turbo_6`  
**commit**: `74c022746112a8940e801e4bcc16eea99e4b1bdc`  
**内核基础**: ACK 6.6.89

## 驱动清单

### 必须驱动（与 DTS 兼容字符串匹配）

| 驱动 | 状态 | 兼容字符串 | ACK 源文件 | ACK Kconfig |
|------|------|-----------|-----------|-------------|
| `sun-gcc` (GCC时钟) | ⏳ 待移植 | `qcom,sun-gcc` | `drivers/clk/qcom/gcc-sun.c` | `CONFIG_SM_GCC_SUN` |
| `sun-tlmm` (Pin控制) | ⏳ 待移植 | `qcom,sun-tlmm` | `drivers/pinctrl/qcom/pinctrl-sun.c` | `CONFIG_PINCTRL_SUN` |
| `sun-pdc` (中断控制器) | ✅ fallback可用 | `qcom,sun-pdc`→`qcom,pdc` | 主线已有 | `CONFIG_QCOM_PDC` |

### 可选 sun 时钟驱动（待移植）

| 驱动 | ACK 源文件 | Kconfig 配置 |
|------|-----------|-------------|
| camcc-sun (相机时钟) | `drivers/clk/qcom/camcc-sun.c` | `CONFIG_SM_CAMCC_SUN` |
| gpucc-sun (GPU时钟) | `drivers/clk/qcom/gpucc-sun.c` | `CONFIG_SM_GPUCC_SUN` |
| dispcc-sun (显示时钟) | `drivers/clk/qcom/dispcc-sun.c` | `CONFIG_SM_DISPCC_SUN` |
| videocc-sun (视频时钟) | `drivers/clk/qcom/videocc-sun.c` | `CONFIG_SM_VIDEOCC_SUN` |
| debugcc-sun (调试时钟) | `drivers/clk/qcom/debugcc-sun.c` | `CONFIG_SM_DEBUGCC_SUN` |
| evacc-sun (EVA时钟) | `drivers/clk/qcom/evacc-sun.c` | `CONFIG_SM_EVACC_SUN` |
| cambistmclkcc-sun | `drivers/clk/qcom/cambistmclkcc-sun.c` | `CONFIG_SM_CAMBISTMCLKCC_SUN` |
| tcsrcc-sun (TCSR时钟) | `drivers/clk/qcom/tcsrcc-sun.c` | `CONFIG_SM_TCSRCC_SUN` |

## 现状

- **2026-06-21**: CI 编译成功（运行 #27898880561），Linux 7.1.1 + SM8650 fallback 驱动
- **阻塞点**: ACK 6.6.89 的驱动需要适配到 Linux 7.1.1 的 API

### DT binding 头文件
ACK 仓库中以下 DT binding 头文件是必需的：
- `include/dt-bindings/clock/qcom,gcc-sun.h` — GCC 时钟 ID 定义
- 其他 sun 时钟头文件（camcc-sun.h, gpucc-sun.h 等）

可通过以下 URL 直接获取：
```
https://raw.githubusercontent.com/OnePlusOSS/android_kernel_oneplus_sm8735/oneplus/sm8735_b_16.0.0_turbo_6/include/dt-bindings/clock/qcom,gcc-sun.h
```

## 移植方法

1. CI 工作流从 OnePlusOSS ACK 仓库直接拉取 sun 驱动源文件
2. 将源文件放入主线内核对应目录（`drivers/clk/qcom/`, `drivers/pinctrl/qcom/`）
3. 添加 Kconfig/Makefile 条目
4. 编译并修复 API 不兼容问题
5. 迭代直到编译通过

## 获取 ACK 6.6.89 源码

```bash
# 从 OnePlusOSS 拉取（推荐）
git clone --depth 1 --branch oneplus/sm8735_b_16.0.0_turbo_6 \
    https://github.com/OnePlusOSS/android_kernel_oneplus_sm8735.git

# 或者从 AOSP 公共源码
git clone --depth 1 --branch android-6.6-2025-01 \
    https://android.googlesource.com/kernel/common
```
