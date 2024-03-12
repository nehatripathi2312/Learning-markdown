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