# Full Review

Runs a complete review of one page (whole-file mode) or of every changed doc file (diff mode), combining `check.md`, `style.md`, and `tech.md` into one report.

## Usage

Read when the user asks for a full or complete review of a page, or a review before merging a docs PR.

- **A file path is given** → whole-file mode.
- **No path, and the user says "review my changes", "review the diff", or similar** → diff mode.
- **No path and nothing changed** → ask which file.

For any page under `agent-manager/docs/reference/cli/`, also run the verification in `cli-reference.md`; its mismatches count as `tech.md` CRITICAL issues.

## Diff mode

Reviews every changed `.md`/`.mdx` file, including uncommitted work.

### Step 1: Base ref

Default to the line the work branched from: `agent-manager/docs-v2` (the main line, which builds Cloud) or, for a fix to a released version, its release branch `agent-manager/release/<version>`. Use `origin/<branch>` if the local one is stale, and another branch if the user names one.

```bash
BASE_REF=agent-manager/docs-v2
MERGE_BASE=$(git merge-base "$BASE_REF" HEAD)
```

### Step 2: Changed files

```bash
git diff --name-only --diff-filter=ACMR "$MERGE_BASE" -- 'agent-manager/**/*.md' 'agent-manager/**/*.mdx'
git status --porcelain -- 'agent-manager/**/*.md' 'agent-manager/**/*.mdx' | awk '$1=="??"{print $2}'
```

Combine and de-duplicate. Also note whether any `_category_.yml`, `redirects.yaml`, `docspace.yaml`, or `editions.yaml` under `agent-manager/` changed, and whether a page was moved or deleted (`git diff --name-status "$MERGE_BASE"`); `check.md` → Menu placement applies to those. On a release branch, any change outside `agent-manager/` is a FAIL (`check.md` → Release branches). If the set is empty, say so and stop.

### Step 3: Run the Platform's checks

```bash
wso2-docs check agent-manager --base "$BASE_REF"
```

A blocking finding (MDX compile error, broken link or image, front matter, Edition marker, redirect coverage) is a `check.md` FAIL for the file it names; its warn-only lint (Vale, cspell, headings) feeds `check.md` and `style.md` as warnings. The last line says `Result: pass` or `Result: fail`.

### Step 4: Review each file, scoped to the diff

- **New file** (absent at the merge base): apply every check in full.
- **Existing file**: read `git diff -U3 "$MERGE_BASE" -- <file>` and apply the checks to changed lines and the sections containing them. Report pre-existing issues elsewhere in the file only under "Pre-existing (not blocking)", if at all.
- **Generated file** (`agent-manager/docs/reference/helm-charts/`): report it as hand-edited (see `check.md` → Generated pages) and skip the rest.

### Step 5: Report per file, then a rollup

Use the combined format below once per file, headed `Reviewing diff: <path> (vs <base ref>, includes uncommitted changes)`, then a final rollup: files reviewed, totals, overall PASS/FAIL.

Also list, for each changed page on the main line, which release branches hold a copy that may need the same fix (do not edit them; `edit.md` → Released versions explains when it applies and how to find them).

## Review sequence

Run each fully, in order, before writing the report:

1. `check.md`: `wso2-docs check`, front matter, headings, code blocks, lists, links, images, admonitions, MDX safety, menu placement, Editions and versions.
2. `style.md`: terminology, voice, formatting, AI patterns, specificity, page-type rules.
3. `tech.md`: every technical claim against `wso2/agent-manager` at the right ref (read `source.md` first).
4. `cli-reference.md` when the page is under `agent-manager/docs/reference/cli/`.

## Combined output

```text
Reviewing: <path>
Page type: <type>
Verified against: wso2/agent-manager @ <ref> (<short sha>)

─────────────────────────────────────
CHECK
<failures and warnings only>

STYLE
<findings only>

TECH
<findings, plus the category coverage summary>

─────────────────────────────────────
OVERALL RESULT: PASS / FAIL
Confidence: High / Medium / Low
Hard gate failures: <n> (<which reference>)
Failures: <n>
Warnings: <n>
```

## Confidence

- **High**: every reference passes with at most two low-severity warnings.
- **Medium**: non-gating failures or warnings remain, or some claims are UNVERIFIED.
- **Low**: any hard gate failed.

## Hard gates

The overall review FAILS if any of these hold:

- `check.md`: any FAIL, including `Result: fail` from `wso2-docs check`.
- `style.md`: 5 or more AI-vocabulary occurrences; any templated rhetorical pivot; 2 or more symmetric contrasts.
- `tech.md` (and `cli-reference.md`): any CRITICAL; 3 or more HIGH; an example UNVERIFIED without reason; an unconfirmed security claim; an unresolved cross-page contradiction; a CRITICAL or HIGH verdict based on memory rather than source.
