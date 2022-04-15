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

### Import into Visual Builder
Before we can begind building we need to do a couple if imports. First, we need to import our Ethers.js library. Within your application right click the resources folder and select 'Import'. Navigate to and select the Ether.js file you saved to your computer. We also need to import our ContractABI.json. This file will inform Visual Builder of the structure of inputs & outputs of our contract call and is a required element. Everytime you compile a smart contract locally one of these files is created so each time you update and redeploy a smart contract you will need to update the ABI on your frontend. (This pattern is necessary for both visual builder & standard JavaScript applications). So right click resources again and import the ABI file.

### Create Frontend Components
 For the Visual Builder frontend, I'm keeping it quite simple. We have to main panels. One shows if there is no wallet detected & contains a connect wallet button. And the other panel will contain our buttons to both read & update the smart contract. The panels are configured via a Bind If and display according to whether or not a MetaMask account is connected to our app. I also have setup some variables to store data from the inputs / outputs. 

### Load Libraries & Write Javascript Functions
For this demonstration we are going to be writing 4 separate functions - 2 for interacting with MetaMask and 2 for interacting with our contract. However before we can write the functions we need to tell Visual Builder to use the files we imported earlier. 

	define(["resources/ethers.js",  'text!resources/contractabi.json'],  function(ethers,  abiString)  {
Then we set our contract address and parse the imported ABI file:

		const  contractAddress  =  "0x06A3F540004A2ba928fcba915D41fF32C084B57b";
		const  contractABI  =  JSON.parse(abiString).abi;

Now we can write the MetaMask functions:
- checkIfWalletIsConnected()
	- This function uses the ethers.js library to see if MetaMask has a previous connection with this application
		
			PageModule.prototype.checkIfWalletIsConnected  =  async  function  ()  {
			try  {
			const  {ethereum}  =  window;
			if  (!ethereum)  {
			console.log("Make sure you have metamask");
			}  else  {
			console.log("We have the ethereum object",  ethereum);
			}
			const  accounts  =  await  ethereum.request({method:  "eth_accounts"});
			if  (accounts.length  !==  0)  {
			const  account  =  accounts[0];
			console.log("Found an authorized account",  account);
			return  account;
			}  else  {
			console.log("No authorized account found");
			}
			}  catch  (error)  {
			console.log(error);
			}
			};
- connectWallet()
	- This function uses the ethers.js library to link MetaMask and our application
	
			PageModule.prototype.connectWallet  =  async  function  ()  {
			try  {
			const  {ethereum}  =  window;
			if  (!ethereum)  {
			alert("Get MetaMask");
			return;
			}
			const  accounts  =  await  ethereum.request({method:  "eth_requestAccounts"});
			console.log("Connected",  accounts[0]);
			return  accounts[0];
			}  catch  (error)  {
			console.log(error);
			}
			};

And for our Contract:
- readContractString()
	- This function calls our countract's public `message()` function - 
	
			PageModule.prototype.readContractString  =  async  function  ()  {
			try  {
			const  {ethereum}  =  window;
			if  (ethereum)  {
			const  provider  =  new  ethers.providers.Web3Provider(ethereum);
			const  signer  =  provider.getSigner();
			const  contract  =  new  ethers.Contract(contractAddress,  contractABI,  signer);
			let  response  =  await  contract.message();
			console.log(response);
			return  response;
			}
			}  catch  (error)  {
			console.log(error);
			}
			};
- setContractString()
	- This function calls our contract's public `update()` function
	
			PageModule.prototype.setContractString  =  async  function  (newString)  {
			try  {
			const  {ethereum}  =  window;
			if  (ethereum)  {
			const  provider  =  new  ethers.providers.Web3Provider(ethereum);
			const  signer  =  provider.getSigner();
			const  contract  =  new  ethers.Contract(contractAddress,  contractABI,  signer);
			let  response  =  await  contract.update(newString);
			console.log(response);
			return  response;
			}
			}  catch  (error)  {
			console.log(error);
			}
			}
			return  PageModule;
			});
	
