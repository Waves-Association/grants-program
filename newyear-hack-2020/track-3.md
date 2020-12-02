# Gravity Interoperability Hackathon: integrating Matic

## Preface

We welcome you to contribute to any project or service in the field of DeFi, Web3, Digital Art, or distributed computing, based on the technologies of Solana, Waves, and Gravity, that also meets the criteria of the hackathon outlined below.

## Criteria

The core functionality of your product or service submission should be based on solving the following technical problems related to cross-chain token transfers using Gravity Protocol. It should also include interaction with external data sources. Each task describes an extension of the logical parts of the Gravity Protocol. We highly recommend that you open pull requests in specific repositories mentioned in the task descriptions while implementing your solution, which should minimize the rewriting of the existing codebase.

## Tasks

I. Gateway
Implementation of the SuSy protocol between Waves and Solana. To ensure compatibility with a new chain, a set of separate smart contracts is required. More importantly, there are two types of contracts: those used by the core of Gravity and application-specific contracts (e.g. SuSy). 

SuSy protocol, as an application layer, conducts gateway communications and is available in a separate repository. First and foremost, it is necessary to implement an IB-port (issue/burn port) that would control token emission for SuSy.

An example of an IB Port for Ethereum and Waves is below: https://github.com/Gravity-Tech/gateway/blob/main/contracts/ethereum/IBPort.sol 
https://github.com/Gravity-Tech/gateway/blob/main/contracts/waves/luport.ride

There should be 2 smart contracts in Solana and Waves networks with the next functionality:

Solana smart contract must support the external call of the attachEventData function with the next parameters:
Token ID (optional since different tokens will have their own gateways)
Amount
Receiver
Only one of 5 admin accounts have a chance to call this method.
After attachEventData Receiver must receive its amount of wrapped tokens
Holder of wrapped tokens has to have an ability to send tokens to the gateway address and this should trigger API (RPC) calls.

For all statements 1-3, an API must be implemented (open and public).

Consider checking the “contracts” directory inside the Gravity-Tech/gateway repository to see examples of gateway contracts’ source code.

After creating the contracts, they need to be compiled into bytecode. The compiled bytecode files should stay in the “/abi” directory.

II. Liquid staking gateway (IB Port)

Implementation of the SuSy gateway that supports auto-staking of WAVES, USDN, EURN tokens on a chosen target chain.

Ethereum Example with auto-staking: https://github.com/Gravity-Tech/gateway/blob/main/contracts/stakable/IBPort.sol

III. Solana – Waves gateway (LU Port)

Implementation of the LU (lock/unlock) port on a chosen target chain.

Waves Example with Lock-Unlock mechanics: https://github.com/Gravity-Tech/gateway/blob/main/contracts/waves/luport.ride 

IV. Nebulas & Extractors for both sides - Waves & Solana
SuSy Protocol contains three basic elements: ports, nebulas, extractors.

Ports (IB & LU) are essentially user smart contracts, in terms of Gravity. These are responsible for burning-issuing and locking-unlocking of a token in a specific chain. 
Nebulas are smart contracts that will interoperate with user smart contracts (ports). These contracts issue USER-SC transaction calls.
Extractors are bridges between two chains. The goal of extractors is to extract (and serialize/deserialize) data from specific target chains. The main interfaces must have two http GET methods: /info and /extract.

In the current SuSy implementation, a byte string is passed between extractors (which is a joined array of strings under the hood). Extractor’s main responsibility is to provide a specific byte string for a particular transfer request. An extractor should cache every returned request for about 10 minutes (adjustable parameter).

An example of an extractor written in Go: https://github.com/Gravity-Tech/gravity-node-data-extractor 

Any programming language is welcome, however it is recommended to extend the existing Go extractor by providing implementations for required methods rather than rewriting from scratch.
V. Adapter implementation
In order to add an implementation for any new chain (including Solana), it must first be supported by the Gravity Network itself. 
An implementation of this task should introduce changes into the gravity-core repository. For a better understanding of what kind of modifications this would entail, consider reviewing the Binance Smart Chain implementation pull request: 
https://github.com/Gravity-Tech/gravity-core/pull/32
Key things to keep in mind:
Take notice of the BlocksInterval parameter for the specific chain (required for a correct network “pulse”).
Include an implementation of the adaptors.IBlockchainAdaptor interface, which is the most descriptive part of each specific chain.

Test recommendations: try deploying contracts & starting a node in customnet to test out your newly written adapter. After testing, create a pull request to Gravity-Tech/gravity-core.
VI. Proof-Of-Reserves System
For wrapped tokens, a system is required that monitors and matches balances on the LU port with those of the wrapped tokens. If there is a discrepancy of some kind, an Emergency Call should be initiated which should record an entry in a smart contract on one of the target chains. Requires the use of Gravity’s and/or ChainLink’s oracles.
VII. Interchain OTC.
Initial liquidity for wrapped tokens on destination chains should be established to allow for cross-chain transfers. ChainLink oracles can be used as a price feed source.
Сreate a pool analogous to Uniswap, but with the price determined by oracles.

Specifications:
User can add liquidity by calling addLiquidity(token_id) function, which adds a certain amount of tokens into the pool
The user’s share is calculated as the dollar value of their share of the total pool
E.g.: A pool has 10 USDT and 5 USDN, sum total is $15.
A user adds 5 USDT to the pool, and their share is now: 5/15 ~ 33.3%

User can remove liquidity by calling removeLiquidity(token_id, amount) function, which removes a combination of token A and token B
E.g.: A user has a 33.3% share. They can withdraw $5 from the total pool of $15.
However, there is 10/15 USDT and 5/15 USDN, which means the user can withdraw (10/15) * 5 $ USDT and (5/15) * 5 USDN.

User can call trade(token_id, token_id, amount) function to swap token A to token B (if the balance of B allows the operation)

Gravity nodes can call setCurrentPrice(token_id, price) function sets the prices in dollars for token A and token B every 60 blocks.

VIII. Deployer
In order to achieve an easier deployment cycle for a specific gateway, a testing-debugging sequence is necessary. During the implementation, you will likely deploy the entire environment multiple times. We suggest that you look into the current Gravity gateway deployer: https://github.com/Gravity-Tech/gateway-deployer

git pull every submodule
Cd into a specific folder for deployment
With Go deployers, update package dependencies before deployment. For instance:
go get -u github.com/Gravity-Tech/gateway
go get -u github.com/Gravity-Tech/gravity-core

## Technical requirements
For evaluation, a technical solution to one of 5 tasks is required, covered by autotests. Programming languages: JavaScript, TypeScript, Go, Python, Rust. Smart contract languages: Ride, Rust. The solution requires documentation with flow-charts and specifications of API/methods.

## Product requirements

Solutions to I-VIII tasks should be integrated as a technological component of a DeFi/Web3 service.

Described functionality should be implemented in the given order (I–VIII).

## Useful links

Gravity White Paper – https://arxiv.org/pdf/2007.00966.pdf 

SuSy White Paper – https://arxiv.org/pdf/2008.13515.pdf

Gravity Tech Github – https://github.com/Gravity-Tech

Hackathon/Grant Application Form – https://github.com/Waves-Association/grants-program/issues
