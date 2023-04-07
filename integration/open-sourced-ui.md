---
description: Dive into our open-source frontend and learn how our UI interacts with Solana!
---

# Open-Sourced UI

**Objective**: focus here is on Solana specific code and not so much on react or css

**\*\*Note**: If you are looking to integrate Friktion with your protocol, go [here](broken-reference) instead!

### Getting Started

Download the repo here: [https://github.com/Friktion-Labs/frontend](https://github.com/Friktion-Labs/frontend)

### Ok, let's dive in.

An important concept this UI uses is a waterfall style data flow. As many UI components have dependencies such as price of assets, user deposit info, and Volt specific data, we need a way to propagate this data based on dependency.

```
Context: MarkPrices10
  Fetches the mark price for all assets in the registry
  Dependencies: none


Context: SubvoltLoader10
  Loads all subvolt data for all subvolts
  Returns {
    allLoaded: boolean,
    subvoltData: Record<GlobalId, Subvolt1Data | null>
  }
  Soft dependencies:
   - MarkPrices10: If markPrices10 is loaded, will calculate and return the info in Subvolt1Data


Context: UserDeposits10
  Loads deposit information for all subvolts for the connected user
  Returns {
    allLoaded: boolean,
    depositsForUser: Record<GlobalId, Subvolt1Data | null>
  }
  Dependencies:
    - SubvoltLoader10 (the dependency is due to voltVaultData being loaded)
    - useConnectedWallet

MarkPrices10
↳ SubvoltLoader10
  ↳ UserDeposits10
    ↳ useCards
      ↳ React components
```

#### More Important files

registry10.tsx - Contains important metadata

AppProviders.tsx - View all the providers as well as see how we use @solana/wallet-adapter-wallets

EvilTwinSisterOfVFAC.tsx - UserDeposits10.tsx actually depends on this file for `depositedForAssets` and `vaultsForAssets`

OwnedTokenAccounts.tsx - Load connected wallet's tokens

transactionHandler.ts - Where the actual use of Friktion's SDK is to deposit/withdraw/etc

### Important Patterns&#x20;

You will notice `AsyncButton09` includes `goodies` object which is what those functions in `transactionHandler.ts` require to perform a transaction.

![goodies object](<../.gitbook/assets/image (47).png>)

Sail is a React library for Solana account management and transaction handling. You can also see the use of sail when fetching Volt data in `SubvoltLoader10.tsx`
