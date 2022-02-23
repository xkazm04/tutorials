# How to use Tatum and MetaMask

*Connect MetaMask and leverage Tatum's  abstraction in no time*

---

In a custodial app, you manage your users' private keys and transfer assets on their behalf. In a non-custodial app, users have their own wallets that hold their private keys.

If you build a non-custodial application, most of your users will probably use MetaMask as their primary wallet. You can easily use Tatum to connect your users' MetaMask wallets to your app and leverage our abstraction, ready-to-go contracts, and simple transfers in a few easy steps.

---

## Connect your MetaMask to a web page

The first step is to connect your web app with MetaMask. To do so, we'll create an Enable button on your web page that will request the accounts handled by MetaMask.

<div class='tabbed-code-blocks'>
```HTML
<button class="enableMetamask">Enable MetaMask</button>
```
```JS
let accounts = [];
document.querySelector('.enableMetamask')
.addEventListener('click', async () => {
    accounts = await ethereum.request({
      method: 'eth_requestAccounts'
    });
    console.log(accounts);
});
```
</div>
When a user clicks on the **Enable** button, the MetaMask UI will prompt you for a connection. It will then expose the managed accounts to the web app. 

In this example, we are assuming that MetaMask is installed as a Chrome extension. For more complex scenarios, take a look at these [official MetaMask guides](https://docs.metamask.io/guide/create-dapp.html#basic-action-part-1).

---

## Prepare a transaction using Tatum

Next, we will leverage a functionality Tatum uses for its Key Management System (KMS) tool to prepare a transaction for MetaMask. We will assemble it in KMS using a `signatureId`, but we will prepare the transaction configuration instead of signing it and broadcasting it to the blockchain.
In this example, we'll deploy an NFT smart contract on the Polygon Mumbai testnet.

 **Example**
<div class='tabbed-code-blocks'>
```json
// 1. Let's call the Deploy NFT with the signatureId present
//    You can use any arbitrary UUID v4 string as a signatureId
const response = await axios.post('https://api-eu1.tatum.io/v3/nft/deploy/', {
  "name": "Test Token",
  "chain": "MATIC",
  "symbol": "TTC",
  "signatureId": "b7ad58f7-d826-4db5-8a52-4f492935a7b4"
}, {
  headers: {
    'x-api-key': '951cabe04de143b98b75c4d4ed4d2d99'
  }
});
// 2. Response is the ID of the prepared transaction
const {
  signatureId
} = response.data;
// 3. Let's grab the transaction and get the configuration
const {
  data
} = await axios.get('https://api-eu1.tatum.io/v3/kms/' + signatureId, {
  headers: {
    'x-api-key': '951cabe04de143b98b75c4d4ed4d2d99'
  }
});
```
</div>
1. First, we'll invoke the [Deploy NFT](https://developer.tatum.io/rest/smart-contracts#/b3A6MzA3NjA5MDg-deploy-nft-smart-contract) operation. We will use a `signatureId` in the call, which can be any arbitrary UUID v4 string. Again, we will not sign the transaction and broadcast it to the blockchain. We will only prepare the transaction config.

<div class='tabbed-code-blocks'>
```json
const response = await axios.post('https://api-eu1.tatum.io/v3/nft/deploy/', {
  "name": "Test Token",
  "chain": "MATIC",
  "symbol": "TTC",
  "signatureId": "b7ad58f7-d826-4db5-8a52-4f492935a7b4"
}, {
  headers: {
    'x-api-key': '951cabe04de143b98b75c4d4ed4d2d99'
  }
});
```
</div>
2. The response is the `signatureId` - the identifier of the prepared transaction that is waiting to be signed.

<div class='tabbed-code-blocks'>
```json
const {
  signatureId
} = response.data;
```
</div>

3. Next, we'll grab the pending transaction.

<div class='tabbed-code-blocks'>
```json
const {
  data
} = await axios.get('https://api-eu1.tatum.io/v3/kms/' + signatureId, {
  headers: {
    'x-api-key': '951cabe04de143b98b75c4d4ed4d2d99'
  }
});
```
</div>

---

## Send the transaction to MetaMask for signature

Now that we have prepared the transaction using KMS, we need to perform a few operations to get it ready for MetaMask.

We need to get the transaction configuration, set the correct sender account, and convert the gas price to be compatible with Metamask.

1. First, let's get the transaction configuration.

<div class='tabbed-code-blocks'>
````json
const txConfig = JSON.parse(data.serializedTransaction);
````
</div>

2. Next, we'll set the correct sender account.

<div class='tabbed-code-blocks'>
````json
txConfig.from = accounts[0];
````
</div>

3. Now, we'll convert the `gasPrice` from the decimal gwei format Tatum uses to the HEX gwei format that MetaMask uses.

<div class='tabbed-code-blocks'>
```json
txConfig.gasPrice = txConfig.gasPrice ? 
parseInt(txConfig.gasPrice).toString(16) : undefined;
```
</div>

4. And finally, let's send the transaction to MetaMask.

<div class='tabbed-code-blocks'>
```json
console.log(await ethereum.request({
  method: 'eth_sendTransaction',
  params: [txConfig],
}));
```
</div>

Below, all the steps of **Deploying an NFT** are assembled together.

<div class='tabbed-code-blocks'>
```json
// 1. Let's call the Deploy NFT with the signatureId present
//    You can use any arbitrary UUID v4 string as a signatureId
const response = await axios.post('https://api-eu1.tatum.io/v3/nft/deploy/', {
  "name": "Test Token",
  "chain": "MATIC",
  "symbol": "TTC",
  "signatureId": "b7ad58f7-d826-4db5-8a52-4f492935a7b4"
}, {
  headers: {
    'x-api-key': '951cabe04de143b98b75c4d4ed4d2d99'
  }
});
// 2. Response is the ID of the prepared transaction
const {
  signatureId
} = response.data;
// 3. Let's grab the transaction and get the configuration
const {
  data
} = await axios.get('https://api-eu1.tatum.io/v3/kms/' + signatureId, {
  headers: {
    'x-api-key': '951cabe04de143b98b75c4d4ed4d2d99'
  }
});
// 4. Get the transaction config
const txConfig = JSON.parse(data.serializedTransaction);
// 5. Set the correct sender account
txConfig.from = accounts[0];
// 6. Put gas price in HEX
txConfig.gasPrice = txConfig.gasPrice ? parseInt(txConfig.gasPrice).toString(16) : undefined;
// 7. Send tx to MetaMask
console.log(await ethereum.request({
  method: 'eth_sendTransaction',
  params: [txConfig],
}));
```
</div>

---

## Recap

To sum it up, we have connected MetaMask to our web app, chosen the operation to perform (deploy an NFT smart contract), used Tatum in KMS mode to prepare the transaction, set the correct sender account and gas price parameters, and let MetaMask sign and broadcast the operation.
The same method can be used for many other blockchain operations, allowing you to connect MetaMask to your non-custodial app for a wide range of use cases.

Everything we have done here is available in this [JSFiddle](https://jsfiddle.net/u7Lmfhqo/5/).
