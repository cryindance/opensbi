# OpenSBI 平台支持 (platform)

## 概述

平台特定代码目录，包含各厂商平台的固件实现和移植支持。

**支持的平台类型：**
- **通用平台 (generic)** - 基于FDT设备树的通用实现
- **特定SoC** - Nuclei UX600、Kendryte K210等
- **FPGA平台** - Ariane、OpenPiton

## 目录结构

```
platform/
├── generic/               # 通用平台（基于FDT）
│   ├── allwinner/        # 全志D1
│   ├── andes/            # Andes AE350
│   ├── nuclei/           # Nuclei SoC系列
│   ├── renesas/          # Renesas RZ/Five
│   ├── sifive/           # SiFive FU540/FU740
│   ├── starfive/         # StarFive JH7110
│   ├── platform.c        # 通用实现
│   └── include/
│       └── platform_override.h
├── nuclei/               # Nuclei原生平台
│   └── ux600/            # UX600处理器
├── kendryte/             # Kendryte
│   └── k210/             # K210双核
├── fpga/                 # FPGA平台
│   ├── ariane/
│   └── openpiton/
└── template/             # 平台移植模板
```

## 平台实现模式

### 模式1: 独立实现（传统）
**示例:** `nuclei/ux600/`, `kendryte/k210/`

特点：
- 完整实现所有平台操作
- 直接定义 `struct sbi_platform_operations`

### 模式2: Override模式（推荐）
**示例:** `generic/andes/`, `generic/nuclei/`

特点：
- 通用 `platform.c` 提供基础实现
- 子平台通过 `platform_override` 扩展
- 基于FDT设备树匹配

### 模式3: 模板参考
**示例:** `template/`

## 平台文件结构

每个平台必须包含：

```
platform/<vendor>/<platform>/
├── platform.c           # 平台实现（必需）
├── objects.mk          # 编译配置（必需）
├── Kconfig             # Kconfig选项
└── configs/
    └── defconfig       # 默认配置
```

## 关键文件

| 文件 | 用途 |
|------|------|
| `template/platform.c` | 平台移植模板 |
| `generic/platform.c` | 通用平台实现 |
| `generic/include/platform_override.h` | Override结构定义 |
| `nuclei/ux600/platform.c` | Nuclei UX600实现 |

## 平台初始化流程

```
fw_platform_init() [generic/platform.c]
    ├── 查找匹配的 platform_override
    ├── 调用 override->fw_init()
    ├── 解析FDT获取hart_count
    └── 返回FDT地址

generic_early_init()
    ├── fdt_reset_init()
    └── 调用 override->early_init()

generic_final_init()
    ├── 调用 override->final_init()
    ├── fdt_cpu_fixup()
    └── fdt_domain_fixup()
```

## platform_override 结构

```c
struct platform_override {
    const struct fdt_match *match_table;
    u64 (*features)(const struct fdt_match *match);
    int (*early_init)(bool cold_boot, const struct fdt_match *match);
    int (*final_init)(bool cold_boot, const struct fdt_match *match);
    int (*fdt_fixup)(void *fdt, const struct fdt_match *match);
    void (*fw_init)(void *fdt, const struct fdt_match *match);
    ...
};
```

## 平台移植步骤

1. **创建目录结构**
   ```bash
   mkdir -p platform/<vendor>/<platform>/configs
   ```

2. **编写 platform.c**
   - 定义硬件资源地址（PLIC、CLINT、UART等）
   - 实现初始化函数
   - 定义 `struct sbi_platform_operations`
   - 定义 `struct sbi_platform`

3. **编写 objects.mk**
   ```makefile
   platform-objs-y += platform.o
   FW_TEXT_START=0x80000000
   FW_DYNAMIC=y
   ```

4. **编写 Kconfig**
   ```kconfig
   config PLATFORM_MYPLATFORM
       select IPI_MSWI
       select IRQCHIP_PLIC
       select SERIAL_UART8250
   ```

## 典型硬件资源

| 资源 | 典型地址 | 配置宏 |
|------|----------|--------|
| PLIC | 0x0c000000 | PLATFORM_PLIC_ADDR |
| CLINT | 0x02000000 | PLATFORM_CLINT_ADDR |
| UART | 0x09000000 | PLATFORM_UART_ADDR |
| Hart数 | 1-8 | PLATFORM_HART_COUNT |

## 平台对比

| 平台 | 模式 | Hart数 | 特点 |
|------|------|--------|------|
| generic/andes | Override | - | Andes AE350 |
| generic/nuclei | Override | 8 | EvalSoC (SMP/L2) |
| nuclei/ux600 | 独立 | 1 | CPU频率测量 |
| kendryte/k210 | 独立 | 2 | 动态时钟检测 |

## 开发注意事项

1. **选择模式**：新平台推荐Override模式（基于FDT）
2. **固件地址**：确保 `FW_TEXT_START` 与实际硬件匹配
3. **UART初始化**：早期启动需要UART输出调试信息
4. **平台ID**：唯一的 `struct sbi_platform` 名称
5. **参考模板**：`platform/template/` 提供完整参考

## 参考文档

- `docs/platform_guide.md` - 平台移植指南
- `docs/platform_requirements.md` - 平台需求
- `docs/platform/platform.md` - 各平台文档
