# How to pay fees for token transfers from another address

*A gas pump for your native assets, ERC-20, ERC-721 and ERC-1155 tokens.*

---

If you’ve ever built or want to build a custodial exchange like Binance or Kraken; a custodial wallet service; or a custodial NFT marketplace, there’s one major headache that you’re bound to encounter: how do you pay the gas fees for sending tokens from your users’ accounts?

Astoundingly, there really hasn’t been a simple way of doing this until now. As an exchange owner, every time you want to send tokens from one of your user’s accounts, you have to calculate the gas fees, send that amount to their account (a transaction that also incurs gas fees), and send the tokens from their account.
That's two transactions, both of which incur gas fees, plus they can result in crypto “dust”, a tiny amount of currency, remaining trapped in their accounts. If you have thousands or even millions of addresses, this method creates an enormous amount of work and fees for you as a custodial provider.

All in all, this is an incredibly inconvenient process - so we decided to fix that.

---

## Gas pump: a smart contract as a user address

Our new gas pump uses a master address that generates user sub-accounts as smart contracts.


<!-- theme: info -->
>Smart contract implementation is available [here](https://github.com/tatumio/smart-contracts/blob/master/contracts/tatum/custodial/CustodialWalletFactory.sol).

You simply top up the master address with a balance of crypto from which you’ll pay all of your gas fees, and every time you send tokens from a user address, the gas fees are deducted from the master address. As with everything else in Tatum, it’s super easy to implement.

#### Generate new gas pump smart contracts

The first step is to create gas pump smart contracts. Each smart contract functions as a wallet/user address that can accept ERC-20, ERC-721, ERC-721, and native assets on the blockchain you deploy it on. 

Any combination of tokens can be transferred in a batch transaction. This is significantly more efficient and saves enormously on gas fees compared to transferring assets one by one.

Enough talk; let's [generate some gas pump wallets](../blockchain/b3A6MzA4ODYyMjE-generate-custodial-wallet-address) on the Polygon Mumbai testnet. In this call, we must designate the number of wallets we want to create in the "batchCount" field.

**Request example**
```json
curl --request POST \
  --url https://api-eu1.tatum.io/v4/blockchain/custodial \
  --header 'Content-Type: application/json' \
  --header 'x-api-key: ' \
  --data '{
  "chain": "ETH",
  "fromPrivateKey": "0x05e150c73f1920ec14caa1e0b6aa09940899678051a78542840c2668ce5080c2",
  "enableFungibleTokens": false,
  "enableNonFungibleTokens": false,
  "enableSemiFungibleTokens": false,
  "enableBatchTransactions": false,
  "fee": {
    "gasLimit": "40000",
    "gasPrice": "20"
  },
  "nonce": 0
}'
```

**Response example**
```json
{
  "txId": "c83f8818db43d9ba4accfe454aa44fc33123d47a4f89d47b314d6748eb0e9bc9",
  "failed": false
}

```


#### Transfer asset from a gas pump address

Your user deposited some ERC-20 or ERC-721 tokens into his account. Now it's time to [move assets to another address](../blockchain/b3A6MzA4ODYyMjM-transfer-assets). You don't have to send any MATIC there to pay for gas; just state which assets should be sent, how much and where. Let's send an ERC-721 token with a `tokenId` of 100 from the address.

**Request example**
```json
curl --request POST \
  --url https://api-eu1.tatum.io/v4/blockchain/custodial/transfer \
  --header 'Content-Type: application/json' \
  --header 'x-api-key: ' \
  --data '{
  "chain": "ETH",
  "custodialAddress": "0x687422eEA2cB73B5d3e242bA5456b782919AFc85",
  "tokenAddress": "0x687422eEA2cB73B5d3e242bA5456b782919AFc85",
  "contractType": 1,
  "recipient": "0x687422eEA2cB73B5d3e242bA5456b782919AFc85",
  "amount": "100000",
  "tokenId": "100",
  "fromPrivateKey": "0x05e150c73f1920ec14caa1e0b6aa09940899678051a78542840c2668ce5080c2",
  "nonce": 1,
  "fee": {
    "gasLimit": "40000",
    "gasPrice": "20"
  }
}'
```

**Response example**

```json
{
  "txId": "c83f8818db43d9ba4accfe454aa44fc33123d47a4f89d47b314d6748eb0e9bc9",
  "failed": false
}
```
As a result, we obtain a transaction hash. As you can see, we have to enter the address of the token we want to transfer, and for an ERC-721, a `tokenId` as well. 
The `contractType` attribute is important, as it tells the wallet which kind of token it is transferring:
- `contractType` 0 - ERC-20 token
- `contractType` 1 - ERC-721 token
- `contractType` 2 - ERC-1155 token
- `contractType` 3 - native asset - MATIC

<!-- theme: info -->
>Since the transfer endpoint is universal and neither ERC-20 tokens nor native assets have tokenIds, this field can be omitted with transfers of such tokens. Similarly, for amount, ERC-721 tokens are always one-of-a-kind, so the amount is omitted. Native assets don't have contract addresses, so this field is omitted when they are transferred.

#### Transfer assets from a custodial address in a batch call

Simple transfers are great, but [batch transfers](../blockchain/b3A6MzA4ODYyMjQ-transfer-multiple-assets) are much more powerful. In this operation, you can basically transfer all assets from an address in one transaction. You can define different recipients for different assets, you can split the balance of the ERC-20 token between two different recipients and much more.

In this example, we are transferring all types of assets we have in the gas pump wallet in one call:

**Request example**
```json
curl --request POST \
  --url https://api-eu1.tatum.io/v4/blockchain/custodial/transfer/batch \
  --header 'Content-Type: application/json' \
  --header 'x-api-key: ' \
  --data '{
  "chain": "ETH",
  "custodialAddress": "0x687422eEA2cB73B5d3e242bA5456b782919AFc85",
  "tokenAddress": [
    "0x687422eEA2cB73B5d3e242bA5456b782919AFc85"
  ],
  "contractType": [
    0
  ],
  "recipient": [
    "0x687422eEA2cB73B5d3e242bA5456b782919AFc85"
  ],
  "amount": [
    "100000"
  ],
  "tokenId": [
    "100000"
  ],
  "fromPrivateKey": "0x05e150c73f1920ec14caa1e0b6aa09940899678051a78542840c2668ce5080c2",
  "nonce": 1,
  "fee": {
    "gasLimit": "40000",
    "gasPrice": "20"
  }
}'
```

**Response example**
```json
{
  "txId": "c83f8818db43d9ba4accfe454aa44fc33123d47a4f89d47b314d6748eb0e9bc9",
  "failed": false
}
```
Once again, we obtain transaction hash as a response.

<!-- theme: info -->
>In case the asset you want to transfer does not have `tokenAddress`, `amount` or `tokenId`, input value 0 instead. We will take care of the rest for you.

To find out more about the API calls we have just used, see our API Reference.