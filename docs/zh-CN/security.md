# 安全说明

本项目**不能直接用于公开执行不可信代码**。

当前 verifier 适合在本地实验信任模型，但不是生产级沙箱。应把所有提交的
artifact 都视为危险输入。

## 当前边界

Verifier 会：

- 将提交的 Python 源码字符串写入临时文件。
- 在子进程中执行该文件。
- 要求存在可调用的 `run(input)` 函数。
- 调用 `run(input)` 时重定向 artifact 的 stdout 和 stderr。
- 对每个 eval case 执行超时限制。
- 要求返回值是 JSON object。
- 将返回 JSON 与 eval case 的 `expected_output` 做精确比较。

## 当前不会强制保证的内容

当前实现不提供硬化隔离：

- `permission_policy.network` 和 `permission_policy.filesystem` 是声明式
  元数据，不是操作系统层面的强制控制。
- `sandbox_policy.network` 目前也是 policy 元数据，不是网络隔离。
- 子进程没有运行在 container、jail 或虚拟化环境中。
- CPU、内存、文件系统、import、环境变量访问和进程创建没有被完整限制。
- 没有认证或授权。
- Mock credit 和 mock escrow 不是金融控制。

## 安全使用建议

- 只在本地机器或隔离测试网络运行。
- 不要把 API 暴露到公网。
- 不要在包含敏感文件、凭证或网络访问权限的机器上提交未知来源 artifact。
- 测试未知 artifact 时使用一次性环境。

## 生产化加固方向

生产设计至少需要：

- Container、WASM、microVM 或同等级别的硬化执行隔离。
- 默认拒绝的网络和文件系统控制。
- CPU、内存、磁盘、进程数量和运行时间资源配额。
- 已认证的卖家和买家身份。
- 持久审计日志。
- 真实支付和结算集成。
- 恶意代码扫描，以及人工或自动治理流程。

在这些能力完成之前，本仓库应仅作为研究原型使用。
