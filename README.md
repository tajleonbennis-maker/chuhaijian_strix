# Strix Pentest Report — 165.154.226.119

基于 **Strix 1.5.3**（开源 AI 渗透测试工具）+ **DeepSeek V4-Pro** 模型，对授权目标 **165.154.226.119**（Supply Chain Security Analyzer 应用服务器）执行的标准模式渗透测试完整报告。

## 测试信息

| 项目 | 详情 |
|---|---|
| 工具 | Strix 1.5.3（headless，standard 模式） |
| 模型 | `deepseek/deepseek-v4-pro` |
| 执行节点 | 192.168.1.39（内网 Ubuntu 22.04 + Docker sandbox） |
| 扫描时间 | 2026-08-18 ~ 2026-08-19（约 3 小时） |
| LLM 消耗 | 367 请求 / 26.5M tokens |
| 扫描状态 | ✅ completed |

## 发现漏洞（4 个）

| # | CWE | 严重度 | CVSS | 端点 | 描述 |
|---|---|---|---|---|---|
| 1 | CWE-306 | 🔴 error | - | `GET /api/tasks`、`GET /api/results/{task_id}` | 未认证访问管理扫描任务与完整结果（目标/资产/API/CVE/供应链数据全暴露） |
| 2 | CWE-1392 | 🔴 error | - | 登录接口 | 默认管理员口令 `admin/admin123` 可直接登录 |
| 3 | CWE-918 | 🟡 medium | - | `POST /api/manual`（`urls` 参数） | SSRF，可让服务器访问任意内网地址/云元数据 |
| 4 | CWE-497 | 🟡 medium | 5.3 | `GET /stream` | SSE 监控端点未认证，泄露内部扫描状态与 LLM 计费数据 |

## 文件说明

- `165漏洞报告.html` — 可视化漏洞报告（推荐直接打开）
- `165_findings.sarif` — SARIF 2.1.0 标准格式原始数据（可导入 GitHub Code Scanning / VS Code SARIF 插件）
- `165_run.json` — 完整运行记录（LLM 用量、时间线、agent 活动）

## 修复建议（按优先级）

1. 修改/移除 `admin/admin123` 默认口令
2. 所有 `/api` 读接口（tasks/results/status）加认证鉴权，与写接口（`POST /api/analyze` 已有 `需要管理员权限` 校验）对齐
3. `/stream` 监控端点限制回环访问或加认证
4. `/api/manual` 的 `urls` 参数加 SSRF 防护（协议白名单 + 内网段/元数据地址黑名单）

---

⚠️ 本报告包含真实漏洞细节与内部网络信息，仓库默认**私有**。仅限授权团队查看。测试目标为自有资产，符合"仅测试你拥有或获明确授权"原则。

## 攻防数据对比

防守方数据（服务器日志 + 数据库）与攻击方发现（Strix 报告）的交叉印证分析，含 SSRF 利用证据链：

- [攻防数据对比.html](攻防数据对比.html) —— 可视化对比报告

## 在线报告（GitHub Pages）

- [攻防数据对比（在线渲染）](https://tajleonbennis-maker.github.io/chuhaijian_strix/攻防数据对比.html)
- [165 漏洞报告（在线渲染）](https://tajleonbennis-maker.github.io/chuhaijian_strix/165漏洞报告.html)
