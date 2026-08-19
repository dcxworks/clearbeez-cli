# Clearbeez CLI

Verify email addresses from your terminal, your scripts, and your AI agents — powered by [Clearbeez](https://clearbeez.com).

A single, dependency-free binary that is **both** a command-line tool and an [MCP](https://modelcontextprotocol.io) server for AI assistants (Claude, Cursor, and others).

- ✅ Single & bulk email verification (deliverability, catch-all, disposable, role account, business/consumer, quality score)
- ✅ Reads CSVs, writes results CSVs, streams progress
- ✅ Script-friendly exit codes and `--json` output
- ✅ Built-in MCP server (`clearbeez mcp`) — no Node, no extra install
- ✅ One static binary per OS; no runtime dependencies

---

## Install

### Download a binary (recommended)

Grab the build for your platform from the [latest release](https://github.com/dcxworks/clearbeez-cli/releases/latest):

| Platform | File |
|---|---|
| macOS (Apple Silicon) | `clearbeez-darwin-arm64` |
| macOS (Intel) | `clearbeez-darwin-amd64` |
| Linux (amd64) | `clearbeez-linux-amd64` |
| Linux (arm64) | `clearbeez-linux-arm64` |
| Windows (amd64) | `clearbeez-windows-amd64.exe` |
| Windows (arm64) | `clearbeez-windows-arm64.exe` |

**macOS / Linux:**
```sh
curl -fsSL https://github.com/dcxworks/clearbeez-cli/releases/latest/download/clearbeez-linux-amd64 -o clearbeez
chmod +x clearbeez
sudo mv clearbeez /usr/local/bin/clearbeez
```

**Windows (PowerShell):** download `clearbeez-windows-amd64.exe`, rename it to `clearbeez.exe`, and add its folder to your `PATH`.

SHA256 checksums are published as `checksums.txt` on each release.

### Homebrew / Scoop

_Coming soon:_

```sh
brew install dcxworks/tap/clearbeez          # macOS / Linux
scoop bucket add clearbeez https://github.com/dcxworks/scoop-bucket && scoop install clearbeez  # Windows
```

### Build from source

Requires Go 1.24+.
```sh
git clone https://github.com/dcxworks/clearbeez-cli
cd clearbeez-cli
go build -o clearbeez .
```

---

## Authenticate

Get an API key from the Clearbeez app → **API Access** (looks like `cb_live_sk_...`), then:

```sh
clearbeez auth login          # prompts for your key, validates it, saves it
clearbeez auth status         # shows the masked key, plan, and credits
```

The key is stored in `~/.clearbeez/config.json` (mode `0600`). Precedence for overrides: `--api-key` flag → `CLEARBEEZ_API_KEY` env var → config file. For CI, skip `auth login` and just set the env var:

```sh
export CLEARBEEZ_API_KEY=cb_live_sk_...
clearbeez verify user@example.com
```

Default API endpoint is `https://api.clearbeez.com`; override with `--base-url` or `CLEARBEEZ_BASE_URL`.

---

## Usage

### Verify one address
```sh
clearbeez verify someone@company.com
clearbeez verify someone@company.com --json
```

Exit codes make it scriptable: **`0`** deliverable, **`1`** not deliverable, **`2`** error.
```sh
clearbeez verify "$EMAIL" && send_welcome_mail "$EMAIL"
```

### Verify a list
```sh
clearbeez verify --file leads.csv                       # writes leads-results.csv
clearbeez verify --file leads.csv --output verified.csv
clearbeez verify --file emails.txt                      # one address per line
clearbeez verify --file leads.csv --column "Work Email" # explicit column
```
The email column is auto-detected. Small lists verify instantly; larger lists are queued and the CLI polls to completion, then writes the results CSV.

### Check credits
```sh
clearbeez usage
clearbeez usage --json
```

### Commands
| Command | Description |
|---|---|
| `clearbeez verify <email>` | Verify one address (exit code reflects deliverability) |
| `clearbeez verify --file <path>` | Verify a CSV/TXT list, write a results CSV |
| `clearbeez auth login` | Validate & store your API key |
| `clearbeez auth status` | Show configured key, plan, credits |
| `clearbeez usage` | Credit balance and monthly usage |
| `clearbeez mcp` | Run the MCP server (see below) |
| `clearbeez version` | Print version |

---

## MCP server (AI agents)

The same binary runs an [MCP](https://modelcontextprotocol.io) server over stdio, exposing three tools to AI assistants:

| Tool | Description |
|---|---|
| `verify_email` | Verify a single address |
| `verify_emails` | Verify up to 50 addresses at once |
| `get_credit_balance` | Plan, remaining credits, monthly usage |

Make sure `clearbeez` is on your `PATH` (or use the full path to the binary), then configure your client.

### Claude Code
```sh
claude mcp add clearbeez -e CLEARBEEZ_API_KEY=cb_live_sk_... -- clearbeez mcp
```

### Claude Desktop
Add to `claude_desktop_config.json` (Settings → Developer → Edit Config):
```json
{
  "mcpServers": {
    "clearbeez": {
      "command": "clearbeez",
      "args": ["mcp"],
      "env": { "CLEARBEEZ_API_KEY": "cb_live_sk_..." }
    }
  }
}
```

### Cursor
`.cursor/mcp.json`:
```json
{
  "mcpServers": {
    "clearbeez": {
      "command": "clearbeez",
      "args": ["mcp"],
      "env": { "CLEARBEEZ_API_KEY": "cb_live_sk_..." }
    }
  }
}
```

Then ask your assistant things like *"verify john@acme.com"* or *"how many verification credits do I have left?"*. If `clearbeez auth login` was already run, the server picks up your stored key and the `env` block is optional.

---

## Releasing (maintainers)

```sh
./build-cli.sh 0.2.0
```
Cross-compiles all six platform binaries (versionless names) plus `checksums.txt` into `dist/`. Upload those to a GitHub Release; the versionless names keep the download links stable across versions.

---

## Support

- Docs & API: https://clearbeez.com
- Issues: https://github.com/dcxworks/clearbeez-cli/issues
