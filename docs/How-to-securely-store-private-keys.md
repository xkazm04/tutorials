# How to securely store private keys

*Set up Tatum KMS to manage your keys*

---

Private keys and mnemonics are the only ways to unlock your crypto assets and approve transactions. If you lose access to them, you will lose access to your assets forever. Likewise, when someone else obtains access to your keys, they can steal all your funds. This is something that you must keep in mind to run your business successfully.

<!-- theme: warning -->
>**NEVER** give access to your mnemonics or private keys to anyone. Your private keys should **NEVER** leave your secure perimeter and should not be sent over the Internet, not even via HTTPS connection. Tatum will **NEVER** ask for your keys or mnemonics, and you should **NEVER** send them to the Tatum API.

There are public endpoints in the Tatum API which accept or produce sensitive information like private keys or mnemonics. These endpoints are only meant for test use and quick prototyping, not for production usage. For the latter, you should leverage one of the following three options:
- [Tatum KMS](https://github.com/tatumio/tatum-kms) - a key management system for your private keys. This tool stores private keys and mnemonics on your server locally, and it signs pending transactions that are  pulled periodically from the Tatum API.
- [Tatum Middleware](https://github.com/tatumio/tatum-middleware) - a local proxy REST API docker image that runs on your server and accepts every API request. It forwards non-sensitive API calls (create an account, get a block) to the Tatum API and resolves sensitive API calls (generate a wallet, send a transaction) locally. Private keys and mnemonics are part of the HTTP API request but never leave your perimeter.
- **Local libraries for different programming languages** - The library takes care of sensitive operations locally, usually on your local server where the backend of your application is running.

We recommend using Tatum KMS because it is the most secure and advanced tool available. In the following sections, you will learn how to set up the KMS and see examples of performing necessary operations in the Tatum API with KMS. 

Follow the guide below to learn how to use Tatum KMS, and also refer to our **Crypto exchange part 2 workshop** below for a demo on how to work with it.

https://www.youtube.com/watch?v=CGgyyTTv0yw&t=209s&ab_channel=Tatum

---

## How does Tatum KMS work?

Tatum KMS stores all your mnemonics and private keys in a wallet storage file. This storage file is an AEC encrypted file and only you know the encryption key for it. 

Every wallet that is stored inside your KMS has a unique identifier called `signatureId`. This `signatureId` is used in communication with Tatum API and represents the wallet used in a specific operation.

After generating and storing all the wallets you want to work with, you then enable the KMS daemon mode. This daemon mode periodically checks for any transactions that are pending signature. 

Every pending transaction has a `signatureId` present. When a pending transaction is matched with a specific wallet in the wallet storage, it is signed locally and sent to the blockchain. Your wallet data is only stored in memory.

---

## Setting up Tatum KMS

Tatum KMS is a command-line tool and provides two modes of operation:
- **Daemon mode** is responsible for periodically pulling pending transactions from the Tatum API
- **CLI mode** is used to generate wallets or private keys
Tatum KMS is shipped as a Node.JS binary, and Node.JS 14+ should be installed on your server.

**Your local server**
```json
npm i -g @tatumio/tatum-kms
```
---

## Generating your wallets

To generate a wallet that is managed by the KMS, run the `generatemanagedwallet` command while in CLI mode.

**Your local server**
```json
tatum-kms generatemanagedwallet BTC
Enter password to access wallet storage:*****
{
  "signatureId": "014073ce-af80-4f9c-8c7c-653ba5880afb",
  "xpub": "xpub6FPGLmppWEemTJ56aq6wcSkjeZN4iEw1CBvQzkusgbJpqyoiPPJASLpbduzKrNF54i348moHyoVGkyz1H2TC3iEPLfacjPFEfTENkD6YzzZ"
}
```
You need to enter a password which decrypts your wallet storage. If this is the first time and the storage file is empty, you have to set up your new password.

<!-- theme: info -->
>The wallet storage is encrypted with an AEC cipher and is stored on your local server. The password you provide is used to encrypt the mnemonics and private keys inside. If you lose your password, you will lose access to your mnemonics.

---

## Enabling daemon mode
Once you have generated all your managed wallets, you need to start the KMS in daemon mode.

**Your local server**
```json
tatum-kms daemon --apiKey YOUR_API_KEY --chain=BTC
Enter password to access wallet storage:*****
```
You need to enter your wallet storage password in order to unlock the storage. This is required whenever you start the daemon. If your daemon stops, you will need to re-enter the password when you start it up again.

---

## Example usage of the API with Tatum KMS

#### Create an account and generate a blockchain address

This service facilitates [creating a wallet and generating an address](../blockchain/b3A6MjgzNjM1MTc-generate-wallet-and-address) on one of the supported blockchains, all in one call. The required parameters can be customized in the request body. 


**Request example**
```json
curl --request POST \
  --url https://api-eu1.tatum.io/v4/blockchain/chainId/account \
  --header 'Content-Type: application/json' \
  --header 'env: ' \
  --header 'x-api-key: ' \
  --data '{
  "Account": {
    "mnemonic": "urge pulp usage sister evidence arrest palm math please chief egg abuse"
  },
  "Address": {
    "xpub": "xpub6EsCk1uU6cJzqvP9CdsTiJwT2rF748YkPnhv5Qo8q44DG7nn2vbyt48YRsNSUYS44jFCW9gwvD9kLQu9AuqXpTpM1c5hgg9PsuBLdeNncid",
    "index": 0
  }
}'
```
**Response example**
```json
{
  "Account": {
    "secret": "snSFTHdvSYQKKkYntvEt8cnmZuPJB"
  },
  "Address": {
    "address": "2MsM67NLa71fHvTUBqNENW15P68nHB2vVXb",
    "xpub": "xpub6EsCk1uU6cJzqvP9CdsTiJwT2rF748YkPnhv5Qo8q44DG7nn2vbyt48YRsNSUYS44jFCW9gwvD9kLQu9AuqXpTpM1c5hgg9PsuBLdeNncid",
    "index": 0
  }
}
```
--- 
**Transfer assets between addresses**

Use this service to [transfer any blockchain assets](../blockchain/b3A6MjgzNjM1MjM-transfer-assets-between-addresses) from one address to another.

**Request example**
```json
curl --request POST \
  --url https://api-eu1.tatum.io/v4/blockchain/chainId/transaction \
  --header 'Content-Type: application/json' \
  --header 'env: ' \
  --header 'x-api-key: ' \
  --data '{
  "sender": {
    "privateKey": "0x05e150c73f1920ec14caa1e0b6aa09940899678051a78542840c2668ce5080c2",
    "secret": "snSFTHdvSYQKKkYntvEt8cnmZuPJB",
    "address": "0x687422eEA2cB73B5d3e242bA5456b782919AFc85",
    "utxoIndex": 0
  },
  "receiver": [
    {
      "address": "0x687422eEA2cB73B5d3e242bA5456b782919AFc85",
      "note": "Hello there",
      "chainId": "ETH",
      "token": {
        "address": "2MsM67NLa71fHvTUBqNENW15P68nHB2vVXb",
        "symbol": "BTC",
        "amount": 0
      }
    }
  ],
  "fee": {
    "price": 50,
    "limit": 50000,
    "currency": "CELO"
  }
}'
```
**Response example**
```json
{
  "signatureId": "26d3883e-4e17-48b3-a0ee-09a3e484ac83"
}

```
<!-- theme: info -->
>Keep in mind that, when a private key is included in the request, you need to enter the `signatureId` of the private key to the wallet storage. Private keys can be stored using the `storemanagedprivatekey` command. You can generate a private key to a managed stored wallet using the `getprivatekey` command. When there is a mnemonic, you need to enter the `signatureId` of the mnemonic belonging to the wallet storage.
>
>In the case mentioned above, the process is a little different. The transaction is not signed and sent to the blockchain yet. Instead, it is enlisted as a pending transaction waiting to be processed by the KMS.
>
>When the KMS [detects a new pending transaction](../custody/b3A6MzYyNjUxNzc-get-pending-transactions-to-sign), it signs it locally and sends it to the blockchain. The transaction must also [be marked  as processed](../custody/b3A6MzA3NjM1NzQ-complete-pending-transaction-to-sign) so as not be sent to the blockchain again.‌
>
>This process is the same for every transaction method in every blockchain. It may seem a little complicated at the beginning but it provides you with the best security available.

Tatum KMS also supports integrations to [Azure Key Vault](https://azure.microsoft.com/en-us/services/key-vault/#product-overview) or [VGS](https://www.verygoodsecurity.com/), enabling you to store your keys and mnemonics there. More information can be found on the [Tatum KMS GitHub pages](https://github.com/tatumio/tatum-kms), where all the source code is available.
