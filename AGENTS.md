# AGENTS.md - QuantTide Handbook 工作指南

本文档是 `quanttide-handbook` 仓库的工作约定。凡是修改、整理、迁移或上线量潮工作手册内容，都按这里执行。

## 项目定位

- `quanttide-handbook` 是量潮工作手册总入口，使用 MyST 生成站点。
- 目录页负责导航，具体内容写在各本地子页。
- 已迁入本仓库的子手册，不再保留外链跳转样式。
- 以后新增的量潮工作手册内容，默认直接在本仓库新建本地页，不先拆成独立仓库。

## 当前已上线

- `quanttide-handbook` 总入口已上线。
- 量潮工作手册中已经并入本仓库并上线的子页，按现状视为既有基线：
  - `engineering/devops.md`
  - `engineering/data-engineering.md`
  - `management/product-development.md`
  - `management/collaboration.md`
  - `engineering/generative-ai.md`
  - `languages-frameworks-tools/python.md`
  - `engineering/cloud-computing.md`
  - `languages-frameworks-tools/flutter.md`
- 后续新增内容从这批页面之后继续排，不再把它们当成待上线对象。

## 目录结构

- 主配置：`myst.yml`
- 首页：`index.md`
- 一级目录页：`business.md`、`engineering.md`、`management.md`、`languages-frameworks-tools.md`、`disciplines-industries.md`
- 子页面放在对应目录下，例如 `engineering/devops.md`、`engineering/cloud-computing.md`

## 写作准则

- 写成“能照着办”的工作手册，不写成章程。
- 不使用“第X章”“第X条”。
- 标题层级优先围绕：适用范围、工作原则、角色分工、流程、验证、检查清单。
- 技术类内容写场景、前置条件、配置、步骤、验证和故障处理。
- 管理类内容写职责、流程、留痕、归档和例外处理。
- 总入口和目录页只做导航，不堆长解释。

## 上线流程

以后所有要在 `quanttide-handbook` 下上线的内容，都先按这套流程走：

1. 确认所属一级目录和子路径。
2. 确认是新本地页，还是保留外链。
3. 已迁入内容统一改成本地页，去掉跳转链接。
4. 同步更新 `myst.yml` 和对应目录页。
5. 本地构建验证。
6. 提交推送。
7. 补充 `CHANGELOG.md`，如影响全局发布，再同步共享计划。

## 构建与发布

- 本地优先使用 `npx --yes mystmd@1.10.1 build --html`。
- 线上发布通过仓库里的 GitHub Actions 和 `deploy-oss.yml`。
- 如果改了站点根路径，记得同步 `BASE_URL`。
- 不要手工绕过发布流程去改线上页。

## 常见约束

- 不要把已经迁入本仓库的手册恢复成外链。
- 不要只改正文不改 TOC。
- 不要在未确认目录的情况下新增页面。
- 不要一次做大范围无关重排。
- 不要删除已上线目录而不先确认影响。

## 变更前检查

- 页面标题和路径是否匹配。
- `myst.yml` 是否同步。
- 目录页是否同步。
- `CHANGELOG.md` 是否补充。
- 本地构建是否通过。
- 线上路径是否明确。
