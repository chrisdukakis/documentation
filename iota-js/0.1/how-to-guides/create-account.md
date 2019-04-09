# Create an account

**An account is a stateful client library that makes it easier to send and receive transactions without worrying about reusing addresses or promoting and reattaching pending transactions.**

You can use accounts to do the following:

* Create and manage conditional deposit addresses (CDAs), which specify whether they're active and may used in withdrawals or deposits.
* Store the state of a seed in a database

## Withdrawls and deposits

Accounts are made up of many addresses that are in either an active or expired state. The state of an address determines whether it can be used in a withdrawal or a deposit.

The term _withdrawal_ is used to refer to the use of CDAs in [input transactions](root://iota-basics/0.1/concepts/bundles-and-transactions.md) where IOTA tokens are withdrawn from an address. A withdrawal can involve multiple expired CDAs, depending on total deposit amount and the balance of the CDAs.

The term _deposit_ is used to refer to the use of CDAs in outputs transactions where IOTA tokens are deposited into an address.

You can't withdraw tokens from an active address, but depositors can deposit tokens into them.

You can withdraw tokens from an expired addresses, but depositors can't deposit tokens into them.

## Seed state

|**Data**| **Function**|
|:-----------------|:----------|
|The last key index that was used to create a CDA| To be able to create a new CDA that has never been used before|
|All active CDAs|To stop withdrawals from CDAs that may receive deposits|
|Pending transfers| To be able to monitor pending transactions and rebroadcast or reattach them if necessary|

You can create multiple accounts, and each one can manage the state of only one unique seed.

:::important:Important
You must not create multiple accounts with the same seed. Doing so could lead to a race condition where the seed state would be overwritten.

If you have never created an account before, you must create a new seed. Existing seeds can't be used in an account because their states are unknown.
:::

## Create a new account

1. Install the libraries

    ```bash
    npm install @iota/account @iota/converter
    ```

2. Create an `account` object with a new seed and connect to a node
   
   ```js
   import { createAccount } from '@iota/account';
   import { trytesToTrits } from '@iota/converter';

   const seed = trytesToTrits('ASFITGPSD9ASDFKRWE...');

   // Local node to connect to;
   const provider = 'http://<node-url>:14265';

   const account = createAccount({
         seed,
         provider
   });
   ```
    :::

3. **Optional:** Pass a **`persistenceAdapater`** factory to your account. This adapter creates a storage object to which the account can save the seed state. By default, the seed state database is created in the root of the project. You can change the path with `persistencePath` option.

   ```JS
   import { createPersistenceAdapter } from '@iota/persistence-adapter-level';

   const account = createAccount({
         seed,
         provider,
         persistencePath: './',
         stateAdapter: createPersistenceAdapter,
         historyAdapter: createPersistenceAdapter
   });
   ```

   :::info:
    The [`@iota/persistence-adapter-level`](https://github.com/iotaledger/iota.js/tree/next/packages/persistence-adapter-level) adapter is the default adapter, but you could edit it for your needs.

    You can't use one store instance for multiple accounts at the same time. A private adapter is instantiated for each account instance.
    :::

4. **Optional** Pass a `timeSource` method that outputs the current time in milliseconds to your `account` object

   ```js
   const account = createAccount({
         seed,
         provider,
         timeSource: () => {
               // Get time with NTP
               // const time = ...
               return time
         }
   })
   ```

:::success:Congratulations! :tada:
You've created an account that will automatically promote and reattach transactions as well as manage the state your CDAs.
:::
