# Working in MCP-workspace

- The workspace owns tools, commands and shared working practices. Application
  code belongs in its own repository. Read that repository's AGENTS.md before
  making changes.
- Use `task --list` to discover shared routines. Define commands and task logic
  directly in Taskfile.yml; do not introduce wrapper scripts for tasks.
- mani.yaml is the source of truth for repositories, paths and language tags.
  Use mani to select repositories.
- Use trunk-based development: work directly on the default branch unless the
  individual repository's instructions say otherwise.
- Make small, focused commits with one logical change per commit.
- Never add `Co-Authored-By` trailers or other commit attribution to Claude,
  Codex, Pi or any other AI tool or agent.
- Push when the task is complete and relevant automated checks pass, if the
  task can be verified automatically. If manual verification is required, wait
  until it has been performed and its result confirmed. Follow repository-specific
  rules where applicable.
- After pushing, check GitHub Actions for the pushed commits and follow runs to
  completion. Report failures and fix those caused by the changes. Local commits
  do not trigger GitHub Actions.
- Run `task status` before working. Run `task sync` when updates are wanted;
  it clones missing repositories and pulls all projects with `--ff-only`.
- For workspace changes, run `task check`; use `task check:repos` to verify local
  checkouts. For application changes, run the relevant `task test:gateway`,
  `task test:vigilo`, `task test:bibliofil`, or `task test` for all applications.
  Configuration checks do not replace application tests.
- Report what was actually verified and any failures or limitations.
- Use synthetic data. Never print or commit credentials, tokens, personal data
  or local environment files.
- Vigilo rotates refresh tokens. Only one active process may hold an account's
  refresh token. Do not import production tokens or start obsolete services.
- `task dev:up` starts an isolated local environment. Logging in with an existing
  Vigilo account requires a planned migration from its active token holder.
- Publishing releases and changing production require an explicit request
  beyond the push authorization above.

## New MCP servers

- Prefer Go for new MCP servers.
- Expose servers remotely through mcp-gateway rather than publishing them
  directly.
- Prefer the Streamable HTTP transport.
- Keep servers stateless where possible, so each request can be handled
  without server-side session state.
- Instrument MCP servers with OpenTelemetry tracing for MCP methods and
  upstream requests. Configure export with the standard `OTEL_*` environment
  variables, and keep it disabled when no OTLP endpoint is set. Never record
  credentials, tokens or personal data in spans.
