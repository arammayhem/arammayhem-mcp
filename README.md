# ARAM Mayhem MCP

Ask your AI assistant about [ARAM Mayhem](https://arammayhem.com/) champion tiers, item builds, augments, curated combos and patch notes. Each answer carries its canonical page URL and data date so you can check and cite it.

**Remote endpoint:** `https://arammayhem.com/mcp`
**Transport:** Streamable HTTP, stateless, read-only, no authentication.
**Setup guide:** https://arammayhem.com/use-with-ai/

Independent fan project, not affiliated with Riot Games.

## Example questions

- What are Jinx's core items in ARAM Mayhem? Cite the page and data date.
- What does Goliath do, and which champions suit it?
- Show the S+ mage champions in the current tier list.
- Find curated Brand combos and explain their augment interaction.
- What changed in patch 26.20?

The MCP reads the same deployed public data as the site. It cannot see accounts, sessions, player profiles, moderation, community submissions or vote identities. A missing build is reported as unavailable. Curated combo tiers are editorial suggestions, not measured win rates. Statistics describe the returned date and may lag a gameplay patch.

## Connect

### Claude

In Claude, use Settings → Connectors → Add custom connector. Enter `https://arammayhem.com/mcp`, name it ARAM Mayhem and enable it in the conversation. Availability depends on your plan.

For a Claude Desktop client that needs a local stdio entry, install Node.js and add this to its MCP configuration. This uses the third-party [mcp-remote bridge](https://github.com/geelen/mcp-remote); it downloads a package and forwards requests to the remote server.

```json
{
  "mcpServers": {
    "arammayhem": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://arammayhem.com/mcp"]
    }
  }
}
```

### Claude Code

```sh
claude mcp add --transport http arammayhem https://arammayhem.com/mcp
```

See the [Claude Code MCP documentation](https://code.claude.com/docs/en/mcp).

### Cursor

Add to `.cursor/mcp.json` or your global MCP configuration:

```json
{
  "mcpServers": {
    "arammayhem": { "url": "https://arammayhem.com/mcp" }
  }
}
```

### ChatGPT custom MCP connections

Where your plan supports it, open Plugins or Apps/Connectors, choose Add custom MCP server, paste this URL, choose no authentication, and enable the connection in a chat. Some accounts need developer mode or workspace admin approval. See [OpenAI's current setup guide](https://developers.openai.com/plugins/quickstart).

```text
URL: https://arammayhem.com/mcp
Authentication: None
```

There is no charge from ARAM Mayhem. Your AI provider's subscription, API prices and limits still apply.

## Optional local stdio bridge

The main awesome-mcp-servers list requires an installable server. For that directory and stdio clients, this repo includes a bridge, with no dataset or tool logic copy.

```sh
git clone https://github.com/arammayhem/arammayhem-mcp.git
cd arammayhem-mcp
npm ci
npm start
```

Or configure a stdio client with `command: "npx"` and `args: ["-y", "github:arammayhem/arammayhem-mcp"]`. It needs Node.js 20 or later and an internet connection. All rate limits still apply. `Dockerfile` exists only for Glama's source-server startup and introspection checks. It does not add another hosted service. Delete `server.js`, `package.json`, `package-lock.json` and `Dockerfile` when stdio support is no longer needed.

## Tools

| Tool | Returns | Inputs |
| --- | --- | --- |
| `get_tier_list` | Champion tiers and public win/pick rates in site rank order | `role`, `tier`, `locale`, `limit`, `offset` |
| `get_champion_build` | Up to three options per item group and measured skill orders | `champion`, `locale` |
| `get_augment` | Description, rarity, availability and published statistics | `augment`, `locale` |
| `search` | Champion and augment matches with page links | `query`, `locale`, `limit` |
| `get_top_combos` | Curated combos by editorial tier | `champion`, `locale`, `limit` |
| `get_patch_notes` | Latest or selected patch changes | `patch`, `locale` |
| `get_site_links` | Canonical public pages and setup guide | `locale` |

`champion`, `augment` and `query` accept ids and names across the site's languages. Champion aliases are supported. Case and punctuation are ignored; ambiguous matches return a clarification error. Locales: `en`, `zh-CN`, `zh-TW`, `ja-JP`, `ko-KR`, `es-MX`, `vi-VN`. Locale tags are case insensitive; English is the default. Role means champion class, not lane: Fighter, Tank, Mage, Assassin, Marksman or Support. Limits default to 10, maximum 30.

## Verify the endpoint

```sh
npx @modelcontextprotocol/inspector --cli https://arammayhem.com/mcp --transport http --method tools/list
npx @modelcontextprotocol/inspector --cli https://arammayhem.com/mcp --transport http --method tools/call --tool-name get_augment --tool-arg augment=Goliath
```

## Implementation and limits

The server is part of the existing site's Cloudflare Worker. It uses the official MCP TypeScript SDK and the site's validated, committed build inputs. This repository holds connection documentation, registry metadata and a small stdio bridge. The bridge forwards the tool list and calls to the remote endpoint using the official MCP SDK. There is no dataset dump, database or separate data pipeline here.

The remote implementation lives in the site's existing Worker. Our code selects compact public fields and supplies citations; the SDK owns MCP wire handling. The site data refresh deploy updates MCP answers too. The public stdio bridge is [server.js](server.js), and it forwards calls without copying data or tool logic.

Cloudflare's native limits allow 60 requests/minute/IP and 600 requests/minute per location, shared across callers. Counters are approximate and local to each Cloudflare location, not a global spending cap. Oversized bodies over 8 KiB are rejected. No D1 or external data request runs per tool call. IPs may be shared by assistants, so retry after 60 seconds on HTTP 429.

To remove it, delete the MCP route and tool module, setup page and footer link; remove the SDK dependency, two rate-limit bindings and this README's endpoint. Existing site data stays in its current release process.

## License

The bridge, documentation and registry metadata in this repository are MIT licensed. Game content belongs to its respective owners. The site's dataset is not included in this license.
