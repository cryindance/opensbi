# OpenSBI 项目知识库

**Generated:** 2025-04-02
**Project:** RISC-V Open Source Supervisor Binary Interface (OpenSBI)
**License:** BSD-2-Clause

## 概述

OpenSBI 是 RISC-V SBI 规范的参考实现，提供平台无关的静态库 libsbi.a 和平台特定的固件示例。

核心组件：
- libsbi.a - 平台无关的 SBI 实现库
- libplatsbi.a - 平台特定的 SBI 库
- 固件示例（FW_DYNAMIC、FW_JUMP、FW_PAYLOAD）

## 项目结构

```
opensbi/
├── docs/              # 项目文档
├── firmware/          # 固件源码（入口点）
│   ├── fw_base.S     # 主启动入口
│   ├── fw_dynamic.S  # 动态固件
│   ├── fw_jump.S     # 跳转固件
│   └── fw_payload.S  # Payload固件
├── include/           # 头文件
│   ├── sbi/          # SBI库头文件
│   └── sbi_utils/    # 工具库头文件
├── lib/               # 核心库源码
│   ├── sbi/          # SBI实现（核心）
│   └── utils/        # 工具库
├── platform/          # 平台支持
│   ├── generic/      # 通用平台
│   ├── nuclei/       # Nuclei平台
│   ├── kendryte/     # K210平台
│   └── fpga/         # FPGA平台
└── scripts/           # 构建脚本
```

## 关键文件速查

| 任务 | 位置 | 说明 |
|------|------|------|
| 启动入口 | `firmware/fw_base.S` | _start 符号，Hart选择和重定位 |
| 主初始化 | `lib/sbi/sbi_init.c` | sbi_init()，所有模块初始化 |
| SBI调用处理 | `lib/sbi/sbi_ecall.c` | sbi_ecall_handler() |
| Hart管理 | `lib/sbi/sbi_hsm.c` | Hart状态管理 |
| 定时器 | `lib/sbi/sbi_timer.c` | SBI TIME扩展 |
| IPI | `lib/sbi/sbi_ipi.c` | 处理器间中断 |
| 平台接口 | `include/sbi/sbi_platform.h` | 平台操作结构体 |
| Nuclei平台 | `platform/nuclei/ux600/` | Nuclei UX600实现 |

## 代码规范

### 格式化 (.clang-format)
- **缩进：** Tab（宽度8）
- **大括号：** 函数后换行
- **对齐：** 连续赋值和尾部注释对齐
- **Include：** 不自动排序

### 编码风格
- 遵循 **Linux 内核编码风格**
- 每个源文件必须包含 `SPDX-License-Identifier: BSD-2-Clause`

### 提交规范
- **前缀格式：** `lib:`、`platform:`、`firmware:`、`docs:`
- **必须包含：** `Signed-off-by: Name <email>`
- **合并方式：** Rebase and Merge

## 构建命令

```bash
# 构建平台无关库
make

# 构建特定平台固件
make PLATFORM=generic
make PLATFORM=nuclei/ux600

# 使用Clang构建
make CC=clang

# 配置菜单
make PLATFORM=generic menuconfig

# 调试构建
make DEBUG=1

# 生成文档
make docs
```

## SBI 扩展

| 扩展 | 文件 | 功能 |
|------|------|------|
| BASE | `sbi_ecall_base.c` | 基础扩展 |
| HSM | `sbi_ecall_hsm.c` | Hart状态管理 |
| TIME | `sbi_ecall_time.c` | 定时器 |
| IPI | `sbi_ecall_ipi.c` | 处理器间中断 |
| RFENCE | `sbi_ecall_rfence.c` | 远程Fence |
| PMU | `sbi_ecall_pmu.c` | 性能监控 |
| SRST | `sbi_ecall_srst.c` | 系统重置 |

## 阅读路径

1. **启动流程：** `firmware/fw_base.S` → `lib/sbi/sbi_init.c`
2. **SBI接口：** `lib/sbi/sbi_ecall.c` + `include/sbi/sbi_ecall_interface.h`
3. **核心模块：** `sbi_hsm.c` → `sbi_ipi.c` → `sbi_timer.c`
4. **平台适配：** `platform/nuclei/ux600/platform.c`

## 注意事项

- 使用 PIE 工具链（如 riscv64-linux-gnu-gcc）生成位置无关代码
- 固件入口通过 a0=hartid, a1=dtb 传递参数
- HSM扩展从OpenSBI v0.7开始，支持有序的Hart启动
- 平台移植可参考 `platform/template/` 模板

## 参考文档

- `docs/contributing.md` - 贡献指南
- `docs/library_usage.md` - 库使用文档
- `docs/platform_guide.md` - 平台移植指南
- `docs/domain_support.md` - 域支持文档
- `docs/firmware/fw.md` - 固件类型文档
