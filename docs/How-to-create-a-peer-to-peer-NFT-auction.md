# How to create a peer to peer NFT auction

---

Peer-to-peer NFT marketplaces like OpenSea allow users to create NFTs with metadata (pictures, videos, songs, 3D art, etc.) and post listings to sell them to one another. OpenSea then takes a percentage of each sale. Users can create fixed-price listings for their NFTs or put them up for auction and sell them to the highest bidder.

 [Creating NFTs](../token/ZG9jOjI4ODY3ODE4-how-to-create-nft) has always been super simple with Tatum. However, implementing peer-to-peer NFT auctions properly can take a bit of work. That's why we have created an out-of-the-box NFT auction functionality that can be deployed in just a few minutes. 

 When an NFT is sold, the creator is automatically paid, the NFT is instantly transferred to the buyer, and you, the owner of the marketplace, automatically receive a percentage of the transaction. Everything happens on the blockchain, so you don’t even need to create a complex, application-level backend to run the listings.

 <!-- theme: info -->

>With this type of marketplace, your users can create auctions to sell ERC-721 or ERC-1155 tokens. These tokens can be purchased with the native assets of the given blockchain (e.g. ETH on Ethereum or MATIC on Polygon) or any ERC-20 token available on your blockchain of choice.
>
>Currently supported blockchains are:
>- Ethereum
>- Celo
>- Polygon
>- Binance Smart Chain
>- Harmony.ONE.
>
>All smart contracts are available [here.](https://github.com/tatumio/smart-contracts/blob/master/contracts/tatum/nft/MarketplaceListing.sol)

---
# Import required libraries

If you are using our JavaScript SDK, the first step is to import all of the required JS libraries for the functions we will be using.

```JavaScript
import { 
    DeployNftAuction,
    deployAuction,
    CreateAuction,
    sendAuctionCreate,
    sendCeloSmartContractReadMethodInvocationTransaction,
    sendAuctionApproveNftTransfer,
    InvokeAuctionOperation,
    sendAuctionSettle,
    sendAuctionBid,
    sendAuctionApproveNftTransfer,
    celoGetCurrentBlock
} from '@tatumio/tatum';
```
---

## Create an auction house

The first step is to [create your own auction house smart contract](../token/b3A6MzA5MzA2OTI-create-nft-auction). This is a one-time operation, and the auction house you deploy will be used for every listing in your application. In this example, we'll deploy our auction house on the Polygon network.

**Request example**
```JavaScript
const body = new DeployNftAuction();
    body.fromPrivateKey = '0xa488a82b8b57c3ece4307525741fd8256781906c5fad948b85f1d63000948236';
    body.feeRecipient = '0x8cb76aEd9C5e336ef961265c6079C14e9cD3D2eA';
    body.auctionFee = 150;
    body.feeCurrency = Currency.CUSD;
    body.chain = Currency.CELO;
    const test = await deployAuction(true, body, 'https://alfajores-forno.celo-testnet.org');
    console.log(test);
```
```cURL+KMS
curl --request POST \
  --url https://api-eu1.tatum.io/v3/blockchain/auction \
  --header 'content-type: application/json' \
  --header 'x-api-key: REPLACE_KEY_VALUE' \
  --data '{
    "feeRecipient": "0x8cb76aEd9C5e336ef961265c6079C14e9cD3D2eA",
    "auctionFee": 150,
    "feeCurrency": CUSD,
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
```cURL+privateKey
curl --request POST \
  --url https://api-eu1.tatum.io/v3/blockchain/auction \
  --header 'content-type: application/json' \
  --header 'x-api-key: REPLACE_KEY_VALUE' \
  --data '{
    "feeRecipient": "0x8cb76aEd9C5e336ef961265c6079C14e9cD3D2eA",
    "auctionFee": 150,
    "feeCurrency": CUSD,
    "chain": "CELO",
    "fromPrivateKey": "0xa488a82b8b57c3ece4307525741fd8256781906c5fad948b85f1d63000948236"
    "fee": {
       "gasLimit": "40000",
       "gasPrice": "20"
       }
   }'
```
The body of the API request should contain the following parameters:
- `chain` - The blockchain on which the marketplace smart contract will be deployed.
- `fromPrivateKey` - The private key for the address that will pay for the deployment of the auction smart contract.
- `feeRecipient` - The address where the fees will be sent, i.e. your address as the auction owner.
- `auctionFee` - This is a percentage of the price that will be charged to the buyer when they make a purchase. The value is the percentage charged * 100, so a 1.5% fee is written as 150. This percentage is the same for all sales made at your auction.
- `feeCurrency` - The currency in which the gas fee for the operation will be paid (only for Celo).
- `fee` - The gas price and limit for the gas fee to pay for the operation. If this fee is absent, the fee will be calculated automatically.
**Response example**
```json
{
    "txId": "0x9ff62d44abaf65018081a6511c84ca8f89d7575d2a1ea058e93c1b7d57ff1807"
}

```
The response is a transaction ID from which we can obtain the contract address using the [get contract address](../blockchain/b3A6MjgzNjM1MTY-get-transaction-by-hash-or-address) endpoint.

---
## Create an auction listing

Once the auction house contract is ready, you can enable auctions for your users. The NFT asset must be present at the seller's address before they create an NFT auction, and the seller must [approve the NFT spending for the auction](../token/b3A6MzA5MzA2OTc-approve-nft-spending-for-the-auction) contract. 

**Request example**
```JavaScript
  const body = new CreateAuction();
      body.fromPrivateKey = '0xa488a82b8b57c3ece4307525741fd8256781906c5fad948b85f1d63000948236';
      body.contractAddress = '0x991dfc0db4cbe2480296eec5bcc6b3215a9b7038';
      body.nftAddress = '0x1214BEada6b25bc98f7494C7BDBf22C095FDCaBD';
      body.endedAt = 100000;
      body.tokenId = '1';
      body.id = tokenId;
      body.isErc721 = true;
      body.seller = '0x48d4bA7B2698A4b89635b9a2E391152350DB740f';
      body.erc20Address = '0x8cb76aEd9C5e336ef961265c6079C14e9cD3D2eA';
      body.feeCurrency = Currency.CUSD;
      body.chain = Currency.CELO;
      console.log(await sendAuctionCreate(true, body, 'https://alfajores-forno.celo-testnet.org'));
```
```cURL+KMS
curl --request POST \
  --url https://api-eu1.tatum.io/v3/blockchain/auction/sell \
  --header 'content-type: application/json' \
  --header 'x-api-key: REPLACE_KEY_VALUE' \
  --data '{
     "chain": "CELO",
     "feeCurrency": "CELO",
     "contractAddress": "0x991dfc0db4cbe2480296eec5bcc6b3215a9b7038",
     "nftAddress": "0x1214BEada6b25bc98f7494C7BDBf22C095FDCaBD",
     "seller": "0x48d4bA7B2698A4b89635b9a2E391152350DB740f",
     "erc20Address": "0x8cb76aEd9C5e336ef961265c6079C14e9cD3D2eA",
     "id": "string",
     "amount": "1",
     "tokenId": "1",
     "endedAt": 100000,
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
```cURL+privateKey
curl --request POST \
  --url https://api-eu1.tatum.io/v3/blockchain/auction/sell \
  --header 'content-type: application/json' \
  --header 'x-api-key: REPLACE_KEY_VALUE' \
  --data '{
     "chain": "CELO",
     "feeCurrency": "CELO",
     "contractAddress": "0x991dfc0db4cbe2480296eec5bcc6b3215a9b7038",
     "nftAddress": "0x1214BEada6b25bc98f7494C7BDBf22C095FDCaBD",
     "seller": "0x48d4bA7B2698A4b89635b9a2E391152350DB740f",
     "erc20Address": "0x8cb76aEd9C5e336ef961265c6079C14e9cD3D2eA",
     "id": "string",
     "amount": "1",
     "tokenId": "1",
     "endedAt": 100000,
     "isErc721": true,
     "fromPrivateKey": "0x05e150c73f1920ec14caa1e0b6aa09940899678051a78542840c2668ce5080c2",
     "nonce": 1,
     "fee": {
       "gasLimit": "40000",
       "gasPrice": "20"
       }
    }'
```
The seller can choose whether they are selling an ERC-721 or ERC-1155  token, and whether they are selling it for ERC-20 tokens or for the native currency of the given blockchain (in our case CELO). Every auction must have a unique `id`. This parameter identifies the auction and is used for performing other operations.

The body of the API request should contain the following parameters:

- `chain` - The chain on which the listing will be created (same as in the first call).
- `fromPrivateKey` - The private key of the address that will pay for the creation of the NFT listing.
- `contractAddress` - The address of the auction smart contract you deployed in the previous call.
- `nftAddress` - The address of the smart contract of the NFT being sold in the auction.
- `tokenId` - The ID of the NFT to be sold in the auction.
- `id` - The ID of the auction selling the NFT.
- `isErc721` - Set to "true" if you are selling an ERC-721 or equivalent token. Set to "false" if you are selling an ERC-1155 token.
- `erc20Address` (optional) - If the seller is selling their NFT for an ERC-20 (or equivalent) token, this field should be included and contain the address of the smart contract of the ERC-20 token.
- `seller` - The address that holds the NFT being sold and to which the funds used to purchase the NFT will be sent.
- `endedAt` - For a time-based auction, this parameter will be the block at which point the auction will end.
- `feeCurrency` - The currency in which the gas fee for the operation will be paid (only for Celo).
- `fee` - The gas price and limit for the gas fee to pay for the operation. If this fee is absent, the fee will be calculated automatically.

<!-- theme: info-->
>To use an **ERC-20 token as a listing currency**, the seller should add an `erc20Address` parameter to the call with the address of the smart contract of the ERC-20 token that is used for the listing. The buyer will then need to approve the ERC-20 token for spending when they purchase the NFT (see below).

<!-- theme: info-->
>To create a **time-based auction**, the `endedAt` parameter must be present in the body of the call, as it is in the example above. When the blockchain reaches the block number you have specified in this field, the auction will end. In the example, the auction will end when block 10,000 is added.
>
>To estimate the time at which the blockchain will reach a specific block height, use [this API call](https://tatum.io/apidoc.php#operation/GetAuctionEstimatedTime).

**Response example**

```json
{
    "txId": "0x53e19422eec0a0bb20b3c2f43c30b8c30745e8e0265fea0359b56c72a0293f9a"
}
```

The response is a transaction ID. The listing is now available in the auction house and available for bids.

---

## Send approval for the NFT to be sold at the auction

Next, we must give the auction smart contract permission to transfer the NFT we are selling. 

**Request example**
```JavaScript
await sendAuctionApproveNftTransfer(true, {
    fromPrivateKey: '0xa488a82b8b57c3ece4307525741fd8256781906c5fad948b85f1d63000948236',
    chain: Currency.CELO, 
    contractAddress: '0xdf82c2f74aa7b629bda65b1cfd258248c9c2b7d3',
    isErc721: true, 
    spender: '0x991dfc0db4cbe2480296eec5bcc6b3215a9b7038', 
    tokenId: '1'
}, 'https://alfajores-forno.celo-testnet.org');
```
```cURL+KMS
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
```cURL+privateKey
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

The body of the API request should contain the following parameters:

- `fromPrivateKey` - The private key of the address that will pay for the gas fees for the approval operation.
- `chain` - The chain on which the operation will take place (the same as in the first two API requests).
- `contractAddress` - The address of the NFT smart contract.
- `isErc721` - Set to "true" if you are selling an ERC-721 or equivalent token. Set to "false" if you are selling an ERC-1155 token.
- `spender` - The address of the auction smart contract that will be transferring the token.
- `tokenId` - The ID of the NFT being sold.
- `feeCurrency` - The currency in which the gas fee for the operation will be paid (only for Celo).
- `fee` - The gas price and limit for the gas fee to pay for the operation. If this fee is absent, the fee will be calculated automatically.
---

## Approve ERC-20 for bidding and for cashback

For a listing for an NFT being sold for ERC-20 tokens, the buyer [must approve the auction smart contract to spend his ERC-20 tokens](https://tatum.io/apidoc#operation/Erc20Approve) before the actual buy operation. Here is the call to approve the auction for spending the ERC-20 token:

**Request example**
```JavaScript
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
```cURL+KMS
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
```cURL+privateKey
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
The body of the API request should contain the following parameters:
chain - The chain on which the transaction will take place (same as in the previous calls).
- `contractAddress` - The address of the ERC-20 smart contract to be approved for spending.
- `spender` - The address of the marketplace smart contract that will spend the ERC-20 token.
- `amount` - The amount of funds to be approved to spend.
- `fromPrivateKey` - The private key of the address that will pay for the gas fees for the approval operation.
- `feeCurrency` - The currency in which the gas fee for the operation will be paid (only for Celo).
- `fee` - The gas price and limit for the gas fee to pay for the operation. If this fee is absent, the fee will be calculated automatically.

<!-- theme: info-->
>For NFTs to payout cashback as ERC-20 tokens, this same approve ERC-20 spending endpoint must be invoked.

---

## Bidding for the auction

Once the token is in the auction, anyone can bid. Only bids higher than the highest current bid are accepted. The previous bid is returned to the bidder. The parameter `bidValue` contains the sum of the asset price and the marketplace fee.

<!-- theme: info -->
>The listing in the example is for 1 CELO and the fee is 1.5%, so the buyer would have to spend 1.015 CELO to buy the asset.

Use the following API call to [bid for the NFT](../token/b3A6MzA5MzA2OTQ-bid-for-asset).

**Request example**
```JavaScript
const bid = new InvokeAuctionOperation();
    bid.fromPrivateKey = '0x4874827a55d87f2309c55b835af509e3427aa4d52321eeb49a2b93b5c0f8edfb';
    bid.contractAddress = '0x8cb76aEd9C5e336ef961265c6079C14e9cD3D2eA';
    bid.id = `${tokenId}1`;
    bid.bidValue = '0.001015';
    bid.feeCurrency = Currency.CELO;
    bid.chain = Currency.CELO;
    bid.fee = {gasPrice: '5', gasLimit: '300000'};
    console.log(await sendAuctionBid(true, bid, 'https://alfajores-forno.celo-testnet.org'));
```
```cURL+KMS
curl --request POST \
  --url https://api-eu1.tatum.io/v3/blockchain/auction/bid \
  --header 'content-type: application/json' \
  --header 'x-api-key: REPLACE_KEY_VALUE' \
  --data '{
    "contractAddress": "0xb36abab0c1365335dd762815aaae40dd1b990f99",
    "id": "1",
    "bidValue": "0.001015",
    "chain": "CELO",
    "signatureId": "26d3883e-4e17-48b3-a0ee-09a3e484ac83",
    "index": 0,
    "nonce": 1,
    "fee": {
       "gasLimit": "30000",
       "gasPrice": "5"
       }
   }'
```
```cURL+privateKey
curl --request POST \
  --url https://api-eu1.tatum.io/v3/blockchain/auction/bid \
  --header 'content-type: application/json' \
  --header 'x-api-key: REPLACE_KEY_VALUE' \
  --data '{
    "contractAddress": "0xb36abab0c1365335dd762815aaae40dd1b990f99",
    "id": "1",
    "bidValue": "0.001015",
    "chain": "CELO",
    "fromPrivateKey": "0x4874827a55d87f2309c55b835af509e3427aa4d52321eeb49a2b93b5c0f8edfb"
    "fee": {
       "gasLimit": "30000",
       "gasPrice": "5"
       }
   }'
```
The body of the API request should contain the following parameters:
chain - The chain on which the transaction will take place (same as in the previous calls).
- `fromPrivateKey` - The private key of the address that will pay for the gas fees for the bid operation.
- `contractAddress` - The address of the auction smart contract.
- `id` - The ID of the auction selling the NFT.
- `bidValue` - How much the bidder is bidding for the NFT.
- `feeCurrency` - The currency in which the gas fee for the operation will be paid (only for Celo).
- `fee` - The gas price and limit for the gas fee to pay for the operation. If this fee is absent, the fee will be calculated automatically.


**Response example**
```json
{
{
    "txId": "0x99c563e81353bc2f24646b784687892398a0b0b2261c9870be3aee46552a063c"
}
}
```
The response is a transaction ID. Having performed this operation, the bidder has sent his bid to the auction house contract.

---
## Settlement of the auction

Once the auction ends and no more bids can be accepted, any party can settle the auction. In this operation, the NFT is transferred to the buyer, the amount is transferred to the seller, and the fee is transferred to the fee recipient (i.e. the auction house owner address). Settlement can't be performed with a live auction.

Use the following API call to [settle an  auction](../token/b3A6MzA5MzA2OTY-settle-auction-of-asset).

**Request example**
```JavaScript
const settle = new InvokeAuctionOperation();
    settle.fromPrivateKey = '0xa488a82b8b57c3ece4307525741fd8256781906c5fad948b85f1d63000948236';
    settle.contractAddress = '0x991dfc0db4cbe2480296eec5bcc6b3215a9b7038';
    settle.id = 1;
    settle.feeCurrency = Currency.CUSD;
    settle.chain = Currency.CELO;
    console.log(await sendAuctionSettle(true, settle, 'https://alfajores-forno.celo-testnet.org'));
```
```cURL+KMS
curl --location --request POST 'https://api-eu1.tatum.io/v3/blockchain/auction/settle' \
--header 'Content-Type: application/json' \
--header 'x-api-key: YOUR_API_KEY' \
--data-raw '{
    "contractAddress": "0x991dfc0db4cbe2480296eec5bcc6b3215a9b7038",
    "id": "1",
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
```cURL+privateKey
curl --location --request POST 'https://api-eu1.tatum.io/v3/blockchain/auction/settle' \
--header 'Content-Type: application/json' \
--header 'x-api-key: YOUR_API_KEY' \
--data-raw '{
    "contractAddress": "0x991dfc0db4cbe2480296eec5bcc6b3215a9b7038",
    "id": "1",
    "chain": "CELO",
    "fromPrivateKey": "0xa488a82b8b57c3ece4307525741fd8256781906c5fad948b85f1d63000948236"
    "fee": {
       "gasLimit": "40000",
       "gasPrice": "20"
       }
    }'
```
The body of the API request should contain the following parameters:
- `chain` - The chain on which the transaction will take place (same as in the previous calls).
- `fromPrivateKey` - The private key of the address that will pay for the gas fees for the settle operation.
- `contractAddress` - The address of the auction smart contract.
- `id` - The ID of the auction selling the NFT.
- `feeCurrency` - The currency in which the gas fee for the operation will be paid (only for Celo).
- `fee` - The gas price and limit for the gas fee to pay for the operation. If this fee is absent, the fee will be calculated automatically.

**Response example**
```json
{
    "txId": "0x9a5aa635b2e55e39dac7007603969776ced92ce1f209f57b7888b7642a56dc6d"
}
```
The response is a transaction ID. 

<!-- theme: info-->
> For an ERC-1155 listings, the only difference is the number of tokens that are being listed.

---

# And that's it!

Now your app's end-users can create NFT auctions and bid on NFTs with ease!
