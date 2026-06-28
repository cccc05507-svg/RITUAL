# Ritual Sovereign Loop Agent — doaym

Sovereign AI agent deployed on **Ritual testnet** (Chain ID: 1979) with self-waking loop via Scheduler.

## Deployed Contract

| Field | Value |
|---|---|
| Contract | `SovereignLoopAgent` |
| Address | `0x325E36f91f17bB822fc08d5b5520CF227eD064F3` |
| Network | Ritual Testnet (1979) |
| Deployer | `0x364caB0Bc97F2C3BDDa737ba7960D4266621C0E0` |
| Agent Name | `doaym` |
| Status | 🟢 Running (armed) |
| Wake Delay | 500 blocks (~15 min) |
| RitualWallet | 1.15 RITUAL |

## What It Does

A sovereign agent contract that wakes itself every 500 blocks via the Scheduler, invokes the Sovereign Agent precompile (`0x080C`) to run ZeroClaw CLI inside a TEE enclave, receives results via `onSovereignAgentResult` callback, and schedules the next wakeup.

```
start() → schedule first wakeUp() → Scheduler fires wakeUp() at +500 blocks
  → invoke precompile 0x080C (ZeroClaw in TEE) → Phase 2 callback onSovereignAgentResult()
  → schedule next wakeUp() at +500 blocks → (loop repeats)
```

## Contract Architecture

```solidity
contract SovereignLoopAgent {
    // System contracts
    address constant SOVEREIGN_AGENT = address(0x080C);      // Sovereign Agent precompile
    address constant ASYNC_DELIVERY = 0x5A16...39F6;          // Phase 2 callback delivery
    address constant SCHEDULER = 0x56e7...D58B;               // Scheduler system contract

    // Lifecycle
    function approveScheduler() external;      // 1. Authorize Scheduler callback
    function start(bytes input) external;      // 2. Start the loop
    function wakeUp(uint256 idx) external;     // 3. Scheduler-fired: run agent + reschedule
    function onSovereignAgentResult(...) external; // 4. Receive Phase 2 result
    function stop() external;                  // Stop the loop
    function restart() external;               // Resume the loop
}
```

## System Contracts

| Contract | Address |
|---|---|
| Sovereign Agent Precompile | `0x080C` |
| AsyncDelivery | `0x5A16214fF555848411544b005f7Ac063742f39F6` |
| Scheduler | `0x56e776BAE2DD60664b69Bd5F865F1180ffB7D58B` |
| RitualWallet | `0x532F0dF0896F353d8C3DD8cc134e8129DA2a3948` |
| TEEServiceRegistry | `0x9644e8562cE0Fe12b4deeC4163c064A8862Bf47F` |

## Configuration

- **CLI Type**: 6 (ZeroClaw)
- **Model**: `zai-org/GLM-4.7-FP8` (Ritual LLM gateway, no external API key)
- **Prompt**: `Say hello world`
- **Wake Delay**: 500 blocks (~15 minutes)
- **Gas per wake**: 800,000
- **TTL**: 30 blocks

## Deployment Steps

```bash
# 1. Deploy contract
forge create src/SovereignLoopAgent.sol:SovereignLoopAgent \
  --rpc-url https://rpc.ritualfoundation.org \
  --private-key <KEY> --broadcast

# 2. Approve Scheduler
cast send <CONTRACT> "approveScheduler()" --private-key <KEY> --rpc-url https://rpc.ritualfoundation.org

# 3. Fund RitualWallet
cast send 0x532F0dF0896F353d8C3DD8cc134e8129DA2a3948 \
  "depositFor(address,uint256)" <CONTRACT> 100000000 \
  --value 1150000000000000000 --private-key <KEY> --rpc-url https://rpc.ritualfoundation.org

# 4. Start loop (input = 23-field ABI-encoded sovereign agent request)
cast send <CONTRACT> "start(bytes)" <REQUEST_INPUT> --private-key <KEY> --rpc-url https://rpc.ritualfoundation.org --gas-limit 2000000
```

## License

MIT — Built using official Ritual documentation patterns.

## Links

- [Ritual Network](https://ritual.net)
- [Developer Docs](https://docs.ritualfoundation.org)
- [Block Explorer](https://explorer.ritualfoundation.org)
- [Agent Explorer](https://agents.ritualfoundation.org)

## Disclaimer

Testnet only. Use a burner wallet. No warranty provided.
