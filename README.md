# Finerd for Gemini CLI

Gemini CLI extension that connects Gemini to your [Finerd](https://finerd.ai) personal finance data. Ask questions like "how much did I spend on restaurants last weekend?" and get answers based on your real transactions.

## Install

```bash
gemini extensions install https://github.com/finerd-Inc/mcp
```

On first use, Gemini will open your browser for a one-time OAuth login at `api.finerd.ai`. No tokens to copy, no local server to run.

## Tools

### Browse

| Tool | Description |
|------|-------------|
| `list_spaces` | List your spaces (families/households) |
| `list_accounts` | List financial accounts with balances |
| `list_categories` | List expense and income categories |
| `list_bills` | List recurring bills and subscriptions |
| `get_bill` | Get full bill details |
| `search_transactions` | Search transactions by date, category, account, or text |
| `get_transaction` | Full transaction details with per-account journal entries |
| `search_merchants` | Search merchants by name |
| `search_tags` | Search tags by name |
| `get_account_balances` | Current balances across accounts |

### Reports

| Tool | Description |
|------|-------------|
| `get_expense_report` | Expenses by category over a period |
| `get_income_report` | Income by source over a period |
| `get_cashflow_report` | Net cash flow over a period |
| `get_networth_report` | Net worth trend over time |

### Write

| Tool | Description |
|------|-------------|
| `create_transaction` | Add a new transaction |
| `create_complex_transaction` | Add a multi-leg transaction (splits, multi-account) |
| `create_transfer` | Record a transfer between accounts |
| `update_transaction` | Edit an existing transaction |
| `delete_transaction` | Delete a transaction |
| `mark_transaction_reviewed` | Mark a transaction as reviewed |
| `create_tag` | Create a new tag |
| `update_tag` | Rename or recolor a tag |

`search_transactions` returns a lightweight list. Use `get_transaction` with an ID from the results for full per-account amounts, currencies, categories, and tags.

## Example prompts

- "What did I spend on groceries in April?"
- "Show me my biggest transactions last week"
- "List all my checking accounts and their current balances"
- "How much do I owe on credit cards?"

## How it works

The extension is a thin manifest that points Gemini at the Finerd MCP server hosted at `https://api.finerd.ai/mcp`. Authentication is OAuth 2.0 via dynamic discovery. No code runs locally.

## Requirements

- An active [Finerd](https://finerd.ai) account
- [Gemini CLI](https://geminicli.com) 0.40 or later

## Links

- [Finerd](https://finerd.ai)
- [Gemini CLI](https://geminicli.com)
- [Model Context Protocol](https://modelcontextprotocol.io)

## License

MIT — see [LICENSE](LICENSE).
