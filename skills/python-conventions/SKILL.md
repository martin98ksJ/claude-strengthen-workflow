---
name: python-conventions
description: Python 项目的命名、类型提示、异常处理、异步、项目结构和测试规范
version: "1.0"
---

> 通用检查项见 ~/.claude/skills/_common/quality-checklist.md
> 开发实践基线见 ~/.claude/skills/_common/dev-practices.md
> 输出格式遵循 CLAUDE.md
> 通用后端架构规范见 backend-conventions

## 输入

- Python 源码文件
- 涉及的模块或功能描述

## 职责（必须做）

### 命名与风格

- [ ] 模块/包名小写下划线（`user_service.py`），类名大驼峰（`UserService`），函数/变量蛇形（`get_user_by_id`）
- [ ] 常量 UPPER_SNAKE（`MAX_RETRY_COUNT`），私有用单下划线前缀（`_internal_method`），name mangling 用双下划线（极少用）
- [ ] 布尔变量/函数用 `is_/has_/can_/should_` 前缀（`is_active`、`has_permission`）
- [ ] 遵循 PEP 8，行宽 88（Black 默认）或 79（PEP 8 原始），用 formatter 统一（Black/Ruff）

### 类型提示

- [ ] 函数签名必须有类型注解（参数 + 返回值），`-> None` 不省略
- [ ] 复杂类型用 `typing` 模块：`list[str]`（3.9+）、`Optional[str]`（可能 None）、`Union[int, str]`
- [ ] 数据类优先 `dataclass` 或 `pydantic.BaseModel`（需校验时），不用裸 dict 传递结构化数据
- [ ] 启用 mypy/pyright 静态检查，CI 中运行类型检查，不用 `# type: ignore` 绕过（除非有注释说明原因）

### 异常处理

- [ ] 捕获具体异常类型（`except ValueError`），不用裸 `except:` 或 `except Exception`（除非顶层兜底）
- [ ] 自定义业务异常继承 `Exception`，携带错误码和上下文信息
- [ ] 异常链保留：`raise NewError("msg") from original_error`，不吞掉原始异常
- [ ] 资源清理用 `with` 语句（文件/连接/锁），不手动 try-finally
- [ ] FastAPI/Flask 用全局异常处理器统一转换为 HTTP 响应，handler 层不 try-except 业务异常

### 项目结构

- [ ] `src/` 布局（`src/myapp/`）或扁平布局（`myapp/`），入口 `__main__.py` 或 `main.py`
- [ ] 按功能域划分模块（`user/`、`order/`），每个模块内 `router.py`/`service.py`/`model.py`/`schema.py`
- [ ] 依赖管理：`pyproject.toml`（推荐）或 `requirements.txt`，锁文件（`uv.lock`/`poetry.lock`）提交到仓库
- [ ] 虚拟环境隔离：`uv`/`poetry`/`venv`，不全局安装项目依赖

### 异步编程

- [ ] IO 密集用 `async/await`（FastAPI/aiohttp），CPU 密集用 `multiprocessing` 或 `concurrent.futures`
- [ ] 异步函数内不调用同步阻塞操作（`time.sleep`/同步 HTTP），用 `asyncio.sleep`/`httpx.AsyncClient`
- [ ] 异步上下文管理：`async with`（数据库连接/HTTP session），确保资源释放
- [ ] 并发控制：`asyncio.Semaphore` 限制并发数，`asyncio.gather` 并行执行无依赖任务
- [ ] 不混用同步和异步：选定一种模式贯穿整个请求链路

### 数据校验与序列化

- [ ] API 入参用 Pydantic model 校验（FastAPI 自动集成），字段加 `Field(...)` 约束（min/max/regex）
- [ ] 响应模型用 `response_model` 声明，自动过滤多余字段，不返回 ORM 对象给前端
- [ ] 日期时间用 `datetime` 类型，序列化格式统一 `%Y-%m-%d %H:%M:%S`，时区处理用 `zoneinfo`（3.9+）
- [ ] 枚举用 `enum.Enum` 或 `StrEnum`（3.11+），不用魔法字符串

### 测试

- [ ] pytest 为主，测试文件 `test_<module>.py`，函数名 `test_<function>_<scenario>`
- [ ] fixture 管理测试数据和依赖注入，`conftest.py` 放共享 fixture
- [ ] mock 用 `unittest.mock.patch` 或 `pytest-mock`，只 mock 外部依赖（DB/HTTP/文件）
- [ ] 异步测试用 `pytest-asyncio`，标记 `@pytest.mark.asyncio`

### 性能

- [ ] 大数据处理用生成器（`yield`）而非列表，避免一次性加载到内存
- [ ] 字符串拼接用 f-string 或 `"".join()`，循环内不用 `+=` 拼接
- [ ] 热路径避免重复计算：`functools.lru_cache`/`functools.cache` 缓存纯函数结果
- [ ] 数据库批量操作用 `bulk_create`/`executemany`，不循环单条插入

### 常见坑（AI 生成代码高频踩雷）

- [ ] 可变默认参数：`def foo(items=[])` 所有调用共享同一个 list，追加会累积；用 `items=None` + 函数内 `items = items or []`
- [ ] 循环变量闭包：`[lambda: i for i in range(5)]` 全部返回 4，闭包捕获变量引用非值；用 `lambda i=i: i` 或列表推导
- [ ] is vs ==：`is` 比较身份（id），`==` 比较值；`a is None` 正确，`a is 256` 不可靠（CPython 小整数缓存 -5~256）
- [ ] 浮点精度：`0.1 + 0.2 != 0.3`；金额计算用 `decimal.Decimal`，比较用 `math.isclose()`
- [ ] 字典遍历时修改：`for k in d: del d[k]` 抛 `RuntimeError`；用 `for k in list(d.keys())` 或构建新字典
- [ ] except 吞异常：`except: pass` 吞掉所有异常包括 `KeyboardInterrupt`/`SystemExit`；至少 `except Exception` + 日志记录
- [ ] 全局变量与 import 循环：模块 A import B，B 又 import A，导致 `ImportError` 或拿到未初始化的模块；用延迟 import 或重构依赖
- [ ] async 阻塞事件循环：在 async 函数中调用 `requests.get()`/`time.sleep()` 阻塞整个事件循环；用 `httpx.AsyncClient`/`asyncio.sleep`
- [ ] 深浅拷贝：`list.copy()`/`dict.copy()` 是浅拷贝，嵌套对象仍共享引用；需要深拷贝用 `copy.deepcopy()`
- [ ] f-string 与日志：`logging.info(f"user {user_id}")` 即使日志级别不够也会执行格式化；用 `logging.info("user %s", user_id)` 延迟求值

## 边界（不做什么）

- 不涉及通用后端架构（分层、日志、安全 → 见 backend-conventions）
- 不涉及具体 Web 框架用法（FastAPI/Django/Flask → 按项目约定）
- 不涉及数据科学/ML 库（NumPy/Pandas/PyTorch → 按项目约定）
- 不涉及部署和环境配置（→ 见 env-strategy、docker-deploy）

## 输出

输出格式遵循 CLAUDE.md，通用检查项见 _common/quality-checklist.md。
