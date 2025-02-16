# **稳定代币市值交集筛选项目**

## **项目简介**

这是一个基于 **Cloudflare Workers** 和 **KV 存储** 构建的服务，用于获取相对稳定且具有前景的代币。用户可以通过指定天数，使用该服务查询多个日期内市值排名的代币交集，筛选出在指定日期范围内市值维持高水平（代表稳定性和市场信任）的代币。
示例：https://coin.14790897.xyz/?days=5

### **核心功能**

1. **市值交集筛选**：

   - 从 CoinGecko 获取每日市值前 500 的代币列表。
   - 根据用户指定的日期范围，筛选出在所有日期中均存在的代币。
   - 返回符合条件的代币及其市值。

2. **结果格式化**：

   - 返回的代币通过 `"symbol/USDT"` 的格式化输出，方便前端直接使用。
   - 附带 `refresh_period` 字段，标识数据刷新周期。

3. **高效存储与查询**：
   - 借助 Cloudflare Workers 的 KV 存储，每天更新并存储代币数据，实现低延迟查询。

---

## **技术栈**

### **主要技术**

- **Cloudflare Workers**: 作为服务器运行环境，用于处理 API 请求、逻辑计算和数据查询。
- **Cloudflare KV**: 用于存储每日代币市值数据，支持快速读写操作。
- **CoinGecko API**: 用于获取每日市值前 500 的代币数据。
- **TypeScript**: 增强代码的类型安全性，避免运行时错误。

### **API 数据来源**

- [CoinGecko API](https://www.coingecko.com/api/documentations/v3)：提供实时加密货币市场数据，包括市值、价格、交易量等。

---

## **项目结构**

```
project-root/
├── src/
│   ├── index.ts       // CF Worker 主逻辑
├── wrangler.json      // Cloudflare Workers 配置文件
├── package.json       // 项目依赖
├── tsconfig.json      // TypeScript 配置文件
└── README.md          // 项目文档
```

---

## **功能实现**

### **1. 定时抓取每日市值数据**

- 使用 CF Worker 的定时触发功能，调用 CoinGecko API 获取每日市值前 500 的代币，并存储到 KV 中。
- 数据格式如下：
  ```json
  {
    "date": "2025-01-01",
    "tokens": [
      { "id": "bitcoin", "symbol": "btc", "market_cap": 600000000000 },
      { "id": "ethereum", "symbol": "eth", "market_cap": 300000000000 },
      ...
    ]
  }
  ```

### **2. 用户指定天数，获取代币市值交集**

- 用户可以指定查询日期范围（例如最近 7 天），服务会返回在这段时间内，市值排名稳定在前 500 的代币。
- 返回格式化的代币列表，包含代币/USDT 交易对和数据刷新周期：
  ```json
  {
  	"pairs": ["BTC/USDT", "ETH/USDT", "DOGE/USDT"],
  	"refresh_period": 900
  }
  ```

### **3. 数据存储和查询效率**

- 所有代币数据在 KV 中以日期键存储（如 `2025-01-01`），支持快速查询。
- 使用内存计算市值交集，结合 KV 提高查询性能。

---

## **部署与运行**

### **1. 配置 Cloudflare Workers**

#### 安装 Wrangler CLI

确保你已安装 Wrangler CLI 用于管理 Cloudflare Workers：

```bash
npm install -g wrangler
```

#### 配置 `wrangler.json`

在项目根目录的 `wrangler.json` 中添加 KV 命名空间配置（需要在cloudflare创建名为coingecko的KV，并获取id填入配置）：

```json
{
	"$schema": "node_modules/wrangler/config-schema.json",
	"name": "coingecko-stable-coinlist",
	"main": "src/index.ts",
	"compatibility_date": "2025-01-21",
	"observability": {
		"enabled": true
	},
	"triggers": {
		"crons": ["0 * * * *"]
	},
	"kv_namespaces": [
		{
			"binding": "coingecko",
			"id": "343d9187f9e14d85bbb5df1320078716"
		}
	],
	"compatibility_flags": ["nodejs_compat"]
}
```

---

### **2. 安装依赖**

```bash
npm install
```

---

### **4. 启动开发服务器**

在本地运行开发服务器：

```bash
npm run dev
```

---

### **5. 部署到生产环境**

确认项目配置无误后，部署到 Cloudflare：

```bash
npm run deploy
```

---

## **使用说明**

### **定时任务**

通过 Cloudflare Workers 配置定时任务，每天调用 CoinGecko API 并更新 KV 数据。
在 `wrangler.json` 中配置 Cron 表达式：

```json
"triggers": {
		"crons": ["0 * * * *"]
	} # 每天零点触发
```

### **API 请求**

#### **获取最近 7 天的稳定代币交集**

请求示例：

```http
GET https://your-worker-url.com/intersection?days=7&limit=50
```

- **参数说明**：
  - `days`：指定查询的天数（必须）。
  - `limit`：返回代币数量的上限（可选，默认为全部）。

返回示例：

```json
{
	"pairs": ["BTC/USDT", "ETH/USDT", "DOGE/USDT"],
	"refresh_period": 900
}
```

---

## **未来扩展**

1. **多货币支持**：

   - 支持用户自定义目标货币（例如 `BTC/EUR`、`ETH/USD`）。

2. **更多过滤条件**：

   - 根据用户需求加入交易量、稳定性等筛选条件。

3. **性能优化**：
   - 缓存结果数据，减少计算压力。

---

## **贡献指南**

欢迎贡献代码！请按照以下步骤提交 Pull Request：

1. Fork 仓库。
2. 创建新分支并提交改动：
   ```bash
   git checkout -b feature/new-feature
   ```
3. 提交 Pull Request。

---

## **许可证**

本项目遵循 **MIT 许可证**，详情请参见 LICENSE 文件。

---

## **联系方式**

如果你对项目有任何问题或建议，欢迎通过以下方式联系我：

- **邮箱**：liuweiqing147@gmail.com
- **GitHub Issues**：在本仓库提交问题

