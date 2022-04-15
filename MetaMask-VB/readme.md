# MetaMask, Smart Contracts, & Visual Builder

### A Step by Step Guide to Implementing Smart Contract Calls & MetaMask on Oracle Visual Builder.

Visual Builder has a lot of great built-in features that allow developers to fetch their data from a variety of sources and consume said data in their application front ends. One can fetch and send data via the built-in REST capabilities. However, what if we wanted to interact with data from a decentralized network such as a smart contract living on a blockchain? First things first: authentication works differently on the blockchain, we need a wallet to sign our transactions. So: in comes MetaMask; a wallet with capabilities of signing blockchain transactions on a wide variety of different blockchain networks. If we pair MetaMask with a special JavaScript library we can use interact with MetaMask using our application in order to send & sign transactions to smart contracts. So let’s go through how this is implemented.

### In order to follow along you will need to do the following:

- Have your own instance of Visual Builder

- A deployed Smart Contract & Contract ABI Artifact – You can use my HelloWorld sample - xxxx on Ethereum Rinkeby Testnet

		 pragma  solidity ^0.8.13;
	
		 contract HelloWorld { 
		 string  public message;

		 constructor(string  memory  initMessage) {
			 message = initMessage;
		 }
		 function  update(string  memory  newMessage) public {
			 message = newMessage;
		}
		}

- [MetaMask browser extension](https://metamask.io/download/)
		 - Download & Create Account 
		 - Add & Configure Rinkeby Testnet - add picture
		 - Faucet test ethereum - when data is changed on the blockchain you need tokens to pay the computing gas fees - add link / picture

- Import & save Ethers.js JavaScript library into a file locally – [https://cdn.ethers.io/lib/ethers-5.2.esm.min.js](https://cdn.ethers.io/lib/ethers-5.2.esm.min.js)

STEPS

### Create Frontend Components
 For the Visual Builder frontend, I'm keeping it quite simple. We have to main panels. One shows if there is no wallet detected & contains a connect wallet button. And the other panel will contain our buttons to both read & update the smart contract 
