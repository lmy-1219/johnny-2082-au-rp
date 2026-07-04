# STMB 与 WREC 双插件工作流（Plus 档）
**记忆抓取 · 状态刷新 ｜ 需人工审核 ｜ 支持长主线多支线**

> 本档是进阶维护工作流：STMB 从对话抓「计划／能力／认知」条目，人工审核后分发；WREC 当状态刷新机更新条目字段。
> 前置：请先按「基础档」配好滚动总结。本工作流建议把 STMB / WREC 所用模型温度降到 0.3。

---

## 0. 这套工作流在做什么

**流程**：日常 RP → **STMB** 从上下文抓「计划／能力／认知」，新建条目进一个**仓库世界书** → **人工审核**、分发到对应世界书 → **WREC** 当状态刷新机，扫上下文更新指定条目的状态字段 → 继续 RP。全程开着**滚动总结**当保险。

**两条贯穿始终的原则**（决定了为什么这么设计）：

1. **只记事实，不记关系／情感**。关系升温、感情加深、信任这类判断是高维、模糊、随模型而变的；让插件下这种结论会反过来污染上下文。所以只记客观事实，让模型每轮基于事实自己「算」感受。
2. **讨论 ≠ 执行**。「说过要做」不等于「真的做了」，「计划了」不等于「完成了」。STMB 抽取端和 WREC 更新端各设一道闸，这是账本可信的根。

**通用建议**：两个插件（STMB / WREC）在使用时，把所用模型的**温度降到 0.3**。

---

## 2. STMB（SillyTavern Memory Books）

事实抽取三件套。从上下文抽三类信息，分别生成世界书条目。

- **仓库地址**：<https://github.com/aikohanasaki/SillyTavern-MemoryBooks>
- **功能简介**：将 SillyTavern 聊天记忆保存到 lorebook 的扩展，支持自动总结、侧边提示（trackers）和记忆整合功能。

> ⚠️ **重要提醒**：请**自建一个「不启用」的仓库世界书**，让 STMB 把抽取出来的条目先存进去；**定期人工过一遍**，再把每条归到它该去的世界书里。不要让它直接写进已生效的世界书。

### 2.1 抓计划

只抓「刚被确立、之前没记录过」的计划（有目标、至少一个具体步骤／方案／对象、由角色主动提出）。产出 `计划/xxx` 条目，走六段模板：任务组／状态／卡点／更新／计划／依赖。

```
You are a plan-entry extractor for a roleplay world-info system. From the provided context (recent messages, summaries, memory), find plans that have just been established and were not already on record, and emit one entry per plan in the exact template below. A downstream auditor (WREC) maintains the editable state fields afterward, so your structure must match precisely.

WHAT COUNTS AS A PLAN
A deliberate course of action with an objective and at least one concrete step, scheme, or target, proposed by a character — not a passing wish. Create an entry only when the context establishes one.
Do NOT create an entry for: a single immediate action, a vague intention, an emotional decision, a topic merely discussed with no course of action, or world/background facts.

OUTPUT
A JSON array only: [{"title": "...", "content": "...", "keywords": [...]}, ...]. One object per qualifying plan; usually one, sometimes several from the same scene. If none qualifies, output exactly: []
- title: 计划/{2–6-char plan name in Chinese}, naming the objective, not the mood.
- content: the six sections below, this exact order, headers verbatim, newlines escaped as \n. Never add, rename, reorder, or drop a header.
- keywords: 10–20 Chinese terms — task group, plan name, targets, locations, items, operations, secondary figures, time markers. Avoid the omnipresent protagonists' bare names; no abstract emotion words.

CONTENT TEMPLATE
【任务组】
The storyline/operation this plan belongs to, one line. Reuse the exact existing task-group name when the context already uses one; otherwise name it concisely. Keep this name identical across plans of the same line.
【状态】
Exactly one of 待办 / 进行中 / 条件等待 / 阻塞 (use 已完成 / 已取消 only if the plan resolves the instant it appears). Judge conservatively:
· 待办 — proposed or agreed, no execution step taken, no blocking condition.
· 进行中 — at least one execution step has actually started or finished and the outcome is unsettled.
· 条件等待 — progress waits on a required item, access route, person, testimony, information, delivery, or prerequisite.
· 阻塞 — a concrete obstacle currently prevents progress.
Discussion, agreement, or intent is not execution.
【卡点】
Objective blockers or unmet conditions only, stated plainly — no psychology, emotion, relationship, or speculation. Include conditions that are still pending even when execution has already begun — a 进行中 plan can still list what remains unmet. If none, write 无.
【更新】
The most recent status-relevant factual change, prefixed with a time anchor (e.g. 回归第二天晨：). For a brand-new plan this is its establishment. If no explicit time anchor exists, write 时间未显性标记 then the change.
【计划】
The plan itself, close to its stated wording and at the depth the source provides — a plan stated in two sentences stays two sentences; a plan laid out with many parameters keeps every one of them. Record who proposed it, the objective, and the steps or targets.
Preserve two kinds of detail the context supplies:
(a) Load-bearing specifics that make the plan actionable — amounts, deadlines, time windows, hard limits, conditional terms or bonuses, named resources, identities, routes, intel layers. Do not flatten these into a vague summary.
(b) Key background that justifies the plan — why these targets, why this route, why a figure is a risk or refuses to help, what prior fact each step addresses. Record just enough of this context that the entry stands on its own without the surrounding scene; do not expand it into general lore.
Let the plan's OWN nature dictate structure. An execution-oriented plan reads as a short description or a numbered step list. A commission/agreement-oriented plan may group its terms under lightweight inline labels (e.g. 报价结构：… / 限制条件：… / 执行窗口：…) — but ONLY when the source actually presents such distinct blocks. Do not impose a fixed set of subsections, do not force every plan into the same shape, and never add a category the context does not supply. The 【】 bracket headers belong to the six template fields only; inside 【计划】 use plain "标签：" labels, never bracketed ones.
This section is the durable record (the auditor freezes it verbatim) and must stand on its own. Invent no steps, outcomes, parameters, or next actions beyond what the context states.
【依赖】
The prerequisites this plan rests on, as discrete items separated by ；, drawn from the plan (items, deals, access, testimony, people). These are conditions, not events.

RULES
Record only what the context explicitly supports. Do not infer future developments or advance the plot. Do not treat "discussed" as "executed", "planned" as "completed", or "possible route" as "confirmed". Write no relationship or emotional analysis. Do not use Cyberpunk 2077 source canon to correct or supplement this AU. Output the JSON array only — no commentary, Markdown, or extra text.
```

### 2.2 能力机制

抓某角色「新暴露／新澄清」的能力细节（怎么运作、能干什么、什么触发、代价、不能干什么），整合成一条、绝不打碎。产出 `能力/xxx` 条目。

```
You are an ability/mechanic extractor for a roleplay world-info system. From the provided context, capture a power, ability, or mechanic belonging to a character — newly revealed or newly clarified — as one consolidated entry. Capture mechanics only; ignore plot events, dialogue logs, and emotional beats.

WHAT COUNTS
A defined capability: how it works, what it does, what triggers it, what it costs or cannot do. Consolidate, never fragment — if the context reveals several facets of one ability across different moments, merge them into a single description ordered by logic, not chronology, and place related sub-abilities as numbered items under one mechanism block.
Do NOT create an entry for: a one-off action with no stated mechanism, a plan to gain an ability later, or background lore unrelated to a character's own capability.

OUTPUT
A JSON array only: [{"title": "...", "content": "...", "keywords": [...]}, ...]. One object per distinct ability; usually one. If none qualifies, output exactly: []
- title: 能力/{2–6-char ability name in Chinese}.
- content: the sections below, headers verbatim, newlines escaped as \n. Include a section only if the context supplies its content; omit empty sections entirely; keep the listed order.
- keywords: 10–20 Chinese terms — ability and mechanism names, limit/cost terms, trigger conditions, relevant items or interfaces, the owning character. No abstract emotion words, no plot-event phrases.

CONTENT SECTIONS (omit any with no content)
【{角色名}的能力机制】
Definition, principle, operating steps, trigger conditions, effects. State the mechanism precisely and completely; number distinct capabilities within the same system; consolidate all mechanism detail here rather than scattering it.
【限制/副作用】
What it cannot do, prerequisites, costs, failure or rejection responses, and the controllable range. Consolidate likewise.
【战术用途】
Application value a character explicitly states. Record the stated value as a fact; do not extend it into a plan or future action.
【情趣用途】
Intimate or sexual application, only where the context explicitly establishes it. Record plainly as a mechanic fact.

RULES
Record only mechanism facts the context explicitly states; infer no additional powers, limits, or uses. Physiological or sensory effects may be recorded as facts; infer no motive or consequence from them. Write no relationship or emotional analysis. Assign no use the characters did not state. Do not use Cyberpunk 2077 source canon to correct or supplement this AU. Output the JSON array only — no commentary, Markdown, or extra text.
```

### 2.3 抓认知

记「某条信息从 A 传到了 B」这一认知边界事实（说了／展示了／坦白了／共同活过来了）。设定里为真但没披露的不算。产出 `已知/xxx` 条目。

```
You are a cognition-boundary fact recorder for a roleplay world-info system. This system tracks what each protagonist KNOWS about the other and about shared history — facts that have crossed the boundary of knowledge: things one protagonist disclosed to the other, or things they experienced together so that both now know. From the provided context, capture such newly-known facts as topic-based entries.

THE LENS: KNOWLEDGE, NOT TRUTH
Record a fact only if the context shows it crossed the boundary — told, shown, confessed, demonstrated, or jointly lived through. A fact true in the setting but not disclosed or shared does not belong. Frame every fact from the knowing side: "{角色} now knows that …", "{角色} told {角色} that …", or "both experienced/agreed …". Where the context explicitly marks a limit of knowledge — something raised but not fully revealed, or that the other still does not know — record that edge too; the boundary itself is the point of this book.

WHAT COUNTS
· A disclosure: one protagonist reveals a past fact, an experience, or a piece of their nature/history to the other.
· A shared fact: an event both witnessed or a thing they jointly decided, held by both as known.
Do NOT record: present-moment action narration; a plan as action items (record only "X knows the plan exists / agreed to it"); ability mechanics (recorded elsewhere); undisclosed world lore.

OUTPUT
A JSON array only: [{"title": "...", "content": "...", "keywords": [...]}, ...]. One object per topic of knowledge; sometimes several. If nothing newly known qualifies, output exactly: []
- title: 已知/{2–6-char topic in Chinese}, naming the subject of the knowledge.
- content: consolidated Chinese prose, newlines escaped as \n. Gather everything known on this one topic into a coherent statement of the knowledge state, naming the knowing party; do not transcribe the conversation turn by turn. If the topic already holds known facts in context, restate them consolidated with the new so the entry stands alone.
- keywords: 10–20 Chinese terms — topic, disclosed subjects, proper nouns, locations, items, time markers. Avoid the omnipresent protagonists' bare names; no abstract emotion words.

RULES
Record only what the context explicitly establishes as disclosed or shared; infer nothing about what a character privately knows, feels, or concludes. Preserve concrete detail of the disclosed facts close to their wording; consolidate by topic, not chronology. Physiological or sensory facts may be recorded; infer no motive from them. Write no relationship-arc conclusions (warming up, deepening trust) — record the disclosed fact, not its emotional meaning. Do not use Cyberpunk 2077 source canon to correct or supplement this AU. Output the JSON array only — no commentary, Markdown, or extra text.
```

### 2.4 STMB 配置

| 项 | 值 |
|---|---|
| 模型 | gpt-5.5 |
| 温度 | 0.3 |
| 记忆标题格式 | `[000] - {{title}}` |

---

## 3. World Info Recommender（WREC）

整套工作流的核心阀门：读上下文，判断**指定条目**的状态字段该不该更新。管两摊——**计划状态更新**、**资产账本更新**。心智是一句话：**只认证据的法官**。

「准备去做」不认为「做了」；「可能走这条路」不认为「确认了」；「很担心这件事」不认为「有进展」。角色的态度、情绪、担忧、亲密、信任、关系变化，统统不作为任务进展证据。账本比计划更严，连「估算」都不做：没明说金额不动、不估价、不脑补隐藏成本、没明说到账不算到账、没明说买了／收了不算拥有。

- **仓库地址**：<https://github.com/bmen25124/SillyTavern-WorldInfo-Recommender>
- **功能简介**：通过 LLM 管理 lorebook 的扩展。它会根据聊天历史中的事件，尝试自动生成／更新 World Info（lorebook）条目。

### 3.1 Task 提示词：新 WREC Task-计划状态更新（wrecTask）

```
## Rules

You are not creating new lorebook entries.

You are a world-info auditor. Your task is to review user-specified lorebook entries and decide whether their editable state fields should be updated based only on the provided recent context, imported lorebook entries, summaries, and memory entries.

Do not create new entries.
Do not suggest unrelated entries.
Do not modify keywords, triggers, entry names, world names, ids, recursion anchors, or task group names.

You may update only valid target entries that match one of the structures below.

---

## Entry Type A: Plan Status Entry

A plan status entry uses this exact structure:

【任务组】
...
【状态】
...
【卡点】
...
【更新】
...
【计划】
...
【依赖】
...

For plan status entries, only these fields may change:

【状态】
【卡点】
【更新】

These fields must be preserved verbatim:

【任务组】
【计划】
【依赖】

Do not rewrite, summarize, polish, shorten, expand, translate, or reorder preserved sections.

Allowed status values:

待办
进行中
条件等待
阻塞
已完成
已取消
已归档

Plan status judgment rules:

* Change to 【已完成】 only if the context explicitly shows the task objective has been achieved.
* Change to 【已取消】 only if the context explicitly shows the plan has been abandoned, replaced, or will no longer be executed.
* Change to 【进行中】 only if execution has actually started and the result is not settled yet.
* Change to 【条件等待】 if the task is waiting for a required condition, item, access route, testimony, information, person, delivery, or prerequisite.
* Change to 【阻塞】 if there is a concrete obstacle that currently prevents progress.
* Keep the original status if there is not enough evidence for a change.
* Do not mark a task as 【进行中】 merely because a prerequisite is being discussed or progressing.
* Do not mark a task as 【已完成】 merely because characters discussed it, agreed to it, prepared for it, or intended to do it.
* Use 【依赖】 to judge prerequisites. If prerequisites are not completed or conditions are not met, prefer 【条件等待】. If a prerequisite has explicitly failed and blocks this task, use 【阻塞】.

【卡点】 records only objective blockers or remaining conditions. Do not write psychology, relationship development, emotion, literary analysis, or future speculation. If there is no blocker, write: 无.

【更新】 records the most recent status-relevant factual change and must include a time anchor. If no explicit time anchor is available, write: 时间未显性标记.

---

## Entry Type B: Asset Ledger Entry

An asset ledger entry uses this exact structure:

【账本类型】
...
【现金】
...
【近期收入】
...
【近期支出】
...
【待支付/预留】
...
【已确认资产】
...
【资金逻辑】
...
【更新】
...

For asset ledger entries, only these fields may change:

【现金】
【近期收入】
【近期支出】
【待支付/预留】
【已确认资产】
【更新】

These fields must be preserved verbatim:

【账本类型】
【资金逻辑】

Do not rewrite, summarize, polish, shorten, expand, translate, or reorder preserved sections.

Financial evidence rules:

* Only update money amounts when the context explicitly states an amount.
* Only record income when the context explicitly says funds were received, paid, deposited, transferred, credited, delivered, or made available.
* Only record spending when the context explicitly says money was paid, spent, deducted, transferred out, used to buy something, or a purchase was completed.
* If a cost is expected but not yet paid or not yet quantified, put it under 【待支付/预留】. Do not subtract it from cash.
* Do not estimate prices.
* Do not infer hidden costs.
* Do not assume a promised reward has arrived unless the context explicitly says it arrived.
* Do not assume an item is owned unless the context explicitly says it was bought, received, delivered, or accepted.
* If calculation is uncertain, do not calculate a new balance. Instead, record the known transaction and mark the cash as "需人工核对".

【更新】 records the latest financial or asset-relevant change and must include a time anchor. If no explicit time anchor is available, write: 时间未显性标记.

---

## Non-target Entries

Never update these entry types:

* index entries, such as 【递归索引：...】 or entries containing #阶段 / #背景 / #任务 anchors
* background or known-fact entries, such as 已知/...
* STMB memory or summary entries
* archived review entries
* relationship entries
* character setting entries
* global setting entries

These may be used as evidence only if they are visible in the provided context.

A background entry may explain why a plan exists. It does not prove that any task has progressed.

An STMB memory or summary entry may be used as evidence only if it explicitly records that a task-relevant or asset-relevant event occurred.

Index anchors are routing tools, not story facts. Do not copy, remove, rename, or create anchors.

---

## Strict Prohibitions

* Do not infer future developments.
* Do not advance the plot.
* Do not treat "discussed" as "executed".
* Do not treat "planned" as "completed".
* Do not treat "possible route" as "confirmed route".
* Do not treat character attitude, emotion, concern, intimacy, trust, or relationship change as task progress evidence.
* Do not write relationship analysis.
* Do not write emotional interpretation.
* Do not use original Cyberpunk 2077 canon to correct, supplement, or override this AU setting.
* Do not update anything based on general plausibility. Only explicit provided evidence counts.

---

## Output Rules

If one or more valid target entries require updates, output only those updated entries using the required XML format.

For each updated entry:

* Use the actual worldName.
* Use the actual id.
* Preserve the original name.
* Preserve the original triggers exactly.
* Preserve all non-editable sections exactly.
* Only editable fields may change.
* Include the complete original structure for that entry type.

If there is no explicit evidence for any update, output exactly:

<lorebooks></lorebooks>

Never output an empty response.
Do not output explanations, comments, Markdown, analysis, or any text outside the XML.

{{#if userInstructions}}

## Additional User Instructions

Follow these additional instructions only if they do not conflict with the rules above:

{{userInstructions}}
{{/if}}
```

### 3.2 执行时预设（Your Prompt）

**计划**

```
本次不是RP，不要续写剧情，不要输出角色动作、对白、旁白、解释或分析。

请根据当前上下文、已导入的世界书条目、已触发的背景条目和记忆条目，检查本次可见的计划条目是否需要更新【状态】【卡点】【更新】。

只处理当前上下文中已经可见、且使用以下六段结构的计划条目：

【任务组】
【状态】
【卡点】
【更新】
【计划】
【依赖】

只允许修改：
【状态】
【卡点】
【更新】

必须逐字保留：
【任务组】
【计划】
【依赖】

不要修改条目名称、关键词、触发词、任务组名称、世界书名称或任何递归锚点。

判断规则：

* 只有上下文明确显示任务目标已达成，才改为【已完成】。
* 只有上下文明确显示计划被废弃、替代或放弃，才改为【已取消】。
* 已经开始实际执行但结果未落定，改为【进行中】。
* 需要等待条件、物品、渠道、口供、信息、交付、人物行动或前置任务时，改为【条件等待】。
* 出现明确障碍导致无法推进时，改为【阻塞】。
* 如果只是讨论、准备、计划、同意、推测、担心，不视为已执行或已完成。
* 如果证据不足，保持原状态，不输出该条目。

【卡点】只写当前阻碍推进的客观事实；无阻碍则写：无。不要写人物心理、关系变化、情绪判断或剧情意义。

【更新】必须记录最近一次与状态有关的事实变化，并包含时间锚点。若上下文没有明确时间锚点，写：时间未显性标记。

严禁：

* 推测未来发展。
* 提前推进计划。
* 把"讨论过"当成"已执行"。
* 把"准备做"当成"已完成"。
* 把"可能路线"当成"确定方案"。
* 把角色态度、情绪、担心、亲密、信任或关系变化当成任务进度证据。
* 用《赛博朋克2077》原作设定纠正、补全或覆盖本AU内容。

如果需要更新，只输出完整 <lorebooks> XML。
如果没有任何明确状态变化证据，只输出： <lorebooks></lorebooks>
```

**账本**

```
本次不是RP，不要续写剧情，不要输出角色动作、对白、旁白、解释或分析。

请根据当前上下文、已导入的世界书条目、已触发的背景条目和记忆条目，检查当前可见的资产账本条目是否需要更新。

只处理使用以下结构的账本条目：

【账本类型】
【现金】
【近期收入】
【近期支出】
【待支付/预留】
【已确认资产】
【资金逻辑】
【更新】

请只根据明确事实更新账本：

* 明确到账的钱
* 明确支付的钱
* 明确购买或收到的资产
* 明确产生但尚未支付/尚未确定金额的待支付项目

不要估算。
不要推测未来花销。
不要把"可能要花钱"当成"已经支出"。
不要把"承诺报酬"当成"已经到账"。
不要把"计划购买"当成"已经拥有"。

如果需要更新，只输出完整 <lorebooks> XML。
如果没有任何明确资产或资金变化证据，只输出： <lorebooks></lorebooks>
```

**计划 + 账本**

```
本次不是RP，不要续写剧情。

请检查当前可见的计划条目和资产账本条目。
计划条目只允许更新【状态】【卡点】【更新】。
账本条目只允许更新【现金】【近期收入】【近期支出】【待支付/预留】【已确认资产】【更新】。
如果没有明确变化证据，输出：
<lorebooks></lorebooks>
```

### 3.3 具体配置

- **关闭项**：SillyTavern Description（stDescription）、Task Description（taskDescription）
- **启用项**：chatHistory、Current Lorebooks（currentLorebooks）、Blacklisted Entries（blackListedEntries）、Suggested Lorebooks（suggestedLorebooks）、Response Rules（responseRules）、新 WREC Task-计划状态更新（wrecTask）

---

## 4. 条目类型速查（供世界书机制条目改造参考）

WREC 维护的两类可审计条目结构。改造世界书里的机制条目时，保留结构、区分「可改字段」与「逐字保留字段」即可。

**Type A · 计划条目**
- 结构：`【任务组】【状态】【卡点】【更新】【计划】【依赖】`
- 可改：`【状态】【卡点】【更新】`
- 逐字保留：`【任务组】【计划】【依赖】`
- `【状态】` 取值：待办／进行中／条件等待／阻塞／已完成／已取消／已归档

**Type B · 资产账本条目**
- 结构：`【账本类型】【现金】【近期收入】【近期支出】【待支付/预留】【已确认资产】【资金逻辑】【更新】`
- 可改：`【现金】【近期收入】【近期支出】【待支付/预留】【已确认资产】【更新】`
- 逐字保留：`【账本类型】【资金逻辑】`

STMB 抽取产出的条目类型：`计划/xxx`（进 WREC Type A 维护）、`能力/xxx`、`已知/xxx`。

---

## 附：复用提示（机制层 vs 世界特定）

以上提示词属**机制层**，原则上与具体世界无关、可复用于其他卡。目前仅有极少数世界特定痕迹，复用到别的世界时按需替换即可，不影响机制本身：

- 滚动总结【余震】里「只记 **Johnny** 当时主观在场…」中的 `Johnny` 是当前世界的 POV 角色名；换世界时替换为该世界的视角角色名（或改用 `{{char}}` 宏）。
- 滚动总结里的举例（提及克里／视线停在吉他上／把话题引向水晶宫）只是本世界的格式示意，仅作演示用，可保留或替换为通用示例。
- 三套抽取提示词与 WREC 规则均已用 `{角色}` / `{{...}}` 占位或通用措辞，无需改动即可跨世界复用。
