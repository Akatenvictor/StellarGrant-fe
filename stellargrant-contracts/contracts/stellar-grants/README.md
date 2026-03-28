# StellarGrant Contract

Soroban smart contract powering the StellarGrant protocol — milestone-based grant escrow on Stellar.

---

## Receipt Events

Two specialised events are emitted to support accounting exports for funders and recipients.

### `PayerReceipt`

Emitted by **`grant_fund`** every time a funder deposits tokens into escrow.

```ts
// Soroban event fields
{
  event_version: u32,
  grant_id:      u64,
  payer:         Address,   // funding entity
  token:         Address,   // token contract address
  amount:        i128,      // deposited amount (base token unit)
  memo:          Option<String>, // optional invoice reference from caller
  timestamp:     u64,       // ledger timestamp
}
```

**Triggering a deposit with a memo:**

```bash
soroban contract invoke \
  --id <CONTRACT_ID> \
  --source <FUNDER_SECRET> \
  -- grant_fund \
    --grant_id 1 \
    --funder <FUNDER_ADDRESS> \
    --amount 1000 \
    --memo '"INV-2024-001"'   # pass null to omit
```

---

### `PayeeReceipt`

Emitted by **`complete_grant`** once per approved milestone when the payout is
transferred to the grant owner.

```ts
// Soroban event fields
{
  event_version:   u32,
  grant_id:        u64,
  recipient:       Address,  // grant owner / payee
  token:           Address,
  amount:          i128,     // per-milestone payout amount
  milestone_index: u32,      // zero-based index of the paid milestone
  timestamp:       u64,
}
```

---

### Querying Receipts

Use the Stellar Horizon API or `soroban events` CLI to subscribe to or replay receipt events.

#### CLI – replay historical receipts for a contract

```bash
# All PayerReceipt events (type 0x0 = contract events)
soroban events \
  --network testnet \
  --contract-id <CONTRACT_ID> \
  --start-ledger <FROM_LEDGER>
```

Filter by event type in the result using the `#[contractevent]`-generated discriminant.
The topic bytes for `PayerReceipt` and `PayeeReceipt` are derived from the struct name
by the Soroban SDK macro.

#### Horizon REST API

```
GET https://horizon-testnet.stellar.org/accounts/<CONTRACT_ID>/effects
```

For on-chain indexed queries, the `grant_id`, `payer`/`recipient`, and `token` fields
are all first-class topics and can be used in filter expressions by any Horizon-compatible
event indexer (e.g. Zephyr, Hubble).

---

## Other Events

| Event | Emitted by | Purpose |
|-------|-----------|---------|
| `GrantCreated` | `create_grant` | New grant opened |
| `GrantFunded` | `grant_fund` | General funding record |
| `PayerReceipt` | `grant_fund` | **Accounting receipt for funder** |
| `MilestonePaid` | `complete_grant` | Milestone payout |
| `PayeeReceipt` | `complete_grant` | **Accounting receipt for payee** |
| `ReviewerDelegated` | `grant_delegate` | Delegation set |
| `DelegationRevoked` | `grant_revoke_delegation` | Delegation removed |
