# OpenClaw DevOps Organizational Memory

## 1. 目标

本项目目标是把 OpenClaw 从一个 AI Chat / Agent Gateway，升级为 DevOps AI Employee 的组织记忆入口。

核心不是让 ChatGPT 每次重新读取整个 GitHub Repository，而是通过 Repository Index 和 DevOps Organizational Memory，把长期积累的架构、Runbook、Incident、RCA、Pattern 沉淀下来。

最终结构：

```text
GitHub
     │
     ▼
Repository Index
     │
     ├── Architecture Memory
     ├── Runbook Memory
     ├── Incident Memory
     ├── RCA Memory
     └── Pattern Memory
              │
              ▼
         OpenClaw
              │
              ▼
          ChatGPT
```

## 2. 为什么要这样做

### 2.1 不直接让 ChatGPT 读整个仓库

直接把整个仓库交给 LLM 会导致：

- Token 成本高。
- 上下文噪声大。
- 每次分析重复理解项目。
- 老问题无法自然沉淀。
- 不同公司、不同系统的经验难以复用。

正确做法是：

1. GitHub 负责代码和变更源。
2. Repository Index 负责仓库结构、文件摘要、依赖关系、服务边界。
3. Organizational Memory 负责长期知识沉淀。
4. OpenClaw 负责检索、路由、工具调用和上下文压缩。
5. ChatGPT 只接收当前任务真正相关的最小上下文。

### 2.2 真正的护城河

别人可以复制代码，但很难复制一个组织长期运行中形成的：

- 架构演进记录。
- 故障处理经验。
- 事故 RCA。
- 运维决策。
- 已验证的修复 Pattern。
- 公司内部服务关系。
- 系统约束和禁忌操作。

这就是 DevOps Organizational Memory。

## 3. 模块划分

## 3.1 Repository Index

Repository Index 是 GitHub 仓库的结构化索引层。

### 输入

- Repository 文件树。
- README / docs。
- Dockerfile / docker-compose。
- Kubernetes manifests。
- Helm charts。
- Terraform / Ansible。
- CI/CD workflows。
- package / go.mod / pom.xml / requirements.txt。
- 最近 PR diff。
- Issue / PR metadata。

### 输出

```json
{
  "repo": "owner/name",
  "default_branch": "master",
  "services": [],
  "entrypoints": [],
  "deployments": [],
  "dependencies": [],
  "ci_cd": [],
  "observability": [],
  "risk_files": []
}
```

### 要求

- 首次全量索引。
- 后续基于 commit diff 增量更新。
- 文件级摘要缓存。
- 服务级摘要缓存。
- 支持按关键词、服务名、错误信息、文件路径检索。

## 3.2 Architecture Memory

Architecture Memory 记录系统结构。

### 内容

- 服务列表。
- 服务依赖关系。
- 数据流。
- 部署方式。
- 入口流量路径。
- 中间件依赖。
- 关键配置。
- 高风险组件。

### 示例

```yaml
service: payment-api
repo: company/payment
runtime: go
entrypoint: cmd/api/main.go
deploy:
  type: helm
  namespace: prod-payment
dependencies:
  - mysql
  - redis
  - kafka
observability:
  metrics: prometheus
  logs: elasticsearch
  alerts: nightingale
risk_notes:
  - order status update must be idempotent
  - kafka consumer lag can affect settlement delay
```

## 3.3 Runbook Memory

Runbook Memory 记录可执行的处理手册。

### 内容

- 告警名称。
- 判断步骤。
- 查询命令。
- 修复步骤。
- 回滚步骤。
- 风险提示。
- 验证方式。

### 示例

```yaml
alert: KubernetesPodCrashLoopBackOff
symptoms:
  - pod restart count increasing
  - previous container exit code exists
checks:
  - kubectl describe pod
  - kubectl logs --previous
  - check recent deployment diff
fix:
  - rollback deployment if new version caused crash
  - increase memory only if OOMKilled is confirmed
verify:
  - restart count stops increasing
  - readiness probe becomes healthy
```

## 3.4 Incident Memory

Incident Memory 记录每次事故事实。

### 内容

- incident_id。
- 时间。
- 服务。
- 环境。
- 影响范围。
- 告警。
- 证据。
- 操作记录。
- 关联 PR / Issue / Commit。

### 示例

```yaml
incident_id: INC-2026-0001
service: logstash-prod
environment: production
start_time: 2026-06-22T10:00:00+08:00
symptom: kafka consumer lag increasing
evidence:
  - logstash restarted several times
  - elasticsearch bulk timeout
  - master cpu spike during warm forcemerge
linked_components:
  - kafka
  - logstash
  - elasticsearch
```

## 3.5 RCA Memory

RCA Memory 记录事故根因分析。

### 内容

- Root Cause。
- Contributing Factors。
- Evidence。
- What Worked。
- What Failed。
- Corrective Actions。
- Preventive Actions。

### 示例

```yaml
rca_id: RCA-2026-0001
incident_id: INC-2026-0001
root_cause: Elasticsearch warm forcemerge caused periodic master CPU spikes, which contributed to Logstash bulk timeout and Kafka lag accumulation.
evidence:
  - lag increased during Logstash downtime
  - ES bulk timeout repeated during warm phase
  - master node CPU spiked after forcemerge completed
actions:
  - delay warm forcemerge
  - reduce concurrent heavy ILM actions
  - monitor Logstash bulk failure rate
```

## 3.6 Pattern Memory

Pattern Memory 是最重要的复用资产。

它不是记录单个事故，而是把多个相似事故抽象成可复用模式。

### 内容

- Pattern 名称。
- 触发条件。
- 典型症状。
- 关键证据。
- 常见误判。
- 推荐处理。
- 自动化检测规则。

### 示例

```yaml
pattern: ES Warm Forcemerge Causes Ingest Backpressure
trigger:
  - ILM warm phase starts
  - forcemerge running
symptoms:
  - Logstash bulk timeout
  - Kafka lag increasing
  - ES master CPU short spike
common_misdiagnosis:
  - assume Kafka is the root cause
  - assume Logstash config is wrong
recommended_actions:
  - check ES ILM explain
  - check master CPU and cluster state updates
  - reduce or delay forcemerge
automation:
  detector: correlate kafka lag + logstash bulk timeout + ES ILM phase
```

## 4. OpenClaw 集成方式

## 4.1 OpenClaw 角色

OpenClaw 在这个系统里不是简单聊天入口，而是：

- GitHub Token 管理入口。
- Repository Reader。
- PR Reader。
- Issue Reader。
- File Reader。
- MCP Server。
- Memory Retriever。
- Context Compressor。
- Tool Router。

## 4.2 MCP Server 能力

建议暴露以下 MCP tools：

```text
repo.index
repo.search
repo.file.read
repo.pr.read
repo.issue.read
memory.architecture.search
memory.runbook.search
memory.incident.search
memory.rca.search
memory.pattern.search
memory.write.proposal
```

注意：

- search 工具默认只返回摘要和引用，不返回全量文件。
- file.read 必须按需读取。
- memory.write.proposal 只生成写入建议，不直接写长期记忆。
- 长期记忆写入需要人工审核。

## 5. Token 控制策略

### 5.1 禁止默认全仓库读取

默认流程：

```text
User Question
  ↓
OpenClaw Search
  ↓
Top-K Relevant Memory / Files
  ↓
Context Compress
  ↓
ChatGPT
```

不要：

```text
User Question
  ↓
Read Whole Repository
  ↓
ChatGPT
```

### 5.2 上下文预算

建议预算：

| 场景 | Token 预算 |
|---|---:|
| Issue 分析 | 2k - 6k |
| PR Review | 4k - 12k |
| 单服务故障分析 | 6k - 16k |
| 跨服务 RCA | 12k - 32k |
| 全仓库架构总结 | 离线任务，不进入实时对话 |

### 5.3 分层检索

优先级：

1. Pattern Memory。
2. RCA Memory。
3. Runbook Memory。
4. Architecture Memory。
5. Incident Memory。
6. Repository Index。
7. File Reader。

只有前面无法回答时，才读取源码文件。

## 6. 数据目录建议

```text
.memory/
  architecture/
  runbooks/
  incidents/
  rca/
  patterns/

.index/
  repos/
  files/
  services/
  commits/

.mcp/
  tools/
  schemas/
  policies/

docs/
  openclaw-devops-organizational-memory.md
```

## 7. 最小版本 v0.1

### v0.1 目标

先不要做大而全的平台，先完成：

1. 对一个 GitHub Repository 建立 Repository Index。
2. 支持按问题搜索相关文件和 Memory。
3. 支持读取 PR diff / Issue 内容。
4. 生成 RCA 草案。
5. 将 RCA 草案保存为待审核 Markdown。

### v0.1 不做

- 不自动合并 PR。
- 不自动改生产配置。
- 不自动写长期记忆。
- 不直接读取全仓库进入 ChatGPT。
- 不把 GitHub Token 暴露给 ChatGPT。

## 8. v0.1 任务拆解

### Epic 1: Repository Index

- [ ] 扫描仓库文件树。
- [ ] 识别技术栈和服务入口。
- [ ] 生成文件级摘要。
- [ ] 生成服务级摘要。
- [ ] 支持增量更新。

### Epic 2: Memory Schema

- [ ] 定义 Architecture Memory schema。
- [ ] 定义 Runbook Memory schema。
- [ ] 定义 Incident Memory schema。
- [ ] 定义 RCA Memory schema。
- [ ] 定义 Pattern Memory schema。

### Epic 3: MCP Tools

- [ ] repo.index。
- [ ] repo.search。
- [ ] repo.file.read。
- [ ] repo.pr.read。
- [ ] repo.issue.read。
- [ ] memory.*.search。
- [ ] memory.write.proposal。

### Epic 4: OpenClaw Workflow

- [ ] Issue -> search memory -> read related files -> RCA draft。
- [ ] PR -> read diff -> search architecture -> review comments。
- [ ] Incident -> collect evidence -> search patterns -> RCA draft。

### Epic 5: Governance

- [ ] Token budget policy。
- [ ] Tool permission policy。
- [ ] Memory write approval policy。
- [ ] Audit log policy。

## 9. 成功标准

v0.1 成功标准：

- 能回答“这个告警/Issue 可能和哪些文件、服务、历史事故有关”。
- 能生成有证据链的 RCA 草案。
- 每次 ChatGPT 调用只输入相关上下文，而不是整个仓库。
- Memory 可以随着事故和 RCA 增长。
- 人工确认后，Pattern Memory 可以沉淀为长期资产。

## 10. 一句话定位

OpenClaw + GitHub + Organizational Memory 的核心价值：

> 让 DevOps AI Employee 不再每次从零开始，而是基于组织长期积累的架构、事故、RCA、Runbook、Pattern 来持续工作。
