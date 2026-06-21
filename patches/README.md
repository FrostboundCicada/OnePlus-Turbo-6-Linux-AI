# SM8735 (Sun) 内核驱动补丁

## 说明

Linux 主线内核（截至 7.1.1，2026-06-19）尚未包含 SM8735 (Snapdragon 8s Gen 4) 
的专用驱动支持。本目录包含从 ACK 6.6.89 移植的必要驱动补丁。

## 驱动清单

| 驱动 | 状态 | 兼容字符串 | 来源 |
|------|------|-----------|------|
| `sun-gcc` (时钟控制器) | ⏳ 待移植 | `qcom,sun-gcc` | ACK 6.6.89 |
| `sun-tlmm` (Pin控制) | ⏳ 待移植 | `qcom,sun-tlmm` | ACK 6.6.89 |
| `sun-pdc` (中断控制器) | ✅ 有 fallback | `qcom,sun-pdc` → `qcom,pdc` | 主线已有 |

## 移植方法

1. 从 ACK 6.6.89 内核源码提取对应驱动文件
2. 适配 Linux 7.1.1 的驱动框架 API 变化
3. 将补丁文件放入此目录
4. GitHub Actions 工作流会自动应用补丁

## 获取 ACK 6.6.89 源码

```bash
git clone --depth 1 --branch android-6.6-2025-01 \
    https://android.googlesource.com/kernel/common
```

高通 vendor 驱动通常在以下分支：
- `android-6.6-2025-01`
- `android-6.6-2025-04`
- 或 OPPO/OnePlus 的 vendor 内核分支
