# How to build a crypto exchange

*Build a backend for your exchange in 30 minutes*

---

## Introduction

A crypto exchange is a web application where users can trade their crypto assets. The operator of the exchange is the one who owns the private keys to all of the user's crypto assets - in this case, we are talking about a custodial exchange leveraging a custodial wallet.

<div youtube-url="https://www.youtube.com/watch?time_continue=1&v=pVBNvh04ixo&feature=emb_title"></div>


<div class="toolbar-note">
A custodial wallet is a wallet where a third party holds the private keys, not the crypto assets owner. The provider has full control over crypto assets, while users only have permission to send or receive payments.
</div>

Every exchange must have wallets for every crypto asset it supports. Every user of the exchange must obtain accounts for every asset they are trading. The exchange operator defines the trading pairs that can be traded by users and usually charges a fee for every trade performed.

Trades are not performed on the blockchain (on-chain), as this would be extremely slow and expensive. All the trades are only virtual transactions between user accounts.

‌ There are four logical groups of actions to create an exchange:
- **Setting up the application** - includes prerequisites like blockchain wallet creation and creating exchange service accounts for gathering fees.
- **Registration of new users in the application** - the steps that need to be executed when new users register in the exchange
- **User application journey** - what kind of actions users take while in the application
- **Trading** - enabling users to trade their assets

<div class="toolbar-note">
> In the next sections, we will use Bitcoin and Ethereum as blockchains to demonstrate the functionalities.
</div>

---

## Setting up the application

This phase is a one-time step that must be done before the launch of the exchange. It involves creating [the blockchain wallets](https://docs.tatum.io/rest/blockchain/generate-bitcoin-wallet) your application will support or creating service fee accounts for the wallet provider.

![SetUpYourApp.jpg](https://stoplight.io/api/v1/projects/cHJqOjExNjE5Mw/images/D3NosL1IKQo)

### Creating blockchain wallets

To [generate a Bitcoin wallet](https://docs.tatum.io/rest/blockchain/generate-bitcoin-wallet), you need to call a request to the Bitcoin/wallet endpoint. This is the same [for Ethereum](https://docs.tatum.io/rest/blockchain/generate-ethereum-wallet). Only the endpoint is different. The result contains two fields - mnemonic and xpub. 

<div class='tabbed-code-blocks'>
```Request
curl --request GET \
  --url 'https://api-eu1.tatum.io/v3/bitcoin/wallet' \
  --header 'x-api-key: YOUR_API_KEY'
```
```Response
{
  "mnemonic": "zebra parent avocado margin ready heart space orchard police junior travel today bag action rough system novel large rain detail route spare add mail",
  "xpub": "tpubDE8GQ9vAXpwkp37PCCRUpCoeShpC4WiCcACxh8r3nnKjfRPRqw3w58EgkfNiBy1MaRqX1oAAxwAxauEUG7vWupSh5m15znGy7vE7aE6CWzb"
}
```
</div>

<div class="toolbar-note">
To generate a wallet for any ERC-20 token like USDT, LINK or others, use the same call as for Ethereum. These tokens are transported on the Ethereum network and leverage the same Ethereum addresses as native Ether.
</div>

<div class="toolbar-warning">
Blockchain wallets here are created using API, which is not a secure way of generating wallets. Your private keys and mnemonics should never leave your security perimeter. To correctly and securely generate a wallet, you can use [Tatum CLI](https://github.com/tatumio/tatum-cli) from the command line or use our complex key management system, [Tatum KMS](https://github.com/tatumio/tatum-kms).
</div>  

### Generating service accounts

For every supported blockchain wallet, a service ledger account should be created. These accounts will be used to gather fees from the trading of the users. A fee will be charged for every trade performed, and the fee will be transferred to the account.

‌Every account can belong to a specific customer in Tatum. Within Tatum, a customer is an entity containing information about a user of your application, such as the customer's country of residence, accounting currency, etc. The customer is only created during the creation of a new account, and the only required field is the external ID. For service accounts, all accounts can be grouped into one service customer.

<div class="toolbar-note">
Accounting currency is part of Tatum's built-in compliance engine. It's enabled by default.
</div> 

Every account should have a properly set up accounting currency. It should be the FIAT currency of the country where the accounting is performed. For example, for an exchange in Germany, accounting should be in EUR, so the accounting currency is EUR. More details are available in the [API Reference](https://docs.tatum.io/rest/virtual-accounts/account-services).

<div class='tabbed-code-blocks'>
```Request
curl --location --request POST 'https://api-eu1.tatum.io/v3/ledger/account' \
--header 'x-api-key: YOUR_API_KEY' \
--header 'Content-Type: application/json' \
--data-raw '{
    "currency": "BTC",
    "accountingCurrency": "EUR",
    "customer": {
      "externalId": "SERVICE_CUSTOMER_EXTERNAL_ID"
    }
}'
```
```Response
{
    "currency": "BTC",
    "active": true,
    "balance": {
        "accountBalance": "0",
        "availableBalance": "0"
    },
    "frozen": false,
    "accountingCurrency": "EUR",
    "customerId": "5fb7bdf6e96d9ab593e191a6"
    "id": "5fb7bdf6e96d9ab593e191a5"
}
```
</div> 

---

## New user registration

After the configuration has been completed and the exchange is live, users register in the ecosystem. 

![NewUser.jpg](https://stoplight.io/api/v1/projects/cHJqOjExNjE5Mw/images/UbOfPUiwqas)

### Creating user accounts

When a new user signs up for the application, [ledger accounts must be created](https://docs.tatum.io/rest/virtual-accounts/create-new-account) for them. Every user should have an account for every supported blockchain asset in the exchange. Every account should be created with the external ID of the customer. This makes it possible to list all accounts for the specific customer. An account should also have the accounting currency set up correctly.

<div class="toolbar-note">
The customer's external ID should be a unique identifier of the user in your application, e.g., your ID or the hash.
</div>

Every account in the private ledger must have a defined currency. The currency cannot be changed in the future. During the creation of the account, the xpub from the blockchain wallet must be entered. This is the first connection between the blockchain and the ledger.

<div class="toolbar-note">
You will use the xpubs from the wallets generated during the application setup.
</div>

<div class='tabbed-code-blocks'>
```Request
curl --location --request POST 'https://api-eu1.tatum.io/v3/ledger/account' \
--header 'x-api-key: YOUR_API_KEY' \
--header 'Content-Type: application/json' \
--data-raw '{
    "currency": "BTC",
    "xpub": "tpubDE8GQ9vAXpwkp37PCCRUpCoeShpC4WiCcACxh8r3nnKjfRPRqw3w58EgkfNiBy1MaRqX1oAAxwAxauEUG7vWupSh5m15znGy7vE7aE6CWzb",
    "accountingCurrency": "EUR",
    "customer": {
      "externalId": "SERVICE_CUSTOMER_EXTERNAL_ID"
    }
}'
```
```Response
{
    "currency": "BTC",
    "active": true,
    "balance": {
        "accountBalance": "0",
        "availableBalance": "0"
    },
    "frozen": false,
    "accountingCurrency": "EUR",
    "xpub": "tpubDE8GQ9vAXpwkp37PCCRUpCoeShpC4WiCcACxh8r3nnKjfRPRqw3w58EgkfNiBy1MaRqX1oAAxwAxauEUG7vWupSh5m15znGy7vE7aE6CWzb",
    "customerId": "5fb7bdf6e96d9ab593e191a6"
    "id": "5fb7bdf6e96d9ab593e191a5"
}
```
</div>

### Generating a blockchain deposit address for the account

Once the account is created, it is not yet synchronized with the blockchain. There is no blockchain address connected to it, only a blockchain wallet, from which addresses will be chosen. To connect a specific address, you need to generate the account's address using the off-chain method [Generate address for the account](https://docs.tatum.io/rest/virtual-accounts/create-new-deposit-address).

<div class='tabbed-code-blocks'>
```Request
curl --location --request POST 'https://api-eu1.tatum.io/v3/offchain/account/5fb7bdf6e96d9ab593e191a5/address' \
--header 'x-api-key: YOUR_API_KEY'
```
```Response
{
    "xpub": "tpubDE8GQ9vAXpwkp37PCCRUpCoeShpC4WiCcACxh8r3nnKjfRPRqw3w58EgkfNiBy1MaRqX1oAAxwAxauEUG7vWupSh5m15znGy7vE7aE6CWzb",
    "derivationKey": 1,
    "address": "mgSXLa5sJHvBpYTKZ62aW9z2YWQNTJ59Zm",
    "currency": "BTC"
}
```
</div>

The result is a blockchain address that has been connected to the ledger account. Any incoming blockchain transaction to this address will be automatically synchronized to the private ledger.

### Enabling notifications for incoming blockchain transactions

It is possible to [enable webhook notifications for every incoming transaction](https://docs.tatum.io/rest/subscriptions/create-new-subscription) to the account. This notification is fired as an HTTP POST request with a JSON body and contains fields like the transaction amount, currency, and account of the incoming transaction. Users should see somewhere in their wallet page that there are pending incoming transactions - their crypto deposits.

<div class='tabbed-code-blocks'>
```Request
curl --location --request POST 'https://api-eu1.tatum.io/v3/subscription' \
--header 'x-api-key: YOUR_API_KEY' \
--header 'Content-Type: application/json' \
--data-raw '{
	"attr": {
        "id": "5fb7bdf6e96d9ab593e191a5",
        "url": "https://webhook.site/"
    },
	"type": "ACCOUNT_INCOMING_BLOCKCHAIN_TRANSACTION"
}'
```
```Response
{
    "id": "5fef7ab888eef2e9e4927913"
}
```
</div>

The result is the ID of the subscription for later deletion.

---

## User journey

The user has successfully signed up, accounts and deposit addresses for all supported blockchains have been created. Let's take a look at what they should see in the application itself.

![UserJourney.jpg](https://stoplight.io/api/v1/projects/cHJqOjExNjE5Mw/images/tJiSvtZlRSE)

### List of the user's accounts with balances

When the user signs in to the application, a [list of their accounts](https://docs.tatum.io/rest/virtual-accounts/list-all-customer-accounts) should be visible. We can obtain all accounts for one user using his customer ID.

<div class="toolbar-note">
The account's balance is available in the accounts list by default and does not have to be queried separately. There are two types of balances: 
- The **account balance** is the total balance of the account without any pending deposits or other trade blockages. 
- The **available balance** is the balance that can be used on a trade or other types of transactions.
</div>

<div class='tabbed-code-blocks'>
```Request
curl --location --request GET 'https://api-eu1.tatum.io/v3/ledger/account/customer/5fb7bdf6e96d9ab593e191a6?pageSize=50' \
--header 'x-api-key: YOUR_API_KEY'
```
```Response
[
  {
    "currency": "BTC",
    "active": true,
    "balance": {
        "accountBalance": "0.001",
        "availableBalance": "0.001"
    },
    "frozen": false,
    "accountingCurrency": "EUR",
    "xpub": "tpubDE8GQ9vAXpwkp37PCCRUpCoeShpC4WiCcACxh8r3nnKjfRPRqw3w58EgkfNiBy1MaRqX1oAAxwAxauEUG7vWupSh5m15znGy7vE7aE6CWzb",
    "customerId": "5fb7bdf6e96d9ab593e191a6"
    "id": "5fb7bdf6e96d9ab593e191a5"
  }
]
```
</div>

### List of recent transactions in any account

Usually, the [last transactions that happened in any of the accounts](https://docs.tatum.io/rest/virtual-accounts/find-transactions-for-a-customer-across-all-of-the-customer-s-accounts) are presented as well.


<div class='tabbed-code-blocks'>
```Request
curl --location --request POST 'https://api-eu1.tatum.io/v3/ledger/transaction/customer?pageSize=50' \
--header 'x-api-key: YOUR_API_KEY' \
--header 'Content-Type: application/json' \
--data-raw '{
    "id": "5fb7bdf6e96d9ab593e191a6"
}'
```
```Response
[
    {
        "amount": "0.001",
        "operationType": "DEPOSIT",
        "currency": "BTC",
        "transactionType": "CREDIT_DEPOSIT",
        "accountId": "5fb7bdf6e96d9ab593e191a5",
        "anonymous": false,
        "reference": "c81a23dd-e162-4e0b-b0ff-e470c64f7b88",
        "txId": "cd63e729ecc513bc22e8632b69a433126d5621c5f11047f34a0cbe144ce9aaac",
        "address": "n22crsZTASULKtLqg3XzD1NwV1HnfrQpcd",
        "marketValue": {
            "currency": "EUR",
            "source": "CoinGecko",
            "sourceDate": 1606164328453,
            "amount": "15.49693999999999884022"
        },
        "created": 1606164532855
    }
]
```
</div>

The user can see the [details of the account](https://docs.tatum.io/rest/virtual-accounts/get-account-by-id) and [transactions connected only to this account](https://docs.tatum.io/rest/virtual-accounts/find-transactions-for-account). 

### Obtaining the deposit address for an account

Usually, it is good to [display the blockchain addresses](https://docs.tatum.io/rest/virtual-accounts/get-all-deposit-addresses-for-account) connected to this account to send a blockchain transaction to the exchange.

<div class='tabbed-code-blocks'>
```Request
curl --location --request GET 'https://api-eu1.tatum.io/v3/offchain/account/5fb7bdf6e96d9ab593e191a5/address' \
--header 'x-api-key: YOUR_API_KEY'
```
```Response
{
    "txId": "97bc1c3c23b179cba837e4060c0d07aa399f7ac7d34d91a7405cb5f801b93c8a",
    "id": "5fbc208c99a159b4e9120c30",
    "completed": true
}
```
</div>

### Withdrawing funds from the exchange to the blockchain

For every blockchain, there is a specific [API call](https://docs.tatum.io/rest/virtual-accounts/send-bitcoin-from-tatum-account-to-address) for performing withdrawals. We will cover Bitcoin in this section, but it works similarly in others. 

<div class='tabbed-code-blocks'>
```Request
curl --location --request POST 'https://api-eu1.tatum.io/v3/offchain/bitcoin/transfer' \
--header 'x-api-key: YOUR_API_KEY' \
--header 'Content-Type: application/json' \
--data-raw '{
    "senderAccountId": "5fbaca3001421166273b3779",
    "address": "mpTwPdF8up9kidgcAStriUPwRdnE9MRAg7",
    "amount": "0.00195",
    "fee": "0.00005",
    "mnemonic": "behave season capable ridge repair creek seat rescue potato divide fox expose wrestle asthma luggage rack afford pistol ridge modify direct picnic magic cannon",
    "xpub": "tpubDF1sYuDKCJr6mGietaVzqGmF2dqdKVBa1DtLJGBX8HXhtHZPv5UBz3WNWU22tiVAYSjqfvfFxMnDs3vM11iQrKej6dq33UCevhiPW9EQAS2"
}'
```
```Response
{
    "txId": "97bc1c3c23b179cba837e4060c0d07aa399f7ac7d34d91a7405cb5f801b93c8a",
    "id": "5fbc208c99a159b4e9120c30",
    "completed": true
}
```
</div>

You can see that the required parameters are the ledger account's identifier, information about the blockchain wallet, the recipient blockchain address, the amount to be sent, and the blockchain fee to be paid.

---

## Trading

Last but not least is the ability to perform trades. Bear in mind that this section describes exchanges like Binance or Coinbase, where the exchange provider is responsible for every trading pair's liquidity that the exchange supports.

<div class="toolbar-note">
You can imagine the liquidity of the pair as how many open buy/sell trades are present on the Order book and how much volume is traded throughout the day. The higher the liquidity, the more precise the chart, and the higher the accuracy of the price of the asset.
</div>

![Trading.jpg](https://stoplight.io/api/v1/projects/cHJqOjExNjE5Mw/images/vlVRcTu4F6M)

### Opening a new trade

Every user can open an unlimited number of trades. Trades can be either BUY or SELL and are connected to the specific trading pair. Trading pairs are created automatically with the first opened trade.

<div class="toolbar-note">
The trading pair consists of 2 assets. Let's discuss the BTC/ETH pair. The first asset is Bitcoin, and the second is Ethereum. When you open a new BUY trade with the pair BTC/ETH, you want to buy Bitcoin for your Ethereum.
</div>

Every trade must have a price and an amount of the asset you want to trade. In Tatum, every trade is a LIMIT trade by default, and you have to wait until the price hits your target.

<div class="toolbar-note">
You want to buy 1 Bitcoin for 40 Ethereum. You must open a BUY BTC/ETH trade with the price set to 40 and the amount set to 1. Your trade remains open until an opposite trade is opened. This opposite trade should be SELL BTC/ETH with the price set to 40 or below and the amount set to 1 or more.
</div>

<div class="toolbar-note">
MARKET trades can be executed by setting the price above or below the highest BUY or lowest SELL.
</div>

When you open a trade, there are two accounts you must enter:
- the ledger account with the currency of the first asset in the trading pair
- the ledger account with the currency of the second asset in the trading pair
The traded amount will be blocked and debited from one of these accounts based on the trade type, and the other account will be credited with the traded asset when the trade is fulfilled.

<div class="toolbar-note">
Let's buy 1 BTC for 40 ETH in BTC/ETH pair. Account 1 is BTC, and account 2 is ETH. 40 ETH will be blocked from the ETH account and then transferred to the ETH account of the opposite SELL trade. 1 BTC will be credited to the BTC account from the BTC account of the opposite SELL trade.
</div>

Every trade must be filled and closed at some point. It is possible to fill only the part of the trade. There might be trade open for selling 1 BTC for 40 ETH, but an opposing trade was executed to buy only 0.5 BTC. Your actual trade will be partially filled for 0.5 BTC and stays open until the rest of the 0.5 BTC is closed.

<div class="toolbar-note">
A ledger transaction is performed for every partial fill of a trade, and assets are transferred to the ledger accounts associated with the trade. Also, the blockage is decreased accordingly.
</div>

Enough of the theory, [let's open a BUY trade](https://docs.tatum.io/rest/exchange/store-buy-sell-trade).

<div class='tabbed-code-blocks'>
```Request
curl --location --request POST 'https://api-eu1.tatum.io/v3/trade' \
--header 'x-api-key: YOUR_API_KEY' \
--header 'Content-Type: application/json' \
--data-raw '{
    "type": "BUY",
    "price": "40",
    "amount": "1",
    "pair": "BTC/ETH",
    "currency1AccountId": "5f914e372e47312bc56d8d3d",
    "currency2AccountId": "5f914e0a2e47312bc56d8d3b"
}'
```
```Response
{
  "id": "5e68c66581f2ee32bc354087"
}
```
</div>

The response is the ID of this open trade. When you [list the blockages](https://docs.tatum.io/rest/virtual-accounts/get-blocked-amounts-in-an-account) on the ETH account, you will see that there is a blockage of 40 ETH. Additional information is present in the blockage, such as the trade ID as a description.

<div class='tabbed-code-blocks'>
```Request
curl --location --request GET 'https://api-eu1.tatum.io/v3/ledger/account/block/5f914e0a2e47312bc56d8d3b?pageSize=10' \
--header 'x-api-key: YOUR_API_KEY' 
```
```Response
[
    {
        "amount": "40",
        "type": "TRADE",
        "description": "5ff0d13a5f4813e187a2a498",
        "accountId": "5e68c66581f2ee32bc354087",
        "id": "5ff0d13a5f4813e187a2a499"
    }
]
```
</div>


No transaction has been performed yet, only the blockage.

### Listing open trades

When you have open trades that are not yet filled, you can list them using the List active trades endpoint. You can [list open BUY](https://docs.tatum.io/rest/exchange/list-all-active-buy-trades) and [SELL](https://docs.tatum.io/rest/exchange/list-all-active-sell-trades) trades using two different endpoints. Keep in mind that by using these endpoints, you list either all open trades across all pairs or only trades for a specific account by providing the account's ID as a query parameter.

<div class='tabbed-code-blocks'>
```Request
curl --request POST \
  --url https://api-eu1.tatum.io/v3/trade/buy \
  --header 'Content-Type: application/json' \
  --header 'x-api-key: ' \
  --data '{
  "id": "5e68c66581f2ee32bc354087",
  "customerId": "5e68c66581f2ee32bc354087",
  "pageSize": 10,
  "offset": 0,
  "pair": "BTC/EUR",
  "count": true,
  "tradeType": "FUTURE_BUY",
  "amount": [
    {
      "op": "gte",
      "value": "1.5"
    }
  ],
  "fill": [
    {
      "op": "gte",
      "value": "1.5"
    }
  ],
  "price": [
    {
      "op": "gte",
      "value": "1.5"
    }
  ],
  "created": [
    {
      "op": "gte",
      "value": "1.5"
    }
  ],
  "sort": [
    "PRICE_ASC"
  ]
}'
```
```Response
[
  {
    "id": "7c21ed165e294db78b95f0f1",
    "type": "BUY",
    "price": "8650.4",
    "amount": "15000",
    "pair": "BTC/EUR",
    "isMaker": true,
    "fill": "1500",
    "feeAccountId": "7c21ed165e294db78b95f0f1",
    "fee": 1.5,
    "currency1AccountId": "7c21ed165e294db78b95f0f1",
    "currency2AccountId": "7c21ed165e294db78b95f0f1",
    "created": 1585170363103,
    "attr": {
      "sealDate": 1572031674384,
      "percentBlock": 1.5,
      "percentPenalty": 1.5
    }
  }
]
```
</div>

<div class="toolbar-note">
The fee can be executed by providing a fee and feeAccountId property when opening a trade. It is always the first currency of the trading pair and is set up as a percent.
</div>

### Listing closed trades

When the trade is closed, there are two ledger transactions. The first one is between accounts of the trading pair's first currency. The second one is between accounts of the second currency in the trading pair. In BTC/ETH example, there is a ledger-to-ledger transaction of BTC and a ledger-to-ledger ETH transaction. Blockages on both trades are deleted, the trade is not active anymore, and it is moved to historical trades.

To see the list of closed trades, you can call the [List all closed trades](https://docs.tatum.io/rest/exchange/list-all-historical-trades) endpoint.

<div class='tabbed-code-blocks'>
```Request
curl --request POST \
  --url https://api-eu1.tatum.io/v3/trade/history \
  --header 'Content-Type: application/json' \
  --header 'x-api-key: ' \
  --data '{
  "id": "5e68c66581f2ee32bc354087",
  "customerId": "5e68c66581f2ee32bc354087",
  "pageSize": 10,
  "offset": 0,
  "pair": "BTC/EUR",
  "count": true,
  "types": [
    "FUTURE_BUY"
  ],
  "amount": [
    {
      "op": "gte",
      "value": "1.5"
    }
  ],
  "fill": [
    {
      "op": "gte",
      "value": "1.5"
    }
  ],
  "price": [
    {
      "op": "gte",
      "value": "1.5"
    }
  ],
  "created": [
    {
      "op": "gte",
      "value": "1.5"
    }
  ],
  "sort": [
    "PRICE_ASC"
  ]
}' 
```
```Response
[
  {
    "id": "7c21ed165e294db78b95f0f1",
    "type": "BUY",
    "price": "8650.4",
    "amount": "15000",
    "pair": "BTC/EUR",
    "isMaker": true,
    "fill": "1500",
    "feeAccountId": "7c21ed165e294db78b95f0f1",
    "fee": 1.5,
    "currency1AccountId": "7c21ed165e294db78b95f0f1",
    "currency2AccountId": "7c21ed165e294db78b95f0f1",
    "created": 1585170363103,
    "attr": {
      "sealDate": 1572031674384,
      "percentBlock": 1.5,
      "percentPenalty": 1.5
    }
  }
]
```
</div>

You can see that the only difference is the fill property, which is the same as the trade amount.

‌That's it. There are many more things to enhance and features to implement, but this should be a good start for you and your exchange.

If you'd like to learn how to implement fiat currencies and securely work with private keys using Tatum KMS in your exchange, please continue to our **Crypto exchange part 2** workshop below.

<div youtube-url="https://www.youtube.com/watch?time_continue=209&v=CGgyyTTv0yw&feature=emb_title&ab_channel=Tatum"></div>



