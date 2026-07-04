# Skill: context-assistant

## Purpose

Context OS 日常使用助手。5 个命令覆盖完整工作流：进 → 干 → 记 → 查 → 出。

## Scope

适用于任何接入 Context OS 的 AI agent（Claude、ChatGPT、Codex 等）。不处理系统架构修改、层结构变更、runtime 代码维护。

## Commands

用户输入格式：`context <命令>`，命令不区分大小写。

---

### context 启动

**作用**：执行 START.md 协议初始化，汇报上次进度。

**执行步骤**：

1. 读 `START.md`
2. 读 `README.md`
3. 读 `context/Current-Context.md`
4. 读 `context/Active-Task.md`
5. 读 `context/Next-Actions.md`
6. 按需读 `memory/` 层（Identity + Habits + Preferences）

**输出格式**：

```
## Context OS 已启动 (v1.5)

### 上次进度
- 项目: {从 Current-Context.md 提取}
- 最后更新: {从 Active-Task.md 提取}
- 状态: {completed / in-progress}

### 续接任务
{从 Next-Actions.md 提取，标号展示}

### 待确认事项
{从 context/Open-Issues.md 提取，无则显示"无"}

准备好了，说吧。
```

**Token 预算**：~2,500 tokens

---

### context 存档

**作用**：执行 Protocol §4 存档流程 + 归档验证。

**执行步骤**：

1. 读 `context/Current-Context.md` — 检查是否已更新
2. 读 `context/Active-Task.md` — 标记完成/延续
3. 读 `context/Next-Actions.md` — 更新下一步
4. 检查 `memory/` — 有无新持久事实需确认
5. 丢弃临时数据（§4.3 六类）
6. 写回 `context/Current-Context.md`（归档本次 session）
7. 写回 `context/Active-Task.md`（状态更新）

**归档验证清单**：

- [ ] Current-Context.md 已更新（不是上次的内容）
- [ ] Active-Task.md 状态已标记
- [ ] Next-Actions.md 有明确的下次任务
- [ ] memory/ 无未确认的写入
- [ ] 临时数据已丢弃

**输出格式**：

```
## Context OS 已存档

### 本次完成
{列出本 session 完成的事项}

### 延续到下次
{列出 carried forward 的事项}

### 验证
{归档验证清单结果，全 PASS 或标注问题}

### memory 更新
{无 / 列出更新的文件和原因}

下次说 "context 启动" 继续。
```

**Token 预算**：~500 tokens（写回，非重新加载）

---

### context 状态

**作用**：扫描 9 层，报告系统健康度。

**执行步骤**：

1. 逐层检查文件是否存在、是否有真实数据（非模板）
2. 检查 `context/` 上次更新时间
3. 检查 `memory/` 有无待确认事实
4. 运行 `runtime/run.py` 验证 hard_gate
5. 统计各层 token 容量

**输出格式**：

```
## Context OS 系统状态

| 层 | 文件数 | 数据状态 | 上次更新 |
|----|--------|---------|---------|
| memory/ | 5 | ✅ 真实数据 | 2026-07-04 |
| context/ | 5 | ✅ 真实数据 | 2026-07-04 |
| projects/ | 6 | ✅ 真实数据 | 2026-07-04 |
| knowledge/ | 5 | ✅ 真实数据 | 2026-07-04 |
| multi-agent/ | 5 | ⚠️ 模板 | — |
| adapters/ | 6 | ⚠️ 1/6 有内容 | — |
| skills/ | 3 | ✅ 2/3 有内容 | 2026-07-04 |
| runtime/ | 15 | ✅ 正常 | 2026-07-04 |
| docs/ | 29 | ✅ 归档 | 2026-07-04 |

### 健康检查
- Hard Gate: ✅ PASS
- 标准启动 token: ~4,941 (3.9% of 128K)
- 有真实数据的层: 7/9

### 待处理
- multi-agent/ 层仍是模板
- adapters/ 4 个占位符未实现
```

**Token 预算**：~300 tokens（汇总输出）

---

### context 记录 <内容>

**作用**：用户说一句话，助手判断写到哪层哪个文件。

**执行步骤**：

1. 分析用户输入的内容类型
2. 按下表匹配目标文件
3. 读取目标文件
4. 追加内容（不覆盖）
5. 确认写入

**路由规则**：

| 内容类型 | 目标文件 | 确认要求 |
|---------|---------|---------|
| 身份事实（"我是X"） | memory/Identity.md | 需用户确认 |
| 偏好（"我喜欢X"） | memory/Preferences.md | 需用户确认 |
| 习惯（重复行为 3+） | memory/Habits.md | 自动写入 |
| 目标（"我想做到X"） | memory/Long-Term Goals.md | 需用户确认 |
| 项目想法 | memory/Project-Ideas.md | 需用户确认 |
| 当前任务 | context/Active-Task.md | 自动写入 |
| 决策 | context/Decisions.md | 自动写入 |
| 阻塞问题 | context/Open-Issues.md | 自动写入 |
| 下一步行动 | context/Next-Actions.md | 自动写入 |
| 知识模式 | knowledge/Patterns.md | 需用户确认 |
| 系统规则 | knowledge/Standards.md | 需用户确认 |

**输出格式**：

```
## 已记录

- 内容: {用户输入的摘要}
- 目标: {文件路径}
- 类型: {持久/会话级}
- 状态: {已写入 / 待确认}

{如果待确认: "确认写入吗？"}
```

**Token 预算**：~200 tokens

---

### context 查询 <关键词>

**作用**：跨层搜索，汇总用户对某主题了解什么。

**执行步骤**：

1. 在 `memory/` 搜索关键词
2. 在 `knowledge/` 搜索关键词
3. 在 `projects/` 搜索关键词
4. 在 `context/` 搜索关键词
5. 汇总匹配结果

**输出格式**：

```
## 查询: "{关键词}"

### memory/ 层
- {文件名}: {匹配行摘要}

### knowledge/ 层
- {文件名}: {匹配行摘要}

### projects/ 层
- {文件名}: {匹配行摘要}

### context/ 层
- {文件名}: {匹配行摘要}

{无匹配的层显示"无匹配"}
```

**Token 预算**：~400 tokens

---

## Rules

1. **不扫描仓库。** 只读 START.md 协议明确允许的目录。
2. **不自动写 memory/。** 持久事实必须用户确认后才能写入。
3. **context/ 自由写。** 会话级数据可随时更新，无需确认。
4. **一个事实一个文件。** 不跨文件重复信息。
5. **存档必验证。** `context 存档` 必须跑归档验证清单。
6. **token 最小化。** 每个命令只加载必要的文件。
7. **中文输出。** 所有面向用户的输出用中文。

## Examples

### 例 1：启动

```
用户: context 启动

助手:
## Context OS 已启动 (v1.5)

### 上次进度
- 项目: Context OS v1.5
- 最后更新: 2026-07-04 18:18
- 状态: Completed

### 续接任务
1. [Medium] v1.6 — 编写日常使用流程文档
2. [Low] multi-agent/ 层模板填充
3. [Low] adapters/ 4 个占位符实现

### 待确认事项
无

准备好了，说吧。
```

### 例 2：记录

```
用户: context 记录 我决定下周开始学 Rust

助手:
## 已记录

- 内容: 决定下周开始学 Rust
- 目标: memory/Long-Term Goals.md
- 类型: 持久（目标）
- 状态: 待确认

确认写入吗？
```

### 例 3：查询

```
用户: context 查询 InSAR

助手:
## 查询: "InSAR"

### memory/ 层
- Identity.md: 研究方向滑坡/InSAR/AI+地质灾害
- Long-Term Goals.md: 深化滑坡监测预警/InSAR研究能力

### knowledge/ 层
无匹配

### projects/ 层
无匹配

### context/ 层
无匹配
```

## Dependencies

无外部依赖。只需 Context OS 文件系统存在且可读写。

## Metadata

- Version: 1.0
- Category: private-domain
- Triggers: ["context 启动", "context 存档", "context 状态", "context 记录", "context 查询"]
