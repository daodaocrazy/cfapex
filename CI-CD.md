# CFAPEX GitHub Actions CI/CD 部署说明

你在 `daodaocrazy/cfapex` fork 仓中可按以下方式实现「手动同步上游 → 手动触发部署」到 Cloudflare Worker 的 CI/CD 流程。

## 原理

1. 上游 `shitouxiaoge-commits/CFAPEX` 有更新时，你在 GitHub 上手动点击「Sync fork」或在 Actions 里运行本工作流的「同步上游」步骤，将上游 `main` 合并到你 fork 的 `main`。
2. 本地或 Actions 里运行 `wrangler deploy`，把 `CFAPEX-main/CFAPEX-worker.1.4.js` 部署到你已经存在的 Cloudflare Worker `cfapex`，并绑定自定义域 `cfapex.daodaocrazy.qzz.io`。

## 1. 添加 GitHub Secret

在你的 fork 仓 `Settings` → `Secrets and variables` → `Actions` 中添加：

- `CLOUDFLARE_API_TOKEN`：Cloudflare API Token，需具备 `Workers - Edit` 权限（建议创建只对 `cfapex.daodaocrazy.qzz.io` 区域生效的受限 Token）。

## 2. 提交 wrangler 配置

把 `wrangler.cfapex.toml.example` 复制一份为 `wrangler.cfapex.toml`，替换其中的占位符：

- `REPLACE_WITH_YOUR_KV_NAMESPACE_ID` → 你已有的 KV Namespace ID（你本机已配置的是 `fdb650a7593e41aab0fa6a063d3ba4b1`）。
- `REPLACE_WITH_YOUR_PASSWORD_A` → 面板访问密码 `A`（你本机已配置的是 `3LIF0fQb8G4u2cEdpUaBZjqztPSWlCsm`）。

将 `wrangler.cfapex.toml` 提交到 fork 仓。**不要**把 `CLOUDFLARE_API_TOKEN` 写进配置文件，它只存在于 GitHub Secrets。

## 3. 同步上游

上游仓 `shitouxiaoge-commits/CFAPEX` 有新版本时，在你的 fork 仓 Actions 页面运行 `CFAPEX CI/CD` 工作流：

- `sync_upstream = true`：自动拉取上游 `main` 并合并到你 fork 的 `main`，推送后触发部署。
- `sync_upstream = false`：跳过同步，直接用当前 fork 仓代码部署。

合并若出现冲突，Actions 会停止，你需要在本地拉取 `main` 手动解决冲突后重新推送。

## 4. 触发部署

同步完成后（或直接跳过同步），在 Actions 页面运行工作流，设置：

- `deploy_worker = true`：部署 Worker 到 Cloudflare。
- `deploy_worker = false`：仅做代码检出，不部署（可用于只同步代码）。

部署成功后，Cloudflare Worker 会自动更新，访问 `https://cfapex.daodaocrazy.qzz.io/<你的 A 密码>` 即可看到最新面板。

## 5. 文件清单

| 文件 | 用途 |
|------|------|
| `.github/workflows/cfapex-cd.yml` | GitHub Actions 工作流，支持手动同步上游 + 手动触发部署 |
| `wrangler.cfapex.toml.example` | Wrangler 配置模板，复制后填自己的 KV ID 和密码 A |
| `CI-CD.md` | 本说明文档 |

## 注意事项

- 上游仓库不含 `.github/workflows`，你的 fork 是独立的，上游更新不会自动覆盖你的 CI 工作流，只需手动同步代码文件即可。
- `CFAPEX-worker.1.4.js` 是已混淆的单文件，部署时直接用 `wrangler deploy --config wrangler.cfapex.toml` 即可，无需本地构建。
- 如需部署到 Pages 而非 Workers，请改用 `wrangler pages deploy CFAPEX-Pages-1.4.zip` 并调整工作流步骤。
