# OpenCLI · 美国经济分析局数据采集台

> 天津财经大学统计学院 · 龚凤乾  
> 基于 Netlify Functions 的 BEA EasyData API 无服务器代理前端

---

## 功能概述

本平台对接美国经济分析局（Bureau of Economic Analysis, BEA）官方 API，提供以下功能：

- **指标目录浏览**：获取 BEA 各数据集下的表名列表，全部译为中文（关键词映射）
- **数据查询**：按表名（TableName）、行代码（LineCode）、频率（年/季/月）检索具体统计数据，起始年份为 1929 年
- **图表渲染**：Chart.js 折线图，含最新值、最大值、最小值、均值摘要卡片
- **数据管道**：一键将查询结果传至 [CLI-Anything 统计多工具台](https://cli-gong.netlify.app)（URL `?pipeline=` 参数）
- **CSV 导出**：带 UTF-8 BOM，Excel 直接打开无乱码

预置快捷指标（均以 1929 年为起始点）：

| 命令 | 说明 | 表名 · 行 |
|------|------|-----------|
| GDP（年度） | 国内生产总值 | T10101 · Line 1 |
| GDP（季度） | 国内生产总值 | T10101 · Line 1 · Q |
| PCE | 个人消费支出 | T10101 · Line 2 |
| GDI | 国内收入总额 | T10301 · Line 1 |
| 净出口 | 商品与服务净出口 | T10101 · Line 15 |
| PCE 价格指数 | 通胀指标 | T20804 · Line 1 |
| GDP 平减指数 | GDP 隐含价格指数 | T10109 · Line 3 |
| 个人收入 | 个人收入 | T20100 · Line 1 |
| 个人储蓄率 | 储蓄占可支配收入% | T20600 · Line 1 |
| 公司利润 | 税前公司利润 | T61000 · Line 1 |
| 国内私人总投资 | 固定投资+库存 | T10101 · Line 7 |
| 政府支出 | 政府消费+投资 | T10101 · Line 22 |

---

## 项目结构

```
opencli-bea-stats/
├── index.html                      # 前端主页面
├── netlify/
│   └── functions/
│       └── bea-proxy.js            # 无服务器代理函数
├── package.json
├── netlify.toml
└── README.md
```

---

## 部署步骤

### 1. 申请 BEA API Key

前往 [https://apps.bea.gov/api/signup/](https://apps.bea.gov/api/signup/) 免费注册，获取 API Key（即时发放）。

### 2. 部署至 Netlify

**方式 A：通过 Netlify CLI**

```bash
npm install
npm run dev          # 本地开发预览（http://localhost:8888）
netlify deploy --prod
```

**方式 B：通过 Netlify 网站**

1. 将项目推送至 GitHub 仓库
2. 在 Netlify Dashboard → "New site from Git" 导入仓库
3. Build command 留空，Publish directory 填 `.`

### 3. 设置环境变量

在 Netlify Dashboard → Site settings → Environment variables 中添加：

```
BEA_API_KEY = 你的BEA API Key
```

**注意**：未设置 `BEA_API_KEY` 时，代理函数将返回 503 错误，前端会提示配置缺失，无法正常查询。

---

## 代理函数说明（bea-proxy.js）

函数统一挂载于 `/.netlify/functions/bea-proxy`，支持三种调用模式：

| 参数 `?m=` | 说明 | 关键参数 |
|-----------|------|---------|
| `getDatasets` | 获取所有数据集列表 | — |
| `getTree` | 获取指定数据集的参数枚举值 | `dataset`, `paramName` |
| _（默认）_ | 查询具体统计数据 | `dataset`, `tableName`, `frequency`, `year`, `lineCode` |

请求头自动注入：
- `Referer: https://apps.bea.gov/`
- `User-Agent: Mozilla/5.0 (compatible; OpenCLI-BEA/1.0; ...)`

响应头：
- `Access-Control-Allow-Origin: *`（宽松 CORS，允许跨域调用）

---

## 数据管道接口

查询完成后可点击「→ 传至统计多工具台」，自动打开 CLI-Anything 并注入数据：

```
https://cli-gong.netlify.app?pipeline=<JSON>
```

JSON 格式：`{ "cmd": "T10101", "p": ["1929","1930",...], "v": [105.1, 91.2,...] }`

---

## 主要依赖

- [BEA API](https://apps.bea.gov/api/) — 美国经济分析局官方 API（免费）
- [Chart.js 4.4](https://www.chartjs.org/) — 图表渲染（CDN）
- [IBM Plex Mono](https://fonts.google.com/specimen/IBM+Plex+Mono) — 等宽字体（CDN）
- Netlify Functions（Node.js 18+）— 无服务器代理

---

## 相关平台

| 平台 | 地址 | 说明 |
|------|------|------|
| CLI-Anything 统计多工具台 | [cli-gong.netlify.app](https://cli-gong.netlify.app) | DeepSeek 驱动的统计分析工作台 |
| OpenCLI 内蒙古数据台 | [opencli-gong.netlify.app](https://opencli-gong.netlify.app) | 国家统计局分省数据 |

---

*© 2025 天津财经大学统计学院 · 龚凤乾*
