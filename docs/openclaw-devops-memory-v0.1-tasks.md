# OpenClaw DevOps Organizational Memory v0.1 Tasks

## 背景

目标是把 OpenClaw 接入 GitHub，形成 DevOps AI Employee 的组织记忆底座。

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

## 核心原则

- GitHub API / Reader 不烧 token，真正烧 token 的是把代码送给 LLM。
- 禁止默认全仓库读取。
- 先 Repository Index，再 Memory Search，最后才按需 File Reader。
- 长期资产不是代码本身，而是 Architecture / Runbook / Incident / RCA / Pattern。
- Memory 写入必须人工审核，不能让 AI 自动污染长期记忆。

## Phase 1: Repository Index

- [ ] 扫描仓库文件树。
- [ ] 识别服务入口。
- [ ] 识别 Docker / Compose / Helm / Kubernetes / Terraform / Ansible。
- [ ] 识别 CI/CD workflow。
- [ ] 识别监控与告警配置。
- [ ] 生成文件级摘要。
- [ ] 生成服务级摘要。
- [ ] 支持 commit diff 增量更新。

## Phase 2: Memory Schema

- [ ] Architecture Memory schema。
- [ ] Runbook Memory schema。
- [ ] Incident Memory schema。
- [ ] RCA Memory schema。
- [ ] Pattern Memory schema。
- [ ] Memory metadata schema：source、confidence、last_verified_at、owner、related_files。

## Phase 3: MCP Tools

- [ ] `repo.index`
- [ ] `repo.search`
- [ ] `repo.file.read`
- [ ] `repo.pr.read`
- [ ] `repo.issue.read`
- [ ] `memory.architecture.search`
- [ ] `memory.runbook.search`
- [ ] `memory.incident.search`
- [ ] `memory.rca.search`
- [ ] `memory.pattern.search`
- [ ] `memory.write.proposal`

## Phase 4: OpenClaw Workflow

- [ ] Issue -> Search Memory -> Read Related Files -> RCA Draft。
- [ ] PR -> Read Diff -> Search Architecture -> Review Suggestion。
- [ ] Incident -> Collect Evidence -> Search Pattern -> RCA Draft。
- [ ] RCA Draft -> Human Review -> Pattern Memory Proposal。

## Phase 5: Token Budget Policy

- [ ] Issue analysis: 2k - 6k tokens。
- [ ] PR review: 4k - 12k tokens。
- [ ] Single service incident: 6k - 16k tokens。
- [ ] Cross-service RCA: 12k - 32k tokens。
- [ ] Whole repository architecture summary: offline task only。

## Phase 6: Governance

- [ ] GitHub Token 不进入 ChatGPT 上下文。
- [ ] File Reader 只按需读取相关文件。
- [ ] Memory write 只生成 proposal，不直接写长期记忆。
- [ ] Pattern Memory 必须人工审核。
- [ ] 高风险操作禁止自动执行：merge PR、改生产配置、删除资源、重启生产服务。

## Non-goals

- [ ] 不自动合并 PR。
- [ ] 不自动改生产配置。
- [ ] 不自动写长期记忆。
- [ ] 不把 GitHub Token 暴露给 ChatGPT。
- [ ] 不把整个 Repository 塞进 ChatGPT。

## Definition of Done

- [ ] 可以针对一个 GitHub Issue 或 Incident，自动找到相关 memory 和文件。
- [ ] 可以生成带证据链的 RCA 草案。
- [ ] 可以控制每次 LLM 调用的上下文预算。
- [ ] 可以通过人工审核把 RCA 升级为 Pattern Memory。
- [ ] 后续每次类似事故可以复用 Pattern，而不是从零开始。
