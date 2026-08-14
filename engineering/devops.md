# DevOps手册

量潮工作手册研发目录下的 DevOps 子手册，聚焦版本、协作、构建、测试、发布、部署、运维和监控。

## 适用范围

- 适用于需要用 `qtcloud-devops` 或标准 CI/CD 流程管理代码、版本和制品的项目。
- 适用于需要把配置文件、CHANGELOG、tag、CI 运行记录统一对齐的场景。

## 基本原则

- 版本、配置、CHANGELOG 和 tag 保持同源。
- 发布前先审计，发布后再验收。
- 能自动化的步骤优先走 CLI 或 CI。
- 所有关键动作都要留痕。

## 契约

DevOps 契约用四个维度描述治理对象：

- `stages`：计划、开发、构建、测试、发布、部署、运维、监控
- `platforms`：GitHub、CI/CD、制品库等载体
- `sources`：版本事实源
- `scopes`：组件或目录级作用范围

## 生命周期

### 计划

明确范围、版本、依赖、验收口径和负责人。

### 开发

先看同步状态，再同步子模块，最后复核状态。

### 构建

确认 CI、依赖来源和产物一致。

### 测试

确认测试通过数、失败数和覆盖率阈值。

### 发布

以 tag 为版本事实源，先审计后发布。

### 部署

将已发布版本推到目标环境并核对可访问性。

### 运维

处理配置、巡检、回滚和故障恢复。

### 监控

持续查看日志、指标和告警。

## 常用命令

```bash
qtcloud-devops code status
qtcloud-devops build status
qtcloud-devops test status
qtcloud-devops release status
qtcloud-devops release audit -v cli/v0.3.2 --scope cli
```

## 发布判断

发布是否成立，不看命令是否跑完，而看版本、tag、Release、制品和审计证据是否一致。
