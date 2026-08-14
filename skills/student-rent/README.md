# 毕业生租房助手（student-rent）接入指南

一套面向应届毕业生/在校生的租房技能，可在 Claude Code / Codex / WorkBuddy / 龙虾 等平台直接找房、算通勤、在地图上标房、避坑。

三层架构：

| 层 | 实现 | 说明 |
|---|---|---|
| 脑 | `SKILL.md` | 毕业生顾问逻辑（找房→通勤→合租→补贴→避坑） |
| 数据 | beike MCP | 房源/通勤/地理编码/政策/行情，30 个工具 |
| 绘制 | 高德 JS API | 交互式 HTML 地图（`map-template.html`） |

## 1. 前置依赖

| 依赖 | 状态 | 获取方式 |
|---|---|---|
| beike MCP key | 需配置 | 见 §2 |
| 高德 JS API key + 安全密钥 | 需申请 | 见 §3 |

## 2. 配置 beike MCP

beike MCP = 「布丁MCP服务」，streamable HTTP 端点 `https://building.ke.com/mcp`，认证用 `Authorization: Bearer <key>`。

- **Claude Code**：在项目根 `.mcp.json`（或 `~/.claude.json` 的 `mcpServers`）添加：
  ```json
  {
    "mcpServers": {
      "beike": {
        "url": "https://building.ke.com/mcp",
        "headers": { "Authorization": "Bearer <你的beike key>" }
      }
    }
  }
  ```
- **Codex / WorkBuddy / 龙虾**：按各自平台的 MCP 配置方式，把上述 `url` + `headers` 挂进去即可，字段通用。

> key 获取：若本机已安装 beike CLI 并登录过，key 保存在 `~/.beike/BEIKE_MCP_API_KEY`；否则到 building.ke.com 申请。不要把 key 写进聊天或提交到仓库。

## 3. 申请高德 JS API key

绘制地图需要高德「Web端(JS API)」key 与安全密钥：

1. 登录 [console.amap.com](https://console.amap.com)，创建应用 + Key，类型选 **Web端(JS API)**。
2. 记下 **Key** 和 **安全密钥（securityJsCode）**。
3. 配置环境变量（供技能运行时读取）：
   ```bash
   export AMAP_JS_KEY='<你的 JS API key>'
   ```

> 高德 JS API 2.0 官方要求 key + securityJsCode 同时提供；但**本套件实测：当前 key 无需安全密钥即可正常渲染**，模板已将 securityJsCode 注释关闭。若你的 key 强制要求安全密钥，再取消注释模板中的 `_AMapSecurityConfig` 并补 `AMAP_SECURITY_CODE`。

## 4. 各平台接入技能

把技能目录（`SKILL.md` + `manifest.json` + `map-template.html`）复制到对应平台的技能目录即可，一份文件到处跑：

| 平台 | 技能目录 |
|---|---|
| Claude Code | `~/.claude/skills/student-rent/` |
| Codex | 对应 `skills/` 目录（复制整目录） |
| WorkBuddy / 龙虾 | 对应 openclaw 技能目录（`instructionOnly` 技能直接加载） |

## 5. 验证用例

1. **beike MCP 数据**：调用 `rent_house_search`（`query`=「望京 合租 3000以内」，`city_name`=北京），应返回结构化房源结果。
2. **通勤数据**：`maps_geo`（`address`=公司地址）→ 拿坐标；`maps_direction_transit_integrated`（`origin`=公司坐标、`destination`=房源坐标、`city`/`cityd`=城市）→ 拿通勤时长/换乘。
3. **地图绘制**：配好 `AMAP_JS_KEY`/`AMAP_SECURITY_CODE` 后，把公司点 + 2 个房源点 + 路线注入 `map-template.html`，浏览器打开确认点/圈/线正常渲染。
4. **端到端 smoke**（Claude Code）：
   > 我拿到北京的 offer，公司在西二旗，预算 3500，帮我找通勤 45 分钟内、可合租的房子。

   预期：触发 student-rent → 找房 → 算通勤 → 生成地图 HTML → 合租取舍 + 1 条避坑提醒，且**不**默认引导加微/约看。

## 6. 能力边界

- 本期只做：找房、通勤、地图可视化、合租、人才公寓/补贴、避坑。
- 不做：小红书等 UGC 点评；高德步行/骑行/周边 POI/天气（beike MCP 未含，如需要后续补独立高德 MCP）。
- 加微、预约带看是可选次要用，非默认主路径。
