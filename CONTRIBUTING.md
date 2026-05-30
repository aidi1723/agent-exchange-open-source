# Contributing / 贡献指南

## English

Thanks for taking interest in the Trusted A2A Capability Exchange prototype.
This project is intentionally small and experimental. Contributions should keep
that focus: verifiable capability listings, quote-locked checkout, mock escrow,
and clear security boundaries.

### Development Setup

```bash
python3 -m venv .venv
. .venv/bin/activate
pip install '.[dev]'
.venv/bin/python -m unittest discover -s tests -v
```

### Contribution Rules

- Keep changes scoped to one behavior or documentation topic.
- Preserve the prototype security language. Do not describe the verifier as a
  production sandbox.
- Add or update tests when changing runtime behavior.
- Keep API examples synchronized with `src/a2a_exchange/app.py`.
- Prefer simple, readable Python over framework-heavy abstractions.

### Pull Request Checklist

- Tests pass with `.venv/bin/python -m unittest discover -s tests -v`.
- README and language-specific docs are updated when behavior changes.
- New public APIs include request and response examples.
- Security-sensitive changes explain the new trust boundary.

## 中文

感谢关注 Trusted A2A Capability Exchange 原型。本项目刻意保持小而实验性。
贡献应围绕可验证 capability listing、quote 锁定 checkout、mock escrow，以及
清晰的安全边界展开。

### 开发环境

```bash
python3 -m venv .venv
. .venv/bin/activate
pip install '.[dev]'
.venv/bin/python -m unittest discover -s tests -v
```

### 贡献规则

- 每次修改聚焦一个行为或一个文档主题。
- 保持原型安全表述，不要把当前 verifier 描述成生产级沙箱。
- 修改运行时行为时同步新增或更新测试。
- API 示例需要与 `src/a2a_exchange/app.py` 保持一致。
- 优先使用简单、可读的 Python，不引入不必要的复杂抽象。

### Pull Request 检查清单

- `.venv/bin/python -m unittest discover -s tests -v` 通过。
- 行为变化时同步更新 README 和对应语言文档。
- 新增公开 API 时包含请求和响应示例。
- 涉及安全边界的修改需要说明新的信任假设。
