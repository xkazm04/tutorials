# How to build a custodial wallet

*Build a bacend for custodial wallet in 30 minutes*

---

A wallet is a simple application where users can sign in, generate accounts in different blockchains, and send and receive payments. Usually, the wallet provider allows users to instantly send transactions to each other within the ecosystem and without fees.

<div class="toolbar-note">
A wallet can store any assets. These could be customer ERC-20 tokens, standard blockchain assets, or government-issued currencies.
</div>

In this tutorial, you will learn how to build a custodial wallet backend on Tatum. All you will have to do after this tutorial is to create a nice frontend for your application.

<div class="toolbar-note">
> A custodial wallet is a wallet where a third party (usually the wallet provider) holds the private keys. The provider has full control over blockchain assets, while users only have permission to send or receive payments.
</div>

There are three logical groups of actions to create a wallet:
- **Setting up the application** - this includes prerequisites like blockchain wallet creation.
- **Registration of new users in the application** - what must be done during the registration phase
- **User application journey** - what kind of actions users take while in the application

---

## Application set up

This phase is a one-time step that must be done before the launch of the wallet. It involves creating [the blockchain wallets](https://developer.tatum.io/rest/blockchain/generate-bitcoin-wallet) your application will support or creating system accounts for the wallet provider.

<div class="toolbar-tip">
[How to create a blockchain wallet]()
</div>

---


## New user registration

After the configuration is done and the application is live, users register in the ecosystem. When a new user signs up for the application, [ledger accounts must be created](https://developer.tatum.io/rest/virtual-accounts/create-new-account) for them. Every account should be created with the external ID of the customer. This makes it possible to list all accounts for the specific customer.

<div class="toolbar-tip">
The customer's external ID should be a unique identifier of the user in your application, e.g., your ID.
</div>

When you create accounts, blockchain [addresses should be created](https://developer.tatum.io/rest/virtual-accounts/create-new-deposit-address) and connected to the ledger accounts. This starts the process of automatic synchronization of the incoming blockchain transactions. It is possible to [enable webhook notifications for every incoming transaction](https://developer.tatum.io/rest/subscriptions/create-new-subscription) to the account.

<div class="toolbar-tip">
[How to automatically scan blockchain addresses and set up webhook notifications]()
</div>

---

## User journey

When the user signs into the application, a [list of their accounts](https://developer.tatum.io/rest/virtual-accounts/list-all-customer-accounts) should be visible. Usually, the [last transactions that occurred in any of the accounts](https://developer.tatum.io/rest/virtual-accounts/list-all-customer-accounts) are presented as well.

<div class="toolbar-note">
The account's balance is available in the accounts list by default and does not have to be queried separately.
</div>

The user can see the [details of the account](https://developer.tatum.io/rest/virtual-accounts/get-account-by-id) and [the transactions connected only to this account](https://developer.tatum.io/rest/virtual-accounts/find-transactions-for-account). Usually, it is good to [display the blockchain addresses](https://developer.tatum.io/rest/virtual-accounts/get-all-deposit-addresses-for-account) connected to this account to send a blockchain transaction.

Finally, there should be the option to [send the transaction from the account to a blockchain](https://developer.tatum.io/rest/virtual-accounts/send-bitcoin-from-tatum-account-to-address) address.

<div class="toolbar-tip">
[How to send a Bitcoin blockchain transaction from a virtual account]()
</div>

Depending on the requirements, it is possible to enable transactions within the ecosystem to be exclusively performed inside the application. These are [ledger transactions](https://developer.tatum.io/rest/virtual-accounts/send-payment), which are feeless and instant.

<div class="toolbar-tip">
[How to send feeless instant transaction]()
</div>

And that's all there is to it. There are many more ways to enhance your app and many more features to implement, but this should be a reasonable start for you and your wallet.































































































































































