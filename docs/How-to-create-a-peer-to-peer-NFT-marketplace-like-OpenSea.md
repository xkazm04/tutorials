# How to create a peer-to-peer NFT marketplace like OpenSea

*Open Sea fixed-priced asset listing in three API calls.*

---

Peer-to-peer NFT marketplaces like OpenSea allow users to create NFTs with metadata (pictures, videos, songs, 3D art, etc.) and post listings to sell them to one another. OpenSea then takes a percentage of each sale. 

If you want to build a backend for your own peer-to-peer NFT marketplace, you’ll definitely want to implement the same functionality. 

[Creating NFTs](../token/ZG9jOjI4ODY3ODE4-how-to-create-nft) has always been super simple with Tatum. But now, you can also create NFT marketplaces and listings using just a few API calls. When the NFT is sold, the creator is automatically paid, the NFT is instantly transferred to the buyer, and you, the owner of the marketplace, automatically receive a percentage of the transaction. Everything happens on the blockchain, so there's no need to create a complex, application-level backend to run the listings.



<!-- theme: info -->
>With this type of marketplace, your users can create listings to sell ERC-721 or ERC-1155 tokens. These tokens can be purchased with the native assets of the given blockchain (e.g. ETH on Ethereum or MATIC on Polygon) or any ERC-20 token available on your blockchain of choice.
>
>Currently supported blockchains are Ethereum, Celo, Polygon, Binance Smart Chain, and Harmony.ONE.
>
>All smart contracts are available [here](https://github.com/tatumio/smart-contracts/blob/master/contracts/tatum/nft/MarketplaceListing.sol).


---
## Deploy your own marketplace

The first step is to [create your own marketplace smart contract](../token/b3A6Mjg1MjQ5NjY-create-nft-marketplace). This is a one-time operation, and the marketplace you deploy will be used for every listing in your application. In this example, we'll deploy our marketplace on the Polygon network.

**Request example**

```json
curl --request POST \
  --url https://api-eu1.tatum.io/v4/token/MATIC/marketplace \
  --header 'Content-Type: application/json' \
  --header 'x-api-key: ' \
  --data '{
  "feeRecipient": "0x687422eEA2cB73B5d3e242bA5456b782919AFc85",
  "marketplaceFee": 150,
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

As you can see in the example above, you must enter a value in the `marketplaceFee` field. This is a percentage of the price that will be charged to the buyer when they make a purchase. The value represents the percentage multiplied by 100, so a 1.5% fee is written here as 150. This percentage is the same for all sales made on your marketplace.

The `feeRecipient` field represents the address where the fees will be sent, i.e. your address as the marketplace owner.

The `txId` field in the response body contains the contract address. 

---

## Create a fixed price asset listing

Once the marketplace is ready, you can enable listings for your users. The NFT asset must be present at the seller's address before they create an NFT listing. The seller can choose the price of the asset, whether they are selling an ERC-721 or ERC-1155  token, and whether they are selling it for ERC-20 tokens or for the native currency of the given blockchain (in our case MATIC).

Every listing must have a unique `listingId`. This parameter identifies the listing and is used for performing other operations.


<!-- theme: info -->
>To use an ERC-20 token as a listing currency, the seller should add an `erc20Address` parameter to the call with the address of the smart contract of the ERC-20 token that is used for the listing.

[The following API call](../token/b3A6Mjg1MjQ5Njc-sell-asset-on-marketplace) will create a fixed-price NFT listing on your marketplace:

**Request example**
```json

curl --request POST \
  --url https://api-eu1.tatum.io/v4/token/BSC/marketplace/sell \
  --header 'Content-Type: application/json' \
  --header 'x-api-key: ' \
  --data '{
  "contractAddress": "0x687422eEA2cB73B5d3e242bA5456b782919AFc85",
  "nftAddress": "0x687422eEA2cB73B5d3e242bA5456b782919AFc85",
  "seller": "stringstringstringstringstringstringstring",
  "erc20Address": "0x687422eEA2cB73B5d3e242bA5456b782919AFc85",
  "listingId": "string",
  "amount": "1",
  "tokenId": "10000",
  "price": "10000",
  "isErc721": true,
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

The listing is now available in the marketplace and the seller should send the NFT asset being sold to the marketplace. The NFT can be sent using one of the following standard NFT transfer methods: [ERC-721](../token/b3A6MzA3Mjc3Mzg-transfer-nft-token) or [ERC-1155](../token/b3A6MzA4NDM4MDI-transfer-multi-token). The NFT marketplace is the recipient (i.e. the `contractAddress` in the call above).

---

## Purchasing an NFT asset from a listing

Once the token is in the marketplace, it can be purchased by any buyer with enough funds in their account - the total price consists of the price of the listing and the marketplace fee.

<!-- theme: info -->
>The listing price in the following example is 1 MATIC and the fee is 2.5%, so the buyer would have to spend 1.025 MATIC to buy the asset.


<!-- theme: info -->
>To use an ERC-20 token as a listing currency, the seller should add an `erc20Address` parameter to the call with the address of the smart contract of the ERC-20 token that is used for the listing.

[Use the following API call](../token/b3A6Mjg1MjQ5Njg-buy-asset-on-marketplace) to purchase an NFT from a listing on the marketplace:

**Request example**
```json
curl --request POST \
  --url https://api-eu1.tatum.io/v4/token/MATIC/marketplace/buy \
  --header 'Content-Type: application/json' \
  --header 'x-api-key: ' \
  --data '{
  "contractAddress": "0x687422eEA2cB73B5d3e242bA5456b782919AFc85",
  "erc20Address": "0x687422eEA2cB73B5d3e242bA5456b782919AFc85",
  "listingId": "string",
  "amount": "1",
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

After performing this operation, the NFT has been transferred to the buyer, the amount has been transferred to the seller, and the fee has been transferred to the fee recipient of the marketplace.

<!-- theme: info -->
> For an ERC-20 listing, the only difference is that the buyer [must authorize the NFT marketplace to spend his ERC-20 tokens](../token/b3A6MzA4OTMzODI-approve-spending-of-erc-20) before the actual operation.
>
> For an ERC-1155 listings, the only difference is the number of tokens that are being listed.

>You can [take a look](https://mumbai.polygonscan.com/tx/0x18117c5e8fdef378c449c06c9306150067b499f04cb8ced8c0de6189d4246f8f) at an example transaction on the Mumbai Testnet. The buyer entered an even bigger amount to buy the NFT, so the rest was returned to him.