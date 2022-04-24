# Visual Builder + MetaMask = Smart Contracts
### A Guide to Implementing Smart Contract Calls using MetaMask on Oracle Visual Builder.

Visual Builder has a lot of great built-in features that allow developers to fetch their data from a variety of sources and consume said data in their application frontends via the built-in REST capabilities. However, what if you wanted to build an application that interacts with smart contracts? 

Becauase authentication works differently on the blockchain, a wallet is needed to sign transactions. So... in comes MetaMask; a wallet with capabilities of signing blockchain transactions on a wide variety of networks. When paired with a special JavaScript library an application can utilize MetaMask to send & sign transactions to smart contracts. Let's go through how this is done.

### In order to follow along you will need the following:

- An instance of Visual Builder - learn more about VBCS [here.](https://www.oracle.com/application-development/visual-builder/)

- A deployed Smart Contract & Contract You can use this `HelloWorld` sample - `0x06A3F540004A2ba928fcba915D41fF32C084B57b` on Ethereum Rinkeby Testnet
	
		pragma  solidity ^0.8.13;
		 contract HelloWorld { 
			 string  public message;

			 constructor(string  memory  _initMessage) {
				 message = _initMessage;
			 }
			 function  update(string  memory  _newMessage) public {
				 message = _newMessage;
			 }
		}
- ABI file for a Smart Contract - You can use the `HelloWorld.json` sample. Learn more about smart contract ABI specifications [here.](https://docs.soliditylang.org/en/v0.8.13/abi-spec.html#:~:text=The%20Contract%20Application%20Binary%20Interface,as%20described%20in%20this%20specification.) In short the ABI file will inform your frontend how a contract's inputs & outputs work and is automatically generated when a contract is compiled. 

- [MetaMask browser extension](https://metamask.io/download/)
		 - Download & Create Account 
		 - Add & Configure Rinkeby Testnet - see screenshots below
		
<p float="left">
  <img src="https://github.com/bbehrman10/o-blogs/blob/main/MetaMask-VB/images/metamask1.png?raw=true" width="45%" />
  <img src="https://github.com/bbehrman10/o-blogs/blob/main/MetaMask-VB/images/metamask2.png?raw=true" width="49%" /> 
</p>
- Faucet test ethereum - when data is changed on the blockchain you need ETH to pay the computing gas fees, this applies to both the mainnet and the testnet. You can request ETH to be sent to your wallet from a faucet - https://faucets.chain.link/
<img src="https://github.com/bbehrman10/o-blogs/blob/main/MetaMask-VB/images/faucet.png?raw=true"/>

### Don't Want to Build It Yourself? No Problem!
You can download a fully working sample `MetaMaskVBDemo` from this repo and import it into your own instance of Visual Builder. All you'll need to do is configure MetaMask, faucet some test ETH, and test away! 

### Instructions:
#### Import into Visual Builder
Before we can begind building we need to do a couple if imports. First, we need to import our Ethers.js library. Within your application right click the resources folder and select 'Import'. Upload the `ethers.js` file saved in the repo. Then we also need to import the `HelloWorld.json` file.

<p>
<img src="https://github.com/bbehrman10/o-blogs/blob/main/MetaMask-VB/images/import.png?raw=true" width="50%"/>
</p>

#### Create Frontend Components
 Next create a frontend to interact with the deployed contract. My example has 2 main panels. Panel 1 shows if there is no wallet detected & contains a `Connect Wallet` button. Panel 2 contains buttons to both read & update the smart contract. The panels are configured via a `Bind If` and display according to whether or not a MetaMask account is connected to the application. There are also variables setup to store values from the form & from the contract.

![frontend](https://github.com/bbehrman10/o-blogs/blob/main/MetaMask-VB/images/frontend.png?raw=true)
![variables](https://github.com/bbehrman10/o-blogs/blob/main/MetaMask-VB/images/variables.png?raw=true)

#### Load Libraries & Write Javascript Functions
In this example there are 4 functions - 2 that interact MetaMask and 2 that interact with the smart contract. However before writing these functions, make sure to define & link to both the Ethers.js library, and the contract ABI file.

	define(["resources/ethers.js",  'text!resources/contractabi.json'],  function(ethers,  abiString)  {
Then we set our contract address and parse the imported ABI file:

		const  contractAddress  =  "0x06A3F540004A2ba928fcba915D41fF32C084B57b";
		const  contractABI  =  JSON.parse(abiString).abi;

Now write the MetaMask functions:
- checkIfWalletIsConnected()
	- This function utilizes MetaMask to see if a wallet is already connected with the application
		
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
	- This function tells MetaMask to connect a wallet to the application
	
			PageModule.prototype.connectWallet  =  async  function  ()  {
				try  {
					const  {ethereum}  =  window;
					if  (!ethereum)  {
						alert("Get MetaMask");
						return;
					}
					const  accounts  =  await  ethereum.request({method:  "eth_requestAccounts"});
					return  accounts[0];
				}  catch  (error)  {
					console.log(error);
				}
			};

And for our Contract:
- readContractString()
	- This function calls the contract's public `message()` function -  this is where the ethers.js library comes in when creating special provider, signer, & contract objects.
	
			PageModule.prototype.readContractString  =  async  function  ()  {
				try  {
					const  {ethereum}  =  window;
					if  (ethereum)  {
						const  provider  =  new  ethers.providers.Web3Provider(ethereum);
						const  signer  =  provider.getSigner();
						const  contract  =  new  ethers.Contract(contractAddress,  contractABI,  signer);
						let  response  =  await  contract.message();
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
							return  response;
					}
				}  catch  (error)  {
					console.log(error);
				}
			}
#### Link Everything Together:
Now that the frontend & functions are defined, all that's left to do is tie everything together with some event listeners and action chains. For each of the buttons head over to the events tab and create a new `click` event and create a new action chain for each event. Within the action chain you will need to drag in the Call JS Function action into the chain and in the drop down select the corresponding function.
![actions](https://github.com/bbehrman10/o-blogs/blob/main/MetaMask-VB/images/actionList.png?raw=true)
<p float="left">
  <img src="https://github.com/bbehrman10/o-blogs/blob/main/MetaMask-VB/images/connectWallet.png?raw=true" width="35%" />
  <img src="https://github.com/bbehrman10/o-blogs/blob/main/MetaMask-VB/images/readContractString.png?raw=true" width="30%" /> 
  <img src="https://github.com/bbehrman10/o-blogs/blob/main/MetaMask-VB/images/setContractString.png?raw=true" width="25%" />
</p>
Make sure to assign the results of the call to the response variable. For the read function the example displays the contract string. For the set string function it returns the hash of the transaction.
<p float="left">
  <img src="https://github.com/bbehrman10/o-blogs/blob/main/MetaMask-VB/images/stringsResult.png?raw=true" width="48%" />
  <img src="https://github.com/bbehrman10/o-blogs/blob/main/MetaMask-VB/images/hashResults.png?raw=true" width="48%" /> 
</p>

#### Try It Out:
Everything is set up now you can hit the play button on the application. Test your wallet & contract. Head on over to [Etherscan Rinkeby](https://rinkeby.etherscan.io/address/0x06A3F540004A2ba928fcba915D41fF32C084B57b) to track transactions as they occur. 
#### Video Demonstration
Coming Soon...
