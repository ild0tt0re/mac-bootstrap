# Repository guidance

## Purpose and layout

This repository bootstraps and updates an Apple Silicon Mac using mise. It is configuration, not an application: there is no build system or automated test suite.

- `config.toml` is the single deployable file, installed manually as `~/.config/mise/config.toml`. It contains runtime versions, packages, repositories, services, hooks, managed zsh blocks, and inline tasks.
- `README.md` documents installation, updates, customization, and optional identity/SSH/history workflows in Italian. Keep user-facing documentation, descriptions, and comments consistent with that language.

## Editing conventions

- Preserve the single-file setup; keep shell tasks inline in TOML unless a change explicitly requires a different architecture.
- Keep the README synchronized with changes to packages, tasks, defaults, prerequisites, and update behavior. Describe the current setup and practical usage; do not retain historical logs, migration notes, or past design decisions.
- The documented baseline is macOS Apple Silicon, Apple Command Line Tools, mise >= 2026.10.7, standard home/config paths, and `/opt/homebrew` CLI paths.
- Runtime/package versions intentionally use `latest`; Node follows stable releases, not only LTS. Updates preserve previous runtimes (`upgrade.auto_prune = false`, `mise upgrade --no-prune`).
- Preserve idempotence and existing user configuration. Initial defaults are written only when files are absent, including checks for symlinks. Managed Git aliases and the two rerere settings are deliberate exceptions. Shell edits belong in the mise-managed blocks.
- Preserve quoting and literal multiline TOML strings for embedded shell. Bash-specific tasks explicitly declare `shell = "bash -c"`; managed shell blocks use zsh syntax.
- Keep credentials and personal identities out of tracked files. Git identities live in local files; SSH tasks reuse existing keys and verify them. History synchronization is manual.

## Validation

Start with static checks that do not apply the setup to the host:

```sh
python3 -c 'import pathlib, tomllib; tomllib.loads(pathlib.Path("config.toml").read_text()); print("TOML OK")'
git diff --check
```

The TOML check requires Python 3.11 or later. For shell changes, extract the relevant parsed string to a temporary file and run syntax checks with the matching interpreter (`sh -n`, `bash -n`, or `zsh -n`). Syntax checks do not validate mise schema or runtime behavior. There is no repository test runner; report what was actually checked.

Do not run bootstrap, update, dotfile application, SSH, identity, or synchronization tasks merely to validate a repository edit: they install software or change the user's machine and remote state. Editing the repository copy does not update the installed global config.

When the user requests host-level verification, the documented command from HOME is `mise run setup:verify`. It checks mise diagnostics/status and runtime availability in a new login shell; it does not verify app accounts or SSH authentication. The README also documents `mise bootstrap --dry-run`, `mise upgrade --dry-run`, and `mise bootstrap packages upgrade --dry-run`; bootstrap dry-run skips hooks and does not expand the final update task. Ensure any such check uses the intended configuration, and do not treat the installed global copy as evidence for uninstalled repository changes.
