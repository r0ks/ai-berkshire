# Hermes Stock Profile — 工作交接日志

> 归档日期：2026-08-29
> 来源 Profile：`stock`（原名 `devops`）
> 说明：本目录记录 stock profile 中与 ai-berkshire 项目相关的全部记忆与技能，供后续接手机器人使用。
> 交接性质：profile 暂时 freeze，工作移交至另一机器人。

---

## 一、持久记忆（MEMORY.md）

来源：`~/.hermes/profiles/stock/memories/MEMORY.md`

### 记忆 1 — 工作范围

> 工作范围限定在 stock profile（~/.hermes/profiles/stock/）。原名 devops，已重命名为 stock。其它 profile（default、chh 等）不在工作范围内，不关注不操作。当前 stock profile 配置模型 deepseek-v4-pro，通过 Telegram 连接，Gateway systemd 服务 hermes-gateway-stock.service。ai-berkshire 项目路径 ~/GitHub/ai-berkshire/，投资角色技能文件路径 ~/invest_skills/。

### 记忆 2 — Telegram 群组多 Bot 投研辩论系统架构决策

> Telegram 群组多 Bot 投研辩论系统架构决策：5 个独立 Bot（1 主持人+4 大师），全部使用长轮询（不用 webhook/nginx）。Gateway 层必须配置 require_mention:true + observe_unmentioned:true + exclusive_bot_mentions:true。专家顺序发言（非并行），@mention 触发。专家之间禁止用 @username 互相称呼。主持人有防自锁机制（收到专家完成信号后只回复"收到"+调度下一位，禁止追问）。完整设计方案在 ~/telegram-group-debate-design-v3.md。

## 二、用户画像（USER.md）

来源：`~/.hermes/profiles/stock/memories/USER.md`

> Prefers a design-first workflow for architectural changes: present detailed design proposal → user reviews and gives second confirmation → only then proceed with implementation. Do not jump to config edits or file changes during the design phase — the user will explicitly say when to proceed with implementation.

## 三、关联技能（telegram-multi-bot）

来源：`~/.hermes/profiles/stock/skills/software-development/telegram-multi-bot/`

| 文件 | 用途 |
|------|------|
| `SKILL.md` | Telegram 多 Bot 架构设计技能（Bot API 10.0 B2B、Gateway 集成、安全防护） |
| `references/sequential-debate-pattern.md` | 顺序 @mention 多轮辩论模式完整设计（8 阶段工作流） |
| `references/investment-data-tools.md` | 投资数据工具按角色分配（ashare_data.py / financial_rigor.py） |

**技能要点**：
- Bot API 10.0 (2026-05-07) 开启 Bot-to-Bot Communication Mode 后，群组内 bot 可见其他 bot 消息
- Gateway 三层过滤：`require_mention` / `observe_unmentioned_group_messages` / `exclusive_bot_mentions`
- 每 Bot 独立 Profile + 独立 Gateway systemd 服务 + 长轮询
- 专家发言结尾信号：`@ab_moderator_bot 分析完成。`
- 主持人防自锁机制（最高优先级指令）
- 群内禁止 @username 互相称呼（防抢话）
- 最多 5 轮辩论，无共识则强制终止

## 四、投资角色技能文件（invest_skills）

来源：`~/invest_skills/`

| 文件 | 角色 | 行数 | 核心能力 |
|------|------|:---:|------|
| `duan-yongping.md` | 段永平 | 221 | 生意本质、商业模式、管理层 |
| `warren-buffett.md` | 巴菲特 | 319 | 护城河、财务质量、估值、数据工具 |
| `charlie-munger.md` | 芒格 | 235 | 逆向思考、竞争格局、行业对比 |
| `li-lu.md` | 李录 | 240 | 文明趋势、长期风险、管理层信号 |
| `team-lead.md` | 主持人 | 258 | 调度、汇总、共识判断、数据校验 |

每文件结构：身份 → 核心框架 → 人格特征 → 群聊互动规则 → 数据获取能力 → 原始来源

## 五、设计方案文档

- `~/telegram-group-debate-design-v3.md` — Telegram 群组六角色多轮辩论投研系统完整设计（8 阶段 + 报告保存 + 数据抽检）
- `~/GitHub/ai-berkshire/skills/investment-team.md` — 参考项目团队协作框架
- `~/GitHub/ai-berkshire/skills/earnings-team.md` — 参考项目财报精读团队框架

## 六、环境快照（2026-08-29）

| 项目 | 状态 |
|------|------|
| Hermes Agent | v0.19.0 (2026.7.20) · upstream a61183b5 |
| Python | 3.11.15 |
| 当前模型 | deepseek-v4-pro（provider: deepseek） |
| 当前 Profile | stock（原 devops） |
| Gateway 服务 | hermes-gateway-stock.service / chh / housing |
| ai-berkshire | 已同步至 58d09b1（2026-08-29） |
| 5 个 ab-* Profile | 未创建（V3 待实施） |
| 5 个 B2B Bot Token | 未配置 |
| Cron jobs | 0 |

## 七、待办（供后续机器人执行）

1. `git pull` ai-berkshire 已完成，tools/ 已含 ashare_data.py 52 周高低点修复
2. 创建 5 个 Hermes Profile：ab-moderator / ab-duan / ab-buffett / ab-munger / ab-lilu
3. @BotFather 注册 5 个 Telegram Bot 并开启 B2B Communication Mode
4. 每个 Profile 写入 config.yaml（require_mention / observe_unmentioned / exclusive_bot_mentions 三键）
5. 每个 Profile 的 SOUL.md 从 ~/invest_skills/*.md 生成
6. 部署 5 个 Gateway systemd 服务（长轮询）
7. 群组集成测试（mention 过滤、全流程辩论、报告保存、数据抽检）
