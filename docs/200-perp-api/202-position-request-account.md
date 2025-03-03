---
sidebar_label: "PositionRequest Account"
description: "Understand and integrate the Jupiter Perp Program."
title: "PositionRequest Account"
---

<head>
    <title>PositionRequest Account</title>
    <meta name="twitter:card" content="summary" />
</head>

This page contains an overview of the **Solana account types** used in the Jupiter Perpetuals Program, and specifically the `PositionRequest` account.

The `PositionRequest` account is a struct which represents a set of parameters and states associated to a request to open or close a position, the `PositionRequest` account consists of mostly similar properties as [`Position` account](./position-account).

:::note `PositionRequest` Account Derivation
It is a Program-Derived Address (PDA) derived from the underlying `Position` account's address, several constant seeds, and a random integer seed which makes each `PositionRequest` account unique.

The is an example [`PositionRequest` account](https://solscan.io/account/DNnX2B1oiYqKLrbLLod1guuaZA28DQwJ8HuHsgDafoQK).
:::

:::info `PositionRequestATA` account
A `PositionRequestATA` account is created for each `PositionRequest` account.

The `PositionRequestATA` account is an [associated token account](https://spl.solana.com/associated-token-account) derived from the `PositionRequest` that contains the tokens from the trader's deposits or withdrawals from withdrawing collateral or closing positions.

The tokens are then transferred to the position token's custody token account or returned to the trader's wallet when the `PositionRequestATA` account is closed.
:::

:::info Take Profit / Stop Loss Requests
`PositionRequest` accounts for non TP / SL requests are closed as soon as the request is executed or rejected.

TP / SL requests are also stored onchain via `PositionRequest` accounts. However, they will only be closed when the TP / SL request is triggered and executed.

Active TP / SL requests can be fetched onchain (through blockchain explorers like Solscan or SolanaFM) by searching for the `PositionRequest` address or public key associated with the TP / SL request.
:::

:::tip Example Typescript Repository
This [repository](https://github.com/julianfssen/jupiter-perps-anchor-idl-parsing) contains Typescript code samples on interacting with the Jupiter Perpetuals program IDL with `anchor` and `@solana/web3.js`

You can also find the [Custody Account fields in the repository](https://github.com/julianfssen/jupiter-perps-anchor-idl-parsing/blob/1a0b5dc71081958895691047a9aa8ba51d2a8765/src/idl/jupiter-perpetuals-idl.ts#L2583) or on a [blockchain explorer](https://solscan.io/account/PERPHjGBqRHArX4DySjwM6UJHiR3sWAatqfdBS2qQJu#anchorProgramIdl).
:::

## Account Details

Each `PositionRequest` account contains the following data:

| Field | Description |
| --- | --- |
| `owner`                   | **Type:** `publicKey`<br /><br />The public key of the trader's account. |
| `pool`                    | **Type:** `publicKey`<br /><br />The public key of the [JLP pool account](./pool-account). |
| `custody`                 | **Type:** `publicKey`<br /><br />The public key of the position's [`custody` account](./custody-account). |
| `collateralCustody`       | **Type:** `publicKey`<br /><br />The public key of the position's collateral custody account.<br /><br />Like the `custody` account, a `collateralCustody` account contains information for the token that's used as collateral for the position (SOL / wETH / wBTC for long positions, USDC / USDT for short positions). The borrow rates for the position will also be calculated based on the position's `collateralCustody`. |
| `mint`                    | **Type:** `publicKey`<br /><br />For opening positions and collateral deposits, mint refers to the input mint requested by the trader.<br /><br />For example, if a trader opens a position by providing the initial margin with SOL, then mint will be equal to SOL's mint address. If the trader deposits collateral in USDC, then mint will be equal to USDC's mint address.<br /><br />For closing positions and collateral withdrawals, mint is equal the to position collateral token's mint address. For example, if a trader closes a long SOL position, mint will be equal to SOL's mint address. If a trader closes a short SOL position, mint is equal to USDC or USDT's mint address depending on the position's collateral. |
| `openTime`                | **Type:** `i64`<br /><br />The time when the request of position is created in UNIX timestamp format. |
| `updateTime`              | **Type:** `i64`<br /><br />The time when the request of position is last updated in UNIX timestamp format. |
| `sizeUsdDelta`            | **Type:** `u64`<br /><br />The USD amount to increase or decrease the position size by. The amount is an integer in the atomic value (before decimals which is 6 for USDC / UST mints).<br /><br />For example, a position request to increase an open position's size by 10 USDC will have a `sizeUsdDelta = 10000000`. |
| `collateralDelta`         | **Type:** `u64`<br /><br />For opening positions and collateral deposits, `collateralDelta` is the token amount to increase or decrease the position collateral size by. The token amount is represented in atomic values (before decimals). |
| `requestChange`           | **Type:** `RequestChange`<br /><br />`requestChange` will be equal to `Increase` for open position and collateral deposit requests, and `Decrease` for close position and collateral withdrawal requests. |
| `requestType`             | **Type:** `RequestType`<br /><br />`Market` for all position requests except for TP / SL requests, which have a different `requestType` known as `Trigger`. |
| `side`                    | **Type:** `Side`<br /><br />`Long` for long positions, `Short` for short positions |
| `priceSlippage`           | **Type:** `u64`<br /><br />The maximum price with slippage for position requests when opening, closing, or updating the position size.<br /><br />- When increasing the size of a long position or decreasing the size of a short position, the request will fail if the current price of the position's token is greater than `priceSlippage`.<br /><br />- When decreasing the size of a long position or increasing the size of a short position, the request will fail if `priceSlippage` is greater than the current price of the position's token. |
| `jupiterMinimumOut`       | **Type:** `u64`<br /><br />For requests that require token swaps, the output amount of the token swap must be greater than or equal to `jupiterMinimumOut`, else the request will fail. |
| `preSwapAmount`           | **Type:** `u64`<br /><br />This is an internal attribute used by the program to calculate the `collateralDelta` for position requests that require token swaps. |
| `triggerPrice`            | **Type:** `u64`<br /><br />The price (USD) used for TP / SL position requests. |
| `triggerAboveThreshold`   | **Type:** `bool`<br /><br />When `triggerAboveThreshold` is true, the TP / SL position request will be triggered when the position's token price is greater than or equal to `triggerPrice`. When `triggerAboveThreshold` is false, the TP / SL position request will be triggered when the position's token price is less than or equal to `triggerPrice`. |
| `entirePosition`          | **Type:** `bool`<br /><br />This attribute is only checked when closing or decreasing position sizes. When `entirePosition` is true, the entire position will be closed (i.e. a close position request). When `entirePosition` is false, the position size will be reduced according to sizeUsdDelta. |
| `executed`                | **Type:** `bool`<br /><br />Determines whether the position request is executed or not. |
| `counter`                 | **Type:** `u64`<br /><br />The random integer seed used to derive the position request address. |
| `bump`                    | **Type:** `u8`<br /><br />The bump seed used to derive the position request address. |
