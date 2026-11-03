# MCP-workspace

One place to develop the gateway and the MCP servers together. Each
application lives in its own repository; this repository holds the tooling
around them.

| Repository | What it is |
|---|---|
| [mcp-gateway](https://github.com/Olsenius/mcp-gateway) | OAuth login and admin page in front of every server |
| [bibliofil-mcp](https://github.com/Olsenius/bibliofil-mcp) | Library loans, reservations and national search |
| [vigilo-connector](https://github.com/Olsenius/vigilo-connector) | Vigilo for parents: messages, timetable, absences |
| [fribo-mcp](https://github.com/Olsenius/fribo-mcp) | Cabin road plowing: register stays |
| [tibi-mcp](https://github.com/Olsenius/tibi-mcp) | Tibi e-books: search, borrow, Kindle, EPUB |
| [spond-no-match-mcp](https://github.com/Olsenius/spond-no-match-mcp) | Spond: the kids' activities, responses and participants |

See [Architecture](architecture.md) for how they fit together.

## Get started

Install Git, Docker and [mise](https://mise.jdx.dev), then:

```bash
mise trust && mise install
task clone      # clone the repositories
task setup      # install Go and Python dependencies
task check      # validate the workspace
```

## Everyday commands

```bash
task status     # branches and local changes in every repository
task sync       # clone missing repos, fast-forward pull all
task test       # all test suites (or test:gateway, test:tibi, ...)
task env:init   # create .env with random local credentials
task dev:up     # start the local stack at http://localhost:18080/admin
task dev:down   # stop it, keep the data
task docs:serve # preview these docs at http://localhost:8000
```

`task --list` shows everything.

## Good to know

- Work directly on `main` in every repository.
- The local stack uses its own volumes and synthetic data, never production.
- Vigilo allows only one active holder of an account's refresh token.
  Do not log in locally with an account in use elsewhere.
- AI agents: read [Guide for agents](agents.md) and `AGENTS.md`.
