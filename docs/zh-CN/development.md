# 开发手册

本文说明如何运行、测试和扩展 Trusted A2A Capability Exchange 原型。

## 环境要求

- Python 3.9 或更新版本。
- 支持 PEP 517 构建的 `pip`。
- 本地机器或隔离网络。不要把当前原型暴露给公网执行不可信代码。

## 安装

```bash
git clone https://github.com/aidi1723/agent-exchange-open-source.git
cd agent-exchange-open-source
python3 -m venv .venv
. .venv/bin/activate
pip install '.[dev]'
```

如果旧版 `pip` 不支持 editable install，可以使用上面的非 editable 安装方式，
或在虚拟环境中升级 pip。

## 运行 API

```bash
python -m a2a_exchange
```

应用会通过 Uvicorn 启动在：

```text
http://127.0.0.1:8000
```

FastAPI 交互式文档地址：

```text
http://127.0.0.1:8000/docs
```

## 运行测试

```bash
.venv/bin/python -m unittest discover -s tests -v
```

当前测试覆盖注册、失败验证、不暴露 artifact 的 discovery、quote 锁定、
checkout、mock credit 扣款、escrow settlement，以及后续注册不影响已有 quote。

## 项目结构

```text
src/a2a_exchange/
  manifest.py    manifest、policy、scorecard、listing 的 Pydantic 模型
  eval_pack.py   可复放 eval case 和 eval pack 模型
  verifier.py    基于子进程的 Python artifact 验证器
  registry.py    线程安全的内存 capability registry
  discovery.py   不暴露 artifact 的 discovery 过滤器
  quote.py       锁定版本的一次性 quote book
  escrow.py      mock escrow 状态机
  credit.py      mock 静态买家 credit guard
  app.py         FastAPI HTTP 接口和 app factory
  __main__.py    Uvicorn 启动入口
tests/
  test_exchange.py   端到端可信交易流程测试
docs/
  en/          英文手册
  zh-CN/       中文手册
```

## 运行时模型

应用默认使用内存状态：

- `CapabilityRegistry` 保存提交的 capability record。
- `CapabilityDiscovery` 返回不含 artifact 的 listing 元数据。
- `QuoteBook` 创建短期、一次性的 quote，并快照 price、artifact hash 和 scorecard。
- `MockCreditGuard` 为每个 buyer 提供静态 mock token 余额。
- `EscrowBook` 记录 mock settlement 状态。

进程重启会清空 registry、quote、balance 和 escrow 状态。

## Capability 合约

Artifact 是 Python 源码字符串。有效 artifact 必须定义：

```python
def run(input):
    return {"result": "json-compatible object"}
```

Verifier 要求 `run(input)` 返回 JSON object，对应 Python 中的 `dict`。
Eval case 的输出会和 `expected_output` 做精确相等比较。

## 开发约定

- 公共行为需要由 `tests/test_exchange.py` 或新的聚焦测试文件覆盖。
- `src/a2a_exchange/app.py` 中的 API schema 需要和 `docs/en/api.md`、
  `docs/zh-CN/api.md` 同步。
- 除非真的加入硬化沙箱，否则不要改变当前安全警告。
- 没有单独设计前，不要直接加入持久化存储、认证或真实支付。

## 常用命令

```bash
# 运行全部测试
.venv/bin/python -m unittest discover -s tests -v

# 启动 API
.venv/bin/python -m a2a_exchange

# 查看 git 状态
git status --short --branch
```
