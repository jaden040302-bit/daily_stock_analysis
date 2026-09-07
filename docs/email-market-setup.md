# 三市场邮件复盘部署

本文件针对当前个人 fork 的基础部署。使用现有行情、新闻、报告和邮件实现；不代表已增加全市场选股、港美股资金流或行业扫描。

## 先填写配置

仓库 Settings → Secrets and variables → Actions。

Secrets：
- `DEEPSEEK_API_KEY`：DeepSeek 官方平台创建的密钥。
- `EMAIL_PASSWORD`：发件邮箱的 SMTP 授权码／Gmail 应用专用密码，不是登录密码。

Repository variables：
- `LITELLM_MODEL`：`deepseek/deepseek-v4-flash`（示例，实际以官方模型列表和账户权限为准）。
- `EMAIL_SENDER`：发件邮箱。
- `EMAIL_RECEIVERS`：收件邮箱。
- `MARKET_REVIEW_REGION`：`cn,hk,us`，仅用于手动触发时的默认范围。
- `REPORT_LANGUAGE`：`zh`。

可选 Secrets：`TAVILY_API_KEYS` 或项目支持的其他新闻搜索服务密钥。仅有模型密钥不等于已经接入新闻搜索；新闻不足时应显示缺失信息，不能补造消息。

本 fork 工作流的基础预检接受 DEEPSEEK_API_KEY、OPENAI_API_KEY、GEMINI_API_KEY、ANTHROPIC_API_KEY、ANSPIRE_API_KEYS 或 AIHUBMIX_KEY 至少一个，并要求 LITELLM_MODEL 或 OPENAI_MODEL、邮件三项配置；复杂 Channels/YAML、本地模型等原项目高级方案需相应调整此预检，不在本次基础部署范围内。

官方入口：
- DeepSeek API Key：https://platform.deepseek.com/api_keys
- DeepSeek 模型说明：https://api-docs.deepseek.com/
- Google 应用专用密码：https://myaccount.google.com/apppasswords
- Google 说明：https://support.google.com/mail/answer/185833?hl=zh-Hans

## 运行安排

- 北京时间周二至周六 08:00 触发美股复盘（UTC cron `0 0 * * 2-6`），覆盖美国周一至周五，保留按纽约当地日期的交易日检查。
- 北京时间周一至周五 18:00 触发 A股和港股复盘（UTC cron `0 10 * * 1-5`）。
- 定时任务固定为 market-only。暂未设置自选股，不会把示例贵州茅台当作用户自选股。未来添加自选股时还需配置按市场分组的定时个股分析；目前 STOCK_LIST 配好后可手动运行 full 或 stocks-only。
- 各市场休市按现有日历跳过；原日历接口不可用时为 fail-open，应检查日志与数据日期，不把旧行情当新行情。
- 08:00/18:00 是触发时间，GitHub 排队、安装依赖和分析耗时会使邮件晚于该时刻到达。美股早报目前是美股复盘，不包含新增的 A/H 盘前跨市场推演模块。
- 报告日期可能是生成日；美股交易日期应以数据内时间戳为准。

## 验证与启用

1. 合并部署变更后填写上面的 Secrets / Variables。
2. 打开 Actions 并启用 fork 的工作流。
3. 选择 每日股票分析 → Run workflow → market-only。休市测试可显式选 force_run，报告须按实际数据日期阅读。
4. 核查模型鉴权、新闻源状态、各市场报告及邮件实际到达情况。
5. 若测试失败，不视为部署成功；查看日志解决后再次验证。

公开仓库不要提交邮箱授权码、API 密钥、持仓成本或账户资金。邮件地址用 Variables/Secrets 配置，不写入公开代码。

## 数据边界与下一阶段

当前项目港美股没有完整板块榜与资金流数据。本次不新增数据服务、不承诺具体买卖点有效性。下一阶段补充行业 ETF 相对强度、量能持续性、可核实资金指标、来源和失效条件；成交额增长不能直接称为新增资金流入。

## 验证范围与回滚

本次离线检查 YAML、嵌入 shell 语法、定时市场路由和无自选股行为；缺少用户密钥，未执行真实模型调用、行情抓取和 SMTP 投递。尚未进行 GitHub Actions 端到端验收。

暂停：Actions 中 Disable workflow。回滚：还原本次工作流改动。此说明仅服务当前中文个人 fork，未同步英文部署文档；通用项目行为未修改。
