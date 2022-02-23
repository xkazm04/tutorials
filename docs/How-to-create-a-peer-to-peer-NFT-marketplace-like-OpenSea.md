# How to create a peer-to-peer NFT marketplace like OpenSea

*Open Sea fixed-priced asset listing in three API calls.*

---

Peer-to-peer NFT marketplaces like OpenSea allow users to create NFTs with metadata (pictures, videos, songs, 3D art, etc.) and post listings to sell them to one another. OpenSea then takes a percentage of each sale. 

If you want to build a backend for your own peer-to-peer NFT marketplace, you’ll definitely want to implement the same functionality. 

[Creating NFTs]() has always been super simple with Tatum. But now, you can also create NFT marketplaces and listings using just a few API calls. When the NFT is sold, the creator is automatically paid, the NFT is instantly transferred to the buyer, and you, the owner of the marketplace, automatically receive a percentage of the transaction. Everything happens on the blockchain, so there's no need to create a complex, application-level backend to run the listings.

<div class="toolbar-note">
With this type of marketplace, your users can create listings to sell ERC-721 or ERC-1155 tokens. These tokens can be purchased with the native assets of the given blockchain (e.g. ETH on Ethereum or MATIC on Polygon) or any ERC-20 token available on your blockchain of choice.
</div>
Currently supported blockchains are:
- Ethereum
- Celo
- Polygon
- Klaytn
- Binance Smart Chain
- Harmony.ONE
All smart contracts are available [here](https://github.com/tatumio/smart-contracts/blob/master/contracts/tatum/nft/MarketplaceListing.sol).


---
## Import required libraries

If you are using our JavaScript SDK, the first step is to import all of the required JS libraries for the functions we will be using.

<div class='tabbed-code-blocks'>
```JavaScript
import { 
    deployMarketplaceListing,
    sendMarketplaceApproveErc20Spending,
    sendMarketplaceBuyListing,
    sendMarketplaceCreateListing,
    sendCeloSmartContractReadMethodInvocationTransaction,
    sendAuctionApproveNftTransfer
} from '@tatumio/tatum';
```
</div>

## Deploy your own marketplace

The first step is to [create your own marketplace smart contract](https://developer.tatum.io/rest/smart-contracts/create-nft-marketplace). This is a one-time operation, and the marketplace you deploy will be used for every listing in your application. In this example, we'll deploy our marketplace on the Celo blockchain.


<div class='tabbed-code-blocks'>
```SDK
const body = new DeployMarketplaceListing()
    body.fromPrivateKey = '0xa488a82b8b57c3ece4307525741fd8256781906c5fad948b85f1d63000948236'
    body.feeRecipient = '0x8cb76aEd9C5e336ef961265c6079C14e9cD3D2eA'
    body.marketplaceFee = 150
    body.feeCurrency = Currency.CUSD
    body.chain = Currency.CELO
    const test = await deployMarketplaceListing(true, body, 'https://alfajores-forno.celo-testnet.org')
    console.log(test)
```
```REST API call with KMS
curl --request POST \
  --url https://api-eu1.tatum.io/v3/blockchain/marketplace/listing \
  --header 'content-type: application/json' \
  --header 'x-api-key: REPLACE_KEY_VALUE' \
  --data '{
    "feeRecipient": "0x8cb76aEd9C5e336ef961265c6079C14e9cD3D2eA",
    "marketplaceFee": 150,
    "chain": "CELO",
    "signatureId": "26d3883e-4e17-48b3-a0ee-09a3e484ac83",
    "index": 0,
    "nonce": 1,
    "fee": {
       "gasLimit": "40000",
       "gasPrice": "20"
       }
    }'
```
```REST API call with Private key
curl --request POST \
  --url https://api-eu1.tatum.io/v3/blockchain/marketplace/listing \
  --header 'content-type: application/json' \
  --header 'x-api-key: REPLACE_KEY_VALUE' \
  --data '{
    "feeRecipient": "0x8cb76aEd9C5e336ef961265c6079C14e9cD3D2eA",
    "marketplaceFee": 150,
    "chain": "CELO",
    "fromPrivateKey": "0xa488a82b8b57c3ece4307525741fd8256781906c5fad948b85f1d63000948236"
    "fee": {
       "gasLimit": "40000",
       "gasPrice": "20"
       }
    }'
```
</div>


The body of the API request should contain the following parameters:
chain - The blockchain on which the marketplace smart contract will be deployed.
`fromPrivateKey` - The private key for the address that will pay for the deployment of the marketplace smart contract.
`feeRecipient` - The address where the fees will be sent, i.e. your address as the marketplace owner.
`marketplaceFee` - This is a percentage of the price that will be charged to the buyer when they make a purchase. The value is the percentage charged * 100, so a 1.5% fee is written as 150. This percentage is the same for all sales made on your marketplace.
`feeCurrency` - The currency in which the gas fee for the operation will be paid (only for Celo).
`fee` - The gas price and limit for the gas fee to pay for the operation. If this fee is absent, the fee will be calculated automatically.

**Response example**
```json
{
    "txId": "0x9ff62d44abaf65018081a6511c84ca8f89d7575d2a1ea058e93c1b7d57ff1807"
}
```

The response is a transaction ID from which we can [obtain the contract address](https://developer.tatum.io/rest/smart-contracts/get-contract-address-from-transaction). 

---

## Create a fixed price asset listing

Once the marketplace is ready, you can enable listings for your users. The NFT asset must be present at the address of the seller before they create an NFT listing. The seller can choose the price of the asset, whether they are selling an ERC-721 or ERC-1155  token, and whether they are selling it for ERC-20 tokens or for the native currency of the given blockchain (in our case CELO).

Every listing must have a unique `listingId`. This parameter identifies the listing and is used for performing other operations.

<!-- theme: info -->
>To use an ERC-20 token as a listing currency, the seller should add an `erc20Address` parameter to the call with the address of the smart contract of the ERC-20 token that is used for the listing.

[The following API call](https://developer.tatum.io/rest/smart-contracts/sell-asset-on-the-nft-marketplace) will create a fixed-price NFT listing on your marketplace:

<div class='tabbed-code-blocks'>
```SDK
const body = new CreateMarketplaceListing()
    body.fromPrivateKey = '0xa488a82b8b57c3ece4307525741fd8256781906c5fad948b85f1d63000948236'
    body.contractAddress = '0xd093bEd4BC06403bfEABB54667B42C48533D3Fd9'
    body.nftAddress = '0x1214BEada6b25bc98f7494C7BDBf22C095FDCaBD'
    body.tokenId = '1'
    body.listingId = '1'
    body.isErc721 = true
    body.price = '1'
    body.erc20Address = '0x8cb76aEd9C5e336ef961265c6079C14e9cD3D2eA'
    body.seller = '0x48d4bA7B2698A4b89635b9a2E391152350DB740f'
    body.feeCurrency = Currency.CUSD
    body.chain = Currency.CELO;
    console.log(await sendMarketplaceCreateListing(true, body, 'https://alfajores-forno.celo-testnet.org'));
```
```REST API with KMS
curl --request POST \
  --url https://api-eu1.tatum.io/v3/blockchain/marketplace/listing/sell \
  --header 'content-type: application/json' \
  --header 'x-api-key: REPLACE_KEY_VALUE' \
  --data '{
      "chain": "CELO",
      "feeCurrency": "CELO",
      "contractAddress": "0xd093bEd4BC06403bfEABB54667B42C48533D3Fd9",
      "nftAddress": "0x1214BEada6b25bc98f7494C7BDBf22C095FDCaBD",
      "seller": "0x48d4bA7B2698A4b89635b9a2E391152350DB740f",
      "erc20Address": "0x8cb76aEd9C5e336ef961265c6079C14e9cD3D2eA",
      "listingId": "string",
      "amount": "1",
      "tokenId": "1",
      "price": "1",
      "isErc721": true,
      "signatureId": "26d3883e-4e17-48b3-a0ee-09a3e484ac83",
      "index": 0,
      "nonce": 1,
      "fee": {
       "gasLimit": "40000",
       "gasPrice": "20"
       }
      }'
```
```REST API with Private key
curl --request POST \
  --url https://api-eu1.tatum.io/v3/blockchain/marketplace/listing/sell \
  --header 'content-type: application/json' \
  --header 'x-api-key: REPLACE_KEY_VALUE' \
  --data '{
      "chain": "CELO",
      "feeCurrency": "CELO",
      "contractAddress": "0xd093bEd4BC06403bfEABB54667B42C48533D3Fd9",
      "nftAddress": "0x1214BEada6b25bc98f7494C7BDBf22C095FDCaBD",
      "seller": "0x48d4bA7B2698A4b89635b9a2E391152350DB740f",
      "erc20Address": "0x8cb76aEd9C5e336ef961265c6079C14e9cD3D2eA",
      "listingId": "string",
      "amount": "1",
      "tokenId": "1",
      "price": "1",
      "isErc721": true,
      "fromPrivateKey": "0xa488a82b8b57c3ece4307525741fd8256781906c5fad948b85f1d63000948236",
      "nonce": 1,
      "fee": {
       "gasLimit": "40000",
       "gasPrice": "20"
       }
      }'
```
</div>

The seller can choose whether they are selling an ERC-721 or ERC-1155  token, and whether they are selling it for ERC-20 tokens or for the native currency of the given blockchain (in our case CELO). Every marketplace listing must have a unique id. This ID identifies the listing and is used for performing other operations.

The body of the API request should contain the following parameters:
chain - The chain on which the listing will be created (same as in the first call).
- `fromPrivateKey` - The private key of the address that will pay for the creation of the NFT listing.
- `contractAddress` - The address of the marketplace smart contract you deployed in the previous call.
- `nftAddress` - The address of the smart contract of the NFT being sold in the listing.
- `tokenId` - The ID of the NFT to be sold in the listing.
- `listingId` - The ID of the listing to sell the NFT.
- `isErc721` - Set to "true" if you are selling an ERC-721 or equivalent token. Set to "false" if you are selling an ERC-1155 token.
- `price` - The price of the NFT being sold.
- `erc20Address` (optional) - If the seller is selling their NFT for an ERC-20 (or equivalent) token, this field should be included and contain the address of the smart contract of the ERC-20 token.
- `seller` - The address that holds the NFT being sold and to which the funds used to purchase the NFT will be sent.
- `feeCurrency` - The currency in which the gas fee for the operation will be paid (only for Celo).
- `fee` - The gas price and limit for the gas fee to pay for the operation. If this fee is absent, the fee will be calculated automatically.

<div class="toolbar-note">
To use an ERC-20 token as a listing currency, the seller should add an erc20Address parameter to the call with the address of the smart contract of the ERC-20 token that is used for the listing.
</div>

```Response
{
    "txId": "0x9ff62d44abaf65018081a6511c84ca8f89d7575d2a1ea058e93c1b7d57ff1807"
}
```

The response is a transaction ID. The listing is now available in the marketplace and the seller must now send approval for the NFT to be sold on the marketplace.

---

## Send approval for the NFT to be sold on the marketplace

Next, we must give the marketplace permission to transfer the NFT we are selling. 

<div class='tabbed-code-blocks'>
```SDK
console.log(await sendAuctionApproveNftTransfer(true, {
    fromPrivateKey: '0xa488a82b8b57c3ece4307525741fd8256781906c5fad948b85f1d63000948236',
    chain: Currency.CELO,
    contractAddress: '0xd093bEd4BC06403bfEABB54667B42C48533D3Fd9',
    isErc721: true,
    spender: '0x991dfc0db4cbe2480296eec5bcc6b3215a9b7038',
    tokenId:'1'
  }, 'https://alfajores-forno.celo-testnet.org'));
const endedAt = (await celoGetCurrentBlock()) + 9;
```
```REST API call with KMS
curl --request POST \
  --url https://api-eu1.tatum.io/v3/blockchain/auction/approve \
  --header 'content-type: application/json' \
  --header 'x-api-key: REPLACE_KEY_VALUE' \
  --data '{
    "chain": "CELO",
    "feeCurrency": "CELO",
    "contractAddress": "0xd093bEd4BC06403bfEABB54667B42C48533D3Fd9",
    "spender": "0x991dfc0db4cbe2480296eec5bcc6b3215a9b7038",
    "isErc721": true,
    "tokenId": "1",
    "signatureId": "26d3883e-4e17-48b3-a0ee-09a3e484ac83",
    "index": 0,
    "nonce": 1,
    "fee": {
       "gasLimit": "40000",
       "gasPrice": "20"
       }
     }'
```
```REST API call with Private key
curl --request POST \
  --url https://api-eu1.tatum.io/v3/blockchain/auction/approve \
  --header 'content-type: application/json' \
  --header 'x-api-key: REPLACE_KEY_VALUE' \
  --data '{
    "chain": "CELO",
    "feeCurrency": "CELO",
    "contractAddress": "0xd093bEd4BC06403bfEABB54667B42C48533D3Fd9",
    "spender": "0x991dfc0db4cbe2480296eec5bcc6b3215a9b7038",
    "isErc721": true,
    "tokenId": "1",
    "fromPrivateKey": "0xa488a82b8b57c3ece4307525741fd8256781906c5fad948b85f1d63000948236",
    "nonce": 1,
    "fee": {
       "gasLimit": "40000",
       "gasPrice": "20"
       }
     }'
```
</div>

The body of the API request should contain the following parameters:

- `fromPrivateKey` - The private key of the address that will pay for the gas fees for the approval operation.
- `chain` - The chain on which the operation will take place (the same as in the first two API requests).
- `contractAddress` - The address of the ERC-20 token used to buy the NFT from the marketplace.
- `isErc721` - Set to "true" if you are selling an ERC-721 or equivalent token. Set to "false" if you are selling an ERC-1155 token.
- `spender` - The address of the marketplace smart contract that will be transferring the token.
- `tokenId` - The ID of the NFT being sold.
- `feeCurrency` - The currency in which the gas fee for the operation will be paid (only for Celo).
- `fee` - The gas price and limit for the gas fee to pay for the operation. If this fee is absent, the fee will be calculated automatically.

---

## Approve ERC-20 for spending and for cashback

For a listing for an NFT being sold for ERC-20 tokens, the buyer [must approve the marketplace to spend his ERC-20 tokens](https://developer.tatum.io/rest/smart-contracts/approve-spending-of-erc-20) before the actual buy operation. Here is the call to approve the marketplace for spending the ERC-20 token:

<div class='tabbed-code-blocks'>
```SDK
const approve = new ApproveErc20();
    approve.contractAddress = '0x874069Fa1Eb16D44d622F2e0Ca25eeA172369bC1';
    approve.spender = '0x8cb76aEd9C5e336ef961265c6079C14e9cD3D2eA';
    approve.chain = Currency.CELO;
    approve.feeCurrency = Currency.CELO;
    approve.amount = '0.001015';
    approve.fromPrivateKey = '0x4874827a55d87f2309c55b835af509e3427aa4d52321eeb49a2b93b5c0f8edfb';
    approve.fee = {gasPrice: '5', gasLimit: '300000'};
    console.log(await sendAuctionApproveErc20Transfer(true, approve, 'https://alfajores-forno.celo-testnet.org'));
```
```REST API call with KMS
curl --request POST \
  --url https://api-eu1.tatum.io/v3/blockchain/token/approve \
  --header 'content-type: application/json' \
  --header 'x-api-key: REPLACE_KEY_VALUE' \
  --header 'x-testnet-type: SOME_STRING_VALUE' \
  --data '{
    "chain": "CELO",
    "amount": "0.001015",
    "spender": "0x687422eEA2cB73B5d3e242bA5456b782919AFc85",
    "contractAddress": "0x687422eEA2cB73B5d3e242bA5456b782919AFc85",
    "signatureId": "26d3883e-4e17-48b3-a0ee-09a3e484ac83",
    "nonce": 0,
    "feeCurrency": "CELO"
    "fee": {
       "gasLimit": "30000",
       "gasPrice": "5"
     }
    }'
```
```REST API call with Private key
curl --request POST \
  --url https://api-eu1.tatum.io/v3/blockchain/token/approve \
  --header 'content-type: application/json' \
  --header 'x-api-key: REPLACE_KEY_VALUE' \
  --header 'x-testnet-type: SOME_STRING_VALUE' \
  --data '{
    "chain": "CELO",
    "amount": "0.001015",
    "spender": "0x687422eEA2cB73B5d3e242bA5456b782919AFc85",
    "contractAddress": "0x687422eEA2cB73B5d3e242bA5456b782919AFc85",
    "fromPrivateKey": "0x4874827a55d87f2309c55b835af509e3427aa4d52321eeb49a2b93b5c0f8edfb",
    "feeCurrency": "CELO"
    "fee": {
       "gasLimit": "30000",
       "gasPrice": "5"
     }
    }'
```
</div>

The body of the API request should contain the following parameters:
- `chain` - The chain on which the transaction will take place (same as in the previous calls).
- `contractAddress` - The address of the ERC-20 smart contract to be approved for spending.
- `spender` - The address of the marketplace smart contract that will spend the ERC-20 token.
- `amount` - The amount of funds to be approved to spend. You need to approve amount to be paid for the listing + the marketplace fee.
- `fromPrivateKey` - The private key of the address that will pay for the gas fees for the approval operation.
- `feeCurrency` - The currency in which the gas fee for the operation will be paid (only for Celo).
- `fee` - The gas price and limit for the gas fee to pay for the operation. If this fee is absent, the fee will be calculated automatically.

<div class="toolbar-note">
For NFTs to payout cashback as ERC-20 tokens, this same approve ERC-20 spending endpoint must be invoked.
</div>


---

## Purchasing an NFT asset from a listing

Once the token is in the marketplace, it can be purchased by any buyer with enough funds in their account - the total price consists of the price of the listing and the marketplace fee.

<div class="toolbar-note">
The listing in the example is for 1 CELO and the fee is 1.5%, so the buyer would have to spend 1.015 CELO to buy the asset.
</div>

Use [the following API call](https://developer.tatum.io/rest/smart-contracts/buy-asset-on-the-nft-marketplace) to purchase an NFT from a listing on the marketplace:

<div class='tabbed-code-blocks'>
```SDK
const body = new InvokeMarketplaceListingOperation()
    body.fromPrivateKey = '0x4874827a55d87f2309c55b835af509e3427aa4d52321eeb49a2b93b5c0f8edfb'
    body.contractAddress = '0xd093bEd4BC06403bfEABB54667B42C48533D3Fd9'
    body.listingId = '1'
    body.amount = '1.015'
    body.feeCurrency = Currency.CELO
    body.chain = Currency.CELO;
    console.log(await sendMarketplaceBuyListing(true, body, 'https://alfajores-forno.celo-testnet.org'));
```
```REST API call with KMS
curl --request POST \
  --url https://api-eu1.tatum.io/v3/blockchain/marketplace/listing/buy \
  --header 'content-type: application/json' \
  --header 'x-api-key: REPLACE_KEY_VALUE' \
  --data '{
    "contractAddress": "0xd093bEd4BC06403bfEABB54667B42C48533D3Fd9",
    "listingId": "1",
    "amount": "1.015",
    "chain": "CELO",
    "signatureId": "26d3883e-4e17-48b3-a0ee-09a3e484ac83",
    "index": 0,
    "nonce": 1,
    "fee": {
       "gasLimit": "40000",
       "gasPrice": "20"
       }
    }'
```
```REST API call with Private key
curl --request POST \
  --url https://api-eu1.tatum.io/v3/blockchain/marketplace/listing/buy \
  --header 'content-type: application/json' \
  --header 'x-api-key: REPLACE_KEY_VALUE' \
  --data '{
    "contractAddress": "0xd093bEd4BC06403bfEABB54667B42C48533D3Fd9",
    "listingId": "1",
    "amount": "1.015",
    "chain": "CELO",
    "fromPrivateKey": "0x4874827a55d87f2309c55b835af509e3427aa4d52321eeb49a2b93b5c0f8edfb"
    "fee": {
       "gasLimit": "40000",
       "gasPrice": "20"
       }
    }'
```

The body of the API request should contain the following parameters:
- `chain` - The chain on which the transaction will take place (same as in the previous calls).
- `fromPrivateKey` - The private key of the address that will pay for the NFT and the gas fees for the approval operation.
- `contractAddress` - The address of the marketplace smart contract.
- `listingId` - The ID of the NFT listing as specified when the listing was created.
- `amount` - The price of the NFT listing + the marketplace fee to be transferred to purchase the NFT - only valid for listings in native - ETH | BSC | MATIC | CELO currencies. For ERC20 based listings, omit this value.
- `feeCurrency` - The currency in which the gas fee for the operation will be paid (only for Celo).
- `fee` - The gas price and limit for the gas fee to pay for the operation. If this fee is absent, the fee will be calculated automatically.

<div class="toolbar-note">
>To use an ERC-20 token as a listing currency, the seller should add an `erc20Address` parameter to the call with the address of the smart contract of the ERC-20 token that is used for the listing.
</div>


**Response:**
```json
{
    "txId": "0x18117c5e8fdef378c449c06c9306150067b499f04cb8ced8c0de6189d4246f8f"
}
```

The response is a transaction Id. In this operation, the NFT is transferred to the buyer, the amount is transferred to the seller, and the fee is transferred to the fee recipient of the marketplace.

<div class="toolbar-note">
You can take a look at an example transaction on the [Mumbai Testnet](https://mumbai.polygonscan.com/tx/0x18117c5e8fdef378c449c06c9306150067b499f04cb8ced8c0de6189d4246f8f). The buyer entered an even bigger amount to buy the NFT, so the rest was returned to him.
</div>

And that's it, just a few API calls and you can create a full peer-to-peer NFT marketplace for your users. For ERC-1155 listings, the only difference is the number of tokens that are being listed.
---

## Workflow tutorials

For how to work with external NFTs, Tatum royalty NFTs, and selling NFTs for native assets or ERC-20 tokens, check out our tutorials:

https://youtu.be/pJY8kF2s-mI

https://youtu.be/pl-p-6u8dyA

https://youtu.be/bBQif9IM9Fw

https://youtu.be/W1Aozjke--0
