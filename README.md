# Compfeed for Claude Code

The official Claude Code plugin marketplace for [Compfeed](https://compfeed.com).

Installing the `compfeed` plugin connects Claude Code to your Compfeed
organization through Compfeed's remote MCP server, so Claude can work with your
Compfeed data directly.

## Install

In Claude Code:

```
/plugin marketplace add Compfeed/claude-code-plugin
/plugin install compfeed@compfeed
```

### 1. Choose your Compfeed organization

The plugin asks for **Compfeed organization** when it is enabled. Enter the slug
from your Compfeed organization URL — the short name that identifies your
organization, for example `paksa`.

Your slug determines the MCP endpoint Claude Code connects to:

| Organization slug | MCP endpoint                                 |
| ----------------- | -------------------------------------------- |
| `paksa`           | `https://api.compfeed.com/mcp/o/paksa`       |
| `acme`            | `https://api.compfeed.com/mcp/o/acme`        |
| `my-company`      | `https://api.compfeed.com/mcp/o/my-company`  |

Nothing is hardcoded — each user configures their own organization. To change it
later, run `/plugin`, select **compfeed**, and choose **Configure**.

### 2. Authenticate with Compfeed

If Claude Code prompts you to sign in, approve the OAuth request in your
browser. Claude Code handles the OAuth flow natively and stores the
credentials for you.

To re-authenticate later, run `/mcp`, select **compfeed**, and authenticate.

### 3. Verify the connection

```
/mcp
```

The `compfeed` server should be listed as **connected**, pointing at
`https://api.compfeed.com/mcp/o/<your-slug>`. Expand it to see the tools
Compfeed exposes.

If it shows *needs authentication*, complete step 2. If it shows a connection
error, confirm your organization slug is spelled exactly as it appears in your
Compfeed URL.

## How organization selection works

`plugins/compfeed/.claude-plugin/plugin.json` declares one user-configurable
option, `organization_slug`, as a required string. `plugins/compfeed/.mcp.json`
references it in the server URL:

```json
{
  "mcpServers": {
    "compfeed": {
      "type": "http",
      "url": "https://api.compfeed.com/mcp/o/${user_config.organization_slug}"
    }
  }
}
```

Claude Code prompts for the value at enable time, stores it per user, and
substitutes it into the URL before connecting. Because the option is required
and has no default, there is no fallback organization: an unconfigured plugin
reports that the option needs to be set rather than connecting somewhere
unintended.

## Development

Test the marketplace from a local checkout before publishing:

```bash
git clone https://github.com/Compfeed/claude-code-plugin
cd claude-code-plugin

# Validate both manifests (warnings are errors)
claude plugin validate . --strict
claude plugin validate plugins/compfeed --strict
```

Then install from the working tree instead of GitHub. `--scope local` keeps the
test confined to this directory, and `--config` skips the interactive prompt:

```bash
claude plugin marketplace add ./ --scope local
claude plugin install compfeed@compfeed --scope local --config organization_slug=YOUR_SLUG

# Confirm the URL was built from your slug
claude mcp list | grep compfeed
claude plugin details compfeed@compfeed
```

Tear the test down again:

```bash
claude plugin uninstall compfeed@compfeed --scope local
claude plugin marketplace remove compfeed
```

After editing a manifest, re-run `claude plugin marketplace update compfeed` so
Claude Code picks up the change.

## Repository layout

```
.claude-plugin/
  marketplace.json              # marketplace: name, owner, plugin list
plugins/
  compfeed/
    .claude-plugin/
      plugin.json               # plugin manifest + organization_slug option
    .mcp.json                   # compfeed MCP server definition
```

## Requirements

- Claude Code 2.1 or newer (native remote HTTP MCP support)
- A Compfeed organization
