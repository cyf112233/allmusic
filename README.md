# 🎵 AllMusic Auto Sync — 自动抓取并发布 AllMusic 服务端 & 客户端 🚀
<br>

欢迎来到 AllMusic Auto Sync —— 一个自动化工作流项目，它会每天检查并抓取（拉取）来自 Coloryr/AllMusic_Server 的最新成功构建与源码（服务端 & 客户端），并在本仓库中自动创建 Release 发布构建产物与源码包。想像一台永不停歇的流水线，把最新稳定构建镜像带到你仓库的 Releases 页面 ✨

---

## 🌈 项目亮点

- 每日自动运行（可手动触发）。
- 智能去重：如果最新成功的 workflow 与上一次相同，则跳过，避免重复发布。
- 下载并保存目标仓库的 artifacts（若存在）以及对应的源码 zip（基于 commit SHA）。
- 自动在本仓库创建 Release，并把构建产物与源码作为 release assets 上传。
- 简单配置：只需提供一个访问目标仓库的 PAT（可选但推荐），并启用 Actions 写权限。

---

## 🔧 能力简介（一句话）

每天自动“爬取”/拉取 Coloryr/AllMusic_Server 的最新成功构建（服务端 & 客户端）、打包源码并在本仓库生成 Release，保持你本仓库对上游稳定构建的镜像与备份。

---

## 📁 文件与工作流位置

工作流文件：`.github/workflows/sync-release.yml`  
（该 workflow 会查询：`Coloryr/AllMusic_Server` 的 actions runs，下载 artifacts 与源码，并在此仓库创建 release。）

---

## 🛠 如何配置（必读）

1. 在本仓库的 Settings → Secrets 中添加：
   - `SOURCE_PAT`（推荐）：用于访问目标仓库 API 与下载 artifacts。若目标仓库是 public，理论上可不填，但会有更严格的速率限制与认证问题。建议权限：public_repo 或 repo（视目标仓库是否私有而定）。
   - `GITHUB_TOKEN`：GitHub 自动提供，无需手动配置（用于创建 release、提交记录等）。

2. 若需要每天按北京时间0点触发，请把 `.github/workflows/sync-release.yml` 中的 cron 改为：
   ```
   '0 16 * * *'
   ```
   （GitHub Actions 的 cron 使用 UTC）

3. 如果你的仓库启用了分支保护或禁止工作流推送，`.last_release_source_run` 的自动提交推送可能会失败；工作流会忽略 push 错误以避免整个任务失败，但请根据需要手动调整分支策略或使用其它持久化机制。

---

## 🧭 工作流行为（细节）

- 查询目标仓库最近的成功 run（status=success）。
- 读取该 run 的 run_id 与 head_sha：
  - 若 run_id 与上次记录（`.last_release_source_run`）相同，视为重复，跳过。
  - 否则，下载该 run 的所有 artifacts（若有）和源码 zip（zipball of head_sha）。
- 在本仓库创建 release（tag: `source-run-<run_id>`），并上传 artifacts 与源码为 assets。
- 将处理过的 run_id 写入 `.last_release_source_run` 并尝试提交回触发分支（失败不会阻塞 release 创建）。

---

## ⚠️ 注意与限制

- Artifact 保留期限：目标仓库上的 artifacts 可能会过期（例如 90 天），过期后无法下载。
- Release 资产大小限制：单个资产有大小上限（约 2GB）；超大文件可能上传失败。
- 速率限制：未使用 PAT 时，API 请求可能受速率限制，导致失败或被限制。
- 去重逻辑：当前基于 run_id；如果你希望改为“基于 head_sha 去重”（同一 commit 不重复发布），我可以替你修改。

---

## ✨ 可选改进建议

- 基于 head_sha 去重（避免不同 run_id 下的相同源码重复发布）。
- 将 release tag 命名规则改为更语义化（例如包含日期、commit short SHA）。
- 在 Release body 中附加更多元数据（workflow 名称、job 列表、测试覆盖率摘要等）。
- 对大文件进行分段上传或使用外部存储（如 GitHub Packages、S3）以规避单文件大小限制。
- 增加通知（失败或成功时发邮件、Slack 或创建 issue）。

---

## 🧾 示例：手动触发测试

1. 在 GitHub 页面中，进入 Actions → 选择 "Sync latest successful run and release" 工作流。
2. 点击 “Run workflow” 来手动触发。
3. 查看日志，确认：
   - 找到最近的成功 run；
   - 是否下载 artifacts & 源码；
   - 是否成功创建 Release；
   - 是否写入 `.last_release_source_run`。

---

## ❤️ 免责声明

本项目通过 GitHub API 拉取目标仓库的构建产物与源码。请确保你有权按照目标仓库的授权条款来下载与再发布这些内容。本项目不会绕过任何访问控制或版权限制。对于目标仓库私有内容或未授权转载，责任由使用者承担。
