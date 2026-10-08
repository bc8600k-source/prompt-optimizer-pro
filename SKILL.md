---
name: prompt-optimizer-pro
description: 综合版 prompt 优化器，结合 61 种框架选择 + CLARITY 质量检查 + 迭代闭环验证。
version: 2.0.0
author: 灵瞳
license: MIT
---

# Prompt Optimizer Pro

综合六个 GitHub skill 的优点：
- dreamor/prompt-optimizer-skill：61 种框架 + CLARITY 质量检查
- bergr7/claude-skill-prompt-optimizer：迭代闭环 + 预算控制
- Agentic-Assets/Agent-Skills：全面覆盖（few-shot/CoT/ReAct/结构化输出）
- ckelsoe/prompt-architect：31 框架 + 5 维评分 + 7 类意图路由
- microsoft/SkillOpt：SKILL.md 文本空间优化 + 验证门控 + 睡眠周期
- treylom/prompt-engineering-guide：7000 行综合参考 + 模型特定策略

---

## 工作流（7 步）

### Step 1: 分析输入

接收用户请求，分类清晰度：
- 完全模糊 → 提供 3 个示例
- 部分清晰 → 问关键问题（Step 4）
- 完全清晰 → 继续

**复杂度评估（必做）：**
- Simple（≤3 元素）→ APE, ERA, TAG, RTF, BAB, PEE, ELI5
- Medium（4-5 元素）→ RACE, COAST, ROSES, Chain-of-Thought, SMART, FOCUS
- Complex（6+ 元素）→ RACEF, CRISPE, RISEN

**5 维评分（来自 prompt-architect）：**

| 维度 | 评分标准 |
|------|----------|
| Clarity | 目标是否明确？模糊词（"东西""事情""可能"）扣分 |
| Specificity | 要求是否具体？命名实体、数量、格式/长度/风格加分 |
| Context | 背景信息是否充分？情境、受众、理由加分 |
| Completeness | what/why/how/output format 是否齐全？ |
| Structure | 结构是否清晰？分段、列表、逻辑顺序加分 |

每项 1-10 分，给出总分（平均分，保留一位小数）。7+ 分通常不需要框架。

### Step 2: 选择框架

根据复杂度和领域选择框架，解释：
1. 为什么选这个框架
2. 置信度（1-10）
3. 替代方案（如果置信度 < 7）

### Step 3: 加载框架细节

从 `frameworks/index.json` 读取框架的 elements、domains、use_cases。

### Step 4: 澄清模糊点

验证 5 个维度：
1. **目标清晰度** — 预期结果是什么？
2. **目标受众** — 谁看 AI 的输出？
3. **上下文完整性** — 背景信息够吗？
4. **格式要求** — 输出格式是什么？
5. **约束条件** — 有什么限制？

**拒绝处理：** 用户拒绝回答 → 用智能默认值继续。

### Step 5: 生成优化 prompt

应用框架构建最终 prompt，提供 1-3 个版本：
- Basic（简单任务默认）
- Enhanced（中等/复杂任务默认）
- Expert（用户明确要求）

### Step 6: 质量验证（CLARITY 检查）

7 项检查，每项 pass/fail：

| 字母 | 元素 | 触发条件 |
|------|------|----------|
| C | Context | 命名具体背景 |
| L | Logic | 规定推理方法或有序子步骤 |
| A | Action | 包含具体祈使动词 |
| R | Role | 分配专家身份 |
| I | Input/Output | 命名输入和输出结构 |
| T | Tone | 约束语气风格 |
| Y | Yardstick | 声明 ≥1 个可衡量验收标准 |

**验证阈值：**
- Simple ≥ 3/7
- Medium ≥ 5/7
- Complex ≥ 6/7

**如果验证失败：**
1. 列出失败项
2. 为每项生成一行补丁
3. 重新应用并重新评分

### Step 7: 迭代验证（可选）

如果用户要求迭代优化：
1. 记录 baseline（当前 prompt 的输出质量）
2. 做最小化修改
3. 对比修改前后的输出
4. 如果改进 → 保留；如果回归 → 回滚
5. 重复直到用户满意或预算耗尽

**预算控制：** 默认 3 轮迭代，用户可调整。

**验证门控（来自 SkillOpt）：**
- 每次修改后在 held-out 测试集上验证
- 只有严格改进才接受修改
- 回归的修改立即回滚
- 记录每次修改的证据（baseline → candidate 分数）

**睡眠周期（来自 SkillOpt-Sleep）：**
- 定期回顾过去会话，提取重复任务
- 在后台重跑任务，验证当前 skill/memory 的效果
- 只接受有验证门控保护的改进
- 适合 cron 定时执行（如凌晨 3:00）

---

## 核心原则

1. **不要过度工程化** — Simple 任务用 Simple 框架 + Basic 版本
2. **保留用户原始意图** — 优化不是重写
3. **解释每次优化** — 为什么改，改了什么
4. **提供多版本** — Basic / Enhanced / Expert
5. **验证后再呈现** — CLARITY 检查不过不输出

---

## 参考文件

### dreamor 框架系统
- `frameworks/index.json` — 61 种框架的结构化元数据
- `references/Quick_Reference.md` — 意图到框架的查找表
- `references/Decision_Tables.md` — 领域映射、版本特征、CLARITY 框架
- `references/Frameworks_Summary.md` — 人类可读的框架概述

### Agentic-Assets 参考
- `references/prompt-patterns.md` — zero-shot, few-shot, CoT, ReAct
- `references/prompt-optimization.md` — 迭代优化、A/B 测试、token 缩减
- `references/evaluation-frameworks.md` — 指标、测试套件、自动评估
- `references/structured-outputs.md` — JSON mode, function calling, schema 设计
- `references/system-prompts.md` — persona 设计、guardrails、上下文管理

### prompt-architect 参考
- `references/prompt-architect/frameworks/` — 31 框架详细文档
- `references/prompt-architect/techniques/few-shot.md` — few-shot 技术
- `assets/templates/` — 框架模板

### SkillOpt 参考
- `references/skillopt-prompts/` — SkillOpt 的优化 prompt 模板

### prompt-engineering-guide 参考
- `references/prompt-engineering-full.md` — 7000 行综合参考（模型特定策略、Context Engineering、FactChecker 等）

---

## 触发词

- 优化 prompt / 优化提示词 / 改写 prompt / 优化指令
- 让提示词更好 / 帮我把这个写成 prompt
- 改进这个 prompt / 让这个 prompt 更好
- 任何模糊/简短的指令需要变成高质量 prompt
- 用户对 AI 输出质量不满意但未指定修复方法

---

## 与其他 skill 的关系

- 本 skill 专注于 **prompt 优化**
- 如果需要 **cron 任务优化** → 用 `scheduled-task-correctness`
- 如果需要 **TTS 配置** → 用 `hermes-voice-tts-config`
- 如果需要 **购买决策** → 用 `purchase-advisory`