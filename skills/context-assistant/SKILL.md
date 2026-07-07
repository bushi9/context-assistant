---
name: context-assistant
type: lazy
always_confirm: false
trigger_keywords:
  - context 启动
  - context 存档
  - context 状态
  - context 记录
  - context 查询
---
# context-assistant

Context OS 日常助手。命令：context <cmd>。

---
## context 启动
执行 START.md 初始化。必须读取：
1. START.md + README.md
2. context/Current-Context.md, Active-Task.md, Next-Actions.md
3. memory/Identity.md（始终读）
4. 若 Next-Actions 引用历史则读 Habits.md + Preferences.md
5. 若用户提及需要则读 Long-Term Goals.md

输出格式：
`
## Context OS 已启动
### 上次进度
- 项目: {Current-Context} | 更新: {Active-Task} | 状态
### 续接任务
{Next-Actions}
### 待确认
{从 Open-Issues.md 提取，无则"无"}
准备好了，说吧。
`

## context 存档
执行存档流程 + 归档验证。
1. 读 Current-Context.md（检查是否已更新）
2. 读 Active-Task.md（标记完成/延续）
3. 读 Next-Actions.md（更新下一步）
4. 检查 memory/ 有无新持久事实需确认
5. 备份 context/ 全部 5 文件
6. 丢弃临时数据
7. 写回 Current-Context.md（归档本次 session）
8. 写回 Active-Task.md（状态更新）
9. 验证写回，失败则用备份恢复

验证清单：
- [ ] Current-Context 已更新（不是上次内容）
- [ ] Active-Task 状态已标记
- [ ] Next-Actions 有下次任务
- [ ] memory/ 无未确认写入
- [ ] 临时数据已丢弃

输出格式：
`
## Context OS 已存档
### 本次完成
### 延续到下次
### 验证
### memory 更新
`

## context 状态
逐层检查 9 层文件状态、更新时间，运行 runtime/run.py 验证 hard_gate，统计 token。
输出：9 层表 + 健康检查(Hard Gate / 启动 token / 有数据层数) + 待处理项。

## context 记录 <内容>
判断内容类型 -> 路由写入。路由表：

| 内容类型 | 目标文件 | 确认要求 |
|---------|---------|---------|
| 身份事实:"我是X" | memory/Identity.md | 需确认 |
| 偏好:"我喜欢X" | memory/Preferences.md | 需确认 |
| 习惯(重复>=3次) | memory/Habits.md | 自动写 |
| 目标:"我要做到X" | memory/Long-Term Goals.md | 需确认 |
| 项目想法 | memory/Project-Ideas.md | 需确认 |
| 当前任务 | context/Active-Task.md | 自动写 |
| 决策 | context/Decisions.md | 自动写 |
| 阻塞问题 | context/Open-Issues.md | 自动写 |
| 下一步行动 | context/Next-Actions.md | 自动写 |
| 知识模式 | knowledge/Patterns.md | 需确认 |
| 系统规则 | knowledge/Standards.md | 需确认 |

分类判断："我是/我叫"=身份 | "我喜欢/不喜欢"=偏好 | 重复>=3次=习惯 | "我要做到/计划"=目标 | "我想做个/有个想法"=项目 | 当前在做=任务 | "我决定/选择"=决策 | "卡住/报错/不行"=阻塞 | "下一步/接下来"=行动 | "发现规律/模式"=知识 | "规定/规则/标准"=系统

输出格式：
`
## 已记录
- 内容: {摘要}
- 目标: {文件路径}
- 类型: {持久/会话级}
- 状态: {已写入/待确认}
{待确认:"确认写入吗？"}
`

## context 查询 <关键词>
在 memory/, knowledge/, projects/, context/ 搜索，其余层若存在同样搜索。汇总匹配结果。
输出格式：
`
## 查询: "{关键词}"
### {层名}
- {文件名}: {匹配行}
`
无匹配的层显示"无匹配"，不存在的层标记"层不存在"。

## Rules
1. 不扫描仓库，只读 START.md 明确允许的目录。
2. memory/ 需用户确认后才写入。context/ 可自由写，无需确认。
3. 一个事实一个文件，不跨文件重复信息。
4. 存档必须跑归档验证清单。
5. 文件不存在或不可读则跳过+标记，不中断流程。
6. 所有面向用户输出用中文。
