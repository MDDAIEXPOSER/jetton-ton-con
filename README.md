# Step 1: Setting up your Development Environment
To develop smart contracts and a front-end, you need to have a development environment set up with the necessary tools and frameworks. For this tutorial, we will use the following tools:
- TONOS CLI: A command-line tool for working with TON blockchain.
- TONDEV: A development tool that provides a local blockchain for TON.
- Solidity: A high-level programming language used to write smart contracts.
- BluePrint: A visual programming tool for designing smart contracts.
- Ton.js: A JavaScript library for interacting with smart contracts on the TON blockchain.

## TONOS CLI: A command-line tool for working with TON blockchain:
```
```
## TONDEV: A development tool that provides a local blockchain for TON:
```
```
## Solidity: A high-level programming language used to write smart contracts:
```
```
## BluePrint: A visual programming tool for designing smart contracts:
```
```
## Ton.js: A JavaScript library for interacting with smart contracts on the TON blockchain:
```
```
# Step 2: Creating the Smart Contract
The first step is to create the smart contract that will handle the sale of NFTs for tokens. For this tutorial, we will use the nft-jetton-sale-smc project available at https://github.com/dvlkv/nft-jetton-sale-smc.

Clone the repository to your local environment and navigate to the contracts directory. In this directory, you will find the Jetton.sol and NFT.sol contracts. Deploy these contracts to your local blockchain using TONDEV or TONOS CLI.

Once deployed, you can use the Jetton contract to create a new token and the NFT contract to mint NFTs.

Now, create a new contract named NFTMarketplace in the same directory as Jetton.sol and NFT.sol. Add the following imports to the beginning of the contract:
```
pragma ton-solidity >= 0.35.0;

import "./Jetton.sol";
import "./NFT.sol";
```

The NFT.sol file contains the implementation for the NFT contract, while the Jetton.sol file contains the implementation for the Jetton token.

Add the following code to the contract to define the NFTMarketplace contract:
```
contract NFTMarketplace {
    address private _owner;

    Jetton private _jettonToken;
    NFT private _nft;

    mapping(uint256 => uint256) private _nftPrices;

    constructor(address jettonTokenAddress, address nftAddress) public {
        _owner = msg.sender;

        _jettonToken = Jetton(jettonTokenAddress);
        _nft = NFT(nftAddress);
    }
}
```

The NFTMarketplace contract has a constructor that takes two addresses: one for the Jetton token contract and one for the NFT contract. The _owner variable is set to the address of the contract creator.

Next, add the following function to the contract to allow users to create NFTs and set their prices:
```
function createNFT(uint256 tokenId, uint256 price) public {
    require(msg.sender == _owner, "Only the owner can create NFTs");

    _nftPrices[tokenId] = price;
    _nft.mint(msg.sender, tokenId);
}
```

Next, add the following function to the contract to allow users to buy NFTs using Jettons:
```
function buyNFT(uint256 tokenId) public {
    require(_nftPrices[tokenId] > 0, "NFT is not for sale");

    uint256 price = _nftPrices[tokenId];

    require(_jettonToken.balanceOf(msg.sender) >= price, "Insufficient Jetton balance");

    _jettonToken.transferFrom(msg.sender, _owner, price);
    _nft.safeTransferFrom(_owner, msg.sender, tokenId, "");

    _nftPrices[tokenId] = 0;
}
```
The buyNFT function takes a tokenId as an argument and first checks if the NFT is for sale by verifying if its price is greater than zero. If the NFT is for sale, the function checks if the buyer has enough Jettons to purchase the NFT. If the buyer has enough Jettons, the function transfers the Jettons from the buyer to the owner of the NFT and transfers the NFT from the owner to the buyer.

Finally, the function sets the price of the NFT to zero to prevent further sales of the NFT.
Step 3: Testing the Smart Contract
Now that you have created the smart contract, it's time to test it. You can use the TONOS CLI or Ton.js library to interact with the smart contract.

To test the smart contract, you can follow the steps below:

1. Deploy the Jetton and NFT contracts to your local blockchain using TONDEV or TONOS CLI.
2. Deploy the NFTMarketplace contract to your local blockchain using TONDEV or TONOS CLI.
3. Call the createNFT function to create a new NFT and set its price.
4. Call the buyNFT function to buy the NFT using Jettons.

You can use the tonos-cli tool or Ton.js library to interact with the smart contract. Here's an example of how to use Ton.js to call the createNFT function:
```
const TonClient = require("@tonclient/core");
const { libNode } = require("@tonclient/lib-node");
const jettonAbi = require("./Jetton.abi.json");
const nftAbi = require("./NFT.abi.json");
const marketplaceAbi = require("./NFTMarketplace.abi.json");

// Set up TonClient
TonClient.useBinaryLibrary(libNode);
const client = new TonClient({
    network: {
        server_address: "http://localhost"
    }
});

// Connect to the deployed contracts
const jetton = client.getContract(jettonAbi, "<deployed jetton address>");
const nft = client.getContract(nftAbi, "<deployed nft address>");
const marketplace = client.getContract(marketplaceAbi, "<deployed marketplace address>");

// Call the createNFT function
async function createNFT(tokenId, price) {
    const result = await marketplace.run({
        functionName: "createNFT",
        input: {
            tokenId: tokenId,
            price: price
        },
        keyPair: null
    });

    console.log(result);
}

createNFT(1, 100);
```
You can use a similar approach to call the buyNFT function.

## Step 4: Creating the Front-End
The front-end of the NFT marketplace will consist of a web interface and a Telegram Bot. The web interface will allow users to browse NFTs for sale and buy them using Jettons.
To create the web interface, you can use any web framework such as React, Angular, or Vue.js. In this tutorial, we will use React to create the web interface.

First, create a new React project:
```
npx create-react-app nft-marketplace
```

Next, install the Ton.js library:
```
npm install ton-client-js
```

Create a new file named config.js to store the addresses of the deployed contracts:
```
export default {
    jettonAddress: "<deployed jetton address>",
    nftAddress: "<deployed nft address>",
    marketplaceAddress: "<deployed marketplace address>"
};
```

Next, create a new file named Ton.js to configure the Ton.js library:
```
import TonClient from "@tonclient/core";
import { libWeb } from "@tonclient/lib-web";
import config from "./config";

// Set up TonClient
TonClient.useBinaryLibrary(libWeb);
const client = new TonClient({
    network: {
        server_address: "http://localhost"
    }
});

// Connect to the deployed contracts
const jetton = client.getContract(config.jettonAddress, require("./Jetton.abi.json"));
const nft = client.getContract(config.nftAddress, require("./NFT.abi.json"));
const marketplace = client.getContract(config.marketplaceAddress, require("./NFTMarketplace.abi.json"));

export default {
    client,
    jetton,
    nft,
    marketplace
};
```

The Ton.js module exports an object containing the TonClient instance and the contracts connected to the deployed addresses.

Next, create a new file named Marketplace.js to implement the front-end logic:
```
import { TonClient } from "@tonclient/core";
import { libWeb } from "@tonclient/lib-web";

TonClient.useBinaryLibrary(libWeb);

const tonClient = new TonClient({
    network: { 
        server_address: "net.ton.dev",
    },
});
```

This code initializes the TonClient with the network address of the TON DevNet.

Next, update the buyNFT function to use TonConnect to sign and send the transaction:
```
async function buyNFT(id, price) {
    const seller = await nftMarketplaceContract.ownerOf({ id });
    const buyer = (await tonClient.crypto.accounts.get())[0].address;
    const sellerWallet = await tonClient.net.query_collection({
        collection: "accounts",
        filter: { id: { eq: seller } },
        result: "boc",
    })[0].boc;
    const buyerWallet = await tonClient.net.query_collection({
        collection: "accounts",
        filter: { id: { eq: buyer } },
        result: "boc",
    })[0].boc;

    const unsignedTransaction = await nftMarketplaceContract.buyNFT({
        id,
        price,
        seller,
        buyer,
    });

    const signedTransaction = await window.tonlabs
        .getSigner({ type: "TonWeb" })
        .sign(unsignedTransaction);

    await tonClient.processing.process_message({
        message_encode_params: {
            abi: {
                type: "Contract",
                value: nftMarketplaceContract.abi,
            },
            address: nftMarketplaceContract.address,
            call_set: {
                function_name: "buyNFT",
                input: {
                    id,
                    price,
                    seller,
                    buyer,
                },
            },
            signer: {
                type: "Keys",
                keys: signedTransaction.keys,
            },
        },
        send_events: true,
    });

    await tonClient.net.wait_for_collection({
        collection: "transactions",
        filter: {
            in_message: {
                dst: { eq: nftMarketplaceContract.address },
            },
            now: { gt: 0 },
        },
        result: "id",
    });
}
```
This code retrieves the addresses of the buyer and seller wallets, creates an unsigned transaction using the buyNFT function of the NFTMarketplace contract, uses TonConnect to sign the transaction, and sends the signed transaction to the TON network using the TonClient.

Now, when the "Buy Now" button is clicked, TonConnect will prompt the user to sign the transaction and send the Jettons to the seller in exchange for the NFT.

Then, add the following code to the beginning of the Marketplace component to initialize the TonClient:
```
import { TonClient } from "@tonclient/core";
import { libWeb } from "@tonclient/lib-web";

TonClient.useBinaryLibrary(libWeb);

const tonClient = new TonClient({
    network: { 
        server_address: "net.ton.dev",
    },
});
```
That's it! You have successfully integrated TonConnect into the buy flow of the NFT marketplace.

