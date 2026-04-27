# ebpf_learn

这个仓库用于对比两种 `execve` 追踪实现路径：

- `execve-tracker`：纯 Rust（Aya）
- `ebpf-execve-counter`：C eBPF + Rust 用户态（libbpf-rs）

目标不是只做一个可用工具，而是把“不同技术栈下的同一问题”放在同一个仓库中，便于学习、实验和横向比较。

## 仓库结构

- `execve-tracker/`：Aya 方案（eBPF + 用户态都用 Rust）
- `ebpf-execve-counter/`：libbpf 方案（eBPF 用 C，用户态用 Rust）

## 两个项目的核心区别

### 1) 语言与工具链

- `execve-tracker`
  - 内核态：Rust（`aya-ebpf`）
  - 用户态：Rust（`aya`）
  - 关键词：Rust-first、单语言体验

- `ebpf-execve-counter`
  - 内核态：C（libbpf 风格 `SEC(...)`）
  - 用户态：Rust（`libbpf-rs`）
  - 关键词：贴近传统 libbpf 生态

### 2) 架构分层

- `execve-tracker` 采用三层结构：
  - `execve-tracker-ebpf`：内核态 eBPF 程序
  - `execve-tracker`：用户态加载与输出
  - `execve-tracker-common`：内核态/用户态共享数据结构（key 定义）

- `ebpf-execve-counter` 采用双层结构：
  - `execve_counter.bpf.c`：内核态 eBPF 程序
  - `src/main.rs`：用户态加载与输出

### 3) 可维护性与学习曲线

- Aya 方案在 Rust 项目里更统一，类型一致性和重构体验更好。
- C + libbpf-rs 方案更贴近大量现有 eBPF 资料与历史项目，便于理解“经典路径”。

### 4) 工程风险点（本仓库实践中的典型）

- 跨语言结构体一致性：C 与 Rust 的 key 布局必须严格一致。
- 挂载点选择：`tracepoint/syscalls/sys_enter_execve` 通常比某些符号级 kprobe/raw tracepoint 更稳。
- 统计读取方式：应遍历 map 真实 key，而不是在用户态“猜 key”。

## 两个项目当前共同能力

- 追踪 `execve` 事件并聚合计数
- 统计维度包含调用者与目标可执行文件
- 支持按用户过滤：`-u/--user`
- 支持窗口统计：`--clear-each-interval`

## 适用场景建议

- 想快速在 Rust 内完成 eBPF 开发：优先看 `execve-tracker`
- 想理解传统 libbpf 工作流或接入历史项目：优先看 `ebpf-execve-counter`
- 想做技术选型：直接对比这两个项目的同类功能实现成本与可维护性

## 后续可扩展方向

- 增加 `execveat` 覆盖
- 增加按 PID/命令名前缀过滤
- 引入 ring buffer 事件流（替代纯 map 轮询）
- 增加统一测试样例与基准脚本
