# Send and receive transactions

**To send and receive transactions in an account, you must use conditional deposit addresses (CDA). CDAs are special addresses that allow you to specify the conditions in which they may be used in account withdrawals and deposits.**

Accounts use CDAs to avoid address reuse. Without CDAs, recipients have no way of knowing whether a sender is about to withdraw tokens from an address before they deposit tokens into it. With CDAs, recipients can create an address that expires after a certain time, allowing senders to make a judgement about whether to deposit tokens into it. If senders aren't sure if a bundle will confirm in time, they can ask the recipient for another CDA.

:::info:
CDAs can be used only in an account and not in the generic [client library methods](root://client-libraries/0.1/introduction/overview.md). As a result, both you and the sender must have an account to be able to use CDAs.
:::

CDAs can be in either an active or expired state. Active addresses are part of the seed state, so you can't withdraw tokens from them, but depositors can deposit tokens into them. Expired addresses are removed from the seed state, so you can withdraw tokens from them, but depositors can't deposit tokens into them.

The workflow of a CDA should be the following:

1. You create a CDA
2. You send the CDA to a depositor
3. Based on the address's state, the depositor must decide whether a bundle will be confirmed in the given timeframe. If depositors decide the timeframe is too small, they should request a new CDA.

## Create a CDA

To create a CDA you specify the following conditions, which are used to determine if it's active or expired:

* **address (required):** An address
* **timeout_at (required):** The time at which the address expires

And one of the following, recommended fields:

* **multi_use (recommended):** A boolean that specifies if the address may be sent more than one deposit. Cannot be used in combination with `expected_amount` in the same CDA.
* **expected_amount (recommended):** The amount of IOTA tokens that the address is expected to contain. When this amount is reached, the address is considered expired. We highly recommend using this condition. Cannot be used in combination with `multi_use` in the same CDA.

:::info:
The combination of both `expected_amount` and `multi_use` in the same CDA is not supported. Both fields are designed for different scenarios. Please refer to the [CDA FAQ](../references/cda-faq.md) for more information.
:::

|  **Combination of fields** | **Withdrawal conditions**
| :----------| :----------|
|`timeout_at` |The CDA can used in withdrawals as long as it contains IOTA tokens|
|`timeout_at` and `multi_use` (recommended) |The CDA can be used in withdrawals as soon as it expires, regardless of how many deposits were made to it. See the [CDA FAQ](../references/cda-faq.md) on when to use addressess with the `multi_use` field set. |
|`timeout_at` and `expected_amount` (recommended) | The CDA can be used in withdrawals as soon as it contain the expected amount. See the [CDA FAQ](../references/cda-faq.md) on when to use addressess with the `multi_use` field set.|

:::warning:Warning
If a CDA was created with only the `timeout_at` field, it can be used in withdrawals as soon as it has a non-zero balance even if it hasn't expired. 

To avoid address reuse, we recommend creating CDAs with either the `multi_use` field or with the `expected_amount` field whenever possible.
:::

1. Pass the CDA fields to the `generateCDA()` method

    ```js
    account.generateCDA({
        timeoutAt: Math.floor(new Date('7-16-2186').getTime() / 1000), // Date in seconds
        multiUse: true,
        expectedAmount: 10000000,
        security: 2,
    })
        .then(cda => {
            // Do something with the CDA
        })
        .catch(err => {
            // Handle errors here...
        })
    ```

:::info:
If you created an account with a `timeSource` method, you can set the `timeoutAt` field.
:::

The `generateCDA()` method returns a CDA object with the following fields:
```js
{
   address: 'AT9GOVPQDDKAJ...ADFA9IRSV', // The last 9 trytes are the checksum
   timeoutAt: 6833278800,
   multiUse: false,
   expectedAmount: 1000
}
```

## Distribute a CDA

Because CDAs are descriptive objects, you can serialize them into any format and distribute them. For example, the `serializeCDAMagnet(cda)` method returns a serialized CDA as a magent link:

```json
iota://MBREWACWIPRFJRDYYHAAME…AMOIDZCYKW/?timeout_at=1548337187&multi_use=1&expected_amount=0
```

1. To parse a magnet link into a CDA, use the `parseCDAMagnet()` method

    ```js
    import { parseMagnetLink } from '@iota/cda'
        
    const magnetLink = 'iota://MBREWACWIPRFJRDYYHAAME…AMOIDZCYKW/?timeout_at=1548337187&multi_use=1&expected_amount=0'
    const { address, timeoutAt, multiUse, expectedAmount } = parseCDAMagnet(
        magnetLink
    );
    ```

## Deposit IOTA tokens into a CDA

1. After making sure that the CDA is still active, use the `sendToCDA()` method to deposit IOTA tokens into it

    ```js   
    account.sendToCDA({
        address: 'AT9GOVPQDDKAJ...ADFA9IRSV', // must include the checksum
        multiUse: false,
        timeoutAt: 6833278800,
        expectedAmount: 10000000,
        value: 10000000
    })
         .then((trytes) => {
            console.log('Successfully prepared transaction trytes:', trytes)
        })
        .catch(err => {
            // Handle errors here...
        });

    // start attachments of transactions
    account.startAttaching({
        depth: 3,
        minWeightMagnitude(9),
        delay: 30 * 1000 // 30s
    })
    
    // or stop
    account.stopAttaching()
    ```

2. **Optional:** To use a magnet link, use `parseCDAMagnet()` and pass the results to the`sendToCDA()` method

    ```js
     const magnetLink = 'iota://MBREWACWIPRFJRDYYHAAME…AMOIDZCYKW/?timeout_at=1548337187&multi_use=1&expected_amount=0'
     const { address, timeoutAt, multiUse, expectedAmount } = parseCDAMagnet(
        magnetLink
    );

    account.sendToCDA({
        address,
        timeoutAt,
        multiUse,
        expectedAmount,
        value: 1 // value to send to the CDA
    })
        .then()
        .catch();
    ```
