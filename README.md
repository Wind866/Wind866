# Wind866

A 股盘后自动化：定时抓行情 → 算观察池 / 出看板 → 推飞书。
所有时间均为北京时间，准点由 Cloudflare Worker 触发，GitHub cron 只作兜底。

## 在跑的项目

| 仓库 | 做什么 | 档期 | 看板 |
| --- | --- | --- | --- |
| [BIGApush](https://github.com/Wind866/BIGApush) | 早晚资讯、盘中观察池、收盘选股（15 种形态策略）、个股查询、档期对账 | 08:13 / 12:53 / 19:33 / 21:53 | — |
| [astock-screener](https://github.com/Wind866/astock-screener) | 股东增减持 / 定增 / 回购 / 分红 事件筛选看板 | 20:33 | [看板](https://wind866.github.io/astock-screener/) |
| [limit-up-relay](https://github.com/Wind866/limit-up-relay) | 涨停次日接力策略：情绪打分、接力观察池、收盘复盘 | 18:53 收盘定稿 | [看板](https://wind866.github.io/limit-up-relay/) |
| [etf-feishu-push](https://github.com/Wind866/etf-feishu-push) | 全市场 ETF 各指数主力净流入 → 近 10 日对照图 | 20:03 | — |

## 共同的工程约定

四个仓库共用同一套骨架，改任意一个时可参照其余三个：

- **数据日期 ≠ 运行日期**：cron 实测会迟到 3~10 小时，甚至跨过午夜。所有任务都按「行情时间戳里的日期」记账，跨夜只会延迟推送，不会丢档或错行。
- **推送幂等**：只有飞书确实返回成功才写台账（`.push-state/ledger.json`），同一档期重复触发会被秒级跳过。
- **降级要说出来**：数据源失效时卡片与日志会明确写出降级口径，不假装数据完整；未配 Webhook 一律判红，不允许「运行页绿色但群里没消息」。
- **漏跑可见**：档期对账每天核对「应到 / 实到」，缺口直接判红。
- **依赖可控**：Action 固定到 commit SHA + dependabot 每周提升级 PR；akshare 钉死版本，上游静默改字段会立刻暴露。
- **密钥零入库**：Webhook、加签密钥、App Secret 只走 GitHub Secrets；CI 带泄露扫描，误提交即判红。

## 约定俗成的排错顺序

1. 先看 Actions run 有没有变红（判红多数是 webhook / 加签 / 数据源）。
2. 再看 `.push-state/ledger.json` 是否把该档期记成已推（重复卡片多源于此）。
3. 最后才怀疑数据源本身——用各仓库的 `--dry-run` / `--demo` / `check-sources.yml` 分离取数问题与渲染问题。

> 所有内容仅供研究复盘，不构成投资建议。市场有风险，投资需谨慎。
