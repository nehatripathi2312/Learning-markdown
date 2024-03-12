# General

These docs describe the v3 API for the dYdX decentralized perpetual contracts exchange. The exchange runs on an L2 (layer-2) blockchain system, and operates independently of previous dYdX protocols and systems, including the v1 and v2 APIs.

Like the previous iteration of dYdX perpetuals, the exchange uses a centralized order book, but remains non-custodial, and settles trades and liquidations in a trustless manner.

 These docs describe the dYdX [layer-2 perpetuals exchange](https://trade.dydx.exchange/portfolio/overview).

# Layer 2: ZK-Rollups
Trades are settled in an L2 (layer-2) system, which publishes ZK (zero-knowledge) proofs periodically to an Ethereum smart contract in order to prove that state transitions within L2 are valid. Funds must be deposited to the Ethereum smart contract before they can be used to trade on dYdX.

By settling trades on L2, the exchange is able to offer much higher trade throughput and lower minimum order sizes, compared with systems settling trades directly on Ethereum (i.e. L1). This is achieved while maintaining decentralization, and the exchange is fully non-custodial.

The L2 system was developed with, and is operated jointly with, Starkware. More information about the L2 design can be found in [Starkware's documentation](https://docs.starkware.co/starkex-docs/). (Note: Some of the details described there may be specific to Starkware's previous StarkEx system and may not apply to the dYdX system.)

# Data Centers
Our data centers are located in the AWS AP-NORTHEAST-1 region (Tokyo).

 It is strictly against our [Terms of Use] (https://dydx.exchange/terms) to use United States based IPs to trade on dYdX.

 # Number Formats
All amounts and prices in the clients and API are represented in “human readable,” natural units. For example, an amount of 1.25 ETH is represented as 1.25, and a price of $31,000.50 per BTC is represented as 31000.5.

# Base URLs
Base URLs for API endpoints are as follows:

## [Production (Mainnet)]:  (https://api.dydx.exchange)
## [Staging (Goerli)]: (https://api.stage.dydx.exchange)

## Testnet
We have one testnet which is on Goerli. To use the testnet, use the above Staging URL for your endpoint. Also use a networkId of 5 (Goerli) instead of 1 (Mainnet).

The user interface for testnet can be found [here] (https://trade.stage.dydx.exchange/portfolio/overview).

The dYdX Goerli USDC token address is 0xF7a2fa2c2025fFe64427dd40Dc190d47ecC8B36e. Users can deposit via the Testnet website.

## Rate-Limits
All rate-limits are subject to change.

Please make use of the WebSockets API if you need real-time data.

Rate Limit - API
Limits are enforced by IP Address for public endpoints, and by both IP Address and Account for private endpoints.

Each request consumes 1 point towards the rate limit. [POST v3/orders](https://dydxprotocol.github.io/v3-teacher/#rate-limit-api) consumes variable points based on the order. Points refresh at the end of each time window. Please take note of the RateLimit-Remaining header to track points usage.

## Response  Headers

|Field |Description|
|------------|----------------|
|RateLimit-Remaining |Points remaining in the time window.|
|RateLimit-Reset|Timestamp that the time window ends, in Epoch milliseconds.|
|Retry-After |Milliseconds until the next time window. Header included only when the limit has been reached.|
|RateLimit-Limit |The maximum amount of points allowed per time window.|

|Request| Limit|
|--------| --------------------|
|GET v3/*| 175 requests per 10 seconds.|
|PUT v3/emails/send-verification-email |2 requests for 10 minutes.|
|DELETE v3/orders| See Cancel-Order Rate Limits|
|POST v3/orders| See Place-Order Rate-Limits|
|POST v3/testnet/tokens| 5 requests per 24 hours.|
|GET v3/active-order |See Active-Order Rate-Limits|
|DELETE v3/active-orders| See Active-Order Rate-Limits|
|All other requests |10 requests per minute.|

## Rate Limit - Websocket
Limits are enforced per <connectionId.>

 If your connection exceeds the request limit, we will terminate the connection, and you will need to reconnect to the websocket. Additionally, sending too many invalid messages will also result in your websocket being disconnected.
|Request| Limit|
|-----------| -----------------------|
|subscribe v3_accounts, v3_markets| 2 requests per 1 second.|
|subscribe v3_orderbook, v3_trades| 2 requests for 1 second per market.|
|ping| 5 requests per 1 second.|

## Cancel-Order Rate Limits
Canceling orders is limited per asset-pair and is intended to be higher than the limit on placing orders.

<DELETE v3/orders> requests are limited to <3> requests per <10> seconds per asset-pair.

<DELETE v3/orders/:id> requests are limited to <250> requests per <10> seconds per asset-pair.

## Place-Order Rate-Limits
Order rate limits are limited to maxPoints spent (per asset-pair) in a fixed window of <windowSec> seconds.

We want to give priority to those who are making the largest orders and who are contributing the most liquidity to the exchange. Therefore, placing larger orders is subject to higher limits (i.e. larger orders carry a lower point cost). The point cost is based on the orderNotional which is equal to the <size * price> of the order.

<Limit-order point consumption is equal to:
orderConsumption = clamp(
  ceil(targetNotional / orderNotional),
  minOrderConsumption,
  maxOrderConsumption
)> 

The <minOrderConsumption> is different for each order type, and can be one of minLimitConsumption, <minMarketConsumption,> or minTriggerableConsumption. Limit orders that are Fill-or-Kill or Immediate-or-Cancel are considered to be market orders for the purposes of rate limiting.

The values of the above variables as of March 15th, 2022 are listed below, but the most up-to-date values can be found in the [v3/config endpoint.] (https://dydxprotocol.github.io/v3-teacher/#get-global-configuration-variables)

|Variable| Value|
|----------| ----------------|
|maxPoint| 1,750|
|windowSec| 10|
|targetNotional| 40,000|
|minLimitConsumption| 4|
|minMarketConsumption| 20|
|minTriggerableConsumption| 100|
|maxOrderConsumption| 100|

## Active-Order Rate-Limits
Querying active orders is limited per endpoint and per asset and is intended to be higher than the respective DELETE and GET endpoints these new endpoints replace.

## DELETE Active-Orders Rate Limits

<DELETE v3/active-orders/*> 

- 425 points allotted per 10 seconds per market.
- 1 point consumed if order id included.
- 25 points consumed if order side included.
- 50 points consumed otherwise.

## GET Active-Orders Rate Limits
<GET v3/active-orders/*>

- 175 points allotted per 10 seconds per market.
- 1 point consumed if order id included.
- 3 points consumed if order side included.
- 5 points consumed otherwise.

## Other Limits

Accounts may only have up to 20 open orders for a given market/side pair at any one time. (For example up to 20 open <BTC-USD> bids).

# Perpetual Contracts

The dYdX Perpetual is a non-custodial, decentralized margin product that offers synthetic exposure to a variety of assets.

## Margin

Collateral is held as USDC, and the quote asset for all perpetual markets is USDC. Cross-margining is used by default, meaning an account can open multiple positions that share the same collateral. Isolated margin can be achieved by creating separate accounts (sub-accounts) under the same user.

Each market has three risk parameters, the <initialMarginFraction>, <maintenanceMarginFraction> and <incrementalInitialMarginFraction>, which determine the max leverage available within that market. These are used to calculate the value that must be held by an account in order to open or increase positions (in the case of initial margin) or avoid liquidation (in the case of maintenance margin).

## Risk Parameters and Related Fields

|Risk| Description|
|------------| -------------------|
|initialMarginFraction| The margin fraction needed to open a position.|
|maintenanceMarginFraction| The margin fraction required to prevent liquidation.|
|incrementalInitialMarginFraction| The increase of initialMarginFraction for each incrementalPositionSize above the baselinePositionSize the position is.|
|baselinePositionSize| The max position size (in base token) before increasing the initial-margin-fraction.|
|incrementalPositionSize| The step size (in base token) for increasing the initialMarginFraction by (incrementalInitialMarginFraction per step).|

## Portfolio Margining
There is no distinction between realized and unrealized PnL for the purposes of margin calculations. Gains from one position will offset losses from another position within the same account, regardless of whether the profitable position is closed.

## Margin Calculation
The margin requirement for a single position is calculated as follows:

<Initial Margin Requirement = abs(S × P × I)
Maintenance Margin Requirement = abs(S × P × M)> 

Where:

- S is the size of the position (positive if long, negative if short)
- P is the oracle price for the market
- I is the initial margin fraction for the market
- M is the maintenance margin fraction for the market

The margin requirement for the account as a whole is the sum of the margin requirement over each market i in which the account holds a position:

<Total Initial Margin Requirement = Σ abs(Si × Pi × Ii)
Total Maintenance Margin Requirement = Σ abs(Si × Pi × Mi)>

The total margin requirement is compared against the total value of the account, which incorporates the quote asset (USDC) balance of the account as well as the value of the positions held by the account:

<Total Account Value = Q + Σ (Si × Pi)>

The Total Account Value is also referred to as equity.

Where:

- Q is the account's USDC balance (note that Q may be negative). In the API, this is called quoteBalance. Every time a transfer, deposit or withdrawal occurs for an account, the balance changes. Also, when a position is modified for an account, the quoteBalance changes. Also funding payments and liquidations will change an account's quoteBalance.
- S and P are as defined above (note that S may be negative)
An account cannot open new positions or increase the size of existing positions if it would lead the total account value of the account to drop below the total initial margin requirement. If the total account value ever falls below the total maintenance margin requirement, the account may be liquidated.

Free collateral is calculated as:

<Free collateral = Total Account Value - Total Initial Margin Requirement> 

Equity and free collateral can be tracked over time using the latest oracle price (obtained from the markets websocket).

## Liquidations

Accounts whose total value falls below the maintenance margin requirement (described above) may have their positions automatically closed by the liquidation engine. Positions are closed at the close price described below. Profits or losses from liquidations are taken on by the insurance fund.

## Close Price for Liquidations

The close price for a position being liquidated is calculated as follows, depending whether it is a short or long position:

<Close Price (Short) = P × (1 + (M × V / W))
Close Price (Long) = P × (1 − (M × V / W))>

Where:

1. P is the oracle price for the market
2. M is the maintenance margin fraction for the market
3. V is the total account value, as defined above
4. W is the total maintentance margin requirement, as defined above

This formula is chosen such that the ratio V / W is unchanged as individual positions are liquidated.