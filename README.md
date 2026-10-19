# airmon-sidebar — DSH 机场监控侧边栏面板

DeepSeek Harness（DSH）侧边栏插件：把本机 **airmon 机场监控管线**（订阅采集 → 快照 diff → 关联检测 → 探活 → 基建画像 → 套餐到期）可视化为一个 9-tab 面板。

> 数据永远在本机生成、本机渲染。插件不打包任何监控数据，也不上传任何东西。

## 功能（v6）

| Tab | 内容 |
|---|---|
| 总览 | KPI 台账 + 机场卡片（点击弹窗：节点表/真实 IP/16轮-24h-7d-30d 走势/探活/基建画像/推荐站评测联动） |
| 到期 | 套餐过期/30 天内到期（数据源 `subscription-userinfo` 响应头） |
| 告警 | 告警流（空块过滤）+ 搜索过滤 |
| 华为 | 华为云基建关联命中 + 台账 |
| 节点 | 全部节点总表（搜索/机场筛选/IP 列） |
| 机场库 | 监控清单 + 免费候选存活监测 + 推荐站评测档案 + 一键接入 |
| 知识 | 协议科普/行业观察离线知识库（AnyTLS/Hysteria2/Reality/VLESS/Trojan/六协议对比/拔线潮/倍率/防骗） |
| 分析 | 30 天走势/协议分布/地区分布/churn/0 拉取榜/RDAP 集中度/双机场对比图 |
| 运行 | run.log 尾部 + 最近一轮成功时间 |

特色：探活徽标（TCP 抽样握手 + RTT）、基建画像评分（端口扇出/高位端口/伪 TLD/RDAP 集中）、推荐站档案 ↔ 监控详情双向跳转。

## 安装

```bash
# 方式一：plugin_manager（推荐）
plugin_manager { "action": "install_bundle", "target": "<本目录绝对路径>" }
plugin_manager { "action": "set_plugin", "target": "@local/airmon-sidebar", "enabled": true }

# 方式二：手工
# 复制本目录到 DSH profile 的 node_modules/@local/airmon-sidebar，
# 并在 profile 的 cordis.patch.yml 里加：
# - insert:
#     - id: airmon-sidebar
#       name: '@local/airmon-sidebar'
```

安装后在 DSH Web GUI 左轨出现「机场监控」图标。`index.js` 的路由变更需重启 DSH；`logic.js`/`panel.html` 热更。

## 数据源（环境变量）

| 变量 | 默认 | 说明 |
|---|---|---|
| `AIRMON_DASHBOARD` | `…/dashboard/index.html` | 仪表盘 HTML（内嵌 payload 解析） |
| `AIRMON_SITES_DIGEST` | `…/data/sites_digest.json` | 推荐站评测档案 |
| `AIRMON_SITEWATCH` | `…/data/sitewatch.json` | 免费候选存活 |
| `AIRMON_PROBE` | `…/data/probe.json` | TCP 探活结果 |
| `AIRMON_IDC` | `…/data/idc_profile.json` | 基建画像 |
| `AIRMON_LASTRUN` | `…/data/last_run.json` | 最近一轮运行台账 |
| `AIRMON_KNOWLEDGE` | `…/data/knowledge.json` | 知识库 |

面板读取不到数据时显示「模板版」提示，不影响安装。独立预览：`GET /airmon-sidebar/api/data` 导出 JSON 存为包内 `data.json`。

## 安全设计

- **白名单透出**：logic.js 对每个数据文件只透出显式列出的键；快照 `raw`（password/_rdap/source）与订阅 URL 永不出 API。
- **负向自检**：各数据生成端写盘前扫描 `token=`/`eyJ…`（JWT）/赋值型 `password`，命中即中止。
- **零内嵌数据**：发布版不打包任何节点地址/机场清单；`data.json` 与数据文件均不入库（见 .gitignore）。
- **API 栅栏**：仅 GET + 同源 iframe（sec-fetch-site 校验 + loopback Host），POST 一律 403。

## 生效规则

| 文件 | 改动后 |
|---|---|
| logic.js / panel.html / data.json | 热更（下次请求生效） |
| client.js | 刷新 GUI 页面 |
| index.js / package.json / cordis.patch.yml | 重启 DSH |

## 卸载

```bash
plugin_manager { "action": "set_plugin", "target": "@local/airmon-sidebar", "enabled": false }
plugin_manager { "action": "remove_bundle", "target": "@local/airmon-sidebar" }
```

无持久化用户数据，卸载即净。

## 免责声明

本项目用于监控**自己购买/自有**的机场订阅与节点资产，请勿用于探测、扫描或干扰第三方服务。推荐站评测档案版权归原站，此处仅作个人离线摘要。

## License

MIT
