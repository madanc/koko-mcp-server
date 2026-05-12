# KoKo Finance MCP Server

A hosted MCP server for AI-powered credit card advice. Search 100+ US credit cards, compare options side by side, analyze portfolio health, and get personalized recommendations — all through the Model Context Protocol.

**Server URL:** `https://kokofinance.net/mcp/`

No API key required. No local installation needed. Just connect your MCP client.

## Quick Setup

### Claude Desktop

Add to `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or `%APPDATA%\Claude\claude_desktop_config.json` (Windows):

```json
{
  "mcpServers": {
    "koko-finance": {
      "type": "http",
      "url": "https://kokofinance.net/mcp/"
    }
  }
}
```

### Claude Code

```bash
claude mcp add koko-finance --transport http https://kokofinance.net/mcp/
```

Or copy the `.mcp.json` from this repo into your project.

### Cursor

Go to **Settings > MCP** and add a new server:
- **Name:** koko-finance
- **Type:** HTTP
- **URL:** `https://kokofinance.net/mcp/`

### Cline / Windsurf

Add to your MCP settings file:

```json
{
  "mcpServers": {
    "koko-finance": {
      "type": "http",
      "url": "https://kokofinance.net/mcp/"
    }
  }
}
```

## Tools (14)

| Tool | Description |
|------|-------------|
| `search_credit_cards` | Natural language search across 100+ US credit cards. Optional filters: `card_type`, `issuer`, `max_annual_fee`, `credit_tier` |
| `compare_cards` | Side-by-side comparison of 2-3 cards with fees, rewards, net value, and break-even. Optional: `credit_tier`, `primary_goal`, `issuer_preferences` (<100ms) |
| `get_card_details` | Full details for a specific card (fees, rewards, benefits) |
| `calculate_card_value` | Annual fee break-even analysis with first-year and ongoing value |
| `optimize_portfolio` | Portfolio health score, per-card KEEP/OPTIMIZE/CANCEL verdicts. Optional: `point_balances`, `benefit_selections`, `credit_tier`, `primary_goal`, `issuer_preferences`, `analysis_type`, `cancel_threshold`, `keep_threshold` (<100ms) |
| `recommend_card_for_category` | Best card to use for a specific spending category, ranked by reward value. Optional: `spending`, `credit_tier`, `primary_goal`, `issuer_preferences` (<100ms) |
| `check_card_renewal` | Should you renew this card? Verdict + downgrade/replacement options. Optional: `benefit_selections`, `acquired_date`, `credit_tier`, `primary_goal`, `renewal_cancel_threshold`, `renewal_keep_threshold` |
| `create_mcp_session` | Session tracking for multi-query conversations |
| `which_card_at_merchant` | Best card from your portfolio at a specific merchant. Auto-detects spending category (e.g. Starbucks → dining) and ranks by reward value |
| `check_merchant_benefits` | Check if any cards have credits at a merchant (e.g. Saks → Amex Platinum $100 credit). Includes earning recommendation |
| `get_card_benefits` | All credits/benefits for a card with value, frequency, schedule, and conditions |
| `get_card_terms` | Schumer Box data — purchase APR, penalty APR, late fees, cash advance fees, promotional APR, grace period |
| `get_card_changes` | Audit log of card data changes (fee updates, benefit changes) with date filtering. Optional: `field` filter |
| `get_program_trends` | Points program valuation history — CPP and transfer partner ratio changes over time |

## Personalization Parameters

Several tools accept optional parameters that improve result relevance. All are optional — when omitted, the server uses sensible defaults (e.g., national average spending, no issuer bias).

### `credit_tier`

Credit score range that affects card eligibility and approval likelihood.

| Value | Meaning |
|-------|---------|
| `"excellent"` | 750+ |
| `"good"` | 700-749 |
| `"fair"` | 650-699 |
| `"poor"` | Below 650 |

**Used by:** `search_credit_cards`, `compare_cards`, `optimize_portfolio`, `recommend_card_for_category`, `check_card_renewal`

### `primary_goal`

The user's main objective for their credit card strategy.

| Value | Meaning |
|-------|---------|
| `"maximize_rewards"` | Get the most points/cashback from everyday spending |
| `"minimize_fees"` | Keep annual fees low while maintaining value |
| `"build_credit"` | Establish or improve credit score |
| `"travel_perks"` | Prioritize lounge access, travel credits, and transfer partners |
| `"simplicity"` | Prefer fewer cards with straightforward reward structures |

**Used by:** `compare_cards`, `optimize_portfolio`, `recommend_card_for_category`, `check_card_renewal`

### `issuer_preferences`

A list of issuer preference objects that express affinity or aversion toward specific card issuers. Each entry has an `issuer` name and a `weight` between -1.0 (strong aversion) and 1.0 (strong preference).

```json
[
  {"issuer": "Chase", "weight": 0.8},
  {"issuer": "American Express", "weight": 0.5},
  {"issuer": "Capital One", "weight": -0.5}
]
```

**Used by:** `compare_cards`, `optimize_portfolio`, `recommend_card_for_category`

### `spending`

A dictionary of monthly spending amounts by category, used to calculate personalized reward values instead of national averages.

```json
{"dining": 500, "groceries": 800, "travel": 300, "gas": 150, "general": 2000}
```

**Used by:** `compare_cards`, `optimize_portfolio`, `recommend_card_for_category`, `check_card_renewal`, `calculate_card_value`

### `cancel_threshold` / `keep_threshold`

Override the default net-value thresholds (in dollars) for KEEP/OPTIMIZE/CANCEL verdicts.

- `cancel_threshold` — Cards below this → CANCEL (default: -50)
- `keep_threshold` — Cards above this → KEEP (default: 50)

**Used by:** `optimize_portfolio`

### `renewal_cancel_threshold` / `renewal_keep_threshold`

Same concept applied to a single card's renewal decision.

**Used by:** `check_card_renewal`

### `acquired_date`

Date the user acquired the card (`YYYY-MM-DD`). Informs tenure-based renewal analysis.

**Used by:** `check_card_renewal`

## Response Metadata

Calculation tools include metadata fields in responses:

- **`spending_source`** — `"user_provided"` or `"national_averages"`. Indicates whether calculations used user-supplied spending or BLS national averages.
- **`data_source`** — Indicates where card data was sourced (e.g., `"koko_db"`).
- **`rewards_by_category`** — Per-category reward breakdown included in compare, portfolio, and value tools.
- **`points_program_key`** / **`portal_cpp`** — Points program identifier and portal cents-per-point valuation, included on all card data.

## Prompts (5)

Pre-built conversation starters that guide the AI through structured workflows:

| Prompt | Description | Parameters |
|--------|-------------|------------|
| `portfolio-review` | Full portfolio analysis with health score and verdicts | `card_names`, `monthly_spending` (optional) |
| `which-card` | Find the best card for a specific purchase | `card_names`, `category`, `amount` (optional) |
| `new-card-finder` | Search for a new card matching your criteria | `spending_focus`, `annual_fee_limit` (optional), `credit_score` (optional) |
| `renewal-check` | Walk through a card renewal decision step by step | `card_name`, `annual_fee` (optional) |
| `card-risk-assessment` | Analyze a card's risk profile — APR exposure, penalty triggers, fee traps | `card_name`, `issuer` (optional) |

## Benefit Selections

The `optimize_portfolio` and `check_card_renewal` tools accept a `benefit_selections` parameter — a list of individual benefit keys the user actually uses. Selected benefits count at 100% utilization; unselected benefits count at 0%.

```
benefit_selections: ["uber", "airline_fee", "dining", "admirals_club"]
```

This gives accurate net-value calculations instead of the default 50% utilization estimate.

**Discover valid keys** by calling the REST endpoint `GET https://kokofinance.net/api/v1/benefit-categories` (no auth required), or see the [developer docs](https://kokofinance.net/developers.html#benefit-categories).

## Example Conversations

**"Review my portfolio"**
> I have a Chase Sapphire Reserve, Amex Gold, and Citi Double Cash. Analyze my portfolio and tell me which cards are worth keeping.

**"Which card should I use?"**
> I'm buying $200 in groceries. I have an Amex Gold, Chase Freedom Unlimited, and Citi Custom Cash. Which card should I use?

**"Find me a new card"**
> I spend a lot on travel and dining. I want a card under $300 annual fee with a good sign-up bonus. What do you recommend?

**"Should I renew my Amex Platinum?"**
> I only use the Uber credit and airline fee credit on my Amex Platinum. Is it still worth paying the $695 annual fee?

**"Which card at Starbucks?"**
> I'm buying coffee at Starbucks. I have a Chase Sapphire Reserve, Amex Gold, and Citi Double Cash. Which card earns the most?

**"Do I have any credits at Saks?"**
> I'm shopping at Saks Fifth Avenue. Do any of my cards have credits there? I have an Amex Platinum and Chase Sapphire Reserve.

**"What benefits does my Amex Platinum have?"**
> Show me all the credits and benefits on my Amex Platinum card — how much is each worth?

## Links

- [KoKo Finance](https://kokofinance.net) — Main site
- [Developer Documentation](https://kokofinance.net/developers.html) — Full API and tool reference
- [MCP Endpoint](https://kokofinance.net/mcp/) — Direct server URL

## Also Available

- **[Claude Code Plugin](https://github.com/KokoFinance/koko-credit-card-plugin)** — Adds the MCP server connection plus an Agent Skill that teaches Claude structured credit card workflows

## License

MIT
