# The Curve Pool That Lies To Quotes

[`PB USDC USDT v4a100`](https://etherscan.io/address/0xe60B5E323D72a914b089f137eC9b3aB91ae24A65) looks like a normal Curve USDC/USDT pool until you inspect the rate oracle.

The trick is simple: the oracle is designed to send off-chain quote simulations into a discounted-rate branch, making USDC look cheaper than it really is. Real transactions use the market-rate branch, so USDT -> USDC quotes promise more USDC than execution supports.

| Metric | Result |
| --- | ---: |
| Gross adverse quote delta | `$225,476` USDC |
| Attacker-wallet net profit | `$34,592.87` |
| Total pool swaps scanned | `124,172` |
| Toxic-direction swaps compared | `68,699` |

The deployer/oracle owner directly profited, and remains the current largest LP.

## The Trick

The trick relies on common `eth_call` defaults: zero gas price and `address(0)` as `tx.origin`. Those calls enter a special oracle branch that applies a `48 bps` discount to the USDC rate:

```solidity
if (tx.origin == address(0) || tx.gasprice == 0) {
    return marketRate * (10_000 - discountBps) / 10_000;
}

return marketRate;
```

That is the quote trap. Off-chain simulations see discounted USDC; real transactions see normal USDC. The pool therefore quotes too much USDC for the toxic USDT -> USDC direction.

The opposite direction, USDC -> USDT, is not the victim-loss direction. There the sign flips: `eth_call` underpromises output, so transaction-context output can look better than the quote.

## Direct Proof

The pool has two rate-oracle adapters:

- USDC oracle: [`0x09e8...FC16`](https://etherscan.io/address/0x09e889BE79da10ABe97116Ef1aFCF5A25509FC16)
- USDT oracle: [`0xA7Bf...Da9`](https://etherscan.io/address/0xA7Bf5E68d96fc20C4FC95420fe20d69bb1301Da9)

USDC oracle behavior:

| Call context | `getRate()` result |
| --- | ---: |
| `from` omitted, `gasPrice = 1` | `0.994871584` |
| nonzero `from`, `gasPrice = 0` | `0.994871584` |
| nonzero `from`, `gasPrice = 1` | `0.999670000` |

Curve pool `stored_rates()`:

| Context | Stored rates |
| --- | --- |
| Via `eth_call` | `[0.994871584, 0.99872873]` |
| Via transaction context | `[0.99967, 0.99872873]` |

Example `get_dy` calls:

| Input | Direction | Via `eth_call` | Via transaction context | Difference |
| --- | --- | ---: | ---: | ---: |
| `1,000` USDT | USDT -> USDC | `1002.988043` USDC | `998.222250` USDC | `-47.52 bps` |
| `1,000` USDC | USDC -> USDT | `995.322982` USDT | `1000.074945` USDT | `+47.74 bps` |

The first row is the toxic path: the quote says the trader should receive more USDC than the transaction-context quote supports.

## Damage Estimate

| Metric | Result |
| --- | ---: |
| Total pool swaps | `124,172` |
| USDT -> USDC swaps compared | `68,699` |
| USDT sold volume | `68.058098M` USDT |
| Actual USDC bought volume | `67.926090M` USDC |
| Sum quoted via `eth_call` | `67.980535M` USDC |
| Sum quoted via transaction context | `67.863652M` USDC |
| Gross adverse quote delta | `225,476.476233` USDC |
| Signed quote-minus-transaction delta | `116,882.591250` USDC |

The `$225,476` number is gross adverse quote delta, not wallet-attributed PnL. It measures how much the `eth_call` quote overpromised in the toxic direction.

In the opposite direction, USDC -> USDT, `eth_call` underpromised output. Transaction-context output exceeded it by `223,957.960803` USDT on a gross basis.

## Attacker Profit And LP Evidence

The deployer/oracle owner is [`0x215c...5fd6`](https://etherscan.io/address/0x215cB1AECdd7D392F793680017960748174b5Fd6). It deployed the pool, owns both oracle adapters, LP'd directly, and later exited its direct LP position.

Direct deployer LP accounting:

| Token | Deposited | Withdrawn | Net |
| --- | ---: | ---: | ---: |
| USDC | `46,801.856700` | `54,801.271263` | `+7,999.414563` |
| USDT | `40,444.440486` | `62,390.861719` | `+21,946.421233` |
| Combined stable units | `87,246.297186` | `117,192.132982` | `+29,945.835796` |

The attacker remains the current largest LP through [`0x159a...d9de`](https://etherscan.io/address/0x159a7a9c58860bce0ea567d7371fb46ef2fdd9de). That address was funded by the owner before entry, shares the same 23-byte delegation code, and later deployed multiple toxic PB USDC/USDT variants.

Included attacker-wallet accounting:

| Bucket | Deposited | Withdrawn / current value | Net |
| --- | ---: | ---: | ---: |
| Deployer/oracle owner, realized | `87,246.297186` | `117,192.132982` | `+29,945.835796` |
| [`0x159a...d9de`](https://etherscan.io/address/0x159a7a9c58860bce0ea567d7371fb46ef2fdd9de) linked LP, marked current | `92,623.188766` | `97,270.220219` | `+4,647.031453` |
| Combined attacker wallets | `179,869.485952` | `214,462.353201` | `+34,592.867249` |

The [`0x159a...d9de`](https://etherscan.io/address/0x159a7a9c58860bce0ea567d7371fb46ef2fdd9de) row is included in the `$34,592.87` headline number. It includes `40,894.236697` already withdrawn plus `56,375.983522` of current balanced-withdraw LP value, so that part is marked current value rather than fully realized profit.

## Why Damage Exceeds Profit

The `$225,476` damage number is quote delta. It is the gross gap between what the toxic quote context promised and what transaction-context pricing supported.

The `$34,592.87` profit number is wallet accounting. It is net of deposits, withdrawals, current LP value, pool inventory, opposite-direction flow, and Curve price dynamics.

Even if the attacker is the only meaningful LP, they do not automatically keep the full quote delta. The quote delta measures deception surface; attacker profit measures end-state wallet value.

## Technical Appendix

### Oracle Reconstruction

This is a human-readable reconstruction of the small oracle adapter runtime shared by the two oracle contracts. Names are reconstructed from selectors and behavior. The USDC and USDT instances differ by immutable `priceFeed` address and slot-0 `discountBps`.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.27;

interface IChainlinkLikeFeed {
    function latestRoundData()
        external
        view
        returns (
            uint80 roundId,
            int256 answer,
            uint256 startedAt,
            uint256 updatedAt,
            uint80 answeredInRound
        );
}

contract ReconstructedCurveRateOracle {
    uint256 public discountBps;
    address public owner;

    address public immutable priceFeed;

    uint256 public constant FEED_DECIMALS = 8;
    uint256 public constant TARGET_DECIMALS = 18;

    constructor(address _priceFeed, address _owner, uint256 _discountBps) {
        priceFeed = _priceFeed;
        owner = _owner;
        discountBps = _discountBps;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "Not owner");
        _;
    }

    function transferOwnership(address newOwner) external onlyOwner {
        owner = newOwner;
    }

    function setDiscountBps(uint256 newDiscountBps) external onlyOwner {
        require(newDiscountBps < 10_000, "Too high");
        discountBps = newDiscountBps;
    }

    function getMarketRate() public view returns (uint256) {
        (, int256 answer,,,) = IChainlinkLikeFeed(priceFeed).latestRoundData();
        require(answer > 0, "Invalid price");
        return uint256(answer) * 10 ** (TARGET_DECIMALS - FEED_DECIMALS);
    }

    function getRate() external view returns (uint256) {
        uint256 marketRate = getMarketRate();

        if (tx.origin == address(0) || tx.gasprice == 0) {
            return marketRate * (10_000 - discountBps) / 10_000;
        }

        return marketRate;
    }
}
```

### Instance Values

USDC oracle [`0x09e8...FC16`](https://etherscan.io/address/0x09e889BE79da10ABe97116Ef1aFCF5A25509FC16):

- `owner`: [`0x215c...5fd6`](https://etherscan.io/address/0x215cB1AECdd7D392F793680017960748174b5Fd6)
- `priceFeed`: [`0x8fFf...18f6`](https://etherscan.io/address/0x8fFfFfd4AfB6115b954Bd326cbe7B4BA576818f6)
- Feed description: `USDC / USD`
- `discountBps`: `48`

USDT oracle [`0xA7Bf...Da9`](https://etherscan.io/address/0xA7Bf5E68d96fc20C4FC95420fe20d69bb1301Da9):

- `owner`: [`0x215c...5fd6`](https://etherscan.io/address/0x215cB1AECdd7D392F793680017960748174b5Fd6)
- `priceFeed`: [`0x3E7d...32D`](https://etherscan.io/address/0x3E7d1eAB13ad0104d2750B8863b489D65364e32D)
- Feed description: `USDT / USD`
- `discountBps`: `0`
