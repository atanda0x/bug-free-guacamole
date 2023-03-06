# How to create a basic BOT

Building a bot that buys, sells, and bids for NFTs in JavaScript involves interacting with various APIs provided by blockchain networks such as Ethereum, Binance Smart Chain, and others. In this example, we'll use the OpenSea API to interact with Ethereum-based NFTs.

Before we start coding, we need to make sure we have the following dependencies installed:

Node.js
NPM
We'll also need to install the following packages:

axios for making HTTP requests
dotenv for loading environment variables
ethers for interacting with Ethereum networks
Let's begin by creating a new Node.js project and installing the required dependencies:

bash
Copy code
mkdir nft-bot
cd nft-bot
npm init -y
npm install axios dotenv ethers
Next, we'll create a .env file to store our API key and wallet information:

makefile

```
OPENSEA_API_KEY=<your-opensea-api-key>
WALLET_ADDRESS=<your-wallet-address>
WALLET_PRIVATE_KEY=<your-wallet-private-key>

```
Note that we'll need to replace <your-opensea-api-key>, <your-wallet-address>, and <your-wallet-private-key> with our actual API key and wallet information.

Now, let's create a new JavaScript file called bot.js and start coding.

First, we'll load our environment variables using dotenv:

```
require('dotenv').config();
const { OPENSEA_API_KEY, WALLET_ADDRESS, WALLET_PRIVATE_KEY } = process.env;
Next, we'll initialize our ethers provider and signer using our wallet private key:

```

```
const { ethers } = require('ethers');

const provider = new ethers.providers.JsonRpcProvider();
const wallet = new ethers.Wallet(WALLET_PRIVATE_KEY, provider);
We'll also create a function to sign transactions using our wallet:

```

```
const signTransaction = async (transaction) => {
  const { gasPrice } = await provider.getGasPrice();
  const { gasLimit } = await transaction.estimateGas();
  const signedTransaction = await wallet.signTransaction({
    ...transaction,
    gasPrice: gasPrice.toString(),
    gasLimit: gasLimit.toString(),
  });
  return provider.sendTransaction(signedTransaction);
};

```

Now, let's create a function to fetch NFTs from OpenSea:

```
const fetchNFTs = async () => {
  const { data } = await axios.get(
    'https://api.opensea.io/api/v1/assets',
    {
      params: {
        owner: WALLET_ADDRESS,
        order_direction: 'desc',
        offset: 0,
        limit: 50,
      },
      headers: {
        'X-API-KEY': OPENSEA_API_KEY,
      },
    },
  );
  return data.assets;
};

```
This function fetches the first 50 NFTs owned by our wallet from OpenSea.

Next, let's create a function to buy an NFT:

```
const buyNFT = async (nft) => {
  const { contractAddress, tokenId } = nft;
  const contract = new ethers.Contract(
    contractAddress,
    [
      'function safeTransferFrom(address _from, address _to, uint256 _tokenId)',
    ],
    wallet,
  );
  const transaction = await contract.populateTransaction.safeTransferFrom(
    WALLET_ADDRESS,
    '0x000000000000000000000000000000000000dEaD', // replace with seller's address
    tokenId,
  );
  return signTransaction(transaction);
};

```
This function buys an NFT by calling the `safeTransferFrom
