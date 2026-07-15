<div align="center">

# Longbow · Contracts

**Solidity core for the Longbow protocol on Robinhood Chain.**

Leveraged longs on `$LONG` with no borrowing, no shorts, and no tokens up front.
Rewards are paid from a finite pre-funded reserve; losses feed the liquidity pool.

[![CI](https://github.com/Longbow-Finance/contracts/actions/workflows/ci.yml/badge.svg)](https://github.com/Longbow-Finance/contracts/actions/workflows/ci.yml)
![Solidity](https://img.shields.io/badge/solidity-0.8.24-363636?logo=solidity)
![Foundry](https://img.shields.io/badge/built%20with-foundry-orange)
![License](https://img.shields.io/badge/license-MIT-c6f24e)

[Website](https://longbowfi.xyz) · [Litepaper](https://longbowfi.xyz/litepaper) · [App & CLI](https://github.com/Longbow-Finance/longbow) · [Explorer](https://robinhoodchain.blockscout.com)

</div>

---

## Overview

A **position** is opened by depositing ETH as collateral and choosing a multiplier. No tokens are
minted to the user at open. As the price of `$LONG` rises, the position accrues reward tokens drawn
from a finite reserve; the maximum reward is **earmarked at open**, so the protocol is always
solvent. On close, the user reclaims their equity in ETH plus any reward tokens. If equity falls to
the maintenance margin, anyone may liquidate the position for a bounty and the remaining collateral
is permanently added to the `$LONG` liquidity pool.

```
maxReward = collateral × multiplier / P0          // earmarked from the reserve at open
reward(P) = maxReward × (P − P0) / P               // 0 at entry, → maxReward as price climbs
equity(P) = collateral × (1 + m × (P − P0) / P0)   // ETH returned on close tracks equity
```

- **No borrowing, no shorts, no interest.** The only direction is long.
- **Solvent by construction.** Rewards can never exceed the earmarked reserve.
- **Losses feed liquidity.** Liquidations and underwater shortfalls are zapped into locked LP.

See the [litepaper](https://longbowfi.xyz/litepaper) for the full mechanism and worked examples.

## Architecture

| Contract | Responsibility |
| --- | --- |
| [`LongToken.sol`](src/LongToken.sol) | Fixed-supply ERC-20 (`ERC20` + `ERC20Permit`). 50% seeds the DEX LP, 50% the reward reserve. |
| [`PositionManager.sol`](src/PositionManager.sol) | Core engine: open/close, collateral custody, reward earmarking, maintenance margin, liquidation. |
| [`oracle/UniswapV2TwapOracle.sol`](src/oracle/UniswapV2TwapOracle.sol) | Manipulation-resistant time-weighted `ETH/LONG` price from the Uniswap V2 pair. |
| [`periphery/UniswapV2LiquiditySink.sol`](src/periphery/UniswapV2LiquiditySink.sol) | Zaps forfeited ETH into the pool and burns the LP tokens — liquidity added permanently. |
| [`interfaces/`](src/interfaces) | `IPriceOracle`, `ILiquiditySink`, and the Uniswap V2 interfaces. |

## Layout

```
src/        core contracts + interfaces
test/       unit, invariant, and fork tests
script/     Deploy.s.sol
lib/        forge-std, openzeppelin-contracts (submodules)
```

## Getting started

Requires [Foundry](https://book.getfoundry.sh/getting-started/installation).

```bash
git clone --recurse-submodules https://github.com/Longbow-Finance/contracts.git
cd contracts
forge build
forge test
```

Already cloned without `--recurse-submodules`? Run `forge install` (or `git submodule update --init --recursive`).

### Test

```bash
forge test -vvv                       # unit + invariant suites
FOUNDRY_PROFILE=intense forge test    # deeper fuzzing / invariants
```

Fork tests are gated on an RPC and skip cleanly when it's unset:

```bash
FORK_RPC_URL=https://rpc.mainnet.chain.robinhood.com forge test --match-path 'test/fork/*'
```

### Deploy

```bash
forge script script/Deploy.s.sol:Deploy \
  --rpc-url https://rpc.mainnet.chain.robinhood.com \
  --private-key <key> --broadcast
```

## Deployments

Robinhood Chain (Arbitrum Orbit L2, chain id `4663`):

| Contract | Address |
| --- | --- |
| `LongToken` | `TBD` |
| `PositionManager` | `TBD` |
| `UniswapV2TwapOracle` | `TBD` |
| `UniswapV2LiquiditySink` | `TBD` |

## Security

These contracts are **unaudited** and provided as-is. The invariant suite checks core solvency
properties (rewards never exceed the reserve; ETH is always backed by open collateral), but that is
not a substitute for an audit. Leverage can result in the total loss of collateral through
liquidation. Do not deposit more than you can afford to lose. Report issues privately before opening
a public issue.

## License

MIT.
