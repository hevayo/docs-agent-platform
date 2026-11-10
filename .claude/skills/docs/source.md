# Agent Manager Source Repo: Locate and Use

The docs in this repo describe **[wso2/agent-manager](https://github.com/wso2/agent-manager)**. That repo is the only acceptable evidence for any claim about how the product behaves. Read this file before writing or verifying any technical content.

## Step 1: Locate a local checkout

Try these in order and stop at the first that works. Store the result as `AM_REPO` and reuse it for the rest of the task.

1. **Env var**: `echo "$AGENT_MANAGER_REPO"`. Use it if it points at a directory containing `agent-manager-service/`.
2. **Git remote in this repo**: some contributors add the source repo as a remote.
    ```bash
    git remote -v | awk '$1=="am"{print $2; exit}'
    ```
    If that prints a local path containing `agent-manager-service/`, use it.
3. **Sibling directory**: `ls ../agent-manager/agent-manager-service >/dev/null 2>&1 && echo ../agent-manager`
4. **Shallow clone** (last resort; ask the user first, since it downloads the repo):
    ```bash
    git clone --depth 1 https://github.com/wso2/agent-manager.git "${TMPDIR:-/tmp}/agent-manager"
    ```
    Use the session scratchpad directory instead of `/tmp` when one is available.

If none work and the user declines a clone, you may read single files over HTTPS (`https://raw.githubusercontent.com/wso2/agent-manager/main/<path>`), but say so, and mark anything you could not check as UNVERIFIED rather than guessing.

## Step 2: Pick the right ref

Each Edition's pages live on their own branch of this repo, at the same `agent-manager/docs/` path, and must be verified against the matching source. Find the branch with `git rev-parse --abbrev-ref HEAD`:

| Docs branch | Edition | Verify against |
|---|---|---|
| `agent-manager/docs-v2` (the main line), or a branch from it | Cloud, and the next Self-hosted version | `main` of `wso2/agent-manager` |
| `agent-manager/release/<v>` (now `2026.11.0`, `1.0.0`), or a branch from it | Self-hosted version `<v>` | tag `amp/v<v>` |

Content marked `<Edition only="cloud">` or `editions: cloud` describes the Cloud deployment, which may not run `main`: if a claim depends on that difference, ask the user which ref Cloud runs. Content marked `selfHostedFrom: next` is not in any release yet, so `main` is the only ref for it.

Before reading source, check where the local checkout sits and whether it is current:

```bash
git -C "$AM_REPO" rev-parse --abbrev-ref HEAD
git -C "$AM_REPO" log -1 --format='%h %cs %s'
git -C "$AM_REPO" status --porcelain | head
```

Do not `checkout`, `pull`, `reset`, or `stash` in the user's checkout without asking: it may hold their in-progress work. To read a file at another ref without touching the working tree, use `git -C "$AM_REPO" show <ref>:<path>` (run `git -C "$AM_REPO" fetch upstream --tags` or `fetch origin --tags` first if the ref is missing, which is safe). If the checkout is a fork, prefer the remote pointing at `wso2/agent-manager`.

If the local branch is behind `main` or on a feature branch, say so in your reply. A feature branch is fine, and often exactly right, when the user is documenting that unmerged change.

## Step 3: Read the aspect guide first

The source repo documents itself. Before searching code, read the root `AGENTS.md` (the aspect map), then the `AGENTS.md` of the aspect your claim touches. They explain the patterns you will grep for.

## Source map

Where each kind of doc claim is implemented. Paths are relative to `AM_REPO`.

| Doc claim is about... | Look in |
|---|---|
| Console screens, navigation, button and field labels | `console/workspaces/pages/<feature>/src/` (for example `mcp-proxies/`, `llm-providers/`, `gateways/`, `deploy/`, `eval/`, `identities/`). Labels are the literal strings in the `.tsx` files. |
| REST endpoints, methods, paths, required permission | `agent-manager-service/api/*_routes.go` (`rr.HandleFuncWithValidationAndAuthz("POST /orgs/{orgName}/...", rbac.<Perm>, ...)`) and the spec `agent-manager-service/docs/api_v1_openapi.yaml` |
| Request/response fields, validation, defaults | `agent-manager-service/docs/api_v1_openapi.yaml`, `agent-manager-service/spec/`, `agent-manager-service/models/`, `agent-manager-service/services/` |
| Roles and permissions | `agent-manager-service/rbac/permissions.go`, `agent-manager-service/rbac/predefined_roles.go` |
| Service configuration, env vars | `agent-manager-service/config/config.go`, `config_loader.go`, `env_reader.go` |
| Platform MCP server tools | `agent-manager-service/mcp/` (`tools/`, `README.md`) |
| Traces, logs, metrics API; observer MCP server | `agent-manager-observer/` (`docs/openapi.yaml`, `handlers/`, `mcp/`) |
| `amctl` commands, flags, output | `cli/pkg/cmd/<group>/` and `cli/AGENTS.md`; see `cli-reference.md` |
| Helm values, chart install options | `deployments/helm-charts/<chart>/values.yaml` and `values.schema.json` |
| Quick start, installer behavior | `deployments/quick-start/` (`install.sh`, `start.sh`, `install-helpers.sh`) |
| Environment and identity provider scripts | `deployments/scripts/` |
| VM and single-cluster installs | `deployments/vm/`, `deployments/single-cluster/`, `deployments/values/` |
| Sandboxing runtimes (gVisor, Kata) | `deployments/k8s/*-runtimeclass.yaml`, `deployments/scripts/setup-sandbox.sh` |
| Python auto-instrumentation | `libs/amp-instrumentation/`, `python-instrumentation-provider/` |
| Evaluators, custom evaluators, monitors | `libs/amp-evaluation/` (`AGENTS.md`, `CONCEPTS.md`), `evaluation-job/`, `agent-manager-service/api/evaluator_routes.go`, `monitor_routes.go` |
| Sample agents used in tutorials | `samples/<sample>/` |
| Audit logging | `agent-manager-service/docs/audit-logging.md` |

Useful searches:

```bash
# A Console label or message, to confirm exact wording and find the screen
grep -rn --include='*.tsx' "Add MCP Proxy" "$AM_REPO/console/workspaces/pages"

# An endpoint
grep -rn '"POST /orgs/{orgName}/mcp-proxies' "$AM_REPO/agent-manager-service/api"

# A config key or env var
grep -rn "TRACE_SAMPLING" "$AM_REPO/agent-manager-service/config" "$AM_REPO/deployments/helm-charts"
```

Exclude `node_modules/`, `vendor/`, and `dist/` from searches; they are dependencies or build output, not source.

## Documenting an upstream change

When the request is "document PR #N" or "update the docs for feature X":

1. Find the change: `gh pr view <N> --repo wso2/agent-manager --json title,body,files` and `gh pr diff <N> --repo wso2/agent-manager`, or `git -C "$AM_REPO" log --oneline -- <path>`.
2. Read the linked design proposal or issue if the PR body points to one (GitHub Discussions, "Design Proposals" category). Treat it like a user-supplied draft (see `edit.md` Step 2): useful for intent, but the merged code wins where they disagree.
3. List every user-visible surface the change touches (Console, CLI, API, Helm values, config) and search `agent-manager/docs/` for pages that already mention each one. Update those pages before creating a new one.

## Citing sources

Whenever you write or verify a claim, note the evidence in your reply (not in the page), as `path:line` at the ref you used, for example `agent-manager-service/api/mcp_proxy_routes.go:29 @ main (a1b2c3d)`. If the evidence is a PR or a design proposal rather than merged code, say that plainly.

## Generated content

Some pages are generated from the source repo and must not be hand-edited:

| Page | Generated from | How to change |
|---|---|---|
| `agent-manager/docs/reference/helm-charts/*.mdx` | `deployments/helm-charts/<chart>/values.schema.json` (fetched over HTTPS at the tag) | Fix the description in the chart upstream, then regenerate: `node scripts/gen-helm-reference.mjs --tag <ref> --out agent-manager/docs/reference/helm-charts`, where `<ref>` is `main` on the main line and `amp/v<v>` on `agent-manager/release/<v>`. Add `--check` to report stale pages (non-zero exit) without writing. |

If you find a wrong value on a generated page, report where the upstream fix belongs instead of patching the `.mdx`.
