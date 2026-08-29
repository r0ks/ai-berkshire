# Sequential @mention Debate Pattern — Full Design

> Session: 2026-07-01 — Ai-Berkshire Telegram multi-bot design
> Updated: 2026-07-24 — added Stages 7-8 (report save + data audit)
> Reference: `~/GitHub/ai-berkshire/skills/earnings-team.md`, `~/invest_skills/`

## Architecture

```
Telegram Group "Ai-Berkshire 投资研究"
├── 👤 rrr (user)           ← initiates research
├── 🤖 @ab_moderator_bot    ← 🎯 Host / Team Lead
├── 🤖 @ab_duan_bot         ← 🏌️ 段永平 (business model)
├── 🤖 @ab_buffett_bot      ← 💰 巴菲特 (moat/financials)
├── 🤖 @ab_munger_bot       ← 🧠 芒格 (competition/risk)
└── 🤖 @ab_lilu_bot         ← 📚 李录 (civilization/long-term)
```

Each bot = one Hermes profile + independent Gateway systemd service.
All use long-polling (default). No webhook/nginx needed.

## Gateway Config (every profile)

```yaml
telegram:
  bot_token: "${TG_<ROLE>_TOKEN}"
  require_mention: true                    # Only respond when @mentioned
  observe_unmentioned_group_messages: true # See all messages as context
  exclusive_bot_mentions: true             # Don't respond when another bot is @mentioned
```

Source: `plugins/platforms/telegram/adapter.py` — `_should_process_message()` L6580,
`_telegram_observe_unmentioned_group_messages()` L5920,
`_explicit_bot_mentions_exclude_self()` L6199.

## Workflow (8 Stages)

### Stage 0: Silent Observation (Default)
All 5 bots in group. All can see every message. None speak unless @mentioned.

### Stage 1: User Initiates
```
User: @ab_moderator_bot 分析腾讯当前的投资价值

Host: 📊 收到。启动四大师投研辩论。
      标的：腾讯控股 (0700.HK)
      信息丰富度评级：A 级
      数据截止日期：2026-07-24  (run `date` before research — AGENTS.md rule)
      
      第一轮 · 独立分析
      @ab_duan_bot 请从生意本质角度分析腾讯
```

### Stage 2: Round 1 — Sequential Independent Analysis
Fixed order: 段永平 → 巴菲特 → 芒格 → 李录

Each expert: analysis (3-5 paragraphs) → ends with `@ab_moderator_bot 分析完成。`

Host response to each: `✅ 收到。{one-line summary}` then immediately @ next expert.

**Host anti-self-lock**: Never add follow-up analysis, questions, or commentary.

### Stage 3: Host Round 1 Synthesis
After all 4 experts done, Host identifies:
- Consensus points (all 4 agree)
- Contradictions (disagreements worth debating)
- Blind spots (topics no one mentioned)

### Stage 4: Round 2+ — Cross-Examination
Same order, but each expert is given the OTHER 3 experts' opinions.

Host prompt format:
```
@ab_duan_bot 第二轮。芒格认为竞争在恶化，巴菲特认为估值不便宜，
李录担忧管理层AI转型能力。请结合这些观点重新分析。
```

### Stage 5: Consensus Judgment
After each round (starting round 2):
- Directional consensus (bullish/bearish/buy/sell/hold) → STOP
- No consensus → continue (max 5 rounds)
- Round 5 reached → force termination, output with disagreement noted

### Stage 6: Final Report
Standard format: one-line conclusion, 4-master scoring table,
consensus vs disagreement, bull/bear thesis, buy checklist,
investment recommendation with price ranges, catalysts, AI limitations statement.

### Stage 7: Report Persistence (NEW in V3)
Host saves final report + all 4 expert analyses to disk:
```
reports/{公司名}/
├── 最终报告_{YYYYMMDD}.md
├── 段永平-生意本质_{YYYYMMDD}.md
├── 巴菲特-财务估值_{YYYYMMDD}.md
├── 芒格-竞争风险_{YYYYMMDD}.md
└── 李录-长期风险_{YYYYMMDD}.md
```

Directory layout matches ai-berkshire `reports/` convention.

### Stage 8: Data Audit (NEW in V3)
Host runs `report_audit.py` quality gate:
```bash
# Extract 15% random sample
python3 ~/GitHub/ai-berkshire/tools/report_audit.py extract \
  --report reports/{公司名}/最终报告_{日期}.md

# Validate each sampled data point from reliable sources

# Pass/fail verdict
python3 ~/GitHub/ai-berkshire/tools/report_audit.py verdict \
  --results '<filled JSON>' --report 最终报告_{日期}
```

Output in group: 📋 数据抽检：15%抽样，N项 → ✅ pass / ⚠️ flagged

## Host SOUL.md Additions

```markdown
## 【防自锁机制 — 最高优先级】

收到专家"分析完成"后：
1. 只回复"✅ 收到" + 一句话要点
2. 立刻 @ 下一位专家
3. 绝对禁止追问/补充分析/展开讨论
4. 只有当前轮次全部完成后，才能做整合分析
```

## Expert SOUL.md Additions

```markdown
## 群内称呼规则（最高优先级）

- ✅ 提到其他大师：纯文本名字（"巴菲特"、"芒格"、"李录"）
- ❌ 禁止 @username 形式提及他人
- ❌ 禁止 @ab_moderator_bot（除结尾"分析完成"信号外）

## 任务完成信号

最后一行必须：@ab_moderator_bot 分析完成。
```

## Exception Handling

| Scenario | Response |
|----------|----------|
| Expert timeout (>5 min) | Skip, continue to next |
| User @ host mid-debate | Host judges: respond or "辩论进行中" |
| [新任务] / [RESET] | Clear state machine, restart |
| Expert misses 3 consecutive rounds | Mark "absent this session" |
| 5 rounds no consensus | Force termination, note disagreement |
| Data audit fails | Report still output, flagged items noted |

## Comparison to Original Ai-Berkshire

| ai-berkshire | Telegram Design |
|-------------|-----------------|
| 4 agents PARALLEL (Task tool) | 4 bots SEQUENTIAL (@mention) |
| Internal SendMessage | Public Telegram group chat |
| Single round synthesis | Multi-round debate (max 5) |
| Team lead synthesizes once | Host integrates each round + judges consensus |
| Roles in Task prompts | Roles in Profile SOUL.md |
| report_audit.py post-report | Same (Stage 8) |
| reports/ directory output | Same (Stage 7) |
