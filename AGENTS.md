# AGENTS.md - Agent 工作指南

本文档为 Agent（包括 CodeBuddy、Claude Code、Cursor、GitHub Copilot 等）在本仓库中工作提供指南。

## 项目定位

量潮工作手册 (quanttide-handbook) 是量潮工作手册体系的入口仓库，采用外部链接和子模块混合架构管理多个独立的手册模块。

## 项目结构

```
├── index.md          # 项目简介
├── README.md         # 项目说明
├── _toc.yml          # Jupyter Book 目录配置
├── _config.yml       # Jupyter Book 配置
├── CHANGELOG.md      # 变更记录
├── entity/           # 实体相关子模块
│   ├── company/      # 企业实体手册 (子模块)
│   └── founder/      # 创始人手册 (子模块)
└── AGENTS.md         # 本文件
```

## 子模块管理

### 当前子模块

| 路径 | 仓库 | 说明 |
|------|------|------|
| `entity/company` | quanttide-handbook-of-business-entity | 企业实体手册 |
| `entity/founder` | quanttide-handbook-of-founder | 创始人手册 |

### 子模块操作规则

1. **禁止**：直接在父仓库修改子模块文件
2. **必须**：在子模块仓库独立提交推送，父仓库只更新引用
3. **发布前**：拉取各子模块最新更新 `git submodule update --remote --merge`

### 常用子模块命令

```bash
# 添加子模块
git submodule add <repo-url> <path>

# 拉取所有子模块最新更新
git submodule update --remote --merge

# 查看子模块状态
git submodule status

# 更新父仓库引用
git add <submodule-path>
git commit -m "Update submodule reference"
```

## 关键文档索引

| 文档 | 用途 |
|------|------|
| [README.md](README.md) | 项目说明 |
| [index.md](index.md) | 项目简介 |
| [_toc.yml](_toc.yml) | 目录配置（外部链接为主） |
| [CHANGELOG.md](CHANGELOG.md) | 变更记录 |

## 标准工作流程

```
1. 理解需求
   ↓
2. 检查相关文档（README.md, _toc.yml）
   ↓
3. 执行操作
   ↓
4. 验证构建（如有内容变更）
   ↓
5. 提交推送
```

## 发布前检查清单

- [ ] 拉取子模块最新更新：`git submodule update --remote --merge`
- [ ] 更新父仓库子模块引用
- [ ] 更新 CHANGELOG.md
- [ ] 验证构建无错误

## 相关参考

- 父仓库 AGENTS.md: `/Users/mac/repos/quanttide/AGENTS.md`
