# student-rent · 毕业生租房助手

面向**应届毕业生 / 在校生**的 AI 租房技能，支持 Claude Code / Codex / WorkBuddy / 龙虾等平台，帮你找房、算通勤、在地图上标房、避坑。

## 它能做什么

- **顾问式访谈**：先问你公司在哪、怎么通勤、能接受多久、预算多少、整租还是合租、对周边有什么要求，再找房
- **真实房源搜索**：接入贝壳 MCP（30 个工具），整租/合租房源、小区、板块、租金行情、政策补贴
- **通勤规划**：地址→坐标 + 公交/地铁/驾车路线，算「公司↔房源」通勤时间
- **地图可视化**：高德 JS API 生成交互地图，标出公司点、房源点、通勤圈、路线
- **避坑指南**：二房东、黑中介、阴阳合同、押金、甲醛等毕业生高频坑

## 一句话安装

对任意 AI 助手（Claude Code / Codex / WorkBuddy / 龙虾等）说一句话即可：

> 帮我安装 codexc/student-rent 这个技能：`npx skills add codexc/student-rent -g -y`

或者自己直接运行：

```bash
npx skills add codexc/student-rent -g -y
```

装好后，首次找房/画地图时会引导你配置两个 key（贝壳 MCP + 高德 `AMAP_JS_KEY`），见下方说明。

## 安装后需配置两项依赖

技能本身免费，但找房和画地图依赖两个 key（见 `skills/student-rent/README.md` 的完整接入指南）：

1. **贝壳 MCP key** —— 注册 beike MCP 到你的 agent：
   ```json
   { "mcpServers": { "beike": {
       "url": "https://building.ke.com/mcp",
       "headers": { "Authorization": "Bearer <你的贝壳key>" } } } }
   ```
2. **高德 JS API key** —— 设环境变量 `AMAP_JS_KEY`（用于地图绘制）

## 触发示例

```
我拿到北京的 offer，公司在西二旗，预算 5500，帮我找通勤 45 分钟内、整租一居的房子
```

技能会自动触发，走「访谈 → 找房 → 算通勤 → 出地图 → 避坑提醒」完整流程。

## 能力边界

- 本期：找房、通勤、地图、合租、人才公寓/补贴、避坑
- 不做：买房/卖房/新房/装修/学区；小红书等 UGC 点评
