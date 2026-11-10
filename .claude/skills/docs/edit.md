# Docs Writing

Write new content into an existing `.mdx` page: a `{placeholder}` left by `new-page.md`, a thin or outdated section, a brand-new section, or an update for a change in `wso2/agent-manager`. **Never fabricate a technical claim.** An empty section, a rough draft, or "add a section on X" is not permission to guess how Agent Manager works. Structure and link format are `check.md`'s job; prose review is `style.md`'s.

## Usage

Read when the user asks to write, draft, add, update, or fill in content on a page that already exists. Read `source.md` first to locate `AM_REPO`. For a brand-new page, use `new-page.md` first.

The user may hand over a draft, design proposal, PR description, or notes. Treat it as one verification source in Step 2, not a replacement for the steps below.

## Step 1: Name the claim each section needs

Before writing, state in one line the concrete fact the section requires. "## Configure the Endpoint" needs something like "the Console path to the endpoint form, each field's label, which fields are required, and the defaults."

## Step 2: Verify each claim

Check each claim against these sources, in order:

1. **User-supplied material** (draft, design proposal, PR body). Treat a stated claim as a lead, then confirm it in code (source 2). If the code contradicts it, show the user both and ask which is correct; do not pick silently. A forward-looking claim about unmerged work may rest on the user's material alone, if the user confirms the page is meant to describe that branch.
2. **The `wso2/agent-manager` source** at the right ref (`source.md` → Step 2 and the Source map). This is the strongest evidence. Console labels come from the literal strings in `console/workspaces/pages/**`; API behavior from `agent-manager-service/api/` and the OpenAPI spec; config from `config/` and the Helm `values.yaml`.
3. **An existing, correct page in this repo**. Link to it or summarize it instead of re-deriving. Check it agrees with source before relying on it.
4. **An external standard** (MCP spec, OAuth RFCs, OpenTelemetry spec, Kubernetes docs), only for claims about the standard itself. How Agent Manager implements that standard still needs source 2.

Training knowledge is not a source.

For every confirmed claim, note the evidence (`path:line @ ref`) for your reply. If the evidence is user material rather than code, say so plainly ("per the design proposal you linked").

## Step 3: If a claim is not verifiable, stop and ask, specifically

Do not write speculative content or ask a vague "what should I write here?" Name the exact missing fact:

> I could not verify **{specific claim}** from the agent-manager source at `{ref}`, the existing docs, or the material you provided. To write this accurately I need:
> - {specific piece of information}
> - {another, if any}
>
> I will draft the section once I have this.

Batch every open question for the page into one message.

## Step 4: Draft only what is confirmed

Write the sections you verified. For anything still blocked, leave the `{placeholder}` as-is, or do not add the sentence yet; say what is missing rather than writing around the gap.

### Write to the style rules while drafting

Before drafting, read `style.md` → Universal Checks and the entry for this page's type, and write to them directly: active voice, second person, one action per numbered step, exact **bold** UI labels copied from source, `inline code` for commands, keys, and paths, no filler or AI vocabulary, no condescension ("simply", "just"), consistent terminology. Match the voice of sibling pages. The review step still runs afterwards, but it should find little to fix.

### Console steps

- Copy every label, button, tab, and menu name exactly as it appears in the Console source, including capitalization.
- Give the navigation path from a stable starting point ("In the Console, navigate to **Organization Settings** → **MCP Proxies**"), not "open the settings".
- Note the scope the reader must be in (organization, project, or environment) when the screen depends on it.
- State the expected result when it is not obvious (what appears, what status to wait for).

### Step count and screen locality

Before finalizing a numbered sequence, count the steps and check each one's screen:

- **10 or more steps in one sequence**: propose a restructure (split pages, group into `## Step N:` phases, fold trivial steps like "Click **Save**" into the previous step) and ask which the user prefers.
- **Back-and-forth between screens with no dependency forcing it**: propose the regrouped order and ask.

Ask both in one message if both apply.

### Diagrams

Do not add a diagram just because one could exist. If you think one would help and the user did not ask, ask first, naming the type (sequence, flowchart, architecture) and why. When drawing one, use a fenced ` ```mermaid ` block and no per-diagram color overrides, and make every label match the terms used in the prose.

### Editions and versions

The main line (`docs-v2`) builds Cloud, and Self-hosted versions are cut from it. Before writing, decide which Editions the content applies to, using the source (`source.md` → Step 2), and mark anything that does not apply to both:

- **A whole page**: front matter `editions: cloud` or `editions: self-hosted`; `selfHostedFrom: next` for a page new since the last Self-hosted release.
- **A block or a phrase**: `<Edition only="cloud">…</Edition>` or `<Edition only="self-hosted">…</Edition>`.

`wso2-docs components Edition` and `wso2-docs help write-a-page` have the exact syntax (folder markers, Edition-only images); check the result in `wso2-docs preview agent-manager`, which outlines the markers and switches Editions. Do not write "on Cloud" or "in self-hosted deployments" in prose where a marker would show the reader only what applies.

Write release versions on the main line as `{edition.version}` (`v{edition.version}` for image tags and `amp/v…` refs), which each Edition fills in. Variables are not filled in a Markdown link address, so a version there stays `v0.0.0-dev`.

### Released versions

After editing a page on the main line, check whether the same page exists on a release branch and whether the change applies there:

```bash
for b in $(git branch -a --format='%(refname:short)' --list '*agent-manager/release/*'); do
  git cat-file -e "$b:agent-manager/docs/{same-relative-path}" 2>/dev/null && echo "$b"
done
```

(If no release branches are listed, `git fetch origin` first; it does not touch the working tree.) Each release branch describes the product at its tag. Only carry a change to a release branch when it fixes something that was also wrong at that release (verify against the tag, per `source.md`), and ask before editing one. The fix is a separate change on that branch, which may touch only files under `agent-manager/`.

## Step 5: Report

List what you wrote, the evidence behind each claim, the Edition markers you added, any open questions or placeholders left, and whether release branches need the same change. Suggest `review.md` before the PR.
