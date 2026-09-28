# 数据源（2026-09-29 实测修订）

## 核心：东财公告列表 → 正文抓取接口链（实测可用）

请求时带桌面端 UA，Referer 头统一为 `https://data.eastmoney.com/`。

### 第 1 步：公告列表接口（拿 art_code）

```
Referer: https://data.eastmoney.com/
GET https://np-anotice-stock.eastmoney.com/api/security/ann?sr=-1&page_size=20&page_index=1&ann_type=A&client_source=web&stock_list={纯数字代码}
```

- `stock_list` 必须用**纯数字代码**（如 `600519`），不带 `.SH`/`.SZ` 后缀；多家用逗号分隔。
- 返回 `data.list[]`，按标题关键词定位目标定期报告：
  - 年报：标题含"年度报告"
  - 半年报：标题含"半年度报告"
  - 季报：标题含"季度报告"（注意区分"第一季度报告"/"第三季度报告"/"半年度报告"）
- 关键字段：`art_code`（如 `AN202608071835112345`，正文抓取的唯一钥匙）、`title`、`display_time`（发布时间）。
- 若列表页数不够（定期报告发布较早），增大 `page_size`（如 100）或翻 `page_index` 继续找。

### 第 2 步：正文抓取接口（拿报告正文）

```
Referer: https://data.eastmoney.com/
GET https://np-cnotice-stock.eastmoney.com/api/content/ann?art_code={art_code}&client_source=web&page_index=1
```

- 返回 JSON：
  - `data.notice_content`：HTML 富文本，需 strip 标签后再提炼数字（定期报告正文很长，建议先按"主要会计数据和财务指标""财务报告"等章节关键字切分再提取）；
  - `data.page_size`：**>1 时必须循环 `page_index` 逐页抓全**，定期报告类正文普遍分页，不翻页会漏掉"财务报告"章节的明细数字；
  - `data.attach_list[].attach_url`：PDF 备选，正文接口抓不到时下载 PDF 解析。
- 抓取成功判定：`success=1` 且 `notice_content` 非空；否则视为失败，执行降级（PDF 备选 → 网页搜索）。
- 实测注记（2026-09-29 贵州茅台 2026 年半年报 `AN202608141827994408`）：
  - `page_size=44`，逐页请求约 1.3s/页，全量抓完约 1 分钟；时间紧可先抓第 1 页（摘要页）做"未核验"标注的快拆，或只翻到"主要会计数据和财务指标""财务报告"章节所在页。
  - strip 标签后表格结构丢失，分产品/分渠道明细数字会散成数列、难以配对行列名；**分产品表建议改用 PDF 备选（`attach_list[].attach_url`）逐页核对，或按关键字上下文逐段人工配对**。
- **不要直接 curl `https://data.eastmoney.com/notices/detail/{代码}/{art_code}.html`**：那是 JS 空壳，纯服务端请求拿不到正文（人工浏览器阅读可走该链接）。

### 备选（正文接口失败时）

1. 巨潮资讯历史公告查询（POST）：`http://www.cninfo.com.cn/new/hisAnnouncement/query`
   form: `stock={纯数字代码}&tabName=fulltext&pageSize=20&pageNum=1&column={sse|szse|bj}&plate=&seDate=&searchkey=`
   注意：2026-09-29 实测按该参数返回 0 条（`totalRecordNum=0`），疑似参数格式已变更，**恢复前不要依赖**。
2. 上交所定期报告栏目：`http://www.sse.com.cn/disclosure/listedinfo/regular/`（页面读取，需浏览器环境）。
3. 深交所公告接口（POST）：`http://www.szse.cn/api/disc/announcement/annList`
   注意：2026-09-29 实测纯服务端 POST 被 WAF 拦截（返回 50x 错误页），需浏览器环境或完整请求头。

## 数字交叉：网页搜索模板

报告正文数字不足或缺失时，用网页搜索补足并至少两源交叉：

- `{公司} {2026年半年报} 营收 归母净利润 毛利率`
- `{公司} {2026年中报} 分红 每10股派息`
- `{公司} {2026年半年报} 经营活动现金流净额 收现比`

取权威财经媒体（格隆汇、新京报、证券时报·券中社、第一财经、东方财富网、新浪财经），注明来源；媒体间数字矛盾时以报告正文为准并注明差异。

## 兜底规则

接口失败或数据对不上时，一律用网页搜索补足并注明来源；补不到的标注"未核验"，不编造数字。
