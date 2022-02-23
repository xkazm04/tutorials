# How to migrate to Tatum’s updated NFT marketplace and auction features

*Update your app's to our new workflow and leverage enhanced NFT features with your on-chain marketplaces and auctions.*

---

With the launch of our new NFTs that pay royalties as percentages and record provenance data, we have updated the workflow for our instantly deployable NFT marketplace and NFT auction features. This new workflow now enables you to use the new NFT provenance and royalty functionality, and no longer requires users to send their NFTs to the marketplace or auction smart contracts. It also allows NFT creators to receive cashback in whatever custom ERC-20 they would like on whatever blockchain they’re minting on.

<!-- theme: danger -->
>The old workflow will be discontinued, so it is necessary for all applications that use our NFT marketplace and auction functionality to migrate as soon as possible.

---

## What’s changed?

Previously, to sell an NFT on a marketplace or auction, users would have to send the NFT to the marketplace or auction smart contracts prior to its sale. Now, users just need to give permission to the NFT marketplace or auction to transfer the NFT.

In addition, if they wish to leverage the percentage royalties functionality with an ERC-20 token, users must also give permission to the NFT smart contract to spend the ERC-20 token pay out royalties when the NFT is sold at a marketplace or auction.

And finally, if NFT creators want to receive cashback in a custom ERC-20 token, they can now add the smart contract address of the ERC-20 in the mint NFT call.

## How to migrate

Our new NFT marketplace and auction workflows have just two different steps than the previous workflows, assuming you’d like to use provenance and percentage royalty functionality. It works like this, with the new steps in bold lettering:
1. Mint an NFT. For more information on how mint royalty NFTs with provenance data and percentage cashback, please refer to this guide.
2. **If you’d like to set cashback to be paid out in any custom ERC-20 token on whatever blockchain you’re minting on, you can now do so by adding an “ERC20” property to the API endpoint body. In this field, enter the smart contract address of the ERC-20 token in which the cashback will be paid out. Please refer to our guide on how to create royalty NFTs with provenance data and percentage cashback for more information.**
3. **Give permission to the NFT marketplace or auction smart contract to transfer the NFT.**

```SDK
console.log(await sendAuctionApproveNftTransfer(true, {
    fromPrivateKey: '0xa488a82b8b57c3ece4307525741fd8256781906c5fad948b85f1d63000948236',
    chain: Currency.CELO, contractAddress: '0x1214BEada6b25bc98f7494C7BDBf22C095FDCaBD',
    isErc721: true, spender: '0x991dfc0db4cbe2480296eec5bcc6b3215a9b7038', tokenId
}, 'https://alfajores-forno.celo-testnet.org'));

 const endedAt = (await celoGetCurrentBlock()) + 9;

```
```REST API call with Private key
curl --request POST \
  --url https://api-eu1.tatum.io/v3/blockchain/auction/approve \
  --header 'content-type: application/json' \
  --header 'x-api-key: REPLACE_KEY_VALUE' \
  --data '{
     "chain": "CELO",
     "feeCurrency": "CELO",
     "contractAddress": "0x687422eEA2cB73B5d3e242bA5456b782919AFc85",
     "spender": "0x687422eEA2cB73B5d3e242bA5456b782919AFc85",
     "isErc721": true,
     "tokenId": "100000",
     "fromPrivateKey": "0x05e150c73f1920ec14caa1e0b6aa09940899678051a78542840c2668ce5080c2",
     "nonce": 1,
     "fee": {
        "gasLimit": "40000",
        "gasPrice": "20"
     }
   }
```
```REST API call with KMS
curl --request POST \
  --url https://api-eu1.tatum.io/v3/blockchain/auction/approve \
  --header 'content-type: application/json' \
  --header 'x-api-key: REPLACE_KEY_VALUE' \
  --data '{
     "chain": "CELO",
     "feeCurrency": "CELO",
     "contractAddress": "0x687422eEA2cB73B5d3e242bA5456b782919AFc85",
     "spender": "0x687422eEA2cB73B5d3e242bA5456b782919AFc85",
     "isErc721": true,
     "tokenId": "100000",
     "signatureId": "26d3883e-4e17-48b3-a0ee-09a3e484ac83",
     "index": 0,
     "nonce": 1,
     "fee": {
        "gasLimit": "40000",
        "gasPrice": "20"
     }
   }
```

4. Create a new marketplace or auction listing. For more information on how to do so, please refer to our guide on creating NFT marketplaces or our guide on creating NFT auctions.
5. If the NFT is listed as for sale for an ERC-20 token, the buyer must then give the ERC-20 smart contract permission to spend ERC-20 tokens at the marketplace or auction.
6. **To pay the royalties as a percentage of the sale price at the NFT marketplace in an ERC-20 token, the buyer must then give permission to the NFT smart contract to spend the ERC-20 token and send royalties to the creators.**

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
```REST API call with Private key
curl --request POST \
  --url https://api-eu1.tatum.io/v3/blockchain/token/approve \
  --header 'content-type: application/json' \
  --header 'x-api-key: REPLACE_KEY_VALUE' \
  --header 'x-testnet-type: SOME_STRING_VALUE' \
  --data '{
     "chain": "CELO",
     "amount": "100000",
     "spender": "0x687422eEA2cB73B5d3e242bA5456b782919AFc85",
     "contractAddress": "0x687422eEA2cB73B5d3e242bA5456b782919AFc85",
     "fromPrivateKey": "0x05e150c73f1920ec14caa1e0b6aa09940899678051a78542840c2668ce5080c2",
     "nonce": 0,
     "feeCurrency": "CELO"
   }
```
```REST API call with KMS
curl --request POST \
  --url https://api-eu1.tatum.io/v3/blockchain/token/approve \
  --header 'content-type: application/json' \
  --header 'x-api-key: REPLACE_KEY_VALUE' \
  --header 'x-testnet-type: SOME_STRING_VALUE' \
  --data '{
     "chain": "CELO",
     "amount": "100000",
     "spender": "0x687422eEA2cB73B5d3e242bA5456b782919AFc85",
     "contractAddress": "0x687422eEA2cB73B5d3e242bA5456b782919AFc85",
     "signatureId": "26d3883e-4e17-48b3-a0ee-09a3e484ac83",
     "nonce": 0,
     "feeCurrency": "CELO"
   }
```

7. Now, the buyer can use the buy asset on marketplace or the bid for asset on auction endpoints to purchase or bid for the NFT from the marketplace or auction.

## And you’re good to go! 

That’s it, just a couple new steps and you’re all updated with our latest NFT marketplace and auction features. We understand it might be a bit of hassle to migrate over to the new features, but we’re confident that the percentage-based royalty and provenance data features are worth it. Plus, instead of transferring NFTs to the marketplace and auction smart contracts, you only need to allow them to be transferred by the smart contract, which is much more efficient overall.

If you have any questions at all or need any help, our tech team is always ready to assist on the Tatum [Discord](https://discord.com/invite/4TWtSP3vxU) or [subreddit](https://www.reddit.com/r/tatum_io/). We hope you enjoy the new features!


