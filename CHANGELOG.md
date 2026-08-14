# CHANGELOG

所有显著变更都将记录在此文件中。

格式基于 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.0.0/)。

版本遵循语义化版本规范：0.0.x（探索期）→ 0.x.y（验证期）→ x.y.z（正式期）

---

## [Unreleased]

### Added

- 新增量潮工作手册总导航页，先上线 5 个一级目录页。
- 新增五个一级目录页：业务、研发、管理、语言/框架/工具、学科/行业。
- 新增 MyST 文档站配置和 qtdocs-site OSS 部署 workflow。

### Changed

- 将研发目录中的 DevOps 手册链接切换到 `docs.quanttide.com/quanttide-handbook/engineering/devops/`。
- 总入口部署不再递归清理 `quanttide-handbook/` 前缀，以保留子手册发布内容。

## [0.0.1] - 2026-03-05

### 探索期

初始化项目结构，建立量潮工作手册基础框架。

- 初始化项目结构
- 添加 `_config.yml` - Jupyter Book 配置
- 添加 `_toc.yml` - 目录配置（外部链接为主）
- 添加 `index.md` - 项目简介
- 添加 `README.md` - 项目说明
- 添加 `CHANGELOG.md` - 变更记录
- 添加 `AGENTS.md` - Agent 工作指南
- 添加子模块：
  - `entity/company` - 企业实体手册 (v0.0.1)
  - `entity/founder` - 创始人手册 (v0.0.1)
