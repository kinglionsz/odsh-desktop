# 贡献

[English](CONTRIBUTING.md) | 中文

感谢你愿意为 Open DeepSeek Harness Desktop 作出贡献！

本仓库是 DeepSeek Harness 的社区维护桌面发行版，并非上游 `deepseek-ai` 仓库。这一区别决定了你的贡献应当提交到哪里，因此在发起 Pull Request 前，请先阅读[范围：什么属于本仓库](#范围什么属于本仓库)。

## 范围：什么属于本仓库

| 改动 | 归属 |
| --- | --- |
| Electron 宿主、安装器、系统集成、插件市场、Profile 诊断与恢复 | **本仓库** |
| 桌面端专属文档、截图、打包脚本、发行产物 | **本仓库** |
| Harness 运行时、agent loop、核心工具或公开 SDK 的改动 | 上游 [DeepSeek Harness](https://github.com/deepseek-ai)——本仓库以版本化基线的方式消费它们 |
| 内联的 Cordis 包（`vendor/`） | 不直接改动；遵循 [vendor/README.md](vendor/README.md) |

本仓库跟随一条上游基线。即使某个改动本身是正确的，只要它属于上游，这里也会拒绝——接受它将使行为偏离我们实际发布的基线。

## 参与方式

### 报告问题

缺陷报告是价值最高的贡献。请在 [GitHub Issues](https://github.com/flaqai/open-deepseek-harness-desktop/issues) 提交，并包含：

- 你做了什么、期望什么、实际发生了什么。
- 你的平台、客户端版本，以及你运行的是开发版还是安装版。
- 当故障涉及插件、Profile 或启动过程时，附上日志或诊断报告。客户端的诊断页可以导出这些内容。

请先搜索已有 Issue。如果某个 Issue 已关闭但仍影响你，请在其下评论，而不是另开一个重复项。

### Pull Request

本仓库欢迎外部 Pull Request。为了让评审可预期：

1. **先开 Issue**，除非是琐碎修复。在写代码前先讨论方案，可以避免与基线冲突的返工。
2. **控制改动范围。** 一个 Pull Request 只做一件事。独立改动应当拆分，而不是打包提交。
3. **遵循仓库约定。** [AGENTS.md](AGENTS.md) 是代码、文档与测试的权威规则集；[docs/development.md](docs/development.md) 覆盖工作流。
4. **推送前运行与改动面匹配的检查**，并说明你实际运行了哪些命令。参见[运行检查](#运行检查)。

标签由维护者在分类处理时添加；你无需为自己的 Pull Request 打标签。

### 插件生态

DeepSeek Harness 的设计支持深度定制。官方仓库中的包并不天然比社区开发的包更重要。本仓库是一种理念、一份示例、一处灵感来源，而不是要求社区遵循的方向：

- 创建插件并分享。为你的 GitHub 项目添加 `dsh-plugin` 话题，让其他人更容易找到它。
- 撰写博客文章和操作指南。
- 回答问题并帮助其他社区成员。

## 开发环境

需要 Node `^22.19 || >=24` 与 pnpm `11.7.0`。

```sh
pnpm install
pnpm run typecheck
pnpm run lint
pnpm run test
```

`pnpm run build` 产出运行时 bundle。真实 API 测试与演示还需要 `DEEPSEEK_API_KEY`；没有 key 时会自动跳过。

## 运行检查

请让证据与改动面匹配，不要默认跑全量测试。CI 负责穷尽式覆盖与平台矩阵。只报告你实际运行过的命令。

| 改动 | 检查 |
| --- | --- |
| 任何源码改动 | 针对受影响包运行 `pnpm run test`，以及 `pnpm run typecheck`、`pnpm run lint` |
| 文档 | `pnpm run doc-sync`（快速检查可用 `pnpm run test:docs`） |
| 包清单或导出 | `pnpm run hygiene` |
| 用户或模型可观察的行为 | 相应的无密钥录制会话快照；参见 [docs/testing.md](docs/testing.md) |
| Provider 或真实 API 行为 | 带 key 运行 `pnpm run test:e2e` |

如果某项检查因与你的改动无关的原因失败，请在 Pull Request 中说明，而不是改无关代码来让它通过。

## 写代码前值得知道的约定

完整规则集见 [AGENTS.md](AGENTS.md)。以下是初次贡献者最常感到意外的几条：

- **模型可见即必须落日志。** 任何进入模型请求的内容都必须能从会话日志重建。新增模型可见输入需要新增会话事件。
- **改插件，不改主循环。** 新行为应挂在已文档化的扩展点上。改动 `agent-loop` 需要同步更新 [docs/architecture.md](docs/architecture.md)。
- **注册即副作用。** 每一处贡献都经由 `ctx.effect()` 或 `ctx.on()`；注册表的 `register()` 返回 disposer。
- **插件中不得硬编码可调参数。** 随部署而变的选项应是可在 `cordis.yml` 中修改的、经过校验的 `Config` 字段。
- **文档与代码同行。** 在同一次改动中更新受影响的 README 与 JSDoc 契约。
- **注释保持局部。** 不要复述代码，也不要在无局部必要时解释远处行为。

以下区域测试较薄，欢迎加固：`storage`、`session-query`、`workspace`。在此处补测试时，优先走真实 Loader 启动路径，而不是用 `ctx.plugin(...)` 手工挂载。

## 语言与文档配对

英文与中文文档具有同等权威。已有对照的文档必须连同其 `.i18n.yaml` 记录一并合并双边内容；只改一侧会使 `verify-translation-pairing` 失败。编辑任一侧后，请同步另一侧并重录配对：

```sh
pnpm run verify-translation-pairing --write CONTRIBUTING.md
```

## 贡献的许可

提交 Pull Request 即表示你同意你的贡献按本仓库相同的条款授权。参见 [LICENSE](LICENSE)。

## 获取帮助

如果你不确定某项改动应归属何处，请先开 Issue 提问，再写代码。一次简短的提问，代价远低于一个被拒绝的 Pull Request。

我们的团队规模很小，可能无法回复每个帖子，但我们会持续关注，并在分配资源时将这些内容纳入考虑。

探索未至之境。
