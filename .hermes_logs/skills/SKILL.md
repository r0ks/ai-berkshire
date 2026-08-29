---
name: telegram-multi-bot
description: "Design multi-bot architectures on Telegram using Bot API 10.0 Bot-to-Bot Communication Mode — patterns, Hermes Gateway integration, routing rules, and safe-guards."
version: 1.0.0
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [telegram, multi-agent, bot, gateway, architecture, design]
    related_hermes_issues: [10452, 21587]
---

# Telegram Multi-Bot Architecture

Design and implement multi-bot collaboration systems on Telegram, leveraging the Bot API 10.0 (May 2026) **Bot-to-Bot Communication Mode** capability. This skill covers architecture patterns, Hermes Gateway integration, routing rules, and safety guards for autonomous multi-agent workflows on Telegram.

## Trigger Conditions

Load this skill when:
- Designing a multi-bot Telegram group with collaborative AI agents
- Implementing bot-to-bot communication in a Hermes Gateway Telegram deployment
- Planning multi-profile/specialized-bot architectures on Telegram
- User asks about "Telegram 群组多 bot 协作" or similar multi-bot patterns
- Any design discussion involving multiple Telegram bots coordinating in a group

## Key API Capability: Bot API 10.0 Bot-to-Bot Communication

Telegram Bot API 10.0 (released May 7, 2026) removed the long-standing restriction that "bots will not be able to see messages from other bots regardless of mode." The new opt-in **Bot-to-Bot Communication Mode** enables:

| Capability | Condition | Mechanism |
|-----------|-----------|-----------|
| Receive other bots' messages in groups | Bot is group admin **OR** Privacy Mode disabled | `message.from.is_bot == true` in updates |
| Receive via @mention only | Default (Privacy on, non-admin) | Standard mention-based routing |
| Bot-to-bot DM | Both bots have B2B Mode enabled | `sendMessage(chat_id="@other_bot", text="...")` |
| Guest Mode interaction | Bot not in group, summoned by @mention | Companion Bot API 10.0 feature |

**Enabling**: Go to @BotFather → select bot → Bot Settings → enable "Bot-to-Bot Communication Mode". Both sender and recipient bots must enable it.

**Identifying bot messages**: After B2B Mode is enabled, the existing `getUpdates`/webhook stream delivers `Update` objects with `message.from.is_bot = true`.

## User Workflow Preference

This user prefers a **design-first, confirm-then-implement** workflow for architectural changes:
1. Present a detailed design proposal (no code/config changes yet)
2. User reviews and provides feedback / second confirmation
3. Only after confirmation: proceed with implementation

Never jump straight to `hermes config set` or file edits during the design phase. The user will explicitly say when to proceed.

## Architecture Patterns

### Pattern 1: Orchestrator + Specialized Workers

One "master" bot (Orchestrator) serves as the user's single interaction point. Specialized worker bots handle sub-tasks. The Orchestrator is the only bot that talks to the user; workers communicate only within the bot mesh.

```
User ←→ Orchestrator (profile: devops)
              ├── Researcher (profile: research)  — web search, papers, docs
              ├── Coder      (profile: coding)    — code gen, review, debug
              └── Ops        (profile: ops)        — deploy, monitor, manage
```

- Orchestrator has `group_privacy: disabled` (sees all messages, including other bots')
- Workers have `mention_gating: true` (only respond to @mention or reply)
- Orchestrator uses `@worker_bot` mentions to delegate in public, or DM for sensitive data
- All interaction visible in the group for user auditability

### Pattern 2: Peer Mesh (All-Visible)

All bots see all group messages. Any bot can respond to any relevant query. No central orchestrator; the bots self-organize.

- All bots are group admins (or all have Privacy Mode disabled)
- All bots have B2B Mode enabled
- Each bot uses `mention_gating: true` to avoid responding to everything
- Requires strong loop prevention (see Safety Guards)

### Pattern 4: Sequential @mention Debate (Multi-Round Consensus)

**Use case**: Investment research, policy analysis, or any scenario where
multiple specialized AI personas must analyze the same topic, debate
disagreements, and converge toward consensus — all visible in a Telegram group.

**Key design principles**:
- One Host bot orchestrates; N Expert bots analyze on demand
- Experts are called **sequentially** (not parallel) via @mention
- Each expert speaks only when @mentioned (Gateway-enforced)
- Multi-round: Round 1 = independent analysis; Round 2+ = cross-examination
- Consensus check after each round; max 5 rounds then forced termination

**Group composition** (N=4 experts example):

| Role | Username | Profile | Function |
|------|----------|---------|----------|
| 🎯 Host | `@ab_moderator_bot` | `ab-moderator` | Orchestrates, consolidates, judges consensus |
| Expert 1 | `@ab_expert1_bot` | `ab-expert1` | Domain analysis (called first each round) |
| Expert 2 | `@ab_expert2_bot` | `ab-expert2` | Domain analysis (called second) |
| Expert 3 | `@ab_expert3_bot` | `ab-expert3` | Domain analysis (called third) |
| Expert 4 | `@ab_expert4_bot` | `ab-expert4` | Domain analysis (called fourth) |

**Round 1 flow** (independent analysis):
```
User: @ab_moderator_bot 分析腾讯
Host:  @ab_expert1_bot 请从X角度分析
Expert1: [...analysis...] @ab_moderator_bot 分析完成。
Host:  ✅ 收到。@ab_expert2_bot 请从Y角度分析
Expert2: [...analysis...] @ab_moderator_bot 分析完成。
Host:  ✅ 收到。@ab_expert3_bot ...
...[all 4 experts complete]...
Host:  📊 第一轮整合 [summarizes, identifies conflicts]
```

**Round 2+ flow** (cross-examination):
```
Host:  @ab_expert1_bot 第二轮。Expert2认为X，Expert3担心Y，Expert4预测Z。
       请结合这些观点重新分析。
Expert1: [responds to others' points + updated analysis] @ab_moderator_bot 分析完成。
...[same for all 4 experts]...
Host:  📊 共识判断 [checks alignment on bullish/bearish/buy/sell direction]
       → Consensus → STOP, output final report
       → No consensus → continue to next round (max 5)
```

**Host anti-self-lock rule** (SOUL.md, highest priority):

When the Host receives an expert's "分析完成" message, it MUST:
1. Reply ONLY "✅ 收到。" + one-line summary
2. Immediately @mention the NEXT expert
3. **NEVER** add follow-up analysis, questions, or commentary on the expert's content
4. Only synthesize after ALL experts in the current round have spoken

Without this rule, the Host engages in infinite back-and-forth with each expert,
never progressing to the next one.

**Consensus judgment criteria**:
- Directional alignment: all experts agree on bullish/bearish/buy/sell/hold
- No fundamental contradictions (e.g. "moat is widening" vs "competitive advantage is eroding")
- If 5 rounds reached without consensus → forced termination, output report with disagreement noted

Full design with SOUL.md templates, config snippets, exception handling, report persistence (Stage 7), and data audit gate (Stage 8):
see `references/sequential-debate-pattern.md`.

## Investment Research Data Tools

When the sequential debate pattern is used for investment research, each role
needs a different subset of data acquisition tools. Two zero-dependency Python
tools from the Ai-Berkshire project integrate via Hermes's `terminal` toolset:

| Tool | Function | Data Source |
|------|----------|-------------|
| `ashare_data.py` | Real-time quotes, financials, valuation, stock search | 腾讯行情 API + 东方财富 |
| `financial_rigor.py` | Exact decimal: market cap, valuation, cross-validation, 3-scenario, Benford | Python Decimal engine (precision=28) |

**Per-role tool allocation**:

| Role | Tools | Rationale |
|------|-------|-----------|
| 巴菲特 | terminal (full suite) | PE/PB/ROE, three-scenario, cross-validation — data-driven core |
| 芒格 | terminal (financials + calc) | Competitor comparison, growth rate verification |
| 李录 | terminal (financials + calc) | Risk signal quantification (receivables, inventory, goodwill) |
| 段永平 | web only | Philosophy is "毛估估" — forcing data tools breaks character |
| 主持人 | terminal (calc only) | Currency unification for final report, data consistency checks |

**Critical rules**:
- **Never mental math** — all valuation calculations go through `financial_rigor.py`'s exact decimal engine
- **Cross-validate** — every key metric from two independent sources, deviation ≤ 1%
- **Currency conversion** — annotate with rate, source, date; use `calc` not floating-point
- **USD/HKD linked rate**: 7.75-7.85 narrow band simplifies HK↔US conversion

Full tool command reference, cross-validation tables, invest_skills file structure, and ai-berkshire sync updates (ashare_data.py 52-week fix, 复权 rules, new tools/skills):
see `references/investment-data-tools.md`.

## Hermes Gateway Integration

### Current Architecture: Independent Gateway Instances (Production-Proven)

As of Hermes Agent v0.17, the Telegram adapter does NOT support single-gateway
multi-bot-token injection. The production-proven pattern is **one Gateway
instance per Bot profile**, each with its own systemd service. All instances
use **long-polling** (`getUpdates`) by default — the Hermes Telegram adapter
sets ``self._webhook_mode = False`` and only switches to webhook when
``TELEGRAM_WEBHOOK_URL`` is explicitly set. No webhook, nginx, or public port
exposure is needed.

```
Profile: ab-moderator → Gateway systemd service (long-polling, bot token A)
Profile: ab-duan      → Gateway systemd service (long-polling, bot token B)
Profile: ab-buffett   → Gateway systemd service (long-polling, bot token C)
...same per bot
```

### Gateway-Level Mention Filtering (Critical Config)

**Source**: ``plugins/platforms/telegram/adapter.py``, three production config keys
that operate at the Gateway layer (before any message reaches the Agent):

| Config key | Type | When true | Source (line) |
|-----------|------|-----------|---------------|
| `require_mention` | bool | Group messages only trigger Agent dispatch when bot is @mentioned, replied-to, or matches regex patterns | ``_should_process_message()`` L6580 |
| `observe_unmentioned_group_messages` | bool | Messages that don't trigger dispatch are STILL recorded in the session transcript (bot can "see" without responding) | ``_telegram_observe_unmentioned_group_messages()`` L5920 |
| `exclusive_bot_mentions` | bool | When message contains @other_bot but NOT @this_bot → skip entirely. Prevents bot A from responding when bot B is being addressed | ``_explicit_bot_mentions_exclude_self()`` L6199 |

**Standard config per bot profile** (applies to ALL bots in a multi-bot group):

```yaml
telegram:
  bot_token: "${TG_<ROLE>_TOKEN}"
  require_mention: true                    # Never speak unless @mentioned
  observe_unmentioned_group_messages: true # See everything as context
  exclusive_bot_mentions: true             # Don't respond when another bot is @mentioned
```

**Behavior matrix** with these three keys active:

```
Group message                        Bot behavior
───────────────────────────────────  ─────────────────────
"今天天气不错"                       → Observed, no response (require_mention)
主持人: "@ab_duan_bot 请分析"        → TRIGGERED (only @ab_duan_bot mentioned)
主持人: "@ab_buffett_bot 请分析"      → IGNORED (exclusive_bot_mentions — different bot)
巴菲特: "护城河确实宽..."             → Observed, no response (require_mention)
```

Result: **only one bot ever speaks at a time**, enforced at Gateway level, not prompt level.

### No-@mention Rule for Bot Output

Since `exclusive_bot_mentions` filters incoming messages but NOT outgoing, each
bot's SOUL.md MUST include a hard rule: when mentioning other bots in replies,
use **plain-text names** (e.g. "巴菲特认为") — **never** ``@ab_buffett_bot``.
Using @username in bot output would trigger Telegram's mention delivery to the
target bot, bypassing the exclusive routing gate and creating contention.

This is enforced at the prompt level — the Gateway cannot filter the bot's own
outgoing text for @mentions before Telegram delivers them.

## Safety Guards (Mandatory)

### Gateway-Level (config.yaml on every bot)

The **primary safety layer** is Gateway config, not prompt-level rules:

| Guard | Config key | Effect |
|-------|-----------|--------|
| Mention-only dispatch | `require_mention: true` | Bot never speaks unless @mentioned |
| Context observation | `observe_unmentioned_group_messages: true` | Bot sees all messages but stays silent |
| Exclusive routing | `exclusive_bot_mentions: true` | Ignores messages that @mention other bots |
| Long-polling (no webhook) | Default | No public endpoint, no port exposure |

### Prompt-Level (SOUL.md on every bot)

Gateway config gates incoming dispatch, but **outgoing** text still passes
through Telegram's mention delivery system. Prompt-level rules are needed:

1. **No-@mention rule**: When a bot mentions another bot in its reply, use
   plain-text names only ("巴菲特认为...") — NEVER ``@ab_buffett_bot``.
2. **Host anti-self-lock**: Host must not engage in follow-up discussion with
   an expert — only acknowledge and move to the next expert.
3. **Completion signal**: Experts must end with ``@host_bot 分析完成。`` as
   a parseable signal for the Host's state machine.

### Loop Prevention

The sequential @mention pattern is naturally loop-resistant because experts
never @mention each other. Additional safeguards:

1. **Max 5 debate rounds** — Host forces termination after round 5
2. **Per-expert timeout** — If expert doesn't respond within 5 minutes, skip
3. **Chain depth limit**: Max 3 consecutive bot messages without user intervention

## Related Hermes Issues

- [#10452](https://github.com/NousResearch/hermes-agent/issues/10452) — Multi-bot Telegram support for gateway routing
- [#21587](https://github.com/NousResearch/hermes-agent/issues/21587) — Telegram Guest Bots + Bot-to-Bot feature request

## References

- `references/bot-api-10-b2b-specs.md` — Detailed Bot API 10.0 B2B specification notes
- `references/sequential-debate-pattern.md` — Full design: sequential @mention multi-round debate pattern with SOUL.md templates, Gateway config, exception handling, and comparison to original Ai-Berkshire reference project
