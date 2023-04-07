# Typescript SDK

NPM package at [https://www.npmjs.com/package/@friktion-labs/friktion-sdk](https://www.npmjs.com/package/@friktion-labs/friktion-sdk)

Open source repository at [https://github.com/Friktion-Labs/sdk](https://github.com/Friktion-Labs/sdk)

Source code for all examples: [https://github.com/Friktion-Labs/sdk/tree/master/examples](https://github.com/Friktion-Labs/sdk/tree/master/examples)

{% hint style="danger" %}
All examples are tested using SDK version 2.3.3. Please use this major version and an equal to or higher minor version to replicate results.t
{% endhint %}

### Basic Setup

#### 1. Install Friktion SDK [from NPM](https://www.npmjs.com/package/@friktion-labs/friktion-sdk)

Friktion requires a minimum of [Node.js 16](https://nodejs.org/en/)&#x20;

`yarn add @friktion-labs/friktion-sdk`

#### 2. Load Friktion & Volt SDK Instance

```typescript
import {
  // sdk WITHOUT a user wallet attached
  VoltSDK,
  // sdk WITH a user wallet attached
  toConnectedSDK,
  FriktionSDK,
} from "@friktion-labs/friktion-sdk";

// SOL Covered Call Volt pubkey
const voltVaultId = new PublicKey(
  "CbPemKEEe7Y7YgBmYtFaZiECrVTP5sGrYzrrrviSewKY"
);

const friktionSDK: FriktionSDK = new FriktionSDK({
  provider: provider, // e.g AnchorProvider
  network: CLUSTER, // e.g mainnet-beta
});

// loads a volt SDK based on the VoltVault account pubkey
// friktionSDK.loadShortOptionsVoltSDKByKey to load a ShortOptionsVoltSDK
// friktionSDK.loadEntropyVoltSDKByKey to load an EntropyVoltSDK
const voltSdk = await friktionSDK.loadVoltAndExtraDataByKey(voltVaultId), // friktionSdk.loadVolt(...) is a lightweight version of sdk with reduced functionality


// creates a ConnectedSDK instance
// toConnectedEntropySDK for getting a ConnectedEntropyVoltSDK (
const cVoltSdk = toConnectedVoltSDK(
  voltSdk,
  connection, // solana cluster connection
  user, // generally, provider.wallet pubkey
  undefined, // only used if depositing from a PDA or other program-owned account
);
```

**3. Use this reference package.json for example dependencies:**

[https://github.com/Friktion-Labs/sdk/blob/master/examples/package.json](https://github.com/Friktion-Labs/sdk/blob/master/examples/package.json)

{% hint style="info" %}
In the below examples, SOL is used as a default placeholder for the deposit token of a volt. The deposit token can be easily found by calling `voltSdk.depositMint()`&#x20;
{% endhint %}

The SDK provides a method for each instruction, returning a `TransactionInstruction` object later to be included in a transaction.

### Deposit

See a [Full Deposit Example](deposit.md) for implementation details.

```typescript
//// DEPOSIT EXAMPLE ////

// depositing 0.00001 SOL
const depositAmount: Decimal = new Decimal(0.00001);
await cVoltSdk.doFullDeposit(depositAmount);
```

### Withdraw

See a [Full Withdraw Example](withdraw.md) for implementation details. Keep in mind that it is impossible to 100% accurately know the value of a volt before the epoch ends, and thus any withdrawal done mid-epoch is based on a (fairly accurate) estimate.

```typescript
//// WITHDRAW EXAMPLE ////

// withdrawing 0.00001 SOL
const withdrawAmount: Decimal = new Decimal(0.00001);
await cVoltSdk.doFullWithdraw(withdrawAmount); // creates and send transactions
```

### Load Balances

See a [Full Example](load-balances.md) for implementation details.

```typescript
// NOTE: in below definitions, 'SOL' would be replaced by the deposit token for that volt.

const {
    totalBalance, // total # SOL user has in Friktion (e.g 0.001 SOL)
    normalBalance, // SOL value of volt tokens in user's wallet
    pendingDeposits, // SOL value of users's unclaimed and pending deposits
    pendingWithdrawals, // SOL value of user's unclaimed and pending withdrawals
    mintableShares, // # of volt tokens user could mint immediately
    claimableUnderlying, // SOL value of user's *mintableShares*
    normFactor, // 10^(number of decimals in SOL mint) = 10^9 for SOL, varies depending on deposit token
    vaultNormFactor, // generally equivalent to normFactor, but may differ if deposit token was migrated (e.g sollet deprecation)
}  = await voltSdk.getBalancesForUser(user // wallet pubkey or PDA);

```

### Calculate Profits/Losses

See a [Full Example](./#calculate-epoch-reward) for implementation details.

This retrieves the pnl denominated in the **deposit token** of the volt. This can be accessed via `voltSdk.voltVault.depositMint`.  This feature is only supported after a certain epoch on each volt. For **full historicals** or **user-specific PnL** (like [this](https://friktion.fi/portfolio/H6S9znd4mWtz3ug5wMt4yAzekKscbCfYkskCrc6y8uRr)), please ask in the [#developers ](https://discord.com/channels/891360797081632768/904575831844724736)channel in discord

```typescript
// denominated in deposit token.
// e.g if the volt made $1000 in premium, SOL is worth $100, 
const epochPnl = await voltSdk.getPnlForEpoch(roundNumber)
// for SOL Call round 25, outputs 325.872 SOL.
```

### Calculate TVL

This gets the **TVL** (total value locked) of a volt (all collateral under control of the volt's on-chain authorities).

```typescript
const tvl = await voltSdk.getTvlInDepositToken()
```

To convert to USD-demoninated TVL, simply multiply by the deposit token price (scraped from coingecko). Or, call the built-in function.

```typescript
const usdTvl = await voltSdk.getTvl();
// or:
// const usdTvl = tvl.mul(await voltSdk.depositTokenPrice());VoltSDK vs. ConnectedVoltSDK
```

For more detailed statistics about TVL, call `getTvlStats()`

```typescript
const {
    tvl
    usdTvl
    strategyDeposits,
    pendingDeposits,
    pendingWithdrawals, // SOL
  } = await voltSdk.getTvlStats();
```

### Print Volt Details

To see the loaded volt's **type, strategy, and name.**

```typescript
const type: VoltType = voltSdk.voltType() // 
// export enum VoltType {
//  ShortOptions = 0,
//  Entropy = 1,
// }

const strategy: VoltStrategy = await voltSdk.voltStrategy()
// export enum VoltStrategy {
//  ShortCalls,
//  ShortPuts,
//  ShortCrab,
//  LongBasis,
// }

const voltNumber: number = await voltSdk.voltNumber()
// ShortCalls => 1
// ShortPuts => 2
// ShortCrab => 3
// LongBasis => 4
//

const name: string = voltSdk.voltName()
// e.g Volt #01: Covered Call
```

It's often useful to print more details of volt in order to visually interpret the current state (e.g stage of rebalancing, total TVL,&#x20;

```typescript
await voltSdk.printState();
```

Truncated output below. To see full output check out the example

```
-------------------------
 ID: CbPemKEEe7Y7YgBmYtFaZiECrVTP5sGrYzrrrviSewKY
-------------------------
Volt #01: Covered Call
volt 1 or 2
returning...
Short (Fri, 01 Jul 2022 02:00:00 GMT $0.019230769230769230769 PUT)

-------------------------
 HIGH LEVEL STATS
-------------------------
Total Value (minus pending deposits) (SOL):  159155.947477497 , ($):  6124320.85893408456
deposit pool:  0.947477497
premium pool:  0.535425 permissioned premium pool:  35795.55105

...
...
```

### Epoch-specific Information

Each Volt epoch contains valuable information about deposits/withdrawals, fees collected, PnL, and the rebalancing process.

```typescript
const currentRound = await voltSdk.getRoundByNumber(voltSdk.voltVault.roundNumber) // Round account
const currentRound2 = await voltSdk.getCurrentRound() // equivalent to currentRound
const currentEpochInfo = await voltSdk.getEpochInfoByNumber(voltSdk.voltVault.roundNumber) // FriktionEpochInfo account
const currentEpochInfo2 = await voltSdk.getCurrentEpochInfo() // equivalent to currentEpochInfo
```

To understand the fields on these account better, read the [Global Accounts](broken-reference) section

### Estimate Fees

Performance fees can be easily calculated given an expected reward (denominated in deposit token).

```typescript
const totalReward = 100 * voltSdk.getDepositTokenNormalizationFactor() // 100 SOL in lamports
const performanceFee = voltSdk.performanceFeeAmount(totalReward) // 1000 bps = 10% * 100 SOL = 10 SOL = 10000000000 lamports
```

while withdrawal fees require an estimation of the underlying received.

```typescript
const expectedWithdrawnUnderlying = 100 * voltSdk.getDepositTokenNormalizationFactor() // 100 SOL in lamports
const withdrawalFee = voltSdk.withdrawalFeeAmount(expectedWithdrawnUnderlying) // 10 bps = 0.1% * 100 SOL = 0.1 SOL = 100000000 lamports
```

### Alternative Strategy Tracking

If there are operational difficulties with public volts, or if a Circuits volt requests to pause the current strategy, all Friktion volts can utilize a simple, low risk **lending aggregator**.

```typescript
const {tvlDepositToken, tvlUsd} = await voltSdk.getEntropyLendingTvl()
```

Entropy describes the set of exchanges + lending platforms that are based off the original [Mango Markets](https://mango.markets) codebase. To view the lower-level objects used to calculate volt TVL in this set of lending platforms, see below.

```typescript
const { 
    entropyGroup,
    entropyAccount, 
    entropyCache 
} = await this.getEntropyLendingObjects(); // may fail if lending has not been initialized
```

### Load All Volts on Friktion UI

```typescript
let friktionSdk = new FriktionSDK({
   provider: new AnchorProvider(
     new Connection("https://api.mainnet-beta.solana.com"),
     Wallet.local(),
     {}
   ), // with mainnet RPC
   network: 'mainnet-beta',
});

const allMainnetVolts: VoltSDK[] = await friktionSdk.getAllVoltsInSnapshot();

friktionSdk = new FriktionSDK({
   provider: new AnchorProvider(
     new Connection("https://api.devnet.solana.com"),
     Wallet.local(),
     {}
   ), // with devnet RPC
   network: 'devnet',
});

const allDevnetVolts: VoltSDK[] = await friktionSdk.getAllVoltsInSnapshot();

```

### VoltSDK vs. ShortOptions/Entropy-VoltSDK vs. ConnectedVoltSDK

**VoltSDK** is a generic wrapper around a single Friktion volt. Useful for retrieving accounts (e.g [VoltVault, ExtraVoltData](broken-reference)) and calculating statistics. It is implemented as an abstract class. Each specific volt type extends **VoltSDK** to form their own SDK. **ShortOptionsVoltSDK** provides helper methods for interacting with _volt 1 & 2,_ while **EntropyVoltSDK** does the same for _volt 3 & 4._&#x20;

**ConnectedVoltSDK** is an extension of **VoltSDK** tied to a specific user, and provides an API for creating any Friktion volt instruction using the provided wallet as an authority. It's also an abstract class, so requires a specific implementation given by **ConnectedShortOptionsVoltSDK** or **ConnectedEntropyVoltSDK.** Examples below:

```typescript
// VoltSDK
constructor(
    readonly sdk: FriktionSDK,
    readonly voltVault: VoltVault,
    readonly voltKey: PublicKey,
    extraVoltData?: ExtraVoltData | undefined
)
```

```typescript
// ConnectedVoltSDK extends VoltSDK
constructor(
  voltSDK: VoltSDK,
  connection: Connection,
  user: PublicKey,
  daoAuthority?: PublicKey | undefined
)
```

```typescript
// load ShortOptionsVoltSDK extends VoltSDK
const solPutLowKey = new PublicKey("2evPXRLaTZj92DM93sdryeszwqoC9C6DoWa1TKHn1AzU");
const solPutLowShortOptionsSdk = await friktionSDK.loadShortOptionsVoltSDKByKey(solPutLowKey);

// convert to ConnectedShortOptionsVoltSDK extends (ConnectedVoltSDK & ShortOptionsVoltSDK)
// this inherits all methods from both parent classes
const solPutLowConnectedSDK: ConnectedSDK = toConnectedSDK(
    solPutLowShortOptionsSdk,
    provider.connection,
    provider.wallet
);
    
// use short options specific helper to get the specific class type
const solPutLowConnectedSDK: ConnectedShortOptionsVoltSDK = toConnectedShortOptionsSDK(
    solPutLowShortOptionsSdk,
    provider.connection,    
    provider.wallet
);


const solBasisKey = new PublicKey("2yPs4YTdMzuKmYeubfNqH2xxgdEkXMxVcFWnAFbsojS2");
const solBasisEntropySdk = await friktionSDK.loadEntropyVoltSDKByKey(solBasisKey);

// mirror methods toConnectedSDK and toConnectedEntropySDK
```
