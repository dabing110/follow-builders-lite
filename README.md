# follow-builders-lite

轻量、只读的 AI Builder 摘要 Skill：消费上游 [`zarazhangrui/follow-builders`](https://github.com/zarazhangrui/follow-builders) GitHub Actions 产出的公开 feed JSON，汇总 X 推文、播客与官方博客，无需任何 API key、定时任务或推送集成。

## 为什么存在

上游仓库负责采集（X / 播客 / 博客），本 Skill 只做「读取公开产物 → 压缩成结构化 JSON → 交给 Agent 写摘要」。不抓取任何网站、不申请凭据、不修改上游。

## 取数通道（三级自动兜底）

脚本按以下顺序尝试 `feed-x.json` / `feed-podcasts.json` / `feed-blogs.json`，任一通道成功即用：

1. **`raw.githubusercontent.com`** — 官方原始地址；部分网络环境（如受限沙箱）会重置连接。
2. **`gh.idayer.com` 镜像** — 常见的 raw 代理；在部分出口 IP 下会被 Cloudflare 以 `error code 1027 / 429` 拦截（防火墙规则，重试无效）。
3. **`api.github.com` Contents API** — 兜底通道，走 GitHub 官方 API，通常不受上述限制；使用 `Accept: application/vnd.github.raw+json` 直接取文件原文。未认证配额 60 次/小时，每期仅消耗 3 次。

可用环境变量 `FOLLOW_BUILDERS_RAW_BASE` 覆盖首选通道；API 兜底始终保留。若三级全挂，脚本返回 `status=partial` + `errors[]`，摘要时按缺源处理，不要编造内容。

## 用法

```bash
python3 scripts/prepare_digest.py --language zh
python3 scripts/prepare_digest.py --language zh --max-field-chars 6000
python3 scripts/prepare_digest.py --local-dir /path/to/follow-builders   # 本地测试
```

输出一个 JSON 到 stdout，含 `stats`、`x`、`podcasts`、`blogs`、`errors` 等字段。仅以该 JSON 作为摘要素材；按内容类型分节，空节省略，链接随条目保留。

## 相关项目

配套的 [AI 日报三源融合](https://github.com/dabing110/ai-daily-merged) 把本 Skill 与何夕2077、AIHOT 一起交叉去重，生成中文日报。

## License

MIT © dabing110
