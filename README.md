# docs-agent-platform

A Content repo of WSO2 Docs (docs.wso2.com). It holds these Doc spaces, one directory each:

| Directory | Doc space | Editions and branches |
|---|---|---|
| `agent-manager/` | Agent Manager | Cloud on `docs-v2`; 2026.11.0 on `agent-manager/release/2026.11.0`; 1.0.0 on `agent-manager/release/1.0.0` |

**Until the cutover**, `main` still holds the legacy Docusaurus site (published at wso2.com/agent-platform/docs/), and the WSO2 Docs main line is the `docs-v2` branch. Fixes merged to `main` in the meantime are brought across with `git merge main` on `docs-v2`.

Each Doc space directory holds `docspace.yaml` (the Doc space manifest), `editions.yaml` (per-Edition variables), `redirects.yaml` (page moves and legacy maps), `docs/` (pages; folders are the menu), `partials/` (snippets shared inside the Doc space) and `assets/` (with `cloud/` and `self-hosted/` for Edition-only files).

The main line holds every Doc space's main line. A `<doc space>/release/<version>` branch is a copy of the whole repo, but it builds only `<doc space>/`, and the checks block changes to any other directory there. Docs releases are tagged `<doc space>/cloud-<YYYY.MM.DD>.<n>` and `<doc space>/<version>-docs.<n>`.

The repo root holds only shared files: this README, `CODEOWNERS`, the workflows (`.github/workflows/docs-bundle-release.yml` releases a Docs bundle; `pr-open-notify.yml` announces new pull requests), the branch ruleset (`.github/branch-ruleset.json`), the editor settings, `AGENTS.md` and `CLAUDE.md` for agents, and `scripts/gen-helm-reference.mjs`, which generates the Helm chart reference pages. The Platform-managed files come from `templates/content-repo/` in `wso2-enterprise/choreolabs-docs`.

The tools are on GitHub Packages. Once per machine, make a classic personal access token with `read:packages` (authorised for the `wso2-enterprise` organisation's SSO if it asks), then run `npm login --scope=@wso2-enterprise --auth-type=legacy --registry=https://npm.pkg.github.com` with your GitHub user name and the token as the password. This repo's `.npmrc` names the registry for `@wso2-enterprise`; it holds no token.

Get started, from setting up your computer to a Docs release going live: the WSO2 Docs Platform's README, https://github.com/wso2-enterprise/choreolabs-docs#readme.
