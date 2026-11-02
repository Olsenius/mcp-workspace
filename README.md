# MCP-workspace

Shared tools and working practices for [mcp-gateway](https://github.com/Olsenius/mcp-gateway),
[vigilo-connector](https://github.com/Olsenius/vigilo-connector),
[bibliofil-mcp](https://github.com/Olsenius/bibliofil-mcp) and
[fribo-mcp](https://github.com/Olsenius/fribo-mcp) and
[tibi-mcp](https://github.com/Olsenius/tibi-mcp), following the workspace-concept pattern: mise for tools,
Task for commands, mani for repositories and AGENTS.md for agent instructions.

Each application keeps its own Git history. This repository owns the surrounding
workspace configuration.

```text
mcp-workspace/
  .github/workflows/workspace.yml
  AGENTS.md
  mise.toml
  mani.yaml
  Taskfile.yml
  compose.yaml
  mcp-gateway/        # separate repository, ignored
  vigilo-connector/   # separate repository, ignored
  bibliofil-mcp/      # separate repository, ignored
  fribo-mcp/          # separate repository, ignored
  tibi-mcp/           # separate repository, ignored
```

Existing checkouts can be linked into the workspace using ignored symbolic
links. Changes through those links affect the original files. On a new machine,
mani clones each project into its configured directory. GitHub SSH access is
required for cloning and synchronization.

## Getting started

Install Git, mise and Docker with Compose, then run from the workspace root:

```bash
mise trust
mise install
mise exec -- task clone
mise exec -- task setup
mise exec -- task check
mise exec -- task check:repos
mise exec -- task --list
```

Tool versions are pinned in mise.toml. Installation and cloning require network
access. Activate mise in your shell to use `task` directly; otherwise use
`mise exec -- task ...`. Python dependencies follow the application's
pyproject.toml; this workspace does not pin its transitive dependencies.

## Daily work

```bash
task status
task sync             # clone missing projects and fast-forward pull all projects
task check            # validate the workspace
task test:gateway
task test:vigilo
task test:bibliofil
task test:fribo
task test:tibi
task test             # all application suites, sequentially
```

Go tests use `go test -race ./...`. Vigilo uses `.venv/bin/python -m pytest -q`
from its repository. Read each repository's instructions before changing code.

mani.yaml is the repository inventory, including this workspace (`path: .`).
The `go` and `python` tags select projects for setup and combined testing.
Project-specific tasks select a project by name through mani. Compose separately
specifies build contexts for the local services.

`task clone` runs `mani sync --parallel`, which clones missing repositories and
updates .gitignore. `task sync` then verifies that the workspace has a commit and
an upstream, before running `git pull --ff-only` in every project in parallel.
A new workspace needs its first commit and a configured upstream before sync.

If one pull fails, other repositories may already have been updated. Inspect
results for every project. No automatic reset, stash or rebase is performed.
Git determines whether local changes prevent a pull. Use `task status` first.

## Local development environment

```bash
task env:init
task check
task dev:up
task dev:status
task dev:down
```

This builds local code and starts the separate Compose project
`mcp-workspace-dev` with its own data volumes. The gateway admin page is at
<http://localhost:18080/admin>. Backends do not publish host ports.
If you change DEV_PORT in the ignored `.env`, update PUBLIC_URL as well.
The admin password is stored in `.env` with mode 600. `env:init` never overwrites
an existing value; it only adds credentials that are missing. `task build` only
builds images; `dev:down` preserves data.

Local HTTP is intended for development. Public clients need a separate HTTPS
and OAuth setup. To run the published images with a tunnel, use the
[gateway's Compose example](https://github.com/Olsenius/mcp-gateway#quick-start-docker-compose).
Setup and validation tasks do not start application services.

Configure test accounts through the gateway. Bibliofil and Fribo start with write
tools hidden (`MCP_READ_ONLY=true`). Vigilo rotates refresh tokens: do not log in with
an account already used by another active token holder without planning the
migration. Production volumes, credentials and login state are not imported.

Start your agent at the workspace root and have it read AGENTS.md. Do not assume
workspace instructions are loaded when starting inside a child repository.

## Workspace checks and CI

GitHub Actions runs `task check` on pushes, pull requests and manual dispatches.
The same command runs locally. It checks:

- Taskfile parsing, mani configuration and mise TOML syntax.
- Compose configuration using synthetic tokens, without loading the local `.env`.
- Go and Python project selection through mani listings.
- Environment initialization in a temporary directory: required settings,
  distinct random credentials, mode 600, repeated execution and protection of
  an existing symlink target, and that a credential missing from an older `.env`
  is added without changing existing values.

CI does not need child repositories, account credentials or running containers.
`task check:repos` separately checks that all mani projects have local Git
checkouts. Application test suites remain in their own repositories; invoke
them through the workspace test tasks when changing application code.

Workspace checks do not prove application behavior or a working deployment.
Container builds and startup have not yet been verified as part of this setup.

Tibi supports public e-book search without credentials. Like the other
servers, its account is set on the gateway admin page (stored in the `tibi-data`
volume), and it starts read-only. Use synthetic data in the dev environment;
`task tibi:call` makes single calls against the real account with credentials
from 1Password.
See `tibi-mcp/README.md` for verification limitations and setup.
