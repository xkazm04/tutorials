# How to create a peer to peer NFT auction

---

Peer-to-peer NFT marketplaces like OpenSea allow users to create NFTs with metadata (pictures, videos, songs, 3D art, etc.) and post listings to sell them to one another. OpenSea then takes a percentage of each sale. Users can create fixed-price listings for their NFTs or put them up for auction and sell them to the highest bidder.

 [Creating NFTs](../token/ZG9jOjI4ODY3ODE4-how-to-create-nft) has always been super simple with Tatum. However, implementing peer-to-peer NFT auctions properly can take a bit of work. That's why we have created an out-of-the-box NFT auction functionality that can be deployed in just a few minutes. 

 When an NFT is sold, the creator is automatically paid, the NFT is instantly transferred to the buyer, and you, the owner of the marketplace, automatically receive a percentage of the transaction. Everything happens on the blockchain, so you don’t even need to create a complex, application-level backend to run the listings.

 <!-- theme: info -->

>With this type of marketplace, your users can create auctions to sell ERC-721 or ERC-1155 tokens. These tokens can be purchased with the native assets of the given blockchain (e.g. ETH on Ethereum or MATIC on Polygon) or any ERC-20 token available on your blockchain of choice.
>
>Currently supported blockchains are Ethereum, Celo, Polygon, Binance Smart Chain, and Harmony.ONE.
>
>All smart contracts are available [here.](https://github.com/tatumio/smart-contracts/blob/master/contracts/tatum/nft/MarketplaceListing.sol)

---

## Create an auction house

The first step is to [create your own auction house smart contract](../token/b3A6MzA5MzA2OTI-create-nft-auction). This is a one-time operation, and the auction house you deploy will be used for every listing in your application. In this example, we'll deploy our auction house on the Polygon network.

**Request example**
```json
curl --request POST \
  --url https://api-eu1.tatum.io/v4/token/MATIC/auction \
  --header 'Content-Type: application/json' \
  --header 'x-api-key: ' \
  --data '{
  "feeRecipient": "0x687422eEA2cB73B5d3e242bA5456b782919AFc85",
  "auctionFee": 150,
  "fromPrivateKey": "0x05e150c73f1920ec14caa1e0b6aa09940899678051a78542840c2668ce5080c2",
  "nonce": 1,
  "fee": {}
}'
```

**Response example**
```json
{
  "txId": "c83f8818db43d9ba4accfe454aa44fc33123d47a4f89d47b314d6748eb0e9bc9",
  "failed": false
}

```

As you can see in the example above, you must enter a value in the `auctionFee` field. This is a percentage of the price that will be charged to the buyer when they make a purchase. The value represents the percentage multiplied by 100, so a 1.5% fee is written here as 150. This percentage is the same for all sales made at your auction.

The `feeRecipient` field represents the address where the fees will be sent, i.e. your address as the auction house owner.

The `txId` field in the response body contains the [contract address](../blockchain/b3A6MjgzNjM1MTY-get-transaction-by-hash-or-address). 

---
## Create an auction

Once the auction house contract is ready, you can enable auctions for your users. The NFT asset must be present at the seller's address before they create an NFT auction, and the seller must [approve the NFT spending for the auction](../token/b3A6MzA5MzA2OTc-approve-nft-spending-for-the-auction) contract. 

**Request example**
```json
curl --location --request POST 'https://api-eu1.tatum.io/v4/blockchain/auction/approve' \
--header 'Content-Type: application/json' \
--header 'x-api-key: YOUR_API_KEY' \
--data-raw '{
    "spender": "0xb36abab0c1365335dd762815aaae40dd1b990f99",
    "contractAddress": "0x6d8eae641416b8b79e0fb3a92b17448cfff02b11",
    "tokenId": "124",
    "isErc721": true,
    "chain": "MATIC",
    "fromPrivateKey": "0x37b091fc4ce46a56da643f021254612551dbe0944679a6e09cb5724d3085c9ab"
}'
```
**Response example**

```json
{
    "txId": "0x53e19422eec0a0bb20b3c2f43c30b8c30745e8e0265fea0359b56c72a0293f9a"
}
```

The seller can choose whether they are selling an ERC-721 or ERC-1155  token, and whether they are selling it for ERC-20 tokens or for the native currency of the given blockchain (in our case MATIC).
Every auction must have a unique `id`. This parameter identifies the auction and is used for performing other operations.

[The following API call](../token/b3A6MzA5MzA2OTM-sell-asset-on-the-nft-auction) will create a time-based NFT auction on your marketplace. You need to define the latest block in which the auction will still accept bids.

**Request example**
```json
curl --request POST \
  --url https://api-eu1.tatum.io/v4/token/MATIC/auction/sell \
  --header 'Content-Type: application/json' \
  --header 'x-api-key: ' \
  --data '{
  "contractAddress": "0x687422eEA2cB73B5d3e242bA5456b782919AFc85",
  "nftAddress": "0x687422eEA2cB73B5d3e242bA5456b782919AFc85",
  "seller": "0x687422eEA2cB73B5d3e242bA5456b782919AFc85",
  "erc20Address": "0x687422eEA2cB73B5d3e242bA5456b782919AFc85",
  "id": "string",
  "amount": "1",
  "tokenId": "100000",
  "endedAt": 0,
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
<!-- theme: info -->
>To use an ERC-20 token as a listing currency, the seller should add an `erc20Address` parameter to the call with the address of the smart contract of the ERC-20 token that is used for the listing.

The listing is now available for bidding in the auction house.

---

## Bidding for the auction

Once the token is in the auction, anyone can bid. Only bids higher than the highest current bid are accepted. The previous bid is returned to the bidder. The parameter `bidValue` contains the sum of the asset price and the marketplace fee.

<!-- theme: info -->
>The listing in the example below is for 1 MATIC and the fee is 2.5%, so the buyer would have to spend 1.025 MATIC to buy the asset.

Use the following API call to [bid for the NFT](../token/b3A6MzA5MzA2OTQ-bid-for-asset).

**Request example**
```json
curl --request POST \
  --url https://api-eu1.tatum.io/v4/token/MATIC/auction/bid \
  --header 'Content-Type: application/json' \
  --header 'x-api-key: ' \
  --data '{
  "contractAddress": "0x687422eEA2cB73B5d3e242bA5456b782919AFc85",
  "erc20Address": "0x687422eEA2cB73B5d3e242bA5456b782919AFc85",
  "id": "string",
  "bidValue": "1.025",
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
Having performed this operation, the bidder has sent their bid to the auction house contract.

---
## Settlement of the auction

Once the auction ends and no more bids can be accepted, any party can settle the auction. In this operation, the NFT is transferred to the buyer, the amount is transferred to the seller, and the fee is transferred to the fee recipient (i.e. the auction house owner address). Settlement can't be performed with a live auction.

Use the following API call to [settle an  auction](../token/b3A6MzA5MzA2OTY-settle-auction-of-asset).

**Request example**
```json
curl --request POST \
  --url https://api-eu1.tatum.io/v4/token/MATIC/auction/settle \
  --header 'Content-Type: application/json' \
  --header 'x-api-key: ' \
  --data '{
  "contractAddress": "0x687422eEA2cB73B5d3e242bA5456b782919AFc85",
  "erc20Address": "0x687422eEA2cB73B5d3e242bA5456b782919AFc85",
  "id": "string",
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

<!-- theme: info -->
> For an ERC-20 listing, the only difference is that the buyer [must authorize the NFT marketplace to spend his ERC-20 tokens](../token/b3A6MzA4OTMzODI-approve-spending-of-erc-20) before the actual operation.
>
> For an ERC-1155 listings, the only difference is the number of tokens that are being listed.

>You can [take a look](https://mumbai.polygonscan.com/tx/0x9a5aa635b2e55e39dac7007603969776ced92ce1f209f57b7888b7642a56dc6d) at an example transaction on the Mumbai Testnet.
