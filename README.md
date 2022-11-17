# Rocket RPC Whitepaper

  - [Introduction](#introduction)
  - [What's wrong with current RPC solutions](#whats-wrong-with-current-rpc-solutions)
  - [How could Rocket RPC help?](#how-could-rocket-rpc-help)
  - [Why Rocket RPC is a better choice?](#why-rocket-rpc-is-a-better-choice)
  - [Let's talk about tech](#lets-talk-about-tech)
  - [Tokenomics - RPC token](#tokenomics---rpc-token)
  - [Further Steps](#further-steps)

## Introduction
After playing with NEAR for more than a year, I found NEAR protocol is such a developer friendly blockchain that enables me to easily build many fancy and powerful dapps. But what bothers me is that the RPC service for NEAR is quite unstable, I frequently had troubles loading some dapps like Paras or even the NEAR wallet due to the RPC service being down.     

For me this issue is not critical, however, for others this could cause potential loss in funds. Just imagine you are a Burrow user and trying to repay your debt when the price of your collaterals are decreasing. Unable to access the RPC will definitely cause you a headache. And unfortunately, significant price fluctuations will most likely lead to RPC being unavailable.      
      
What I'm trying to build is a new RPC service for NEAR called **Rocket RPC**. It provides different tiers of services so that each user can decide whether he needs a more stable RPC and pay for it according to his usage and the quality of service. There are some existing third-party RPC services out there like Infura, however they are just in one-dimensional: they only differentiate service quality in the user dimension. Unlike those services, Rocket RPC is of *two-dimensional*: we provide different quality of services in both the user and the dapp dimension.       
       
In order to understand how Rocket RPC could help, lets first take a look at some existing RPC solutions in NEAR and what problems do they have.

## What's wrong with current RPC solutions
- ### Official RPC
The official RPC service provided by NEAR (https://rpc.mainnet.near.org) is no doubt the mostly used RPC so far. It is maintained by the NEAR development team.      

The problems with official RPC are: 
1. All users/dapps share the same service, there is no guarantee of QoS. And this is why many dapps including the web wallet is quite unstable under heavy load.
2. When the network load is high, basically everyone will be affected and not able to access the blockchain. Even if a user do have the willing to pay for a better service, there is no such an option for him.

- ### BlockPi DRPC
BlockPi DPRC (https://public-rpc.blockpi.io/http/near-testnet) is a decentralized RPC service maintained by BlockPi team. And this RPC service is essentially a load balancer who distributes traffic across other validator node. It works better than the official one when only part of the network is busy, so that it could route requests to the idle ones.

The problems with BlockPi DPRC are:
1. When all nodes are busy, it will still be unavailable because they don't own any node.
2. Same with the official RPC, all users share the same service, no guarantee of QoS.

- ### Infura (or other similar services)
These RPC services including Infura and gateway.fm are third-party hosted, they give each user a different API key with which Infura can use to provide different tiers of service.

The problems with these services are:
1. Looks good. But in reality most NEAR users cannot use Infura service when playing with their favorite dapps because which RPC endpoint to use is up to the developer and hardcoded in the webpage of dapps, users cannot change it.
2. In this case why doesn't dapps choose Infura instead of official RPC to provide better experience to its users? The fact is they cannot simply switch to Infura. Because if they do so they will have to put the RPC's url (which contains the API key) into the source code of their webpage which could be read by anyone in the Internet. This will make their API key public, thus losing the exclusivity as long as the guaranteed quality of service.

- ### Self-hosted RPC
Besides all above solutions, one can also try to host a RPC server by himself. And the problems are quite obvious: first it is too expensive for an individual user to run an RPC server. And for dapps who want to make their service more stable. They have to build a backend service on top of the RPC server which only accepts requests to their own contract . Otherwise it will become a public one that anyone can use just like the case of Infura.

## How could Rocket RPC help?
Rocket RPC is made to solve the problems of existing RPC services mentioned above. Users can choose the level of service according to their specific needs, and different protocols can enjoy dedicated RPC services in their dapps without worrying about the leakage of api keys.         

Specifically, when a user logs in on a dapp, Rocket RPC issues a jwt token using the user's private key in the browser's local storage. Each time the user subsequently calls the RPC service, the jwt token will be passed to the RPC server, and the server will use the token for authentication and traffic distribution.        

It is worth noting that the jwt issued here is not like a signed message in Ethereum that can only represent the identity of the current user. Thanks to NEAR's unique account system, the jwt issued by Rocket RPC can simultaneously reflect the identity of the user and the contract information that the current dapp is going to interact with, including the address of the contract and the method to be called. Therefore, when the Rocket server receives this jwt, it can perform very fine-grained authentication and traffic management, which no other blockchain can do!        

For a user, when he wants to enjoy a higher level of RPC service in a certain (or all) dapp, he only needs to buy some RPC tokens and deposit them into the escrow contract (we will cover this shortly) and that's it. The Rocket RPC service will automatically deduct fees based on the user's current privilege level and RPC call usage. All billings use the RPC token, which also gives the token a strong usage scenario.       

## Why Rocket RPC is a better choice?
- ### Compared to Official/BlockPi RPC
Each protocol can replace `near-api-js` with `rocket-api-js`, the sdk provided by Rocket RPC, so that all users' access will be carried out using Rocket RPC. A user is of free tier by default but when he needs a higher quality of service guarantee, he only needs to deposit RPC tokens and it will take effect immediately. However, if the official RPC is used, users cannot choose to improve the quality of service at all.

- ### Compared to Infura
As we have discussed, Infura on NEAR has very few usage scenarios. Except for cli users and sender wallet users, Infura cannot be selected in any dapps. The reason is that it is impossible for wallets or dapps to use Infura publicly in their websites, which would reveal their API keys.       

With the Rocket RPC, there is no need to worry any more. Each user will get an exclusive jwt token after logging in,  dapps will no longer have any globally-scoped tokens. The problem of api key leakage is eliminated.

## Let's talk about tech
Rocket RPC is made up of the following components:

- ### rocket-api-js
> repo: https://github.com/near-rocket-rpc/rocket-api-js       

In order to generate jwt token when a user logs in, `rocket-api-js` is built upon `near-api-js` and is designed to be compatible with its interface, which minimizes the code change for developers to adapt.       

To login with `rocket-api-js` is very simple: 
```javascript
  import { connect } from 'rocket-near-api-js';

  const keyStore = new keyStores.BrowserLocalStorageKeyStore();
  const connectionConfig = {
    networkId: "testnet",
    keyStore: keyStore, // first create a key store 
    nodeUrl: process.env.RPC_NODE_URL,
    walletUrl: "https://wallet.testnet.near.org",
    helperUrl: "https://helper.testnet.near.org",
    explorerUrl: "https://explorer.testnet.near.org",
  };

  const near = await connect(connectionConfig);
```

When a logged-in account is detected, `rocket-api-js` will automatically use the stored function call key in local storage to issue jwt token, and use this as the credential for subsequent communication with the Rocket RPC server.        

The token here is in line with the official JWT standard, so theoretically any client or server can verify this token.       

- ### Rocket Smart Contracts
> repo: https://github.com/near-rocket-rpc/rocket-contracts         

    - #### RPC Token Contract
      Users who want to enjoy higher-level of RPC services need to pay, and in Rocket RPC this is through the RPC Token. RPC Token is a standard NEP141 token. Every time a user requests the Rocket RPC service (non-free version), a certain amount of RPC Tokens will be deducted.    
      
      The contract is deployed on testnet at: `token.rocket0.testnet`
      
    - #### Escrow Contract
      The escrow contract is used to store the token that the user intends to pay for the RPC service. Since the NEP141 standard does not support the approve mechanism like Ethereum, we require users to deposit their RPC tokens to the escrow contract in advance.       
      
      Aftet a use deposits the RPC token, the Rocket server will be able to use the escrow contract to deduct fees according to his service level and usage.

      The contract is deployed on testnet at: `escrow.rocket0.testnet`

- ### Rocket RPC Bakcend Service
> repo: https://github.com/near-rocket-rpc/rocket-rpc-service         

The backend service is the core of Rocket RPC. In general, the Rocket RPC service will be responsible for processing the request sent by the client, extracting the jwt from it and performing authentication. After the authentication is completed, the request will be route to an upstream node according to the service level of the user. In addition, the Rocket RPC service will also deduct the user's RPC token according to the user's usage, and will also monitor the change of user's RPC token balance in order to respond to the service level change in real time.         

Here are detailed explanation of some core modules:     

1.  JWT authentication module

    The Rocket server must authenticate the jwt tokens sent by clients. A jwt will contain the following information: encryption algorithm, NEAR account id, public key that issued the jwt and digital signature.

    ```
    sample jwt token:
    eyJhbGciOiJFRDI1NTE5IiwidHlwIjoiSldUIn0.eyJzdWIiOiJsb2NhbG5ldy50ZXN0bmV0IiwicHVia2V5IjoiZWQyNTUxOTpBZmc1SDdHM01xcUtNVjZ0TmY3cWZUZzI2RVRwa1E3WXpNemEzOUQyWldYcSJ9.tZwbAVVJhn5RwXf_ORFDJH-KanS1AzRyA4YrPHu-TpHvXZrGTAZEVuSEDwKp2SHCj8DzGykt3b683j_p6Fw9DQ

    decoded jwt token body:
    {
      "alg": "ED25519",
      "typ": "JWT"
    }
    {
      "sub": "localnew.testnet",
      "pubkey": "ed25519:Afg5H7G3MqqKMV6tNf7qfTg26ETpkQ7YzMza39D2ZWXq"
    }
    ```
    
    The Rocket server will first query the NEAR blockchain of whether the public key is owned by the specified user, and record the scope of the public key (which contracts and methods can be called).     
    
    When the ownership of the public key is confirmed, the public key will be used to verify the signature part of jwt, and if the verification is passed, the user is allowed to access.

2.  Proxy module

    From the perspective of HTTP requests, the Rocket server works like a proxy, accepting client requests and forwarding these requests to the corresponding upstream RPC servers according to the user's service level.
    
    The proxy module is implemented using the `koajs` and `koa-http-proxy` libraries, and has an internal rate limiting mechanism to protect the upstream server when a large number of requests arrive.


3.  Charging module

    Each request of the user will cost RPC tokens, but it is obviously not feasible to deduct the fee on-chain every time, since the gas fee will become uncontrollable. Therefore, the Rocket server will record user actions in a Postgres database first. Then a regular cronjob task will call the escrow smart contract to perform the actual deduction operation.
    
    Due to the delayed deduction mechanism, the Rocket service must be able to track changes in the user's RPC token balance in real time, otherwise there may be problems such as overspending. And that's where NEAR Lake comes in.
    
4.  NEAR Lake
    
    We use [NEAR Lake framework](https://github.com/near/near-lake-framework-js) to listen to user deposit events in the Escrow contract. In this way, whenever the user's deposit changes, the Rocket server can be notified in time. This allows the Rocket server to completely maintain the user's balance change history in the escrow contract, thus ensuring the accuracy of the deduction.           


- ### Rocket RPC Dashborad
> repo: https://github.com/near-rocket-rpc/rocket-dashboard         

In order to allow users to better understand their service usage, we also made a dashboard to display the user's RPC token balance, current service level, historical RPC call records and other information.      

The dashboard uses websocket to communicate with the Rocket server, so that the user's RPC usage information can be displayed to the user in real time.     

![screenshot](dashboard.jpg)

- ### Overall Workflow

The overall workflow of Rocket RPC looks like:
![workflow](workflow.png)

## Tokenomics - RPC token
The last thing I want to discuss is the tokenomic model of RPC token. There are many ways for it to capture value and make both users and protocols benefit.

- From protocols perspective
    - Protocols on NEAR (like Ref.finance) can collaborate with Rocket RPC to provide a better RPC service to all its user, no matter whether they are already using Rocket RPC or not. To be specific, a protocol can sign a contract with Rocket RPC in the form of token swap so that Rocket RPC can use a dedicated RPC route for all its users by default. This means a lot to the protocol since it can significantly improve its availablity during high traffic load. And for Rocket RPC, the treasury now holds tokens of another protocol, this will be used to support the price of RPC tokens.
    - Ptotocols can also earn by users accessing their dapps. Remember we will charge users for every RPC calls they made. We will give back part of the profits back to the protocol in the form of RPC tokens. In this way the team and community of each protocol can be incentived, which could eventually benefit the entire NEAR ecosystem.

- From users perspective
    - Users need to pay RPC tokens for service and this also provides RPC token a way to deflate. They can purchase tokens from DEX or CEX and deposit to the escrow contract, when they are charged, RPC tokens will be burned.
    - However since the model requires users to deposit tokens in advance for their future usage, there could be a huge opportunity cost because they could have used those tokens in other places to make more profits. In order to solve this problems we could utilize the staking machanism. Both RPC tokens and tokens of other protocols, which come from the collaborations mentioned above, can be used to incentivize deposition. 

## Further Steps
The current version of Rocket RPC is just a POC for the metaBUIDL hackathon. I believe there are a lot more to build to make it production ready. The most important ones I can think of are:

1. A full management system for RPC nodes. It should be able to auto-scale nodes, monitor their status and upgrade the NEAR daemon version.
2. Improve the traffic routing algorithm to provide better service to all our users.
3. Pitch major protocols to collaborate with us and use Rocket RPC in their dapps.
