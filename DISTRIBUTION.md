# KoKo Finance MCP Server - Distribution & Publishing Guide

This document tracks all MCP server distribution channels, submission status, and publishing procedures.

## Overview

KoKo Finance MCP server is distributed through multiple channels:
- **Remote HTTP Server**: `https://kokofinance.net/mcp/` (hosted on Google Cloud Run)
- **Official MCP Registry**: `net.kokofinance/koko-finance`
- **GitHub Repos**: Two repos for different use cases
- **Directory Listings**: Multiple community directories

---

## 1. MCP Server Architecture

### Server Details
- **URL**: `https://kokofinance.net/mcp/`
- **Transport**: Streamable HTTP (FastMCP with Server-Sent Events)
- **Tools**: 14 (search, compare, details, calculate value, optimize portfolio, recommend, renewal check, session, which-card-at-merchant, merchant benefits, card benefits, card terms, card changes, program trends)
- **Prompts**: 5 (portfolio-review, which-card, new-card-finder, renewal-check, card-risk-assessment)
- **Backend**: FastAPI Python, deployed on Google Cloud Run

### Source Code Location
- **Main Project**: `kokos-credit-card-assistant` repo (private)
- **MCP Server Code**: `v3_mvp/backend/mcp_server.py`
- **Mount Point**: Attached to FastAPI app in `v3_mvp/backend/main.py`

---

## 2. GitHub Repositories

### Repo 1: koko-mcp-server (This Repo)
**Purpose**: MCP server connector for any MCP client
**URL**: https://github.com/KokoFinance/koko-mcp-server
**Audience**: Claude Desktop, Cursor, Cline, Windsurf users

**Contents**:
- `.mcp.json` - Auto-connect config
- `README.md` - Tools, prompts, setup instructions
- `server.json` - Registry metadata
- `LICENSE` - MIT

**Use Cases**:
- Submit to MCP directories
- Quick setup for any MCP client
- Official listing page for the server

### Repo 2: koko-credit-card-plugin
**Purpose**: Claude Code plugin with Agent Skill
**URL**: https://github.com/KokoFinance/koko-credit-card-plugin
**Audience**: Claude Code users only

**Contents**:
- `.claude-plugin/plugin.json` - Plugin metadata
- `.mcp.json` - MCP server connection
- `skills/credit-card-advisor/SKILL.md` - Workflow instructions
- `skills/credit-card-advisor/references/tool-guide.md` - Detailed tool docs

**Unique Features**:
- Agent Skill teaches Claude 5 credit card workflows
- Output formatting rules (including neutral value presentation)
- Edge case handling

---

## 3. Official MCP Registry

### Publication Details
- **Registry**: https://registry.modelcontextprotocol.io/
- **Namespace**: `net.kokofinance/koko-finance`
- **Version**: 1.0.0
- **Published**: 2026-02-20 19:41:18 UTC
- **Status**: Active

### DNS Authentication
**Domain**: `kokofinance.net`
**TXT Record** (at root domain):
```
v=MCPv1; k=ed25519; p=LA+yZ26MqSjVoGM7m7huM4O1cJKdKrYtI4/8TJB9XNM=
```

**Private Key**: Stored locally at `v3_mvp/koko-mcp-server/mcp-private-key.pem`
⚠️ **Important**: Keep this private key secure. Required for publishing updates.

### Publishing Process

**Prerequisites**:
- `mcp-publisher` CLI installed (`brew install mcp-publisher`)
- DNS TXT record in place at `kokofinance.net`
- Private key file (`mcp-private-key.pem`)

**Steps**:
```bash
cd v3_mvp/koko-mcp-server

# 1. Update version in server.json (follow semver)
# Edit server.json, increment "version": "1.0.0" → "1.0.1"

# 2. Authenticate with DNS
PRIVATE_KEY=$(openssl pkey -in mcp-private-key.pem -text -noout | grep -A3 "priv:" | tail -n +2 | tr -d ' :\n')
mcp-publisher login dns --domain kokofinance.net --private-key $PRIVATE_KEY

# 3. Publish
mcp-publisher publish

# 4. Verify
curl "https://registry.modelcontextprotocol.io/v0/servers?search=net.kokofinance"
```

### Schema Requirements
- **Current Schema**: `https://static.modelcontextprotocol.io/schemas/2025-12-11/server.schema.json`
- **Description**: Max 100 characters
- **Version**: Semantic versioning (e.g., "1.0.0", "2.1.0-alpha")
- **Repository**: Object with `url` and `source` fields
- **Remotes**: Array with `url` and `type` ("streamable-http")

---

## 4. Anthropic Submissions

### Connectors Directory
**Submitted**: 2026-08-14
**Endpoint**: `https://kokofinance.net/mcp/` (streamable HTTP)
**Form Data**:
- **Resources**: None
- **Prompts**: portfolio-review, which-card, new-card-finder, renewal-check, card-risk-assessment
- **Tools**: 14 (full list submitted)
- **Auth**: OAuth 2.0 + Dynamic Client Registration
- **Categories**: Financial Services, Commerce and Shopping, Data & Analytics
- **Reviewer test account**: `kokofinance.mcpreview@gmail.com`

**Status**: Awaiting Anthropic review (no published turnaround timeline)

### OpenAI MCP Directory
**Submitted**: 2026-08-17
**Auth**: Mixed (OAuth + API key), US only, English only
**Domain verification**: `/.well-known/openai-apps-challenge`

**Status**: Awaiting review

### Claude Code Plugin Directory
**Submitted**: February 2026
**Repo**: https://github.com/KokoFinance/koko-credit-card-plugin
**Form URL**: https://clau.de/plugin-directory-submission
**Skill Name**: `credit-card-advisor`
**Platforms**: Both Claude Code and Claude Work

**Status**: Pending review

### Confirmed working (2026-08-17)
- Claude.ai: Google OAuth flow → connected → tools callable
- Claude Desktop: Google OAuth flow → connected → tools callable
- OpenAI: API key auth (`Bearer koko_...`) → connected → tools callable

---

## 5. Community Directories

### Completed
| Directory | Status | Details |
|-----------|--------|---------|
| Official MCP Registry | ✅ Published | `net.kokofinance/koko-finance` — needs republish, see §3 |

### Pending/Next Steps
| Directory | Submission Method | Repo to Submit |
|-----------|------------------|----------------|
| [Cursor Directory](https://cursor.directory/plugins/new) | Auto-detect from GitHub (sign in required) | koko-mcp-server + koko-credit-card-plugin (separate listings) |
| [mcp.so](https://github.com/chatmcp/mcp-directory/issues/1) | GitHub issue | koko-mcp-server |
| [awesome-mcp-servers (punkpeye)](https://github.com/punkpeye/awesome-mcp-servers) | PR under Finance category | koko-mcp-server |
| [awesome-mcp-servers (wong2)](https://github.com/wong2/awesome-mcp-servers) | PR | koko-mcp-server |
| [mcpservers.org](https://mcpservers.org/) | Submit form on site | koko-mcp-server |
| [Cline MCP Marketplace](https://github.com/cline/mcp-marketplace) | PR | koko-mcp-server |

**Note**: PulseMCP only indexes from Official MCP Registry (already covered).

---

## 6. MCP Server Prompts

Added to `v3_mvp/backend/mcp_server.py` (lines 988-1113):

### Prompt 1: portfolio-review
**Parameters**: `card_names` (required), `monthly_spending` (optional)
**Use Case**: Full portfolio analysis with health score and optimization strategies

### Prompt 2: which-card
**Parameters**: `card_names` (required), `category` (required), `amount` (optional)
**Use Case**: Find best card for specific purchase category

### Prompt 3: new-card-finder
**Parameters**: `spending_focus` (required), `annual_fee_limit` (optional), `credit_score` (optional)
**Use Case**: Search for new card matching criteria

### Prompt 4: renewal-check
**Parameters**: `card_name` (required), `annual_fee` (optional)
**Use Case**: Walk through a card renewal decision step by step

### Prompt 5: card-risk-assessment
**Parameters**: `card_name` (required), `issuer` (optional)
**Use Case**: Analyze a card's risk profile — APR exposure, penalty triggers, fee traps

**Discovery**:
- Available via `/mcp/prompts/list` endpoint
- Show up as slash commands in Claude Code
- Appear in attachment menu in Claude.ai

---

## 7. Important Value Presentation Rules

**Issue**: Net value must not be presented as cash earned or money saved.

**MCP Server Changes** (`mcp_server.py`):
- Updated verdict strings in `calculate_card_value` (lines 530-540)
- Added disclaimer `note` field
- Good: "Estimated rewards and credits exceed the annual fee by ~$125"
- Bad: "This card earns you $125" or "Effective cost: $425"

**Agent Skill Changes** (`koko-credit-card-plugin`):
- Added "Value Presentation (Important)" section in SKILL.md
- Explicit good/bad examples
- Updated tool-guide.md with neutral language

---

## 8. Maintenance & Updates

### When to Republish to Registry

**Update `server.json` version and republish when**:
- MCP server tools are added/removed/changed
- Prompts are added/removed/changed
- Server URL changes
- Description or metadata updates

**No republish needed for**:
- Backend bug fixes (server behavior changes)
- UI/frontend updates
- Database changes

### Version Numbering
Follow semantic versioning:
- **Major (2.0.0)**: Breaking changes (tool signatures change, tools removed)
- **Minor (1.1.0)**: New features (new tools, new prompts)
- **Patch (1.0.1)**: Bug fixes, metadata updates

### Deployment vs. Publishing
- **Deploy to Cloud Run**: Updates the live server at `kokofinance.net/mcp/`
- **Publish to Registry**: Updates the registry metadata (requires version bump)
- These are independent operations

---

## 9. Quick Reference

### Connection URLs
- **Remote Server**: `https://kokofinance.net/mcp/`
- **Developer Docs**: `https://kokofinance.net/developers.html`
- **Homepage**: `https://kokofinance.net`

### Registry Search
```bash
# Search Official MCP Registry
curl "https://registry.modelcontextprotocol.io/v0/servers?search=kokofinance"

# Check DNS TXT record
dig kokofinance.net TXT +short | grep MCP
```

### File Locations
- **MCP Server Code**: `v3_mvp/backend/mcp_server.py`
- **Registry Config**: `v3_mvp/koko-mcp-server/server.json`
- **Private Key**: `v3_mvp/koko-mcp-server/mcp-private-key.pem`
- **Plugin Skill**: `claude-plugin/skills/credit-card-advisor/SKILL.md`

---

## 10. Troubleshooting

### DNS Authentication Fails
**Error**: "no MCP public key found in DNS TXT records"
**Solution**:
1. Verify TXT record at root domain: `dig kokofinance.net TXT`
2. Wait 5-10 minutes for DNS propagation
3. Ensure record value matches exactly (including quotes)

### Schema Validation Errors
**Error**: "deprecated schema detected"
**Solution**: Update `$schema` URL in server.json to latest version

**Error**: "description too long"
**Solution**: Keep description ≤100 characters

### Version Already Exists
**Error**: "version already published"
**Solution**: Increment version in server.json before republishing

---

**Last Updated**: 2026-08-29
**Maintained By**: KoKo Finance Team
