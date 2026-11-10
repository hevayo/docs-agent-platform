# Technical Accuracy Review

Review a page as a senior Agent Manager engineer would: catch claims that are wrong, subtly wrong, outdated, or oversimplified to the point of being wrong, before a reader tries to follow them. Every verdict rests on the `wso2/agent-manager` source, not memory.

Not in scope: writing quality (`style.md`) and structure (`check.md`).

## Usage

Read when the user asks to verify a page's technical accuracy, or before merging docs with new technical claims. Read `source.md` first: locate `AM_REPO` and choose the ref that matches the branch the page is on (`docs-v2`, the main line → `main`; `agent-manager/release/<v>` → tag `amp/v<v>`). If no path is given, ask which file.

If the source repo cannot be reached, mark affected claims UNVERIFIED with that reason. Never mark them correct by assumption.

## Step 1: List the claims

Read the whole page. As working notes (not output), list every non-trivial claim, grouped by category: Console navigation and labels; resource behavior (scope, lifecycle, what happens on deploy or update); API endpoints and fields; CLI commands and flags; configuration, env vars, and Helm values; installation steps; security behavior; protocol or standard claims; code and command examples; numbered steps (each one claims the action works and produces the stated result).

## Step 2: Verify by category

Mark non-applicable categories N/A with a short reason. For each claim, record the evidence as `path:line @ ref`.

### 1. Console flows and labels

Find the screen in `console/workspaces/pages/<feature>/src/`. Confirm every bold label, button, tab, and field matches the literal string, the navigation path exists at the stated scope (organization, project, environment), required fields are marked required, and defaults match. Behavior that only exists at runtime (async status changes, error toasts) and cannot be confirmed from source → UNVERIFIED, flagged for manual check in a running instance.

### 2. Resource behavior

Scope, lifecycle, and side effects (for example "an MCP Proxy deploys one artifact per configured environment", "agents pick up proxy changes on their next deployment"). Verify in `agent-manager-service/services/` and related controllers. These are the claims most likely to drift after a refactor.

### 3. API endpoints

Path, method, and required permission from `agent-manager-service/api/*_routes.go` (`HandleFuncWithValidationAndAuthz("<METHOD> <path>", rbac.<Perm>, ...)`). Request and response fields, types, and required-ness from `agent-manager-service/docs/api_v1_openapi.yaml`. Observer endpoints from `agent-manager-observer/docs/openapi.yaml`. Never assume an endpoint exists because the page describes it.

### 4. CLI commands

Follow `cli-reference.md` → "Update or verify a page" for any `amctl` usage, even on pages outside `agent-manager/docs/reference/cli/`.

### 5. Configuration and Helm values

Env vars and defaults from `agent-manager-service/config/`; Helm keys, types, and defaults from `deployments/helm-charts/<chart>/values.yaml` and `values.schema.json`. Check key casing and nesting exactly. A default that changed since the page was written is a common failure.

### 6. Installation and scripts

Quick start and installer steps against `deployments/quick-start/` (`install.sh`, `start.sh`); environment and identity provider scripts against `deployments/scripts/`; VM installs against `deployments/vm/`. Check prerequisite versions, command names, flags, and the order steps must run in.

### 7. Security claims

For any "prevents / protects / secures / validates" claim: confirm the mechanism in source (the middleware, gateway policy, or RBAC permission that enforces it) and against one independent authority (OWASP, the relevant RFC's security considerations, the MCP spec). The claim must hold unconditionally or carry the qualifier it needs. Watch for: API keys vs. JWTs conflated; "the gateway authenticates the agent" when it only checks an API key; default credentials described as safe for production; permissions stated without the scope they apply at.

### 8. Protocols and standards

MCP, OAuth 2.0 / OIDC, JWT, OpenTelemetry: fetch the spec (`WebFetch`) and confirm the mechanism, not just the wording. OAuth 2.0 is authorization, OIDC adds authentication; signed JWTs are tamper-evident, not private; OpenTelemetry head sampling happens in the SDK before export.

### 9. Code and command examples

Syntax parses; package names, imports, and env var names match what the code reads (`libs/amp-instrumentation/`, `samples/`); no unflagged placeholders in runnable code; every required step before the example is on the page.

### 10. Numbered steps

Prerequisites are complete; order matters where the page implies it does; each step gives everything the next needs; stated results match the product. A missing step is a false claim by omission.

### 11. Cross-page consistency

Search `agent-manager/docs/` on the branch you are on for the same topic (to read another branch without checking it out: `git grep -n '<term>' <branch> -- agent-manager/docs`). If this page says X and another says not-X, that is CRITICAL; name both locations and which one source supports.

### 12. Internal consistency and diagrams

No contradictions within the page (a field "optional" in one place and "required" in another). Mermaid diagrams use the same names, counts, and order as the prose.

### 13. Edition scope

On the main line, each claim must hold in every Edition it renders in. Content with no Edition marker shows in Cloud and the Self-hosted versions; check that Self-hosted-only behavior (installation, Helm values, upgrades, cluster access) is marked `only="self-hosted"` and Cloud-only behavior `only="cloud"`, and that `selfHostedFrom: next` content is not already in the latest release (`check.md` → Editions and versions). A claim shown in an Edition where it is false is HIGH; a release version written literally instead of `{edition.version}` is MEDIUM. Use `wso2-docs preview agent-manager --edition <id>` to see what each Edition shows.

## Step 3: Reading pass

Read end to end as a first-time reader following along. Would these steps produce a working result? Is there a sentence that sounds right but is wrong? Would you push back on anything in code review?

## Severity

- **CRITICAL** (hard gate): flatly wrong and will cause failure or distrust. A nonexistent endpoint, flag, field, or Console label; a wrong default that misconfigures an install; a backwards security claim; a cross-page contradiction.
- **HIGH**: defensible but misleading enough that a reader would hit a wall. A missing prerequisite, a step that only works under unstated conditions, an unqualified conditional security claim.
- **MEDIUM**: correct but imprecise, outdated terminology, or missing a qualifier.
- **LOW**: a minor imprecision the reader probably does not need.
- **UNVERIFIED**: cannot be checked from available sources; state why. Not a gate by itself, but a human must resolve it before merge.

## Output format

```text
Reviewing: agent-manager/docs/guides/platform-administration/register-mcp-proxy.mdx
Verified against: wso2/agent-manager @ main (a1b2c3d)
Claims identified: 24

CRITICAL ISSUES

C1: Wrong button label
  Location: Step 2, line 61
  Page says: "Click **New Proxy**."
  Source: console/workspaces/pages/mcp-proxies/src/MCPProxies.Organization.tsx:88 uses "Add MCP Proxy"
  Suggested rewrite: "Click **Add MCP Proxy**."

HIGH ISSUES
  (same format)

MEDIUM ISSUES
  (same format)

LOW ISSUES
  L1: one line each

UNVERIFIED
  U1: "The status changes to Deployed within a minute" (line 97). Timing is runtime behavior; confirm in a running instance.

CATEGORY COVERAGE
  ✅  Console flows: 9 labels, 3 paths checked
  ✅  Resource behavior: ...
  N/A CLI commands: none on page
  ...

─────────────────────────────────────
Result: PASS / FAIL
C: 1 · H: 0 · M: 0 · L: 1 · Unverified: 1
```

## Hard gates

The review FAILS if any of:

1. Any CRITICAL issue.
2. 3 or more HIGH issues.
3. Any code or command example UNVERIFIED without a stated reason.
4. Any security claim not confirmed in source plus one independent authority.
5. An unresolved cross-page contradiction.
6. Any CRITICAL or HIGH verdict based on training knowledge instead of source or a fetched spec.
