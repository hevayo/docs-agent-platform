# New Doc Page Scaffold

Create a new `.mdx` page in the `agent-manager/` Doc space, place it in the menu, ground it in the Agent Manager source, and verify it passes `wso2-docs check`. The page should be ready for content (or already contain verified content) and CI should pass with no follow-up steps.

## Usage

Read when the user asks to create, add, or write documentation for a new concept, how-to, tutorial, or reference page, including "document this feature" or "write docs for X" when no existing page covers the topic. Read `source.md` first; this file assumes you have located `AM_REPO`. Read `wso2-docs help write-a-page` too: it is the authority on page types, front matter, and components, and this file does not restate them.

## Phase 1: Collect Information

Collect everything up front. If anything is missing, ask for all of it in one message, not one question at a time.

1. **Page type**: one of the Platform's page types (`how-to`, `concept`, `reference`, `tutorial`, `quick-start`, `release-notes`; each one's purpose and structure is in `wso2-docs help write-a-page`). A `quick-start` page is rare; confirm with the user before creating one.
2. **Title**: the page's H1 and browser title, set in `title:` front matter.
3. **Feature in the source repo**: the PR, issue, design proposal, or code area in `wso2/agent-manager` this page documents. If the user does not know, search for it (`source.md` → Source map) and confirm what you found. This is what the page will be verified against.
4. **File path**: under `agent-manager/docs/`. Infer from the type and the existing folders: concepts in `concepts/`, how-tos in `guides/<category>/` (for example `guides/platform-administration/`, `guides/securing-agent-endpoints/isolation-tiers/`), tutorials in `tutorials/`, reference in `reference/` (CLI pages: see `cli-reference.md`). File names are kebab-case and usually match the title (`Register an MCP Proxy` → `register-mcp-proxy.mdx`).
5. **Editions**: new pages go on the main line (`docs-v2`), which builds Cloud. Ask whether the feature is Cloud only, Self-hosted only, or both, and whether it is new since the last Self-hosted release (then `selfHostedFrom: next`; see `edit.md` → Editions and versions). Ask whether a released version also needs the page on its release branch (`agent-manager/release/<version>`); do not add it there unless the user confirms.

### Check for an existing page first

Before creating anything, search for a page that already covers this topic (2 or 3 significant title words, skipping generic ones like "guide" or "configure"):

```bash
grep -rli "{KEY_TERM}" agent-manager/docs --include='*.mdx'
grep -rn "^title:\|^sidebar_label:" agent-manager/docs --include='*.mdx' | grep -i "{KEY_TERM}"
```

Read enough of each hit (title, intro, headings) to judge genuine overlap, not a shared keyword. If a genuine match exists, stop and ask:

> I found an existing page that may already cover this: **{EXISTING_TITLE}** (`{EXISTING_PATH}`). Should I edit that page instead, or is this a genuinely different page?

**Edit existing** → stop this file and follow `edit.md`. **Different page** → continue.

## Phase 2: Gather the facts from source

Following `source.md`, read the aspect `AGENTS.md` and the implementing code for the feature. Build a short working list (not written to the page) of the facts the page needs, each with its `path:line` evidence: Console navigation and exact labels, fields and their defaults, endpoints, CLI commands, config keys, limits, error cases.

Anything the page needs that you cannot confirm from source goes into the Phase 1-style batched question in `edit.md` Step 3. Never fill a gap with a plausible guess.

## Phase 3: Create the page from its template

Create the file with the Platform's template, which has the front matter and the page type's headings:

```bash
wso2-docs new <page-type> agent-manager --title "{TITLE}" --folder <path under docs/> [--file <name>] [--description "{DESCRIPTION}"]
```

Pass `--folder` every time (without it, the tool uses the type's default folder, such as `get-started/` for tutorials, while this Doc space keeps tutorials in `tutorials/`). Keep the template's headings and fill them in.

**Front matter**: the template sets `title`, `description`, and `type`; the rules for those and any other key are the Platform's (`wso2-docs help write-a-page`). Then:

- Add `sidebar_position` (every page in this repo has one). See Phase 4.
- `description`: replace the template's placeholder with one sentence of at most 160 characters that does not start with "This page/guide/document".
- `sidebar_label`: only when the nav label should be shorter than the title (common in `reference/cli/`).
- Edition markers from Phase 1 (`editions:`, `selfHostedFrom:`), if any.

**Content**:
- If you verified facts in Phase 2, write them into the template now, following `edit.md` Step 4 (the style rules apply while drafting, not only at review).
- Anything still unverified stays as a literal `{PLACEHOLDER}` (inside `inline code`, so the page still compiles). Do not invent prerequisites, steps, field names, defaults, or links.
- A well-founded guess (for example, from a sibling page's structure) goes in your reply as a suggestion to confirm, never directly into the file.
- Screenshots go in `agent-manager/assets/img/<feature>/` and are referenced relatively from the page (`../../../assets/img/<feature>/<name>.png` from `guides/<category>/`) with descriptive alt text. Never invent image paths; leave `{SCREENSHOT: description}` if the user has not supplied the image.
- Diagrams use a fenced ` ```mermaid ` block, never ASCII art. No per-diagram `%%{init}%%` or `classDef` color overrides.
- Links to other pages are relative and include the `.mdx` extension.

For a `reference` page about CLI commands, use the template in `cli-reference.md` instead; Helm chart pages are generated (see `source.md`). For a tutorial, the prerequisites usually point at the [Quick Start Guide](../get-started/quick-start.mdx).

## Phase 4: Place in the menu

There is no `sidebars.ts`: the folder a page sits in is its place in the menu, and `sidebar_position` orders it among the pages and subfolders in that folder (a subfolder's own position is the `position` in its `_category_.yml`).

### Step 1: Pick the folder

In priority order:
1. **Directory match**: an existing folder that holds pages on the same feature.
2. **Topic match**: the existing folders under `guides/` are `installation/`, `evaluation-and-observability/`, `securing-agent-endpoints/`, `platform-administration/`, and `agent-identity-and-access-management/`. `reference/` has `cli/`, `helm-charts/`, and top-level pages.
3. **Reader task**: place by what the reader is doing at that moment, not a shared keyword.

Top-level folders are fixed by the Platform's skeleton; add new folders only below them. If nothing fits, propose a new folder with a `_category_.yml` shaped like its siblings (`label`, `position`, `collapsed: false`), positioned in reading order, not alphabetically.

### Step 2: Pick the position

List the siblings in order:

```bash
grep -H "^sidebar_position:" agent-manager/docs/<target-dir>/*.mdx | sort -t: -k3 -n
grep -H "^position:" agent-manager/docs/<target-dir>/*/_category_.yml
```

Place the page after the pages it depends on and before pages that build on it; otherwise at the end (highest value + 1, or `1` if there are none). Concepts are ordered in the sequence a reader meets them (organization → environment → project → agent → ...), not alphabetically. If the page goes between two siblings with consecutive positions, the siblings after it need renumbering.

### Step 3: Get approval

Present and wait for approval: folder, `sidebar_position` (after/before which page), any sibling renumbering, any new `_category_.yml`, and the `sidebar_label` if one is used. Revise if the user wants something else. Then apply it.

## Phase 5: Verify

```bash
wso2-docs check agent-manager
```

It runs the pull request's blocking checks (MDX compile, front matter, links, images, Edition markers, the preview build) and warn-only lint; it is done when the last line says `Result: pass`. Look at the page in `wso2-docs preview agent-manager` (add `--edition 1.0.0` or another version to see a Self-hosted Edition). Then run `check.md` against the new file.

## Phase 6: Report

Tell the user: the file created; its folder, `sidebar_position`, and any renumbering; the Edition markers; the `wso2-docs check` result; which facts you verified and against what (`path:line @ ref`); every remaining `{PLACEHOLDER}` and what information it needs; and that `review.md` is available once content is filled in.
