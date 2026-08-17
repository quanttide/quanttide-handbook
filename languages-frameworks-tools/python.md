# Python手册

量潮工作手册语言、框架和工具目录下的 Python 子手册，记录 Python 项目的初始化、依赖管理、代码风格、CLI、SDK、缓存、测试、Web 服务和发布约定。Python 可以写脚本，也可以写服务、SDK、数据任务和批处理流程，但都要把环境、依赖、入口、测试和文档说清楚。

## Python 使用场景

- 一次性脚本、命令行工具、后台服务、数据处理任务和批处理流程。
- 可复用 Python 包、SDK、API Client 和内部工具库。
- 需要虚拟环境、依赖锁定、测试和发布记录的 Python 项目。
- 需要被 CI、定时任务、云函数或其他系统稳定调用的执行入口。
- 需要 AI 辅助起草、解释错误、重构模块或补充测试的代码工作。

## 工程约束

- 先固定 Python 版本和环境管理方式，再写代码。
- 建议使用`poetry`管理依赖、构建和发布。
- 依赖必须可复现，不能只停留在本机临时安装状态。
- 入口、参数、配置和错误输出要稳定，方便别人调用和排查。
- 测试覆盖关键路径和边界，再进入发布或自动化调度。
- 代码、配置、文档和运行示例要一起维护。

## 项目初始化

Python 项目进入长期维护前，要先整理项目资产。

常见目录和文件：

| 对象 | 用途 |
|------|------|
| `tests/` | 单元测试，与代码模块一一对应 |
| `integrated_tests/` | 集成测试，与代码模块一一对应 |
| `pyproject.toml` | Python 项目配置文件，类似 Node 的`package.json` |
| `CHANGELOG.md` | 发布版本时记录变更 |
| `__init__.py` | 推荐保留，用于导入信息和包初始化 |
| `__main__.py` | 项目以包形式对外调用或作为命令行入口时使用 |

提交前要确认：

- Python 版本是否明确。
- 依赖管理方式是否明确。
- `pyproject.toml`是否完整。
- 测试目录是否和模块结构对应。
- 入口模块是否清楚。

## 包管理和配置

建议使用`poetry`作为包管理工具。它用于管理项目依赖、构建和发布。

`pyproject.toml`中要关注：

- 编程语言分类选择 Python 3。
- Django 项目需要声明 Django 框架分类。
- 包含文件和排除文件要配置正确，避免`tests`、`example`等目录被错误打包。
- 版本号、包名、作者、许可证和依赖要同步维护。

## 代码风格和命名

Python 代码优先遵循 Python 语言习惯，不为了跨语言一致性牺牲易用性。

类命名中，要特别处理 API、SDK、OSRM、COS、EIAM 等缩写：

- 行业通用概念如 API、SDK 在类名中保留全大写。
- 产品名称或专有缩写如 OSRM、COS、EIAM 使用首字母大写方式。
- 推荐`OsrmAPIClient`，避免`OSRMAPIClient`这种可读性较差的名字。

示例：

```text
QCloudAPIClient
OsrmAPIClient
CosAPIClient
EiamAPIClient
```

对外 API 命名要符合 Python 习惯，使用蛇形命名，而不是强行套用 Java 风格。

## CLI

命令行工具优先使用`typer`作为主要开发框架。

CLI 项目至少要说明：

- 命令入口。
- 参数和选项。
- 帮助信息。
- 退出码。
- 配置文件位置。
- 错误输出。

## Web 服务

Web 服务框架选择：

- 首选`FastAPI`。
- `Django`和`Flask`作为备选框架。

选择时要写清：

- 是否需要 Django 管理后台。
- 是否需要高性能异步 API。
- 是否只是轻量原型。
- 部署方式是云函数、容器、虚拟机还是传统服务器。
- 配置、数据库、缓存和密钥如何注入。

## SDK 和 APIClient

Python SDK 的核心类通常是 APIClient，用于和云 API 通信并处理异常。

基础设计：

1. 提供`BaseAPIClient`类，处理传参、实例初始化和通用请求。
2. 密钥传递和身份认证按 API 机制命名，例如`access_token`。
3. 通用 API 请求接口可命名为`request_api`。
4. 子模块用 Mixin 封装，再组合成用户使用的`APIClient`。

组合方式：

- `BaseAPIClient` + 所有子模块 Mixin = 通用`APIClient`。
- `BaseAPIClient` + 单个 Mixin = 某个子模块的专用`APIClient`。

这样可以减少巨大单类造成的版本管理冲突，也方便多人维护同一类 API。

## 缓存

缓存框架要对比标准库`tempfile`和社区库`diskcache`。

`tempfile`适合：

- 创建安全的临时目录。
- 使用系统临时文件目录。
- 隔离缓存到安全区域。

`diskcache`的优势：

- 成熟。
- Apache 基金会项目。
- 有 Django 兼容特性。
- 底层也使用`tempfile`创建临时文件目录。

因此需要磁盘缓存时，可以优先考虑`diskcache`，并明确缓存目录、清理规则和并发访问风险。

## 测试

建议使用`pytest`或`unittest`进行单元测试。

测试至少覆盖：

- 正常输入。
- 空输入。
- 错误输入。
- 外部依赖失败。
- 环境变量缺失。
- 回归风险。

集成测试放在`integrated_tests/`，用于验证代码模块之间或外部系统之间的协作。

### 环境变量测试

使用`unittest`时，可以用`mock.patch.dict`模拟环境变量：

```python
mock.patch.dict(os.environ, {"<key>": "value"})
```

注意：类级别装饰器会在`setUp`之后运行。如果`setUp`方法里的代码先使用了环境变量，类装饰器可能无效；这种情况下应把装饰器加在`setUp`方法上。

## 典型任务

- 自动化脚本：重点看参数、日志、幂等性和失败重试。
- 命令行工具：重点看 Typer 入口、帮助信息、退出码、配置文件和版本号。
- Web 服务：重点看 FastAPI/Django/Flask 选型、接口契约、错误响应和部署方式。
- SDK：重点看 APIClient、Mixin、鉴权、异常处理和对外命名。
- 数据采集和处理：重点看输入来源、字段口径、质量检查和输出格式。
- 文档转换和批处理：重点看文件编码、路径规则、异常文件和处理报告。
- AI 辅助工作流：重点看提示词、输入脱敏、输出校验和人工确认。

## 代码交付标准

- 有明确的运行入口、参数说明和示例命令。
- 有可复现的环境说明和依赖锁定方式。
- 有`pyproject.toml`和必要的包配置。
- 有`tests/`和必要的`integrated_tests/`。
- 有日志、错误输出或处理报告，便于定位失败。
- 有发布说明、`CHANGELOG.md`和必要的回退方式。
- SDK 类名、API 名和模块划分符合 Python 习惯。

## 提交前检查

- Python 版本是否明确
- `poetry`或其他依赖管理方式是否明确
- `pyproject.toml`是否完整
- `tests/`和`integrated_tests/`是否对应代码模块
- CLI 是否有稳定入口和帮助信息
- Web 服务框架选择是否写清
- APIClient、Mixin 和鉴权设计是否清楚
- 缓存目录和清理规则是否明确
- 发布是否可回退
- 文档是否同步
