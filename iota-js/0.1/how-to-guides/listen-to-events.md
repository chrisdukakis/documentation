# Listen to events in an account

**An account object emits events when they happen. An example of an event is when you send funds to other accounts or receive a deposit. You can listen for these events and act on them.**

## Prerequisites

This guide assumes that you understand the concept of events in Node.js (https://nodejs.org/api/events.html)
Event listeners are used to assign callbacks to specific event types.
Also listeners may be [removed](https://nodejs.org/api/events.html#events_emitter_removelistener_eventname_listener), and they should be, once the subscription to a
specific event is no longer required.

## Listening to deposit and withdrawal events

Deposit and withdrawal events are emitted as soon as a depositing or
withdrawing bundle is detected. Each of those bundles may trigger events in
two steps, one for it's **pending** state, and one for it's **included** (confirmed) state.

Callbacks are given an object as argument, which contains the
relevant address and the complete depositing or withdrawing bundle.

1. Attach listeners for deposit and withdrawal events:
    ```js
    account.on('pendingDeposit', ({ address, bundle }) => {
        console.log('Address:', address, 'Tail transaction hash:', bundle[0].hash);
        // ...
    })

    account.on('includedDeposit', ({ addresses, bundle }) => {
        console.log('Address:', address, 'Tail transaction hash:', bundle[0].hash);
        // ...
    })

    account.on('pendingWithdrawal', ({ address, bundle }) => {
        // ...
    })

    account.on('includedWithdrawal', ({ addresses, bundle }) => {
        // ...
    })
    ```

    Do not forget to subscribe to `error` events which are usefull for dubugging
    your application, and reacting to exceptions thrown in the background.

    ```js
    account.on('error', (error) => {
        console.log(`Something went wrong: ${error}`);
    })
    ```

2. Generate a CDA

    ```js
    const cda = account
        .generateCDA({
            timeoutAt: Date.now() + 24 * 60 * 60 * 1000,
            expectedAmount: 100,
        })
    ```

3. Send a deposit to the CDA above
    ```js
    cda
        .tap(cda => console.log('Sending to:', cda.address))
        .then(cda =>
            account.sendToCDA({
                ...cda,
                value: 100,
            })
        )
        .catch(error => console.log(error))
    ```

    :::info:
    All relevant transactions that exist in database of connected node
    trigger events uppon account startup.
    :::

You should be able to see a pair of an address and a tail transaction hash once
transaction is detected and one once confirmed!

## Full list of account events

|**Event name**|**Callback argument**|
| :----------| :----------|
|`pendingDeposit`|`{ address, bundle }`|
|`includedDeposit`|`{ address, bundle }`|
|`pendingWithdrawal`|`{ address, bundle }`|
|`includedWithdrawal`|`{ address, bundle }`|
|`selectInput`|`{ transfer, input }`|
|`prepareTransfer`|`{ transfer, trytes }`|
|`getTransactionsToApprove`|`{ trytes, { trunkTransaction, branchTransaction } }`|
|`attachToTangle`|`transactionObjects`|
|`broadcast`|`transactionObjects`|
|`error`|`Error`|

:::success:Congratulations! :tada:
You're account is now emitting events that you can listen to and act on.
:::
