# Contributing

English | [中文](CONTRIBUTING.zh.md)

Thank you for your interest in Open DeepSeek Harness Desktop!

This repository is a community-maintained desktop distribution of DeepSeek Harness. It is not the upstream `deepseek-ai` repository. The distinction matters for where your contribution belongs, so read [Scope: what belongs here](#scope-what-belongs-here) before opening a pull request.

## Scope: what belongs here

| Change | Where it goes |
| --- | --- |
| Electron host, installer, system integration, plugin market, profile diagnostics and recovery | **This repository** |
| Desktop-specific docs, screenshots, packaging scripts, release artifacts | **This repository** |
| Changes to the Harness runtime, agent loop, core tools, or public SDKs | Upstream [DeepSeek Harness](https://github.com/deepseek-ai) — this repository consumes them as a versioned baseline |
| Vendored Cordis packages (`vendor/`) | Not directly; follow [vendor/README.md](vendor/README.md) |

This repository tracks an upstream baseline. A change that belongs upstream will be declined here even when it is correct, because accepting it would fork behavior away from the baseline we ship.

## Ways to contribute

### Reporting issues

Bug reports are the highest-value contribution. Open a [GitHub Issue](https://github.com/flaqai/open-deepseek-harness-desktop/issues) with:

- What you did, what you expected, and what happened instead.
- Your platform, the client version, and whether you run a development or installed build.
- Logs or a diagnostic report when the failure involves plugins, profiles, or startup. The client's diagnostics page can export this.

Please search existing issues first. If an issue is closed but still affects you, comment on it rather than opening a duplicate.

### Pull requests

External pull requests are welcome. To keep review predictable:

1. **Open an issue first** for anything beyond a trivial fix. Discussing the approach before writing code avoids work that conflicts with the baseline.
2. **Keep the change scoped.** One concern per pull request. Split independent changes instead of bundling them.
3. **Follow the repository conventions.** [AGENTS.md](AGENTS.md) is the authoritative rule set for code, documentation, and tests; [docs/development.md](docs/development.md) covers the workflow.
4. **Run the checks that match your change** before pushing, and report which commands you ran. See [Running checks](#running-checks).

Pull request titles and labels follow the repository taxonomy: one `kind/*` label, and every applicable `area/*` label. Maintainers will apply labels if you are unsure.

### The plug-in ecosystem

DeepSeek Harness is designed to be deeply customizable. Packages in an official repository are not inherently more important than packages created by the community. This repository is an idea, a showcase, and a source of inspiration, not a mandate:

- Create a plugin and share it. Add the `dsh-plugin` topic to your GitHub project so others can find it.
- Write blog posts and how-to guides.
- Answer questions and help other community members.

## Development setup

Requires Node `^22.19 || >=24` and pnpm `11.7.0`.

```sh
pnpm install
pnpm run typecheck
pnpm run lint
pnpm run test
```

`pnpm run build` emits the runtime bundles. Real-API tests and demos additionally need `DEEPSEEK_API_KEY`; they self-skip without it.

## Running checks

Match the evidence to the surface you changed; do not default to the full suite. CI owns exhaustive coverage and the platform matrix. Report only the commands you actually ran.

| Change | Check |
| --- | --- |
| Any source change | `pnpm run test` for the affected packages, `pnpm run typecheck`, `pnpm run lint` |
| Documentation | `pnpm run doc-sync` (or `pnpm run test:docs` for a quick pass) |
| Package manifest or exports | `pnpm run hygiene` |
| User-visible or model-visible behavior | The relevant keyless recorded-session snapshot; see [docs/testing.md](docs/testing.md) |
| Provider or real-API behavior | `pnpm run test:e2e` with a key |

If a check fails for reasons unrelated to your change, say so in the pull request rather than adjusting unrelated code to make it pass.

## Conventions worth knowing before you write code

The full rule set is [AGENTS.md](AGENTS.md). The rules that most often surprise first-time contributors:

- **Model-visible implies logged.** Anything that reaches a model request must be reconstructable from the session log. A new model-visible input requires a session event.
- **Plugins, not loop changes.** New behavior goes on documented extension points. Changing `agent-loop` requires updating [docs/architecture.md](docs/architecture.md).
- **Registrations are effects.** Every contribution goes through `ctx.effect()` or `ctx.on()`; a registry's `register()` returns the disposer.
- **No hardcoded tunables in plugins.** Deployment-varying choices are validated `Config` fields changeable from `cordis.yml`.
- **Documentation accompanies code.** Update affected README and JSDoc contracts in the same change.
- **Comments stay local.** Do not restate code or explain distant behavior without local need.

Thinly tested areas that welcome hardening: `storage`, `session-query`, `workspace`. When adding tests there, prefer exercising the real Loader startup path over hand-mounting with `ctx.plugin(...)`.

## Language and documentation pairing

English and Chinese documentation carry equal authority. A document that has a counterpart must merge both sides plus its `.i18n.yaml` record together; editing one side alone fails `verify-translation-pairing`. After editing either side, bring the other along and record the pair:

```sh
pnpm run verify-translation-pairing --write CONTRIBUTING.md
```

## Licensing of contributions

By submitting a pull request you agree that your contribution is licensed under the same terms as this repository. See [LICENSE](LICENSE).

## Getting help

If you are unsure where a change belongs, open an issue and ask before writing code. A short question costs far less than a declined pull request.

We are a small team and may not reply to every post, but we read them and consider them when allocating resources.

Into the unknown.
