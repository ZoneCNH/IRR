# Execution Pipeline

> 执行管线定义：谁处理什么、交付物如何流转、各阶段质量门禁。

## 管线阶段

```
Goal → Spec → Plan → Matrix → Tasks → Prompt → Code → Evidence
  │       │       │       │        │        │       │        │
  └─┬─┘   └─┬─┘   └─┬─┘   └──┬──┘  └──┬──┘  └─┬─┘   └──┬──┘
   Goal    Spec    Plan    Matrix   Tasks    Prompt   Code    Evidence
  architect spec   planner  matrix   planner  prompt   executor verifier
```

## 阶段职责

| 阶段 | 制品 | 负责人 | 输入 | 输出 |
|------|------|--------|------|------|
| Goal | goal.md | goal-architect | 业务需求 | 模块目标、边界、成功标准 |
| Spec | SPEC.md | goal-spec | goal.md | 23 节规格、FR/BR/NFR/AC |
| Plan | IMPLEMENTATION-PLAN.md | goal-planner | SPEC.md | 实现计划、阶段划分 |
| Matrix | TRACEABILITY.md | goal-matrix | SPEC.md + Tasks | §1-§7 全链路追溯矩阵 |
| Tasks | tasks/*.md | goal-planner | Matrix | 原子任务定义（含 Scope/AC/Deps） |
| Prompt | prompt/*.md | goal-prompt-builder | Tasks + Matrix | AI 编码 Context Package |
| Code | *.go / *.ts | task-executor | Prompt + Spec | 实现代码 + 测试 |
| Evidence | evidence/* | goal-evidence | Code + Matrix | AC 验证证据、benchmark、coverage |

## 质量门禁

| 阶段 | 门禁 | 验证方式 |
|------|------|----------|
| Spec | 23 节结构完整性 | spec-lint |
| Matrix | 追溯链闭合（FR→AC→TC→Task） | traceability-check |
| Matrix | 覆盖率仪表盘 FR/BR/NFR/AC/TC 全链路 | matrix-structural-score |
| Tasks | 每个 FR 有 >=1 Task | task-coverage-check |
| Code | 编译/测试/覆盖率 >=80% | CI pipeline |
| Code | race/vet/lint/secret scan | CI pipeline |
| Evidence | 每条 AC 有结构化证据 | evidence-lint |

## 执行模式

### 单模块管线

```
1. 检查 module/{name}/ 是否存在 goal.md + SPEC.md
2. 若无 → goal-architect + goal-spec 创建
3. 检查 TRACEABILITY.md 是否 §1-§7 完整
4. 若无 → goal-matrix 创建/对齐
5. 检查 tasks/ 是否有任务文件
6. 若无 → goal-planner 创建
7. 按 Task 顺序执行 task-executor
8. 收集 evidence → goal-evidence 归档
```

### 多模块批量管线

```
1. 全量扫描 → 按缺失制品分级（P0/P1/P2）
2. P0 模块优先（缺 goal/spec/trace）
3. P1 模块其次（缺 task）
4. 同优先级可并行
5. 每个模块独立管线闭环
```

## 角色分派

| 角色 | Agent | 触发条件 |
|------|-------|----------|
| 架构设计 | goal-architect | 新模块、架构变更 |
| 规格编写 | goal-spec | SPEC.md 缺失或版本升级 |
| 实现规划 | goal-planner | 新功能、Tasks 拆分 |
| 追溯矩阵 | goal-matrix | TRACEABILITY.md 缺失或结构不完整 |
| 代码实现 | task-executor | Task 进入 Code 阶段 |
| 证据收集 | goal-evidence | Code 阶段完成后 |
| 质量审查 | goal-lint / goal-reviewer | 每个阶段完成后 |

## 交付物标准

### TRACEABILITY.md §1-§7 标准

```
§1 FR 追溯表（含 Task 列）
§2 BR 追溯表（含 Task 列）
§3 NFR 追溯表（含 Task 列）
§4 TC→FR 反向追溯
§5 全局 AC 注册表
§6 覆盖率仪表盘（FR/BR/NFR/AC/TC 总数/Done/覆盖率）
§7 变更历史
```

### TASK-*.md 标准

```
# TASK-{MODULE}-{###} {标题}

## Objective
## Scope  
## Covers (FR/BR/NFR 编号)
## Deliverables
## Acceptance Criteria (3-7 条可验证)
## Dependencies
```

### goal.md 标准

```
# {模块名} Goal
| 字段 | 值 | (模块/层级/仓库/版本/状态/更新)
## 目标
## 非目标  
## 成功标准 / 核心能力
## 当前阻塞 (P0/P1/P2)
```
