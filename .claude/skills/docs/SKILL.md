---
name: docs
description: Handles every WSO2 Agent Manager documentation task in this repo in one skill: scaffolding a new page, writing content into an existing page, checking structural standards, reviewing writing quality and tone, verifying technical accuracy against the wso2/agent-manager source repo, reviewing CLI or Helm reference pages, or running the full pre-merge review. Use whenever asked to create, write, add, document, or fill in docs; check, lint, or verify a doc page; review a doc's style, tone, or quality; verify a doc's technical accuracy; update a CLI reference page; or do a full review before merging a docs PR.
allowed-tools: Read Write Edit Bash WebFetch WebSearch
---

# WSO2 Agent Manager Documentation Skill

One skill for every documentation task in `docs-agent-platform`, split into reference files so only the relevant one loads into context. This file is a dispatch table, not the rules themselves. The actual instructions live in the reference files below.

If the target path is a `SKILL.md` or any file under `.claude/skills/` or `.agent/skills/`, stop: these are agent instructions, not documentation, and none of these rules apply.

## The source of truth is the Agent Manager repo

This repo only holds the docs. Every product behavior the docs describe (Console flows, `amctl` commands, REST endpoints, config keys, Helm values, instrumentation, evaluators) is implemented in **[wso2/agent-manager](https://github.com/wso2/agent-manager)**. Any action that writes or verifies a technical claim (`new-page`, `edit`, `tech`, `cli`, `review`) must first read `source.md` to locate a local checkout of that repo and find the right files in it. Never describe how the product works from memory.

## The repo is a WSO2 Docs Content repo

The pages are the `agent-manager/` Doc space: pages in `agent-manager/docs/` (folders are the menu, ordered by `_category_.yml` and `sidebar_position`), images in `agent-manager/assets/`, and the manifests `agent-manager/docspace.yaml`, `editions.yaml`, and `redirects.yaml`. The WSO2 Docs Platform builds, checks, and releases it with its own tools:

- `wso2-docs` in these files means `npx @wso2-enterprise/docs` (needs the one-time GitHub Packages login in the repo's README) or, from a Platform checkout, `node <platform>/bin/wso2-docs.mjs`.
- `wso2-docs help` and `wso2-docs help <task>` (start with `write-a-page`) are the authority on WSO2 Docs authoring rules: page types, front matter, components, Edition markers, admonitions, and checks. Read the task help before changing anything. Where these files and the help disagree on a Platform rule, the help wins.
- `wso2-docs preview agent-manager [--edition cloud|2026.11.0|1.0.0]` serves a live preview; `wso2-docs check agent-manager` runs the pull request's blocking checks (there is no `npm run build`).

**Editions**: the `docs-v2` branch is the main line and builds the **Cloud** Edition. Each Self-hosted version is a release branch, `agent-manager/release/<version>` (now `2026.11.0` and `1.0.0`), holding that version's pages at the same `agent-manager/docs/` path. On the main line, content that applies to only one kind of Edition carries an Edition marker (`edit.md` → Editions).

## Step 1: Match the request to a reference file

### Explicit form

If invoked as `/docs <action> [file-path]`, or the request plainly names one of these action words, map directly:

| action | reference |
|---|---|
| `new-page` | `new-page.md` |
| `edit` | `edit.md` |
| `check` | `check.md` |
| `style` | `style.md` |
| `tech` | `tech.md` |
| `cli` | `cli-reference.md` |
| `review` | `review.md` |

### Natural-language form

| The user is asking to... | Read |
|---|---|
| Create a new page, or "write docs for X" / "document feature Y" when nothing covers the topic yet | `source.md`, then `new-page.md` |
| Write or fill in content on an existing page (a gap, a new section, a feature change) | `source.md`, then `edit.md` |
| Update docs to match a change or PR in `wso2/agent-manager` | `source.md`, then `edit.md` (or `new-page.md` if no page covers it) |
| Check, lint, or verify structural standards (front matter, headings, links, menu placement, Edition markers) | `check.md` |
| Review writing quality, tone, AI-sounding prose, voice consistency | `style.md` |
| Verify technical accuracy (Console steps, API, config, Helm, security claims) | `source.md`, then `tech.md` |
| Create or update a page under `agent-manager/docs/reference/cli/`, or verify one against `amctl` | `source.md`, then `cli-reference.md` |
| Run the full pre-merge review (structure + style + tech) | `review.md` |
| Review my changes / review the diff, no file named | `review.md` (diff mode) |

Default to `review.md` if the request just says "review this doc" without naming a dimension.

### Path-based overrides

- `agent-manager/docs/reference/cli/**` → prefer `cli-reference.md` over `tech.md` for any verify or update request.
- `agent-manager/docs/reference/helm-charts/**` → these pages are **generated** by `scripts/gen-helm-reference.mjs` from the charts in `wso2/agent-manager`. Do not hand-edit them. Tell the user to fix the chart's `values.yaml` / `values.schema.json` descriptions upstream and regenerate (see `source.md` → "Generated content").
- On a release branch (`agent-manager/release/<version>`) → you are editing a released version. Read "Release branches" in `check.md` before touching it.

Read the matched file(s) now and follow them exactly. When a reference file mentions another reference by name ("the check reference", "`style.md`"), read that file in this same directory.

## Step 2: State which reference(s) you are following

Before doing the work, say in one line which file(s) you read (for example "Following source.md and new-page.md"). This keeps the dispatch visible.
