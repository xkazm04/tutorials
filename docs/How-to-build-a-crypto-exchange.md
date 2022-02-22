# How to build a crypto exchange

*Build a backend for your exchange in 30 minutes*

---

## Introduction

A crypto exchange is a web application where users can trade their crypto assets. The operator of the exchange is the one who owns the private keys to all of the user's crypto assets - in this case, we are talking about a custodial exchange leveraging a custodial wallet.

https://www.youtube.com/watch?time_continue=1&v=pVBNvh04ixo&feature=emb_title

<!-- theme: info -->
> A custodial wallet is a wallet where a third party holds the private keys, not the crypto assets owner. The provider has full control over crypto assets, while users only have permission to send or receive payments.

Every exchange must have wallets for every crypto asset it supports. Every user of the exchange must obtain accounts for every asset they are trading. The exchange operator defines the trading pairs that can be traded by users and usually charges a fee for every trade performed.

Trades are not performed on the blockchain (on-chain), as this would be extremely slow and expensive. All the trades are only virtual transactions between user accounts.

‌ There are four logical groups of actions to create an exchange:
- **Setting up the application** - includes prerequisites like blockchain wallet creation and creating exchange service accounts for gathering fees.
- **Registration of new users in the application** - the steps that need to be executed when new users register in the exchange
- **User application journey** - what kind of actions users take while in the application
- **Trading** - enabling users to trade their assets

<!-- theme: info -->
> In the next sections, we will use Bitcoin and Ethereum as blockchains to demonstrate the functionalities.

---

## Setting up the application

This phase is a one-time step that must be done before the launch of the exchange. It involves creating the blockchain wallets your application will support or creating service fee accounts for the wallet provider.

![SetUpYourApp.jpg](https://stoplight.io/api/v1/projects/cHJqOjExNjE5Mw/images/D3NosL1IKQo)

### Creating blockchain wallets

To generate a Bitcoin wallet, you need to call a request to the Bitcoin/wallet endpoint. This is the same for Ethereum. Only the endpoint is different. The result contains two fields - mnemonic and xpub. 

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

<!-- theme: info -->
> To generate a wallet for any ERC-20 token like USDT, LINK or others, use the same call as for Ethereum. These tokens are transported on the Ethereum network and leverage the same Ethereum addresses as native Ether.

<!-- theme: warning -->
> Blockchain wallets here are created using API, which is not a secure way of generating wallets. Your private keys and mnemonics should never leave your security perimeter. To correctly and securely generate a wallet, you can use [Tatum CLI](https://github.com/tatumio/tatum-cli) from the command line or use our complex key management system, [Tatum KMS](https://github.com/tatumio/tatum-kms).  

### Generating service accounts

For every supported blockchain wallet, a service ledger account should be created. These accounts will be used to gather fees from the trading of the users. A fee will be charged for every trade performed, and the fee will be transferred to the account.

‌Every account can belong to a specific customer in Tatum. Within Tatum, a customer is an entity containing information about a user of your application, such as the customer's country of residence, accounting currency, etc. The customer is only created during the creation of a new account, and the only required field is the external ID. For service accounts, all accounts can be grouped into one service customer.

<!-- theme: info -->
> Accounting currency is part of Tatum's built-in compliance engine. It's enabled by default.























































































































