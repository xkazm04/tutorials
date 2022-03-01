# How to build a non-custodial wallet

*Build a backend for a non-custodial wallet in 30 minutes*

---

A wallet is a simple application where users can sign in, generate accounts in different blockchains, and send and receive payments. They are owners of the private keys for every blockchain address, and the wallet provider has no access to them.

![non_custodial_wallet_infografics___-20.jpg](https://stoplight.io/api/v1/projects/cHJqOjExNjE5Mw/images/THdyQVqdQI0)

<div class="toolbar-note">
A wallet can store any assets. These could be customer ERC-20 tokens, standard blockchain assets, or government-issued currencies.
</div>

![non_custodial_wallet_infografics___-21.jpg](https://stoplight.io/api/v1/projects/cHJqOjExNjE5Mw/images/i3xHOEFaHYc)

In this tutorial, you will learn how to build a custodial wallet backend on Tatum. All you will have to do after this tutorial is to create a nice frontend for your application.

<div class="toolbar-note">
A non-custodial wallet is the type of wallet where users have full control over their funds and the associated private key.
</div>

There are two logical groups of actions that must be done to create a wallet:
- **Registration of new users in the application** - what must be done during the registration phase
- **** - what kind of actions users take while in the application

![non_custodial_wallet_infografics___-22.jpg](https://stoplight.io/api/v1/projects/cHJqOjExNjE5Mw/images/3BQCcEwJpBg)

---

## New user registration

When a new user signs up for the application, [blockchain wallets](https://docs.tatum.io/rest/blockchain/generate-bitcoin-wallet) and [ledger accounts must be created](https://docs.tatum.io/rest/virtual-accounts/create-new-account). Every user should have their own blockchain wallet. Every account should be created with the external ID of the customer. This makes it possible to list all accounts for the specific customer.

![non_custodial_wallet_infografics___-23.jpg](https://stoplight.io/api/v1/projects/cHJqOjExNjE5Mw/images/qVO5r2rjAwE)


When you create accounts, blockchain [addresses should be created](https://docs.tatum.io/rest/virtual-accounts/create-new-deposit-address) and connected to the ledger accounts. This starts the process of automatic synchronization of incoming blockchain transactions. It is possible to [enable webhook notifications for every incoming transaction](https://docs.tatum.io/rest/subscriptions/create-new-subscription) to the account.

<div class="toolbar-tip">
[How to automatically scan blockchain addresses and set up webhook notifications](https://docs.tatum.io/guides/ledger-and-off-chain/how-to-automatically-scan-blockchain-addresses-and-set-up-webhook-notifications)
</div>

---

## User journey

When the user signs into the application, a [list of their accounts](https://docs.tatum.io/rest/virtual-accounts/list-all-customer-accounts) should be visible. Usually, the [last transactions that occurred in any of the accounts](https://docs.tatum.io/rest/virtual-accounts/find-transactions-for-a-customer-across-all-of-the-customer-s-accounts) are presented as well.

![non_custodial_wallet_infograficS-24.jpg](https://stoplight.io/api/v1/projects/cHJqOjExNjE5Mw/images/Q1VwnariEFQ)

<div class="toolbar-note">
The account's balance is available in the accounts list by default and does not have to be queried separately.
</div>

The user can see the [details of the account](https://docs.tatum.io/rest/virtual-accounts/get-account-by-id) and [transactions connected only to this account](https://docs.tatum.io/rest/virtual-accounts/find-transactions-for-account). Usually, it is good to [display the blockchain addresses](https://docs.tatum.io/rest/virtual-accounts/get-all-deposit-addresses-for-account) connected to this account to send a blockchain transaction.

Finally, there should be the option to [send the transaction from the account to a blockchain](https://docs.tatum.io/rest/virtual-accounts/send-bitcoin-from-tatum-account-to-address) address.

![non_custodial_wallet_infografics___-25.jpg](https://stoplight.io/api/v1/projects/cHJqOjExNjE5Mw/images/cvBtehCERFQ)


<div class="toolbar-note">
When developing a non-custodial wallet, the [Bitcoin transaction](https://docs.tatum.io/rest/virtual-accounts/send-bitcoin-from-tatum-account-to-address) should have the **attr** field present in the request body. This will be used as **a change address** for the rest of the unspent funds, and it **must be the first address** connected to the ledger account.
</div>

So there you have it. There are many more things that you can enhance in your app and many more features to implement, but this should be a reasonable start for you and your wallet.



