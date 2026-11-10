# `amctl` CLI Reference Pages

Create, update, or verify pages under `agent-manager/docs/reference/cli/` against the `amctl` source in `wso2/agent-manager`. The CLI is the source of truth: every command, argument, flag, type, default, and description on these pages must match it.

## Usage

Read for any request touching `agent-manager/docs/reference/cli/**`: documenting a new command or flag, updating a page after a CLI change, or verifying a page. Read `source.md` first to locate `AM_REPO` and pick the ref.

## Where the CLI lives

| What | Where (relative to `AM_REPO`) |
|---|---|
| Conventions (factory, output, scope resolution) | `cli/AGENTS.md` |
| Root command and global flags | `cli/pkg/cmd/root.go` |
| Command groups | `cli/pkg/cmd/<group>/` (`agent`, `api`, `context`, `gateway`, `llmprovider`, `project`, `skills`), plus `login.go`, `version.go` |
| One subcommand | `cli/pkg/cmd/<group>/<verb>.go` (for example `agent/deploy.go`), or a nested folder (`agent/build/`) |
| Flags | `cmd.Flags().<Type>Var[P](&opts.X, "<name>", "<short>", <default>, "<description>")` in the command file |
| Required flags | `cmd.MarkFlagRequired("<name>")` |
| Error codes (JSON mode) | `cli/pkg/clierr/` |

`amctl` is a Cobra CLI, so the help output is generated from the same definitions. When Go is available, the fastest accurate view of a command is its help text:

```bash
cd "$AM_REPO/cli" && go run ./cmd/amctl <group> <verb> --help
```

This needs no server or sign-in. If Go or the module cache is unavailable, read the command file directly.

## Page map

| Page | Documents |
|---|---|
| `index.mdx` | Installation pointer, global flags, command list (the CLI folder's landing page) |
| `login.mdx`, `version.mdx` | Top-level commands |
| `context.mdx`, `project.mdx`, `agent.mdx`, `llm-provider.mdx`, `gateway.mdx`, `skills.mdx`, `api.mdx` | One page per command group, every subcommand on it |

A new command group gets a new page in `agent-manager/docs/reference/cli/` (`type: reference`), with a `sidebar_position` that keeps the folder in the order a user meets the commands (login, context, project, agent, ...; renumber the siblings after it if needed). A new subcommand goes on its group's page. Installation steps belong in `agent-manager/docs/guides/installation/cli-installation.mdx`, not here.

## Update or verify a page

1. **List the commands in source**: every subcommand registered under the group (`cmd.AddCommand(...)`), including hidden ones (`Hidden: true`), which are not documented.
2. **List the commands on the page**: the Subcommands table and each `## amctl <group> <verb>` section.
3. **Diff them**: missing from the page → add; on the page but gone from source → remove (confirm with the user if it looks like a rename).
4. **For each command**, compare against source:
    - Synopsis line (`Use:` field, including `<required>` and `[optional]` arguments).
    - Short description (from `Short:` / `Long:`, rewritten to the style rules if needed, but not changed in meaning).
    - Arguments (`Args:` validator and how `args[i]` is used).
    - Every flag: name, shorthand, type, default, required, and description. Defaults must be the literal default in the flag definition; `""`, `0`, `false`, and `nil` are shown as `(none)` or `` `false` `` matching the page's existing convention.
    - Inherited flags (`--org`, `--project`, `--json`) listed under "Options inherited from parent commands" only when that command actually uses them (`ResolveScope` with `needsOrg` / `needsProject`).
    - Behavior the page states (confirmation prompts, fallback to the linked project, JSON envelope shape) matches the command's `runX` function.
5. **Examples**: every example must parse against the current flags. Do not invent output; if you show output, take it from the command's render code or ask the user for a real run. On the main line, a release version in an example or its output is `{edition.version}` (as on `version.mdx`), not a literal version.

Report each difference with the source location (`cli/pkg/cmd/agent/deploy.go:219 @ main`).

## Template for a subcommand section

Matches existing pages. Subcommand sections are separated by `---` lines, which is the established convention on these pages only.

````mdx
---

## amctl {group} {verb}

{One sentence: what the command does. Mention confirmation prompts or scope fallbacks here.}

```text
amctl {group} {verb} <{arg}> [flags]
```

### Arguments

| Name | Description |
|------|-------------|
| `<{arg}>` | {description} |

### Options

| Name | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--{flag}` | `-{s}` | {string/bool/int/stringArray} | {default or _required_} | {description} |

### Example

```bash
amctl {group} {verb} {realistic-args}
```
````

Drop the `Short` column when no flag in the section has a shorthand, as existing pages do. Also add the command to the page's Subcommands table with an anchor link (`[`{verb}`](#amctl-{group}-{verb})`).

## After editing

Run `wso2-docs check agent-manager` (its preview build checks the anchors in the Subcommands table), then `check.md` on the page. Style rules for reference pages (`style.md` → Reference) apply: no second person, tables over prose.
