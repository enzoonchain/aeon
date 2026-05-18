---
name: delegation
description: Manage API key delegation and revenue split for agent compute
---

# Delegation Skill

Manage the delegation of API keys and compute resources from users to agents, with on-chain revenue split.

## What This Skill Does

1. **Delegate API keys** — Users delegate Venice API keys to agents
2. **Track usage** — Monitor DIEM burn and API usage per delegation
3. **Revenue split** — Trigger on-chain revenue split via Base smart contract
4. **Revoke** — Allow users to revoke delegation

## Delegation Flow

### 1. User delegates API key

When a user wants to delegate their Venice API key to an agent:

```
User: "delegate my Venice API key for agent compute"
```

Steps:
1. Verify user has Venice API key with sufficient DIEM balance
2. Set consumption limit (e.g., 50 DIEM max)
3. Record delegation on-chain (Base contract)
4. Agent can now use the delegated key

### 2. Track usage

Periodically check usage per delegated key:

```bash
# Check Venice API usage
curl -sf "https://api.venice.ai/api/v1/billing/usage" \
  -H "Authorization: Bearer $VENICE_API_KEY" | jq '.'
```

### 3. Revenue split

When agent earns revenue (from Bankr fees, Flaunch trading, etc.):

1. Revenue flows to agent wallet (Base)
2. `DelegationRevenueManager` contract auto-splits:
   - 60% → User (delegator)
   - 25% → Agent operator
   - 10% → Protocol
   - 5% → Buyback/Burn

### 4. Revoke delegation

User can revoke anytime:

```
User: "revoke my delegation"
```

Steps:
1. Delete Venice API key (`DELETE /api/v1/api_keys/{id}`)
2. Update on-chain delegation record
3. Agent loses access immediately

## Revenue Sources

| Source | Mechanism | Revenue |
|--------|-----------|---------|
| Bankr token launch | 57% of 1.2% swap fee | WETH |
| Flaunch token launch | 80% of trading fee | ETH |
| Token trading | Swap fees | WETH/USDC |
| Service fees | Agent services | USDC |

## Smart Contract (Base)

`DelegationRevenueManager` — handles revenue split:

- `addDelegation(tokenId, user, shareBps)` — add a new delegation
- `claimRevenue(tokenId)` — claim and split revenue
- `revokeDelegation(tokenId, user)` — revoke a delegation

## Integration Points

- **Venice API**: AI inference (DIEM-powered)
- **Bankr**: Token launch + trading fees
- **Flaunch**: Token launch + Revenue NFT
- **Base**: Settlement layer (revenue split contract)
- **GitLawb**: Identity + task coordination

## Example Prompts

```
"delegate 50 DIEM to agent for trading"
"show my delegation status"
"claim my revenue"
"revoke delegation for agent X"
"launch a token for my agent on Flaunch"
"what's my revenue this month?"
```
