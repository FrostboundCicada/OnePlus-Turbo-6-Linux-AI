# 一加 Turbo 6 硬件规格

## 基本信息

| 项目 | 数据 |
|------|------|
| 设备名称 | OnePlus Turbo 6 |
| 处理器 | Qualcomm SM8735 (Snapdragon 8s Gen 4) |
| 平台代号 | **sun**（高通内部） / **erhai**（一加内部） |
| Android 内核 | ACK 6.6.89 |
| Android 版本 | Android 16 (B) |

## SoC 详情 - SM8735 "Sun"

### CPU

| 核心 | 寄存器 | 类型 | capacity | DPC | 所属集群 |
|------|--------|------|----------|-----|---------|
| CPU0 | 0x0 | 中核 | 1792 | 238 | Cluster0 |
| CPU1 | 0x100 | 中核 | 1792 | 238 | Cluster0 |
| CPU2 | 0x200 | 中核 | 1792 | 238 | Cluster0 |
| CPU3 | 0x300 | 中核 | 1792 | 238 | Cluster0 |
| CPU4 | 0x400 | 中核 | 1792 | 238 | Cluster0 |
| CPU5 | 0x500 | 中核 | 1792 | 238 | Cluster0 |
| CPU6 | 0x10000 | 大核 | 1894 | 588 | Cluster1 |
| CPU7 | 0x10100 | 大核 | 1894 | 588 | Cluster1 |

### 内存

- LPDDR5X
- 12GB / 16GB 配置

### 存储

- UFS 4.0（嵌入式）

### PMIC

- PMK8550（主电源管理）
- PM8550 系列

### 关键外设

| 外设 | 接口 |
|------|------|
| UART (调试串口) | QUPV3 SE7 (2uart) |
| UFS | ufshc_mem |
| SD卡 | sdhc_2 |
| USB | QUSB / QMP |
| PCIe | sun-pcie |

## 设备树文件结构（Android 内核）

```
vendor_dt/kernel_platform/qcom/opensource/devicetree/
├── qcom/
│   ├── sun.dtsi              ← SoC 主设备树 (4105行)
│   ├── sun.dts               ← SoC 实例
│   ├── sun-pinctrl.dtsi      ← GPIO 引脚配置
│   ├── sun-regulators.dtsi   ← 电压调节器
│   ├── sun-thermal.dtsi      ← 热管理
│   ├── sun-usb.dtsi          ← USB
│   ├── sun-pcie.dtsi         ← PCIe
│   └── sun-qupv3.dtsi        ← QUPV3 外设
└── oplus/
    ├── sun_overlay_common.dtsi     ← 一加公用覆盖层
    ├── erhai_overlay_common.dtsi   ← 一加 Turbo 6 通用配置
    ├── erhai-24976-sun-overlay.dts ← 一加 Turbo 6 正式版
    └── ...
```
