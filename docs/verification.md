# Clearing verification

ACH has completed public and private clearing on NEAR testnet. This page brings the completed runs, contract state and automated checks together so the core financial behavior can be inspected directly.

**Implementation:** [`823a0f0`](https://github.com/NSEVEjk0/ACH/tree/823a0f05a0f443f26076682a9dc54368d4827b07), protocol v4, workflow revision 2. This documentation was compiled on **15 November 2026** from the completed 12/14 November checks and a new read-only chain snapshot. No new financial transactions, AI calls or OutLayer jobs were submitted to assemble it.

## Evidence at a glance

| Evidence | Recorded result | Inspect |
| --- | --- | --- |
| Finalized clearing state, 15 November | **26 settled rounds: 19 public and seven private**; six completed withdrawals | [Round and withdrawal snapshot](evidence/clearing-20261115.json) |
| Token backing at the same finalized block | **572.960 tUSDC** in both actual token custody and ledger balances; zero pending withdrawals | [Backing snapshot and query list](evidence/clearing-20261115.json) |
| Hosted financial journey, 12 November | **52 passing checks** covering real public/private work, settlement, withdrawal/redeposit, repeated work and private-result recovery | [Archived hosted acceptance](https://github.com/NSEVEjk0/ACH/blob/093b5ca4231c3b749dcef1c6fc6f551b64d76811/docs/v4/validation-20261112.md#final-hosted-acceptance) |
| Implementation checks, 14 November | **299 passing automated tests**, 71 local browser checks, 37 hosted-interface checks and 17 live read-only checks | [Dated test summary](evidence/tests-20261114.json) |

These are test executions and accounting records. The six-decimal tUSDC token has no monetary value; account and round counts are not customer-adoption figures.

## Finalized deployment and backing

The snapshot was collected at **2026-11-15 16:43:30 UTC**, finalized block **277830851**, hash `GVLbUzjsEMfoUbDky8CSW76cgGsrt1qpAh5CJ6kkUpBs`. Every query in that snapshot used the same block.

| Field | Value |
| --- | --- |
| Network | NEAR testnet |
| Vault | [`ach-vault-v4r2-e53eefae.testnet`](https://testnet.nearblocks.io/address/ach-vault-v4r2-e53eefae.testnet) |
| Settlement token | `ach-usdc-v4r2-e53eefae.testnet` |
| Vault code hash | `3Mh1JPqCL6kVvT8qZLmU6eexd3CyzMDAqGBatjUdLQCJ` |
| Pinned private-worker SHA-256 | `d88bf904db9899a5a9ea56bea1ba7e417a26058a53fca00210e5f20e1384e26a` |
| Ledger balances | `572960000` base units |
| Pending withdrawals | `0` base units |
| Tracked token custody | `572960000` base units |
| Independently queried token `ft_balance_of(vault)` | `572960000` base units |

The token contract's separate balance query confirms actual custody, rather than relying only on the vault's own backing statement. The equality checked is `ledger balances + pending withdrawals = actual token custody`.

## Completed clearing cases

The `reference` names below identify records in the [snapshot](evidence/clearing-20261115.json). Each record includes its full round ID, commitment hash, public net positions, approval count and settlement time. Call `get_round({"id": "..."})` on the vault to inspect the original record.

| Reference | Agreed group | Gross bills | Net settlement | Final state |
| --- | --- | ---: | ---: | --- |
| `public_service` | Three service agents, three invoices | 0.280 tUSDC | **0.020 tUSDC** | SETTLED |
| `private_service` | Three agents, three encrypted invoice commitments | Private | **0.020 tUSDC** | SETTLED |
| `single_purchase` | Two agents, one invoice | 0.100 tUSDC | 0.100 tUSDC | SETTLED |
| `public_twelve_agents` | Twelve signers, twelve synthetic invoices | 0.980 tUSDC | **0.020 tUSDC** | SETTLED |
| `private_twelve_agents` | Twelve signers, six private pairs | Private | 0.500 tUSDC | SETTLED |
| `maximum_invoice_fixture_1` and `_2` | Twelve signers, 48 public invoices per fixture | Synthetic large-integer inputs | 0 tUSDC | SETTLED |

The three-service workflow demonstrates offsetting payments with Research funding the net debit. The single-purchase case confirms that a one-way payment retains its full amount. The twelve-agent private fixture contains six separate pairs without offsets; it demonstrates private group execution rather than a funding reduction. The maximum-invoice fixtures exercise identifier lengths, integer arithmetic and invoice-count capacity; their artificial gross values are not economic volume.

The [12 November validation](https://github.com/NSEVEjk0/ACH/blob/093b5ca4231c3b749dcef1c6fc6f551b64d76811/docs/v4/validation-20261112.md#live-artifact-and-scale-verification) also records maximum observed operation gas of **119.721 TGas** and **116.268 TGas** for the two 48-invoice fixtures. Current round bounds and the next-version roadmap are documented in [Architecture](architecture.md#v1-release-and-scaling-work).

### Live private execution and withdrawals

The [archived live record](https://github.com/NSEVEjk0/ACH/blob/093b5ca4231c3b749dcef1c6fc6f551b64d76811/docs/v4/live.json) includes real NEAR AI Cloud work, OutLayer invoice verification and the public/private round IDs still present in the finalized snapshot.

- [Private invoice verification 1](https://testnet.nearblocks.io/txns/84xs4VDfqJADdCpbJSUkcKVnwDsCzMsyBDSBEhWh1j4f), [verification 2](https://testnet.nearblocks.io/txns/HhD18aT7zeMzBxcDmsfF53yoCsqZvKX31sTDu99hRDg6) and [verification 3](https://testnet.nearblocks.io/txns/FAdC2jAK21a4wA5n61gxpnwJRVF3Sy1nGGG7ECGzePUC) used the pinned worker.
- [Private netting execution](https://testnet.nearblocks.io/txns/8ZmSWGrewvCVBzXShs6yCBKBPMa2aaFBM6o3PwLMzzrP) returned the committed net positions. Its corresponding `private_service` round is SETTLED.
- [Public-flow owner withdrawal](https://testnet.nearblocks.io/txns/AHG9vvNToS71XNa7iB7jSMMjw6UjKxx9BCuM3AwMw8gP) and [private-flow owner withdrawal](https://testnet.nearblocks.io/txns/9s1oZA8g8qkzxjwSW3u9icGT6AFLdHriQuhnnpuXqPdu) each withdrew 0.020 tUSDC. The snapshot retains their COMPLETED withdrawal records.

The historical hosted acceptance checked decrypted private SVG bytes against the original artifact, reopened private results after workspace switches, and reconciled the original money operations without repeating them. The [release record](https://github.com/NSEVEjk0/ACH/blob/093b5ca4231c3b749dcef1c6fc6f551b64d76811/docs/v4/release.json) binds that run to its source, deployed vault code and private-worker hash.

The four core source files for the vault, funded-purchase logic, protocol library and private worker are byte-identical between that archived release and implementation commit `823a0f0`; their paths and SHA-256 fingerprints are included in the snapshot's `source_comparison`. Later service, SDK and interface changes are covered by the separately dated checks below; the older hosted run is not represented as a fresh run of every later UI change.

## Automated and browser verification

The **14 November** audit used an isolated copy of implementation commit `823a0f0`. All 299 automated cases passed across the recorded runs.

| Suite | Passing cases |
| --- | ---: |
| Rust workspace | 46 |
| Core netting library | 10 |
| Client SDK | 34 |
| Intents SDK | 16 |
| Legacy clearing agent | 14 |
| Coordinator, sponsor and service agents | 83 |
| Security and interface behavior | 34 |
| Actual v4 contract Wasm | 4 |
| Legacy contract security | 31 |
| Legacy clearing contract | 24 |
| Legacy dependency protections | 3 |
| **Total** | **299** |

The production web build, vault/token/worker builds and cross-language private-worker verification passed. Browser and endpoint coverage:

| Check group | Passing checks | Scope |
| --- | ---: | --- |
| Local browser | 71 | Actual vault/token Wasm, six native private-worker executions and deterministic service fixtures; funded public/private groups, wallet operations, repeat work, history, recovery and layouts |
| Hosted interface | 37 | Public pages, deployment binding and workspace interface behavior with API fixtures |
| Live read-only | 17 | Hosted endpoints, backend service health and finalized deployment/code-hash checks |

The combined legacy process reached its outer 900-second runner allowance after the 31 security tests passed. The remaining 24 clearing and three dependency cases passed in a separate 175.95-second run. This records all cases passing, without claiming that the complete root `npm test` command finished in one uninterrupted run.

The 14 November audit submitted no new public-network financial transactions and made no live AI/OutLayer calls or mainnet Intents transfers. Its local and interface checks complement the separately recorded live financial journeys. Exact commands, durations, scope and original report digests are in [the machine-readable test summary](evidence/tests-20261114.json). Build and test instructions remain in [Development](development.md#tests).

## Recheck without a wallet

This Node command reads current finalized state, then pins the backing, token and representative-round queries to that same block. It needs no credentials or funded workspace and sends no transactions. Compare its printed round IDs and hashes with the snapshot. For an exact historical comparison, the snapshot's block hash can be used as `block_id` on an archival testnet RPC.

```bash
node --input-type=module <<'JS'
const rpc = "https://test.rpc.fastnear.com";
const vault = "ach-vault-v4r2-e53eefae.testnet";
const token = "ach-usdc-v4r2-e53eefae.testnet";
async function view(account, method, args, block) {
  const params = {
    request_type: "call_function", account_id: account, method_name: method,
    args_base64: Buffer.from(JSON.stringify(args)).toString("base64"),
    ...(block ? { block_id: block } : { finality: "final" })
  };
  const response = await fetch(rpc, {
    method: "POST", headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ jsonrpc: "2.0", id: "ach-verify", method: "query", params }),
    signal: AbortSignal.timeout(30000)
  });
  if (!response.ok) throw new Error("RPC HTTP " + response.status);
  const reply = await response.json();
  if (reply.error) throw new Error(JSON.stringify(reply.error));
  if (reply.result.error) throw new Error(reply.result.error);
  return { block: reply.result.block_hash,
    value: JSON.parse(Buffer.from(reply.result.result).toString()) };
}
const config = await view(vault, "get_config", {});
const publicId = "pg-be5a3222915fa7067635386a77783765-4b373d6939de8b9a99fd526ed607bee6";
const privateId = "pg-f49b51a6fa44e671d55924a58b05a3cc-5d2a638ec6d7fc02e14d761af67fa2d9-net-0";
const [backing, custody, publicRound, privateRound] = await Promise.all([
  view(vault, "get_backing_status", {}, config.block),
  view(token, "ft_balance_of", { account_id: vault }, config.block),
  view(vault, "get_round", { id: publicId }, config.block),
  view(vault, "get_round", { id: privateId }, config.block)
]);
for (const result of [backing, custody, publicRound, privateRound])
  if (result.block !== config.block) throw new Error("Block mismatch");
const claims = BigInt(backing.value.total_balances) + BigInt(backing.value.pending_withdrawals);
if (claims !== BigInt(custody.value) ||
    BigInt(backing.value.tracked_token_custody) !== BigInt(custody.value))
  throw new Error("Backing mismatch");
for (const { value: round } of [publicRound, privateRound])
  if (round.status !== "SETTLED" || round.net_settlement_volume !== "20000")
    throw new Error("Unexpected representative round");
console.log(JSON.stringify({
  block: config.block, protocol: config.value.protocol_version,
  settled_rounds: config.value.settled_rounds, actual_token_base_units: custody.value,
  backing: backing.value,
  rounds: [publicRound.value, privateRound.value].map(r => ({
    id: r.input.id, hash: r.hash, status: r.status,
    private: r.input.private, net_base_units: r.net_settlement_volume
  }))
}, null, 2));
JS
```

The core evidence here concerns NEAR vault clearing and private invoice execution. The optional browser Intents feature has its own [dry-quote verification report](v4/intents-route-verification-20261113.md); those previews are not counted as executed cross-chain payments.
