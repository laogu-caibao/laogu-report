# 定期报告深拆

`laogu-report`

定期报告深拆 skill：输入公司名称/代码 + 报告期，逐项拆解年报/半年报/季报。

## 一键安装

仓库地址（点击复制）：

`https://github.com/laogu-caibao/laogu-report`

**方式一：克隆**

```bash
git clone https://github.com/laogu-caibao/laogu-report.git
```

**方式二：下载 ZIP**

https://github.com/laogu-caibao/laogu-report/archive/refs/heads/main.zip

**导入使用**

- Claude Code / Muse：把仓库中的 `SKILL.md` 放到 `~/.claude/skills/laogu-report/` 下即可调用。
- 豆包智能体 / Workbuddy 等：按各平台的 skill 上传流程导入 `SKILL.md`。
- 一次装好全部 16 个：用 [laogu-mcp](https://github.com/laogu-caibao/laogu-mcp)，`uvx laogu-mcp` 一键安装。
## 文件结构

- `SKILL.md` — 主流程（平台中立，需用户输入公司+报告期，不自动运行）
- `references/sources.md` — 数据源：东财公告列表→正文抓取接口链、巨潮/上交所/深交所备选、网页搜索交叉模板

## 输出结构

- 报告摘要（报告期、营收、归母净利、毛利率、同比）
- 利润表拆解（营收结构、毛利率、期间费用率、归母净利同比与归因）
- 资产负债表关键项（商誉/净资产、存货、应收账款/合同负债、货币资金 vs 有息负债）
- 现金流勾稽（经营现金流 vs 净利润、收现比）
- 杜邦三因子（ROE=净利率×周转率×杠杆，附去年同期对比与变化归因）
- 分红方案（每10股派息、股息率、分红比例）
- 风险点清单（3-5 条）

## 使用方式

调用时提供：公司名称/代码 + 报告期，例如"贵州茅台 2026 年半年报"。

---
## 出品

**老谷拆财报** —— 以数据为刃，剖市场真相

- 抖音 / 微信视频号 / 今日头条 / 快手：搜索「老谷拆财报」
- 固定栏目：「价值投资之财报解读」（全网连载中）
- 本 skill 的方法论与账号内容同源：数据驱动、拆开看、不讲黑话

### 扫码关注

| 微信视频号 | 抖音 |
|---|---|
| ![视频号二维码](docs/qrcode-shipinhao.jpg) | ![抖音二维码](docs/qrcode-douyin.png) |
| 扫一扫，关注视频号 | 抖音号：gubaobao22 |

> 作者声明：个人观点，仅供参考，不构成投资建议。
