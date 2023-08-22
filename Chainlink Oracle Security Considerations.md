# Chainlink Oracle Security Considerations

[Chainlink](https://chain.link/) allows smart contract developers to receive a wide variety of off-chain data, with the most commonly used features being receiving off-chain randomness and off-chain pricing data. Integrating your smart contracts with Chainlink provides a unique set of potential security vulnerabilities that attackers can exploit; here are the common vulnerabilities that smart contract developers & auditors need to look out for.

# Not Checking For Stale Prices

Many smart contracts use Chainlink to request off-chain pricing data, but a common error occurs when the smart contract doesn’t check whether that data is stale. Consider this [stale pricing data finding](https://github.com/sherlock-audit/2023-05-USSD-judging/issues/31) from Sherlock’s USSD audit:

```
// @audit no check for stale price data
(, int256 price, , , ) = priceFeedDAIETH.latestRoundData();

return
    (wethPriceUSD * 1e18) /
    ((DAIWethPrice + uint256(price) * 1e10) / 2);
```

Oracle data feeds can return stale pricing data for a variety of [reasons](https://ethereum.stackexchange.com/questions/133242/how-future-resilient-is-a-chainlink-price-feed/133843#133843). If the returned pricing data is stale, this code will execute with prices that don’t reflect the current pricing resulting in a potential loss of funds for the user and/or the protocol. Smart contracts should always check the updatedAt parameter returned from latestRoundData() and compare it to a staleness threshold:

```
// @audit fixed to check for stale price data
(, int256 price, , uint256 updatedAt, ) = priceFeedDAIETH.latestRoundData();

if (updatedAt < block.timestamp - 60 * 60 /* 1 hour */) {
   revert("stale price feed");
}

return
    (wethPriceUSD * 1e18) /
    ((DAIWethPrice + uint256(price) * 1e10) / 2);
```

The staleness threshold should correspond to the heartbeat of the oracle’s price feed. This can be found on [Chainlink’s list of Ethereum mainnet price feeds](https://docs.chain.link/data-feeds/price-feeds/addresses/?network=ethereum) by checking the “Show More Details” box, which will show the “Heartbeat” column for each feed. For networks other than Ethereum mainnet, make sure to select your desired L1/L2 on that page before reading the data columns.
More examples: [[1](https://code4rena.com/reports/2022-07-juicebox#h-01-oracle-data-feed-can-be-outdated-yet-used-anyways-which-will-impact-payment-logic), [2](https://code4rena.com/reports/2021-10-mochi#m-05-chainlinks-latestrounddata-might-return-stale-or-incorrect-results), [3](https://code4rena.com/reports/2021-11-vader#m-10-should-check-return-data-from-chainlink-aggregators), [4](https://code4rena.com/reports/2021-12-perennial#m-03-chainlinks-latestrounddata-might-return-stale-or-incorrect-results), [5](https://code4rena.com/reports/2021-12-yetifinance#m-02-should-check-return-data-from-chainlink-aggregators), [6](https://code4rena.com/reports/2022-04-phuture#m-02-chainlinks-latestrounddata-might-return-stale-or-incorrect-results), [7](https://code4rena.com/reports/2022-04-backd#m-05-chainlinks-latestrounddata-might-return-stale-or-incorrect-results), [8](https://code4rena.com/reports/2022-08-olympus#m-24-naz-m1-chainlinks-latestrounddata-might-return-stale-results), [9](https://github.com/sherlock-audit/2022-09-knox-judging/issues/137), [10](https://code4rena.com/reports/2021-05-fairside#m-09-should-check-return-data-from-chainlink-aggregators), [11](https://code4rena.com/reports/2022-12-tigris#m-24-chainlink-price-feed-is-not-sufficiently-validated-and-can-return-stale-price), [12](https://github.com/sherlock-audit/2023-02-blueberry-judging/issues/94), [13](https://solodit.xyz/issues/missing-checks-for-chainlink-oracle-spearbit-connext-pdf), [14](https://solodit.xyz/issues/chainlink-oracle-return-values-are-not-handled-property-halborn-savvy-defi-pdf), [15](https://solodit.xyz/issues/m-02-check-for-stale-data-before-trusting-chainlinks-response-zachobront-none-dyad-markdown), [16](https://code4rena.com/reports/2022-01-yield#m-01-oracle-data-feed-is-insufficiently-validated), [17](https://code4rena.com/reports/2022-08-olympus#m-18-inconsistency-in-staleness-checks-between-ohm-and-reserve-token-oracles), [18](https://github.com/sherlock-audit/2023-03-Y2K-judging/issues/70), [19](https://github.com/sherlock-audit/2023-02-gmx-judging/issues/174)]

# Not Checking For Down L2 Sequencer

When using Chainlink with L2 chains like Arbitrum, smart contracts must [check whether the L2 Sequencer is down](https://github.com/sherlock-audit/2023-01-sentiment-judging/issues/16) to avoid stale pricing data that appears fresh - Chainlink’s official documentation provides an [example](https://docs.chain.link/data-feeds/l2-sequencer-feeds#example-code) implementation. Smart contract auditors should look out for missing L2 sequencer activity checks when they see price code callinglatestRoundData() in projects that are to be deployed on L2s.
More examples: [[1](https://github.com/sherlock-audit/2023-02-bond-judging/issues/1), [2](https://github.com/sherlock-audit/2022-11-sentiment-judging/issues/3), [3](https://github.com/sherlock-audit/2023-04-blueberry-judging/issues/142), [4](https://github.com/sherlock-audit/2023-02-gmx-judging/issues/151), [5](https://github.com/sherlock-audit/2023-05-perennial-judging/issues/37)]

# Same Heartbeat Used For Multiple Price Feeds

Smart contracts often use multiple oracle price feeds to track prices for multiple assets. It is an error to assume that the same time interval heartbeat can be used as a staleness check for every feed, as [different feeds can have different heartbeats](https://github.com/sherlock-audit/2023-04-jojo-judging/issues/449). Consider this [code](https://github.com/sherlock-audit/2023-04-jojo/blob/main/smart-contract-EVM/contracts/adaptor/chainlinkAdaptor.sol#L43-L55) from [JOJO’s Sherlock audit](https://audits.sherlock.xyz/contests/70):

```
function getMarkPrice() external view returns (uint256 price) {
  int256 rawPrice;
  uint256 updatedAt;
  // @audit first feed
  (, rawPrice, , updatedAt, ) = IChainlink(chainlink).latestRoundData();

  // @audit second feed
  (, int256 USDCPrice,, uint256 USDCUpdatedAt,) = IChainlink(USDCSource).latestRoundData();

  require( // @audit feed #1 stale check using same heartbeatInterval
  block.timestamp - updatedAt <= heartbeatInterval,
  "ORACLE_HEARTBEAT_FAILED"
  );
           // @audit feed #2 stale check using same heartbeatInterval
  require(block.timestamp - USDCUpdatedAt <= heartbeatInterval, "USDC_ORACLE_HEARTBEAT_FAILED");
  uint256 tokenPrice = (SafeCast.toUint256(rawPrice) * 1e8) / SafeCast.toUint256(USDCPrice);
  return tokenPrice * 1e18 / decimalsCorrection;
}
```

In this example, the first price feed has a heartbeat of 1 hour while the second has a heartbeat of 24 hours, so they require different heartbeats to be used in their staleness checks. The appropriate heartbeats can be found on [Chainlink’s list of Ethereum mainnet price feeds](https://docs.chain.link/data-feeds/price-feeds/addresses/?network=ethereum) by checking the “Show More Details” box, which will show the “Heartbeat” column for each feed. For networks other than Ethereum mainnet, make sure to select your desired L1/L2 on that page before reading the data columns.

# Oracle Price Feeds Not Updated Frequently

Care must be taken when selecting which price oracle(s) to use; [using an oracle price feed that isn’t updated frequently](https://github.com/sherlock-audit/2022-09-notional-judging/issues/67) will result in calculations being performed with inaccurate prices that don’t reflect the true value of the asset.
Chainlink Oracles are currently the safest choice, but even then, care must be taken regarding which price feed to choose; [similar price feeds can have different heartbeat & deviation thresholds](https://github.com/sherlock-audit/2023-03-olympus-judging/issues/2); the longer the heartbeat & higher the deviation threshold, the more the oracle price can differ from the true, current price.
Smart contracts developers should use & auditors should check that price feeds with the lowest heartbeat & deviation thresholds are being used to ensure the oracle’s reported price is as close as possible to the true, current price.
More examples: [[1](https://github.com/sherlock-audit/2022-11-isomorph-judging/issues/256), [2](https://github.com/sherlock-audit/2023-02-olympus-judging/issues/210), [3](https://solodit.xyz/issues/strategyfloorfromchainlink-will-often-revert-due-to-stale-prices-spearbit-looksrare-pdf), [4](https://code4rena.com/reports/2022-05-aura#m-02-crvdepositorwrappersol-relies-on-oracle-that-isnt-frequently-updated)]

# Request Confirmation < Depth of Chain Re-Orgs

When requesting randomness, the [REQUEST_CONFIRMATION parameter must be greater than the depth of common chain re-organizations](https://github.com/pashov/audits/blob/master/solo/NFTLoots-security-review.md#c-01-polygon-chain-reorgs-will-often-change-game-results) on the target chain(s) the contract is to be deployed on, as chain re-organizations re-order blocks & transactions, which can affect the returned randomness.
This could result in a winner becoming a loser or vice-versa due to the chain re-organization re-ordering the randomness request, resulting in a different randomness result. This parameter is found in the contract that inherits from VRFConsumerBaseV2:

```
contract VRFv2Consumer is VRFConsumerBaseV2 {
    // @audit REQUEST_CONFIRMATIONS = how many blocks confirmed
    // before receiving randomness. Must be greater than depth
    // of common chain reorganisations that occur on target chain.
    //
    // eg polygon has 5+ block re-orgs per day with depth > 3 blocks
    // and frequently has re-orgs with depth < 30 blocks
    //
    // when your transaction for requesting randomness from VRF is moved
    // to a different block then the returned randonmness can change
    // meaning the winner as determined by the returned randonmness
    // can also change!
    uint16 internal constant REQUEST_CONFIRMATIONS = 3;
```

This parameter will often have the value of 3 because this is the default value in the official Chainlink [tutorial](https://docs.chain.link/getting-started/intermediates-tutorial), so is simply copied without much thought by developers. Smart contract developers & auditors should confirm whether the value of REQUEST_CONFIRMATIONS is suitable for the targeted chain(s) the smart contract will be deployed on. If the smart contract is to be deployed upon multiple chains, a different value for REQUEST_CONFIRMATIONS may be required for each deployment.
More examples: [[1](https://docs.chain.link/vrf/v2/security#choose-a-safe-block-confirmation-time-which-will-vary-between-blockchains)]

# Assuming Oracle Price Precision

When working with Oracle price feeds, developers must account for [different price feeds having different decimal precision](https://solodit.xyz/issues/h-01-incorrect-handling-of-pricefeeddecimals-code4rena-y2k-finance-y2k-finance-contest-git); it is an error to assume that every price feed will report prices using the same precision. Generally, [non-ETH pairs report using 8 decimals, while ETH pairs report using 18 decimals](https://ethereum.stackexchange.com/questions/92508/do-all-chainlink-feeds-return-prices-with-8-decimals-of-precision).
If precision is assumed, there is plenty of room for developer mistakes to be made since, for example, [ETH/USD reports using 8 decimals](https://etherscan.io/address/0x5f4eC3Df9cbd43714FE2740f5E3616155c5b8419#readContract#F3), as it is considered a non-ETH pair since the price of ETH is being reported in USD. There are also price feeds such as [AMPL/USD](https://etherscan.io/address/0xe20CA8D7546932360e37E9D72c1a47334af57706#readContract#F3) that report using 18 decimals which breaks the general rule that USD price feeds report in 8 decimals.
Smart contracts can call [AggregatorV3Interface.decimals()](https://docs.chain.link/data-feeds/api-reference#decimals) to get the exact number of decimals for the price feed being called.
More examples: [[1](https://solodit.xyz/issues/h01-uniswap-chainlink-rate-difference-always-reverts-for-certain-currencies-openzeppelin-notional-audit-markdown), [2](https://code4rena.com/reports/2021-06-tracer#h-06-wrong-price-scale-for-gasoracle), [3](https://code4rena.com/reports/2021-12-vader#h-05-oracle-returns-an-improperly-scaled-usdvvader-price), [4](https://github.com/sherlock-audit/2022-08-sentiment-judging/blob/main/019-H/019-h.md), [5](https://code4rena.com/reports/2022-09-y2k-finance#h-01-incorrect-handling-of-pricefeeddecimals), [6](https://solodit.xyz/issues/chainlink-oracle-can-crash-with-decimals-longer-than-18-halborn-savvy-defi-pdf), [7](https://code4rena.com/reports/2022-06-connext#m-11-tokens-with-decimals-larger-than-18-are-not-supported), [8](https://code4rena.com/reports/2022-10-inverse#m-15-oracle-assumes-token-and-feed-decimals-will-be-limited-to-18-decimals), [9](https://solodit.xyz/issues/m03-undocumented-decimal-assumptions-openzeppelin-1inch-limit-order-protocol-audit-markdown), [10](https://github.com/sherlock-audit/2023-05-USSD-judging/issues/236)]

# Incorrect Oracle Price Feed Address

Some projects will hard-code oracle price feed addresses. Others will have addresses in deploy scripts to be set during contract deployment. Wherever the addresses are located, auditors should check that they point to the correct oracle price feed. Examine this [code](https://github.com/sherlock-audit/2023-05-USSD/blob/main/ussd-contracts/contracts/oracles/StableOracleWBTC.sol#L10-L19) from Sherlock’s USSD contest:

```
// @audit correct address here, but wrong address in constructor
// chainlink btc/usd priceFeed 0xf4030086522a5beea4988f8ca5b36dbc97bee88c;
contract StableOracleWBTC is IStableOracle {
    AggregatorV3Interface priceFeed;

    constructor() {
        priceFeed = AggregatorV3Interface(
            // @audit wrong address; this is ETH/USD not BTC/USD !
            0x5f4eC3Df9cbd43714FE2740f5E3616155c5b8419
        );
    }
```

Here the correct address for the BTC/USD price feed appears in the comment, but in the constructor, the address of the ETH/USD price feed incorrectly appears. Auditors should verify that price feed addresses are correct by referring to [Chainlink’s list of Ethereum mainnet price feeds](https://docs.chain.link/data-feeds/price-feeds/addresses/?network=ethereum). For projects being deployed on L2s or alternate L1s, auditors should verify that correct price feed addresses are being used for those networks.
More examples: [[1](https://github.com/sherlock-audit/2023-02-blueberry-judging/issues/152)]

# Oracle Price Updates Can Be Front-Run

Some stablecoin protocols that allow users to deposit collateral and mint/burn stablecoin tokens based upon prices of collateral assets can be subject to having value extracted from the protocol by having their [oracle updates sandwich attacked](https://github.com/sherlock-audit/2023-03-olympus-judging/issues/1).
Oracle price updates may be too slow behind real-world prices due to only updating after a set deviation % of price change, plus attackers may see oracle updates in the mempool & front-run them. This is a complicated problem to solve; potential solutions involve:

- adding a small fee to mint/burn operations to make the frequent minting/burning which occurs in sandwich attacks less profitable,
- [when a user makes a deposit, enforce a delay that prevents them from withdrawing in a short amount of time to prevent](https://github.com/0xLienid/sherlock-olympus/pull/3/files) sandwich attacks,

For more information, check out [Angle’s Research Series](https://blog.angle.money/angle-research-series-part-1-oracles-and-front-running-d75184abc67) & [Synthetix History with Frontrunning](https://blog.synthetix.io/frontrunning-synthetix-a-history/).
More examples: [[1](https://solodit.xyz/issues/oracle-front-running-could-deplete-reserves-over-time-addressed-consensys-bancor-v2-amm-security-audit-markdown), [2](https://github.com/sherlock-audit/2023-05-USSD-judging/issues/836)]

# Unhandled Oracle Revert Denial Of Service

Calls to [Oracles could potentially revert](https://code4rena.com/reports/2022-07-juicebox#m-09-unhandled-chainlink-revert-would-lock-all-price-oracle-access), which may result in a complete Denial-of-Service to smart contracts which depend upon them. Chainlink multisigs can immediately block access to price feeds at will, so just because a price feed is working today does not mean it will continue to do so indefinitely. Smart contracts should handle this by:

- wrapping calls to Oracles in try/catch blocks and dealing appropriately with any errors,
- providing functionality to replace or update oracle feeds after they are configured.

If a configured Oracle feed has malfunctioned or ceased operating, but the smart contract does not have any alternative data source, nor does the contract allow updates to data sources, that contract will be permanently bricked.
This would be especially bad for stablecoin protocols and lending/borrowing platforms where large amounts of user value are stored in the form of collateral that would no longer be able to be withdrawn due to calls to the price oracles reverting.
More examples: [[1](https://code4rena.com/reports/2022-10-inverse#m-18-protocols-usability-becomes-very-limited-when-access-to-chainlink-oracle-data-feed-is-blocked), [2](https://github.com/sherlock-audit/2023-02-blueberry-judging/issues/161)]

# Unhandled Depeg Of Bridged Assets

Consider a Lending & Borrowing protocol where:

- users can deposit a wrapped asset such as WBTC (wrapped BTC) and borrow against it,
- the protocol uses Chainlink’s BTC/USD feed to price WBTC,

If the WBTC bridge is compromised and WBTC depegs from BTC, the protocol will continue to price WBTC using the BTC/USD price, even though WBTC will instantly become worth far less than native BTC due to the bridge compromise.
Users could then buy WBTC for a far lower value than native BTC, deposit it into the protocol, and borrow against it using the value of native BTC. This would allow attackers to drain the protocol in the event of a bridge compromise leading to a depeg event.
To help address this issue, the protocol could use [Chainlink’s WBTC/BTC price feed](https://etherscan.io/address/0xfdFD9C85aD200c506Cf9e21F1FD8dd01932FBB23) to monitor for a depeg event.
More examples: [[1](https://github.com/sherlock-audit/2023-05-USSD-judging/issues/310)]

# Oracle Returns Incorrect Price During Flash Crashes

Chainlink price feeds have in-built minimum & maximum prices they will return; if during a flash crash, bridge compromise, or depegging event, an asset’s value falls below the price feed’s minimum price, [the oracle price feed will continue to report the (now incorrect) minimum price](https://github.com/sherlock-audit/2023-02-blueberry-judging/issues/18).
An attacker could:

- buy that asset using a decentralized exchange at the very low price,
- deposit the asset into a Lending / Borrowing platform using Chainlink’s price feeds,
- borrow against that asset at the minimum price Chainlink’s price feed returns, even though the actual price is far lower.

This attack would let the [attacker drain value from Lending / Borrowing platforms](https://rekt.news/venus-blizz-rekt/). To help mitigate such an attack on-chain, smart contracts could check that minAnswer < receivedAnswer < maxAnswer.
This attack could also potentially be mitigated off-chain via off-chain monitoring, which compares Chainlink’s latest reported price to other off-chain sources such as centralized exchanges and/or liquid indexes which aggregate multiple off-chain price sources to produce one index price; if external sources are reporting prices lower than Chainlink’s minAnswer, off-chain monitoring could disable the smart contract’s price feed for that asset, forcing any transactions to revert.
Developers & Auditors can find Chainlink’s oracle feed [minAnswer, maxAnswer] values by:

- looking up the price feed address on [Chainlink’s list of Ethereum mainnet price feeds](https://docs.chain.link/data-feeds/price-feeds/addresses/?network=ethereum) (or select other L1/L2 for price feeds on other networks),
- reading the “aggregator” value, e.g., for [AAVE / USD price feed](https://etherscan.io/address/0x6Df09E975c830ECae5bd4eD9d90f3A95a4f88012#readContract#F2),
- reading the [minAnswer](https://etherscan.io/address/0xdF0da6B3d19E4427852F2112D0a963d8A158e9c7#readContract#F19) & [maxAnswer](https://etherscan.io/address/0xdF0da6B3d19E4427852F2112D0a963d8A158e9c7#readContract#F18) values from the aggregator contract

More examples: [[1](https://github.com/sherlock-audit/2023-05-USSD-judging/issues/598), [2](https://github.com/sherlock-audit/2023-05-ironbank-judging/issues/25)]

# Placing Bets After Randomness Request

Market participants should not be able to [place a bet or other input after a randomness request](https://code4rena.com/reports/2023-03-wenwin#m-04-possibility-to-steal-jackpot-bypassing-restrictions-in-the-executedraw) has been made, which will return the result of a lottery or other form of draw, as an attacker would be able to front-run the randomness response by inspecting the winning result then purchasing a ticket with the winning inputs.
More examples: [[1](https://docs.chain.link/vrf/v2/security#dont-accept-bidsbetsinputs-after-you-have-made-a-randomness-request)]

# Re-requesting Randomness

[Re-requesting randomness](https://docs.chain.link/vrf/v2/security#do-not-re-request-randomness) allows VRF service providers to withhold fulfillment if the outcome is not favorable to them, wait for the re-request, then return the randomness only if it is now favorable to them.
