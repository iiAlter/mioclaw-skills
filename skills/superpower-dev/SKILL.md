---
name: superpower-dev
version: 1.0.0
description: Software development methodology for Mioclaw agents. Use before any implementation task - enforces brainstorming → planning → TDD → review workflow.
---

# Superpower Dev Workflow

软件开发方法论，适用于Mioclaw。**任何代码任务开始前必须走此流程。**

## 核心原则

- **先想后做** — 不许上来就写代码
- **小步快跑** — 每步2-5分钟，可验证
- **测试先行** — 没看到测试失败就不写代码
- **两阶段Review** — 先查是否符合spec，再查代码质量

## 工作流

```
┌─────────────────────────────────────────────────────┐
│  1. Brainstorming (构思)                            │
│     → 理解上下文，问清需求，展示设计                │
│     → 用户确认设计后才进入下一步                     │
├─────────────────────────────────────────────────────┤
│  2. Writing Plans (计划)                            │
│     → 拆解成2-5分钟的小任务                         │
│     → 每个任务精确到文件路径和验证步骤              │
├─────────────────────────────────────────────────────┤
│  3. Subagent Implementation (执行)                  │
│     → 每个任务派发独立subagent                       │
│     → 两阶段review: spec合规 → 代码质量            │
├─────────────────────────────────────────────────────┤
│  4. Verification & Finish (收尾)                    │
│     → 验证测试通过                                  │
│     → 清理 worktree，分支处理                      │
└─────────────────────────────────────────────────────┘
```

---

## 1. Brainstorming 构思阶段

**触发条件：** 用户提出任何需要写代码的需求

**流程：**
1. 探索项目上下文（文件、文档、最近commit）
2. 逐一问清需求（一次只问一个问题）
3. 提出2-3个方案及trade-off，给出推荐
4. 展示设计（分块，用户逐步确认）
5. 写设计文档 → 保存到 `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md`
6. Spec review循环（派发subagent检查，用户最终确认）
7. 进入writing-plans阶段

**Hard Gate：** 在用户确认设计之前，**不许写任何代码**。

---

## 2. Writing Plans 计划阶段

**触发条件：** 设计已获用户批准

**原则：**
- 假设执行者对项目零上下文，但是个有经验的工程师
- YAGNI (You Aren't Gonna Need It)
- DRY
- TDD

**Plan文档格式：**

```markdown
# [Feature Name] Implementation Plan

**Goal:** [一句话描述目标]

**Architecture:** [2-3句架构说明]

---
## Task 1: [任务名]

**Files:**
- Create: `path/to/file.py`
- Modify: `path/to/existing.py:123-145`
- Test: `tests/path/to/test.py`

- [ ] **Step 1: 写失败的测试**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

- [ ] **Step 2: 运行测试，确认失败**
- [ ] **Step 3: 写最简代码让测试通过**
- [ ] **Step 4: 运行测试，确认通过**
- [ ] **Step 5: Commit**
```

---

## 3. Subagent Implementation 执行阶段

**每个任务执行流程：**

```
派发implementer subagent
        ↓
    实现 + 测试 + commit + 自审
        ↓
   派发spec reviewer subagent
        ↓
   代码符合spec？ ──no──→ 打回修复
        ↓ yes
   派发code quality reviewer subagent
        ↓
   代码质量过关？ ──no──→ 打回修复
        ↓ yes
     标记任务完成
```

**Model选择：**
- 机械性实现（1-2文件，清晰spec）→ 用便宜快速的模型
- 集成/判断任务（多文件协调）→ 用标准模型
- 架构/设计/审查 → 用最强模型

---

## 4. Verification & Finish 收尾阶段

**触发条件：** 所有任务完成

**检查清单：**
- [ ] 所有测试通过
- [ ] 代码符合原始spec
- [ ] 无临时调试代码残留
- [ ] Commit消息规范

**分支处理选项：**
1. Merge到main
2. 创建PR
3. 保留分支继续开发
4. 丢弃分支

---

## TDD RED-GREEN-REFACTOR

```
┌──────────┐     ┌──────────┐     ┌──────────┐
│   RED    │ ──→ │  GREEN   │ ──→ │ REFACTOR │
│ 写失败测试│     │ 写最简代码│     │  清理代码 │
└──────────┘     └──────────┘     └──────────┘
     ↑                │                │
     └────────────────┴────────────────┘
            确认测试通过才下一步
```

### RED阶段
写一个最小测试，展示预期行为。

**Good:**
```python
def test_retries_failed_operations_3_times():
    attempts = 0
    def operation():
        nonlocal attempts
        attempts += 1
        if attempts < 3:
            raise Error('fail')
        return 'success'
    
    result = retry_operation(operation)
    assert result == 'success'
    assert attempts == 3
```

**Bad:**
```python
def test_retry_works():
    mock = Mock()
    mock.side_effect = [Error(), Error(), 'success']
    # ...
```

### GREEN阶段
写**最少量代码**让测试通过。不要"顺便"加功能。

### REFACTOR阶段
在测试通过的前提下，清理代码。保持测试始终绿色。

---

## Systematic Debugging 调试方法

**4阶段根源分析：**

1. **收集证据** — 错误信息、复现步骤、相关日志
2. **定位范围** — 是哪部分代码的问题？
3. **追踪根因** — 顺着调用链找到真正原因
4. **验证修复** — 确认问题真正解决，不是表面掩盖

**防御性调试：**
- 如果等待某个条件，设置最大等待时间和轮询间隔
- 考虑边界条件和竞态条件
- 检查是否有隐藏的side effect

---

## 适用判断

| 场景 | 用这个skill？ |
|------|--------------|
| 新功能开发 | ✅ 必须 |
| Bug修复 | ✅ 必须 |
| 重构 | ✅ 必须 |
| 临时脚本/原型 | ❌ 可跳过 |
| 配置修改 | ❌ 可跳过 |
| 纯阅读/分析 | ❌ 不适用 |

**记住：** "这太简单了不需要设计"是最大的误区。简单项目才是最容易出问题的。
