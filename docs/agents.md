# Guide for agents

This page expands on `AGENTS.md` in the workspace root. When they disagree,
`AGENTS.md` wins. Each application repository may add its own rules in its
own `AGENTS.md`; read it before changing that repository.

## Orientation

- Start in the workspace root, not inside a child repository, so that the
  workspace instructions are loaded.
- Run `task status` first. It shows branch and local changes for every
  repository. Do not overwrite or commit changes you did not make.
- Run `task sync` only when updates are wanted. It pulls with `--ff-only` and
  never resets, stashes or rebases. If one pull fails, others may already
  have succeeded; report each result.
- `task --list` shows every routine. Add new routines to `Taskfile.yml`
  directly, never as wrapper scripts.
- `mani.yaml` is the inventory of repositories, paths and language tags
  (`go`, `python`). Select repositories with mani, for example
  `mani exec --projects tibi-mcp '...'` or `mani exec --tags go '...'`.

## Repository map

| Repository | Language | Test task | Own `AGENTS.md` | Image |
|---|---|---|---|---|
| mcp-workspace (`.`) | YAML, Python snippets | `task check` | this root | none |
| mcp-gateway | Go | `task test:gateway` | no | GHCR |
| bibliofil-mcp | Go | `task test:bibliofil` | no | GHCR |
| fribo-mcp | Go | `task test:fribo` | yes | GHCR |
| tibi-mcp | Go | `task test:tibi` | yes | GHCR |
| vigilo-connector | Python | `task test:vigilo` | yes | GHCR |
| spond-no-match-mcp | Python | `task test:spond` | no | GHCR |

Child repositories are ignored by this repository's Git. Some are symbolic
links to checkouts elsewhere; edits through a link change the original files.
Commit inside the child repository, not in the workspace.

## Making a change

1. Decide which repository owns the change. Application code belongs in its
   own repository; tools, commands and shared practices belong here.
2. Work directly on `main` (trunk-based), unless the repository says otherwise.
3. Keep commits small with one logical change each. Do not add
   `Co-Authored-By` trailers or any other AI attribution.
4. Verify (see below), then push when checks pass. If verification needs a
   human, wait for their confirmation before pushing.
5. After pushing, follow the GitHub Actions runs for the pushed commits to
   completion (`gh run list`, `gh run watch`). Fix failures caused by your
   change and report the rest.
6. Report what you actually verified, and what you could not verify.

## Verification

| Change | Run |
|---|---|
| Workspace files (`Taskfile.yml`, `mani.yaml`, `compose.yaml`, `mise.toml`) | `task check` |
| Documentation in `docs/` or `mkdocs.yml` | `task docs:build` |
| Local checkouts present | `task check:repos` |
| Application code | the matching `task test:<name>`, or `task test` for all |

Configuration checks do not replace application tests. `task check` proves
neither application behaviour nor a working deployment.

CI for this repository runs `task check`, `task docs:build` and a whitespace
check on every push. Application repositories run their own workflows and
publish images to GHCR.

## Local environment

- `task env:init` writes `.env` with mode 600 and random credentials. It never
  prints or overwrites existing values. Never print or commit `.env`.
- `task dev:up` builds from the local checkouts and starts the Compose
  project `mcp-workspace-dev` on `127.0.0.1:18080`. Only the gateway is
  published. `task dev:down` keeps the data volumes.
- Servers start with `MCP_READ_ONLY=true`. Account settings are entered on
  the gateway admin page and stored in each server's volume.
- Use synthetic data. Do not import production tokens, volumes or login state.

## Safety rules

- Never print, log or commit credentials, tokens, personal data or local
  environment files. This also applies to OpenTelemetry spans.
- **Vigilo rotates refresh tokens.** Only one active process may hold an
  account's refresh token. Logging in locally with an account that is used
  elsewhere logs out the other holder. Plan the migration with the user
  first, and never start obsolete services.
- `task tibi:call` makes exactly one call against the real Tibi account with
  credentials from 1Password. Use it only when asked; do not retry writes
  (`borrow_ebook`, `send_to_kindle`, `set_kindle_email`).
- Publishing releases and changing production need an explicit request, in
  addition to permission to push.

## Adding an MCP server

Prefer Go and use fribo-mcp and tibi-mcp as reference implementations.

The server must:

- run only behind mcp-gateway, with no public port, login or OAuth of its own;
- serve Streamable HTTP at `/mcp` with `Authorization: Bearer <MCP_AUTH_TOKEN>`;
- serve `GET /healthz` without auth and `GET /healthz?check=1` with real checks;
- serve `GET /config` and `POST /config`, with `CONFIG_TOKEN` defaulting to
  `MCP_AUTH_TOKEN`. Keys are environment variable names, secrets are never
  returned, environment-set values are `locked`, and saved settings apply
  without a restart;
- store settings in `DATA_DIR` (default `/data`, files mode 0600), with
  environment variables taking precedence;
- be stateless per request where possible;
- trace MCP methods and upstream requests with OpenTelemetry, configured by
  standard `OTEL_*` variables and disabled without an OTLP endpoint.

Then register it in all of these places:

- [ ] `mani.yaml` (path, URL, language tag)
- [ ] `compose.yaml` here: service, volume, `UPSTREAMS` and `UPSTREAM_TOKEN_<NAME>`
- [ ] `Taskfile.yml` here: `test:<name>` and the new token in `env:init` and `check:env`
- [ ] mcp-gateway `docker-compose.yml`, `.env.example` and `README.md`
- [ ] the docs in `docs/` (repository tables and architecture diagram)
- [ ] a GHCR image workflow in the server's own repository

## Documentation

The docs are built with MkDocs Material from `docs/` and `mkdocs.yml`.
`task docs:serve` previews them, `task docs:build` builds them in strict mode
into the ignored `.local/site/` directory. Diagrams are Mermaid code blocks
(` ```mermaid `). Keep `index.md` short and human-oriented; put detail for
agents on this page. Update the diagrams when servers or flows change.
