# OpenSBI 核心库 (lib/sbi)

## 概述

OpenSBI 的核心 SBI 实现库，包含 SBI 扩展处理、硬件抽象和系统服务。

**关键统计：**
- 47 个源文件
- 12 个 SBI 扩展实现
- 核心功能：Hart管理、中断、定时器、性能监控

## 文件组织

```
lib/sbi/
├── sbi_init.c              # 主初始化入口
├── sbi_ecall.c             # SBI调用分发核心
├── sbi_ecall_*.c           # 各SBI扩展实现
├── sbi_hsm.c               # Hart状态管理
├── sbi_ipi.c               # 处理器间中断
├── sbi_timer.c             # 定时器管理
├── sbi_pmu.c               # 性能监控单元
├── sbi_trap.c              # 异常/中断处理
├── sbi_domain.c            # 域支持
├── sbi_console.c           # 控制台输出
├── sbi_system.c            # 系统控制（重启/关机）
└── sbi_tlb.c               # TLB管理
```

## SBI 扩展文件

| 文件 | 扩展 | 说明 |
|------|------|------|
| sbi_ecall_base.c | BASE (0x10) | 基础扩展：版本查询、扩展探测 |
| sbi_ecall_hsm.c | HSM (0x48534D) | Hart状态管理：启动/停止/挂起 |
| sbi_ecall_time.c | TIME (0x54494D45) | 定时器：设置下一次事件 |
| sbi_ecall_ipi.c | IPI (0x735049) | 处理器间中断 |
| sbi_ecall_rfence.c | RFENCE (0x52464E43) | 远程Fence：TLB同步 |
| sbi_ecall_pmu.c | PMU (0x504D55) | 性能监控单元 |
| sbi_ecall_srst.c | SRST (0x53525354) | 系统重置 |
| sbi_ecall_dbcn.c | DBCN (0x4442434E) | 调试控制台 |
| sbi_ecall_susp.c | SUSP (0x53555350) | 系统挂起 |
| sbi_ecall_cppc.c | CPPC (0x43505043) | 电源管理 |
| sbi_ecall_legacy.c | 0x0-0x9 | v0.1兼容扩展 |
| sbi_ecall_vendor.c | 0x09xxxxxx | 厂商特定扩展 |

## 核心流程

### SBI 调用处理流程
```
sbi_trap_handler()
    └── sbi_ecall_handler()
        └── sbi_ecall_find_extension(extid)
            └── ext->handle() → 各扩展处理
```

### 初始化流程 (sbi_init.c)
```
sbi_init()
    ├── sbi_hart_init()
    ├── sbi_console_init()
    ├── sbi_timer_init()
    ├── sbi_ipi_init()
    ├── sbi_platform_init()
    ├── sbi_ecall_init()
    ├── sbi_hsm_init()
    ├── sbi_pmu_init()
    └── sbi_domain_init()
```

## 扩展注册机制

### 1. 定义扩展结构体
```c
struct sbi_ecall_extension ecall_hsm = {
    .extid_start = SBI_EXT_HSM,
    .extid_end = SBI_EXT_HSM,
    .register_extensions = sbi_ecall_hsm_register_extensions,
    .handle = sbi_ecall_hsm_handler,
};
```

### 2. 添加到构建 (objects.mk)
```makefile
carray-sbi_ecall_exts-$(CONFIG_SBI_ECALL_HSM) += ecall_hsm
libsbi-objs-$(CONFIG_SBI_ECALL_HSM) += sbi_ecall_hsm.o
```

### 3. 初始化时自动注册 (sbi_ecall_init)

## 关键接口

### Hart 管理 (sbi_hsm.c)
- `sbi_hsm_hart_start()` - 启动指定Hart
- `sbi_hsm_hart_stop()` - 停止当前Hart
- `sbi_hsm_hart_get_state()` - 获取Hart状态

### IPI (sbi_ipi.c)
- `sbi_ipi_send_smode()` - 发送S-mode IPI
- `sbi_ipi_send_halt()` - 发送停机IPI

### 定时器 (sbi_timer.c)
- `sbi_timer_event_start()` - 设置定时器事件
- `sbi_timer_event_stop()` - 停止定时器

### PMU (sbi_pmu.c)
- `sbi_pmu_num_counters()` - 获取计数器数量
- `sbi_pmu_counter_start()` - 启动计数器
- `sbi_pmu_counter_stop()` - 停止计数器

## 开发注意事项

1. **新增扩展：** 参考 sbi_ecall_hsm.c 模式，实现 handle 回调函数
2. **条件注册：** 使用硬件检测决定是否注册（如 PMU 检查 sbi_pmu_get_device()）
3. **错误码：** 使用 sbi_ecall_interface.h 中定义的 SBI_ERR_* 常量
4. **Trap处理：** 严重错误通过设置 trap 结构体转发给 S-mode

## 头文件对应

- `include/sbi/sbi_ecall.h` - 扩展结构体定义
- `include/sbi/sbi_ecall_interface.h` - 扩展ID常量、错误码
- `include/sbi/sbi_hsm.h` - Hart管理接口
- `include/sbi/sbi_ipi.h` - IPI接口
- `include/sbi/sbi_timer.h` - 定时器接口
- `include/sbi/sbi_pmu.h` - PMU接口
