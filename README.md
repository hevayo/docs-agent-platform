# docs-agent-platform

A Content repo of WSO2 Docs (docs.wso2.com). It holds these Doc spaces, one directory each:

| Directory | Doc space | Editions and branches |
|---|---|---|
| `agent-management/` | Agent Management | Cloud on `docs-v2/main`; 2026.11.0 in `agent-management/2026.11.0/`; 1.0.0 in `agent-management/1.0.0/` |

**Until the cutover**, `main` still holds the legacy Docusaurus site (published at wso2.com/agent-platform/docs/), and the WSO2 Docs main line is the `docs-v2/main` branch, generated from `main` until the cutover: don't commit to it by hand.

Each Doc space directory holds `docspace.yaml` (the Doc space manifest), `editions.yaml` (per-Edition variables), `redirects.yaml` (page moves and legacy maps), `docs/` (pages; folders are the menu), `partials/` (snippets shared inside the Doc space) and `assets/` (with `cloud/` and `self-hosted/` for Edition-only files).

The main line holds every Doc space's main line, and each released Self-hosted Edition as a folder `<doc space>/<version>/` (its own `docs/`, `assets/` and `redirects.yaml`). Docs releases are tagged `<doc space>/cloud-<YYYY.MM.DD>.<n>` and `<doc space>/<version>-docs.<n>`.

The repo root holds only shared files: this README, `CODEOWNERS`, the workflows (`.github/workflows/docs-bundle-release.yml` releases a Docs bundle; `pr-open-notify.yml` announces new pull requests), the branch ruleset (`.github/branch-ruleset.json`), the editor settings, `AGENTS.md` and `CLAUDE.md` for agents, and `scripts/gen-helm-reference.mjs`, which generates the Helm chart reference pages. The Platform-managed files come from `templates/content-repo/` in `wso2-enterprise/choreolabs-docs`.

The tools are on GitHub Packages. Once per machine, make a classic personal access token with `read:packages` (authorised for the `wso2-enterprise` organisation's SSO if it asks), then run `npm login --scope=@wso2-enterprise --auth-type=legacy --registry=https://npm.pkg.github.com` with your GitHub user name and the token as the password. This repo's `.npmrc` names the registry for `@wso2-enterprise`; it holds no token.

Get started, from setting up your computer to a Docs release going live: the WSO2 Docs Platform's README, https://github.com/wso2-enterprise/choreolabs-docs#readme.
