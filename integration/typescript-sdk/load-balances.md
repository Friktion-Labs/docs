# Load Balances

Loading user balances is vital to understand a user's position in a Friktion volt. This is made non-trivial by the pending deposit/withdrawal epoch system Friktion employs. The following example demonstrates how to understand the balances returned by our [SDK](./)

### Code

```typescript
import { FriktionSDK, VoltSDK } from "@friktion-labs/friktion-sdk";
import { AnchorProvider, Wallet } from "@project-serum/anchor";
import { Connection, PublicKey } from "@solana/web3.js";

// SOL Covered Call Volt
const voltVaultId = new PublicKey(
  "CbPemKEEe7Y7YgBmYtFaZiECrVTP5sGrYzrrrviSewKY"
);
const PROVIDER_URL = "https://api.mainnet-beta.solana.com";
const CLUSTER = "mainnet-beta";
const provider = new AnchorProvider(
  new Connection(PROVIDER_URL),
  Wallet.local(),
  {}
);
const friktionSDK: FriktionSDK = new FriktionSDK({
  provider: provider,
  network: CLUSTER,
});
const user = provider.wallet.publicKey;

(async () => {
  const voltSdk: VoltSDK = await friktionSDK.loadVoltSDKByKey(voltVaultId);
  console.log(
    "balances for wallet = ",
    user.toString(),
    "\n",
    await voltSdk.getBalancesForUser(user)
  );
})();

```

### Example

User flow followed by the **diff** in output of `getBalancesForuser()`  at every step. &#x20;

#### 1. Epoch Begins

User starts epoch with **15 volt tokens** in wallet. They made one claimable pending deposit in the previous epoch for **0.1 SOL**. No pending withdrawals. There are **100 volt tokens** in circulation and **1 SOL inside the volt**. Thus, each volt token is worth **0.01 SOL**, 5 volt tokens = 0.05 SOL, 10 volt tokens = 0.1 SOL.

```typescript
{
  totalBalance: 0.25, // total # SOL user has in Friktion (e.g 0.001 SOL)
  normalBalance: 0.15, // SOL value of volt tokens in user's wallet
  pendingDeposits: 0.1, // SOL value of users's unclaimed and pending deposits
  pendingWithdrawals: 0.0, // SOL value of user's unclaimed and pending withdrawals
  mintableShares: 0.1, // # of volt tokens user could mint immediately. from the claimable deposit
  claimableUnderlying: 0.0, // SOL value of user's *mintableShares*
  normFactor: 1000000000, // 10^(number of decimals in SOL mint) = 10^9 for SOL, varies depending on deposit token
  vaultNormFactor: 1000000000 // generally equivalent to normFactor, but may differ if deposit token was migrated (e.g sollet deprecation)
}
```

#### 2. Initiates Pending Withdrawal

User burns 5 volt tokens to create a pending withdrawal. There are now 95 volt tokens in circulation

```typescript
{
//  ...
 normalBalance: 0.1,
 pendingWithdrawals: 0.05,
//  ...
}
```

#### 3. Claims Pending Deposit

user claims 0.1 SOL deposit from last epoch, receiving 10 volt tokens.

```typescript
{
//  ...
  normalBalance: 0.2,
  pendingDeposits: 0.0,
  mintableShares: 0.0
//  ...
// }
```

#### 4. Initiates Pending Deposit

User deposits 0.25 SOL, creating a pending deposit for the current epoch.

```typescript
{
// ...
  totalBalance: 0.5,
  pendingDeposits: 0.25,
// ...
}
```

#### 5. Epoch Ends

Current epoch ends with 0.1 SOL of rewards for the volt. New epoch starts with 1.3 SOL ( 1.0 - 0.05 + 0.25 + 0.1) in the volt. User's pending deposit becomes claimable, pending withdrawal becomes claimable.

```typescript
{
  totalBalance: 0.525,
  normalBalance: 0.22,
  pendingDeposits: 0.25,
  pendingWithdrawals: 0.055,
  mintableShares: 0.22727272727,
  claimableUnderlying: 0.055,
}
```

#### 6. Claims Pending Withdrawal

User claims pending withdrawal. receives 0.055 SOL to wallet.

```typescript
{
// ...
  totalBalance: 0.47,
  normalBalance: 0.22,
  pendingWithdrawals: 0.0,
  claimableUnderlying: 0.0,
// ...
}
```

#### 7. Claims Pending Deposit

User claims pending deposit. receives 0.22727272727 volt tokens.

```typescript
{
// ...
  normalBalance: 0.47,
  pendingDeposits: 0.0,
  mintableShares: 0.0
// ...
}
```
