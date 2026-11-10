# Docs Standards Checker

Validate one `.mdx` file against this repo's structural standards and report every violation. Writing quality is `style.md`'s job; technical accuracy is `tech.md`'s.

## Usage

Read when the user asks to check, lint, or verify a doc page, or before merging any `.mdx`. If no path is given, ask which file. Read the full file once, then evaluate each rule.

The WSO2 Docs Platform enforces its own rules (front matter, MDX compile, links, images, Edition markers, admonitions, folder skeleton) in `wso2-docs check agent-manager`; `wso2-docs help check` lists them. Run it, report its blocking findings as FAILs and its warnings as WARNs, then apply the checks below, which add this repo's rules and the judgment calls the tool cannot make.

## Checks

### 1. Front matter

- The file starts with a `---` YAML block. Missing → FAIL.
- `title`, `description`, and `type` are required by the Platform; `wso2-docs check` fails on a missing one, a `description` over 160 characters, an unknown `type`, or an unknown key. The allowed keys and values are the Platform's (`wso2-docs help write-a-page`, and the template from `wso2-docs new <type>`); do not restate them here.
- `sidebar_position` present → OK; missing → FAIL (this repo sets it on every page; without it the page sorts to the end of its folder).
- `description` starts with "This page/guide/document" → FAIL. A `description` still carrying the migration's `# generated from the first paragraph; a writer should improve it` comment, or the `wso2-docs new` placeholder text, → WARN.
- `type` should match the page's content and folder (`style.md` → Step 1). A mismatch → WARN.

### 2. No H1 in the body

The page's H1 comes from `title`. A `# ` heading in the body → FAIL (it would be a second H1). Ignore `#` lines inside fenced code blocks.

### 3. Heading hierarchy

A jump of more than one level downward (H2 → H4, or the first heading below H2) → FAIL, with line numbers. Going back up is fine. Ignore `#` lines inside fenced code blocks.

### 4. Code blocks

Every opening fence needs a language tag (` ```bash `, `yaml`, `json`, `python`, `go`, `ts`, `text`, `mermaid`, ...). Unlabeled → WARN with line number. New blocks always need one; retrofitting untouched old blocks is out of scope.

Commands the reader runs should be copy-pasteable: no leading `$ ` prompt inside a `bash` block unless output is shown in the same block → WARN.

### 5. Nested list indentation

A sub-list under a list item must be indented so its marker lines up with the parent item's text (4 spaces under `1. ` or `- ` is safe). 1 to 3 spaces of indentation for a nested marker often renders as a flat paragraph in MDX → FAIL, with line numbers.

### 6. Ordered vs. unordered lists

Actions the reader performs in sequence must be a numbered list. Bullets are for unordered sets: options, prerequisites, properties. A bulleted sequence of imperative steps → FAIL.

### 7. Internal links

- Links to other pages in this Doc space are **relative file paths with the `.mdx` extension** (`./register-mcp-proxy.mdx`, `../../concepts/gateway.mdx`, `./gateway.mdx#how-the-egress-gateway-is-selected`). Each release branch holds its own copy of the pages, so these stay correct in every Edition.
- Links to another Doc space use `docs:<doc space>/<edition|latest>/<page>` (`wso2-docs help write-a-page`).
- An absolute site path (`/docs/...`, `/next/...`, `/v1.0.0/...`, `/latest/...`) or a full URL to the docs site for an internal page → FAIL.
- A relative link without `.mdx` → WARN (works only by URL resolution and breaks easily).
- The only reliable link check is `wso2-docs check agent-manager`: its preview build fails on broken links, anchors, and images. Do not mark link fixes done without it.

### 8. Images

- Every image has descriptive alt text. `![](...)` → FAIL (`wso2-docs check` blocks a new image without alt text).
- Images live under `agent-manager/assets/img/<feature>/` and are referenced by a relative path from the page (`../../../assets/img/alerting/monitor-alerting-section.png` from `agent-manager/docs/guides/<category>/`). Missing file → FAIL:
    ```bash
    grep -o '](\.\./[^)]*assets/[^)]*)' <file> | sed 's/^](//; s/)$//' | while read p; do [ -f "$(dirname <file>)/$p" ] || echo "missing: $p"; done
    ```
- Dark and Edition-only variants (`name.dark.png`, `name.cloud.png`, `name.self-hosted.png` beside `name.png`) follow the Platform's paired-image rules (`wso2-docs help write-a-page`); the page references the base file only.

### 9. Admonitions

Only the Platform's five types (`:::note`, `:::tip`, `:::info`, `:::warning`, `:::danger`; `wso2-docs check` warns on others, such as `:::caution`, and on more than two in a row). Each opened `:::` is closed. Unclosed block → FAIL.

### 10. MDX safety

Bare `<` or `{` in prose outside code (for example `<namespace>` or `{orgName}` written as plain text) breaks MDX compilation. They must be in `inline code` or a code block → FAIL, with line numbers. `{edition.*}` variables are the exception (see 13). Imports (`import Tabs from '@theme/Tabs';`) must be used, and every used component must be imported → FAIL otherwise. Content repos hold no components of their own; `wso2-docs components` lists the ones pages can use. Shared snippets live in `agent-manager/partials/`.

### 11. Menu placement

There is no `sidebars.ts`: the folders under `agent-manager/docs/` are the menu. A page appears in the folder it sits in, ordered by `sidebar_position` among its siblings; a folder takes its label and position from its `_category_.yml` (`label`, `position`). Files starting with `_` are not pages.

- A new folder without a `_category_.yml` → WARN (every folder in this Doc space sets its `label` and `position` there).
- A top-level folder under `agent-manager/docs/` that is not one of the Platform's skeleton folders → FAIL (`wso2-docs check` reports it).
- `sidebar_position` that collides with a sibling's, or puts the page out of reading order → WARN:
    ```bash
    grep -H "^sidebar_position:" "$(dirname <file>)"/*.mdx | sort -t: -k3 -n
    ```
- A page moved or renamed in this change without a `moves:` entry (`from`, `to`) in `agent-manager/redirects.yaml` → FAIL (`wso2-docs check` reports it).

**Placement** (judgment, WARN only): does the folder match the page's topic and type? If clearly wrong (an installation guide under `guides/agent-identity-and-access-management/`), propose a better folder and position as a suggestion.

### 12. Generated pages

If the file is under `agent-manager/docs/reference/helm-charts/` and the diff touches it by hand → FAIL: these are generated (see `source.md` → Generated content).

### 13. Editions and versions

On the main line (`agent-manager/docs-v2`, which builds Cloud):
- Content that applies to only Cloud or only Self-hosted must carry an Edition marker (front matter `editions: cloud|self-hosted`, `selfHostedFrom: next` for a page new since the last release, or `<Edition only="cloud">…</Edition>` / `<Edition only="self-hosted">…</Edition>` inline or around a block). Self-hosted-only content (installation, Helm values, upgrades) with no marker, or wording like "on Cloud" / "in self-hosted deployments" that should be a marker → WARN, for `tech.md` to confirm. `wso2-docs check` validates the marker syntax; `wso2-docs components Edition` describes it.
- A release version written literally (`1.0.0`, `v1.0.0`, `amp/v1.0.0`) where it should track the Edition → WARN: write `{edition.version}` (`v{edition.version}` for image tags and `amp/v…` refs). Link addresses are not filled, so a version inside a Markdown link address stays `v0.0.0-dev`; do not flag that.

### 14. Release branches

If the work is on a release branch (`agent-manager/release/<version>`):
- The change touches only files under `agent-manager/` → FAIL otherwise (`wso2-docs check` blocks it).
- The content must describe the product at that version's tag (`amp/v<version>`), not `main`. Flag any wording about a feature added after the release → WARN, for `tech.md` to verify.

## Output format

```text
Checking: agent-manager/docs/guides/platform-administration/register-mcp-proxy.mdx

WSO2-DOCS CHECK
  ✅  Result: pass

FRONT MATTER
  ✅  title, description, type: how-to, sidebar_position: 4

HEADINGS
  ✅  no body H1, hierarchy sequential

CODE BLOCKS
  ⚠️   line 88: unlabeled code block

LISTS
  ❌  line 54: bulleted list describes sequential steps; use a numbered list

INTERNAL LINKS
  ❌  line 17: absolute link /next/concepts/gateway; use ../../concepts/gateway.mdx

IMAGES
  ✅  all images have alt text and exist

MDX SAFETY
  ✅  no bare < or { in prose

MENU
  ✅  guides/platform-administration/ (Platform Administration), position 4 of 9

EDITIONS
  ✅  no Edition-specific content without a marker

─────────────────────────────────────
2 failures · 1 warning
```

If everything passes, end with `✅  All checks passed`.
