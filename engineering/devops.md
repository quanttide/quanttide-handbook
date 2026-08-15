# DevOps手册

量潮工作手册研发目录下的 DevOps 子手册，讲的是怎么把一次发布从“把代码推上去”变成有版本、有审计、有制品、有回退、有验收的工程动作。DevOps 关注的不只是 CI 是否变绿，还要确认版本、配置、CHANGELOG、tag、制品和线上结果是否互相对应。

## 发布治理边界

- 使用 `qtcloud-devops`、GitHub Actions 或类似 CI/CD 流程管理项目。
- 需要把版本号、配置文件、CHANGELOG、tag、构建产物和部署结果统一起来。
- 多个 scope 共存，需要分别构建、分别发布、分别审计。
- 发布失败会影响线上访问、包分发、文档站点或团队协作节奏。

## 流水线治理原则

- 版本、配置、CHANGELOG、tag 和制品必须互相对得上。
- 自动化优先，人工只做例外处理，例外要写明原因。
- 每个关键动作都要留痕，包括审计、构建、发布、部署和验收。
- 发布前先审计，发布后再验收，失败后先定位原因再重跑。
- scope 级别的差异要显式写出来，不靠口头约定。
- 能用标准发布工具时，不手工绕过版本、tag 或制品流程。

## 四维契约

DevOps 契约用四个维度描述治理对象。

### `stages`

`stages` 定义时序维度，也就是每一步在什么时候检查什么。

- `build` 负责构建
- `test` 负责测试与覆盖率门槛
- `release` 负责发布前准备和发布动作

常见字段：

- `command`：执行命令
- `threshold`：测试覆盖率阈值
- `changelog`：CHANGELOG 路径
- `pre_publish`：发布前命令列表

### `platforms`

`platforms` 定义载体维度，也就是这些动作跑在哪些平台上。

- `source_control`：源代码管理平台
- `pipeline`：CI/CD 平台
- `artifact_registry`：制品库

### `sources`

`sources` 定义事实源维度，也就是版本号从哪里读。

- `auto`：自动识别
- `cargo`：从 `Cargo.toml` 读取
- `pyproject`：从 `pyproject.toml` 读取
- `pubspec`：从 `pubspec.yaml` 读取
- `package.json`：从 `package.json` 读取
- `tagOnly`：只认 git tag

### `scopes`

`scopes` 定义上下文维度，也就是同一仓库里不同组件怎么分别治理。

常见字段：

- `dir`：组件目录
- `language`：语言
- `framework`：框架
- `build_tool`：构建工具
- `registry`：制品库
- `release`：该 scope 的发布配置
- `test_threshold`：该 scope 的测试阈值
- `ci_workflow`：该 scope 对应的 CI 文件

## 发布责任角色

- 发布负责人确认发布意图、版本、范围、窗口和回退条件。
- 开发者负责代码、配置、测试、文档和变更说明。
- CI 负责跑构建、测试、打包、发布和部署流水线。
- 审计人负责核对事实源、发布目标、权限和证据链。
- AI 负责对齐事实源、提示缺口、整理步骤和草拟发布记录。

## DevOps 生命周期

### 计划

计划阶段先把边界定清楚。发布计划要能回答发什么、不发什么、失败了怎么退。

- 明确要做什么，不做什么
- 明确 scope、版本、依赖和验收口径
- 明确负责人和回退方案
- 明确是否需要分组件发布

输出物：

- 版本计划
- scope 列表
- 变更范围
- 验收标准

### 开发

开发阶段先看同步状态，再改内容。没有确认工作区和依赖状态时，不直接进入发布准备。

- 先检查子模块或依赖仓库是否同步
- 改完后再检查一次状态
- 需要的话同步远端最新提交
- 避免在未确认状态下直接往下游推

常见命令：

```bash
qtcloud-devops code status
qtcloud-devops code sync
qtcloud-devops code sync <name>
```

### 构建

构建阶段确认代码能生成目标产物。构建命令、依赖来源和产物路径要能在 CI 中复现。

- 检查构建命令
- 检查依赖来源
- 检查产物路径
- 检查 CI 是否能稳定复现

常见命令：

```bash
qtcloud-devops build status
```

### 测试

测试阶段确认产物不只是能编，还要能用。测试结果要和发布 scope 对齐。

- 看测试通过数、失败数和跳过数
- 看覆盖率
- 看阈值是否达标
- 看是否存在 scope 级别差异

常见命令：

```bash
qtcloud-devops test status
```

### 发布

发布阶段把已经通过构建和测试的提交变成可被获取的版本。发布动作必须能回到具体 tag、制品和记录。

- 先确认版本号和 tag 是否一致
- 再确认 CHANGELOG 是否有对应条目
- 再确认目标平台和制品库
- 最后才执行发布

常见命令：

```bash
qtcloud-devops release status
qtcloud-devops release audit -v cli/v0.3.2 --scope cli
qtcloud-devops release publish -v cli/v0.3.2-rc.1 --registry crates -y
qtcloud-devops release publish -v cli/v0.3.2 --registry crates -y
qtcloud-devops release publish -v cli/v0.3.2 -y --local
```

### 部署

部署阶段把已发布版本送到目标环境。

- 检查目标环境是否已配置好
- 检查是否需要清理旧版本
- 检查资源路径和访问地址
- 检查部署后是否能打开

### 运维

运维阶段处理日常运行中的问题。

- 处理配置变更
- 处理回滚
- 处理故障恢复
- 处理发布后的补救动作

### 监控

监控阶段持续看运行是否健康。

- 看日志
- 看指标
- 看告警
- 看发布后是否有异常回退

## 发布规则

### 版本与标签

版本事实源以 git tag 为准。常见写法：

- `vX.Y.Z`
- `vX.Y.Z-prerelease`
- `scope/vX.Y.Z`
- `scope/vX.Y.Z-prerelease`

发布前要确认：

- 配置文件版本
- CHANGELOG 条目
- tag 名称
- scope 名称

### 发布目标

发布目标由契约、scope、CI 和版本计划共同决定。

| 目标 | 常见制品 |
|------|----------|
| GitHub Release | release notes、源码归档、二进制包 |
| crates.io | Rust crate |
| PyPI | Python package |
| npm | JavaScript / TypeScript package |
| pub.dev | Dart / Flutter package |
| 容器镜像仓库 | Docker / OCI image |
| 内部制品库 | 内部二进制、数据包、模型、文档包 |

### CLI 发布

接入 `qtcloud-devops` 后，优先走 CLI 发布。

```bash
qtcloud-devops build status
qtcloud-devops test status
qtcloud-devops release status
qtcloud-devops release audit -v cli/v0.3.2 --scope cli
qtcloud-devops release publish -v cli/v0.3.2 --registry crates -y
```

CLI 发布通常包含：

1. 校验版本号和 tag
2. 校验 scope、配置和 CHANGELOG
3. 检查工作区状态
4. 创建并推送 tag
5. 触发发布 CI
6. 由 CI 或 CLI 完成制品发布
7. 执行发布后审计

### 手动操作

手动发布只在工具不可用、项目未接入自动化或负责人明确批准时使用。

手动最小流程：

1. 跑发布前审计
2. 从约定分支或提交发布
3. 创建并推送 tag
4. 创建 Release
5. 上传制品
6. 跑发布后审计

常见手动命令：

```bash
git tag cli/v0.3.2
git push origin cli/v0.3.2
gh release create cli/v0.3.2 --title "cli/v0.3.2" --notes-file release-notes.md
```

### 发布后处理

发布完成后，不能只看命令有没有结束，要看结果是否闭合。

- tag 已推送
- Release 可访问
- 制品版本可查询
- 下载或安装可用
- CI/CD release job 成功

## 发布审计

发布审计要回答两个问题：现在能不能发，发完是不是真的成功了。

### 审计对象

- 发布范围
- 版本事实
- 代码状态
- 构建与测试
- 安全
- 发布目标
- 证据

### 自动审计

自动审计优先看工具和平台返回值。

```bash
qtcloud-devops release status
qtcloud-devops release audit -v cli/v0.3.2 --scope cli
qtcloud-devops build status
qtcloud-devops test status
gh release view cli/v0.3.2 --json tagName,url,assets
gh run view <run-id> --json status,conclusion,url
```

### 手动审计

手动审计用来补足工具暂时看不到的部分。

- 版本格式是否正确
- scope 与目录是否一致
- CHANGELOG 是否写的是用户可感知变化
- 工作区是否干净
- 发布凭证是否放在受控环境里
- 发布目标是否列全

### 审计结论格式

审计结论最好包含：

- 发布对象
- 发布路径
- 发布目标
- 发布前结果
- 发布后结果
- 后续处理

## 发布配置模板

```yaml
stages:
  build:
    command: cargo build --release
  test:
    command: cargo test
    threshold: 80.0
  release:
    changelog: CHANGELOG.md
    pre_publish:
      - cargo publish

platforms:
  source_control: github
  pipeline: github_actions
  artifact_registry: crates

sources:
  version:
    type: auto

scopes:
  cli:
    dir: src/cli
    language: rust
    build_tool: cargo
    registry: crates
    release:
      changelog: CHANGELOG.md
      pre_publish:
        - cargo publish
    test_threshold: 90.0
```

## 发布前检查

- 版本号是否和 tag 对上
- CHANGELOG 是否补齐
- 构建和测试是否通过
- 发布目标是否明确
- 审计证据是否保留
