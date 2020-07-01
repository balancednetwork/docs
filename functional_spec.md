# Balanced Network


Functional Specification Version 0.1.0

June 30th, 2020


## Purpose

This document will bring clarity and agreement to the functional specifications and development process, leading to faster and more efficient implementation.


## Objective

A DAO comprises people, smart contracts, and the interfaces between them. This specification describes the architecture, design, and implementation of the smart contracts, web interfaces, and back-end servers for the Balanced Network DAO.

The initial objective is to build a scalable system of frontend, backend, and smart contracts to support the Balanced DAO as described in the whitepaper. It should scale to many users, and be extensible at a later time to many types of pegged assets. This functional specification describes in detail how that will be accomplished.


## Summary

The Balanced Network DAO will incentivize depositing ICX collateral to back loans of tokens, the value of which will be pegged to real-world assets.



*   Deposited ICX will be staked and the value of the participant’s collateral will increase daily as staking rewards are claimed.
*   Tokens are minted at the time of loan creation and destroyed when they are returned.
    *   If tokens are returned by the original borrower, they simply reduce the loan balance.
    *   If tokens are returned by someone who does not have a loan, a proportionate amount of every borrower’s collateral is liquidated to raise the funds needed to pay out an equivalent value in sICX.

The DAO will incentivize depositing collateral and taking out loans by issuing Balance Tokens, which will reward the holder with dividends produced from platform fees, and control over the votes of the ICX stored in the Staking Pool.

Pegged assets are guaranteed their value in ICX by the Balanced DAO. If, at any time, a holder of a pegged asset would like to retire this token in exchange for its equivalent value in ICX, the Balanced DAO will honor such an exchange.


## Table of Contents

* [Balanced Terminology](/functional_spec.md#Balanced-Terminology)
* [Front End Module](/functional_spec.md#Front-End-Module)
* [Backend Module](/functional_spec.md#Backend-Module)
* [Smart Contracts](/functional_spec.md#Smart-Contracts)
* [Equations and Algorithms](/functional_spec.md#Equations-and-Algorithms)
* [Flow of a Position](/functional_spec.md#Flow-of-a-Position)


## Balanced Terminology

This section will cover all terminology relevant to Balanced in order for readers to better understand this paper.

General terminology:

*   **Loan** - A borrower’s outstanding debt
*   **Collateral** - The ICX deposited into Balanced needed to take out a loan
*   **Position**- The combination of a Borrower’s collateral and loan
*   **Bad debt** - Debt that is accrued by Balanced when a position is liquidated
*   **Staking Pool** - A smart contract that stakes, delegates, and claims rewards for all ICX deposited into Balanced
*   **Oracle** - A software service that provides accurate and timely pricing information to inform Balanced of the value of pegged assets



Types of users:

*   **Borrower** - Any user of Balanced DAO that has deposited collateral and borrowed a pegged asset
*   **Trader** - A user that retires pegged assets without a position
*   **Liquidity provider** - A user that provides liquidity on the Balanced DEX and mines BAL for this service



Types of assets:

*   **ICX** - the native cryptocurrency of the ICON public blockchain, and the only cryptocurrency used as collateral in Balanced
*   **sICX** - a representation of a user’s share of the ICX held in the Staking Pool
*   **Pegged asset** - Any token originated by borrowers on Balanced
*   **ICON Dollar (ICD)** - A pegged asset pegged to the value of 1 USD
*   **Balance Tokens (BAL)** - The token that entitles the holder to a share of dividends from the Balanced DAO



Types of fees:

*   **Origination fee**- The fee paid after borrowing a pegged asset
*   **Decentralized exchange fee** - The fee paid when executing trades on the Balanced Decentralized Exchange
*   **Arbitrage fee** - The fee paid by traders when retiring pegged assets



Types of events:

*   **Retire** - When a trader exchanges a pegged asset for its equivalent value in sICX
*   **Repay** - When a borrower returns pegged asset and lowers their position
*   **Mining** - When a user receives BAL from the Balanced DAO for completing certain actions
*   **Locked** - When a borrower’s collateral is locked as a result of under-collateralization
*   **Liquidation** - When a borrower’s collateral is liquidated as a result of under-collateralization



Types of ratios:

*   **Mining ratio** - The ratio of collateral value to loan value that allows a borrower to mine BAL. This ratio is 500% and a borrower must be above this ratio to mine BAL
*   **Mandatory collateral ratio** - The ratio of collateral value to loan value that causes the borrower’s collateral to be locked. This ratio is 400%
*   **Liquidation ratio** - The ratio of collateral value to loan value that causes a liquidation. This ratio is 150%


## Front End Module


### Overview

The Balanced front end is designed to be accessible to as many people as possible. It will be fast and easy to use, so anyone with ICX can open a position within their first 30 seconds on the platform.

Balanced has three pages: Home, Vote, and Trade. Each page is made up of sections that help users perform specific tasks. They're designed to be modular, so it's easy to add, remove, expand, or reposition sections as Balanced evolves.

In the first iteration, most tasks assume a single pegged asset – ICON Dollars – and it will be modified to accommodate additional tokens later on. Until then, all values will be in USD unless otherwise stated.

The details that follow describe how a user will interact with the Balanced front end to complete specific tasks.

[Try the interactive prototype](https://balanced.network/prototype/) to see how much of this looks and works in practice.


### Authentication

Balanced users can sign in using the ICONex extension in their browser, or through a DApp browser like the one in MyIconWallet. They'll need to sign every transaction unless they turn on auto-sign. The tasks on each page assume they haven't.


### Home

The Home page consists of 6 sections: 

*   Collateral and Loan (together referred to as the Position Manager)
*   Position Details
*   Wallet
*   Rewards
*   Activity

With them, a user can: 

*   Open a position
*   View and manage a position
*   View and manage assets
*   Claim rewards
*   View activity
*   Switch wallets
*   View and manage notifications


#### Open a position

A user can open a position by using the Position Manager to deposit collateral and take out a loan.

**Deposit collateral**

Any unstaked ICX in a user's wallet is shown as available in the Collateral section. To deposit it, the user clicks Adjust and uses the slider or text field to choose the amount.

As they adjust it, other values in the Loan and Position Details sections also change so they can preview the outcome:



*   Amount available to loan / Loan limit
*   Collateral balance

When the user clicks Confirm, a prompt will confirm the amount of collateral to deposit, the collateral balance before and after, and a note about the unstaking period. The user clicks Confirm, then signs a transaction to deposit collateral.

**Borrow a pegged asset**

The user then follows a similar process to borrow a pegged asset. They click Adjust in the Loan section, and as they use the slider or text field to choose the amount, other values in the Collateral and Position Details sections also change so they can preview the outcome:



*   Amount of locked collateral
*   Loan value
*   Their percentage of debt on the network
*   Locked and liquidation prices
*   Risk ratio

When the user clicks Confirm, a prompt will confirm the loan amount, the loan balance before and after, and the origination fee. The user clicks Confirm, then signs a transaction to borrow the pegged asset.


#### View and manage a position

The user can then monitor their position from the Position Details section, which includes:



*   Collateral amount
*   Loan amount
*   Loan limit
*   Their network debt 
*   ICX price when all collateral is locked
*   ICX price when all collateral is liquidated

The risk ratio helps the user assess their open position. It ranges from 900% to 150% (unless their risk ratio is lower) and points out the:



*   Mining ratio at 500% (Earn threshold)
*   Mandatory collateral ratio at 400% (Locked)
*   Liquidation ratio at 150%

Rebalancing shows the user how much of their collateral has been sold to repay their loan. They can click to see the numbers for the last day, week, or month, and use the chart to compare that data to previous intervals.

The user can use the Position Manager to adjust their position based on this information, with a few caveats:



*   They can only withdraw collateral up to the Locked position. They'll need to repay their loan to withdraw everything.
*   They can only repay their loan up to the Used position. They'll need to buy more of the pegged asset to repay the rest.

When the user hovers over the locked or used portion, a tooltip will tell them the specific amount and how to access it.

When a user adjusts their collateral or loan, the values in the Collateral, Loan, and Position Details sections update dynamically so they can preview the change.

If a user withdraws collateral, they're prompted to swap their sICX for ICX for a small fee.


#### View and manage assets

The user can check their assets in the Wallet section. They'll see the balance and current value of each. To interact with a specific asset, they can click the row or the button on the right to expand the section. Most assets will only have actions like send and stake, but ICX will include additional details, like how much is available or held in staking contracts.

**Send an asset**

Most assets have Send as the primary call to action. The user clicks it to open the send interface, then enters the address and amount to send. They then click Send and sign the transaction.

**Stake Balance Tokens**

The primary call to action for Balance Tokens is Stake. The user clicks it, the section expands, and they use the slider to adjust the amount of Balance Tokens staked. They then click Adjust and sign the transaction.

If they choose to reduce their BAL stake, they'll see a message that the unstaking period takes 3 days.


#### Claim rewards

The user can check the Rewards section to see how many Balance Tokens and network fees they've earned since they last claimed, and the current USD value.

The user clicks “Claim rewards”, and sees a prompt to confirm and stake their new Balance Tokens

The user adjusts the staked amount with the slider, then clicks Stake and signs a transaction.

The first time a user stakes Balance Tokens, they'll be prompted to visit the Vote page to choose P-Reps to vote for. 


#### View activity

The Activity section shows Balanced transactions that affect the user's position:



*   Collateral deposited and withdrawn
*   Loan borrowed and repaid
*   Assets sent or received
*   Balance Tokens mined
*   Network fees earned
*   Position rebalanced
*   Position liquidated

The user can click Date, Activity, or Amount to adjust the sorting, and use the filter to view specific activities, like payments or position changes.


#### Change wallets

The user can click the wallet icon in the top right to:



*   Reveal the wallet list
*   Add another wallet
*   Switch between wallets
*   Sign out

To retrieve the address for their active wallet, they can hover over it and click to copy it or view a QR code.


#### View and manage notifications

After the user opens a position, a badge will appear on the notification icon in the top right. When they click it, they'll see a message about system/browser-level notifications: turn them on, but don't rely on them entirely.

Notifications may include:



*   Alerts as the risk ratio increases:
    *   Adjust your position to earn Balance Tokens (mining ratio)
    *   Your collateral will be locked soon (mandatory collateral ratio)
    *   Your collateral is close to liquidation (forced liquidation ratio)
*   Claim rewards
*   ICX/BAL have finished unstaking and are available to use
*   Incoming payments
*   Successful trades
*   Trades that fall outside the 3% reward range for sICX/ICX

The user can adjust which notifications to receive in the settings (yet to be designed).

When a user receives a notification, a click will take them to the right place to act on it.


### Vote

The Vote page consists of 3 sections:



*   Vote Details
*   P-Rep Updates
*   P-Reps

With them, a user can: 



*   View and adjust votes
*   Discover and vote for P-Reps

Parts of the vote interface are still being designed: the voting flow, vote history, and inner P-Rep pages.

Users will eventually participate in governance for the Balanced DAO here, but it won't be included in the first release.


#### View and adjust votes

At the top of the Vote page are the user’s vote details:



*   Voting power for 1 BAL
*   Number of BAL staked and unstaked
*   Their own voting power

They can use the slider to quickly adjust their Balance Token stake, which also adjusts their voting power. After they click Confirm, they'll need to sign a transaction.

The pie chart shows the user their current vote allocation. Only the top 5-6 P-Reps will be displayed beside the chart. If the user voted for more, they can click View all to see every P-Rep and the amount of votes allocated to them.

The first-run voting flow is still being designed, but will include the option for users to evenly distribute their votes amongst the top 100 P-Reps to reduce the impact of apathetic voters.

To change their votes, the user clicks Adjust. They'll see how many votes are unallocated, and can use the sliders to change how many to give to each P-Rep. They can also use the search box to find and add other P-Reps to vote for.

When the user is finished, they click Confirm to save their changes, then sign a transaction.

If the user wants to see Balanced vote details, including the network stake, network power, and the overall vote allocation, they can click to view it.


#### Discover and vote for P-Reps

**View P-Rep updates**

When a P-Rep publishes an update for a project on the[ icon.community website](https://icon.community/iconsensus/prep_all_projects/), it appears as a P-Rep update in Balanced. The P-Rep Updates section shows this information for the three most recent:



*   P-Rep
*   Project
*   Update title
*   Update summary
*   When it was published

The user can click to view all P-Rep updates in this summary format, ordered from most to least recent.

If the user clicks an update, they're taken to the P-Rep's project page to view the full update and other project details. Breadcrumb navigation will appear at the top to help the user navigate through the inner pages.

**Search and browse P-Reps**

If the user wants to find a specific P-Rep, they can use the P-Rep section to search or browse the complete list. Each P-Rep listing includes their:



*   Current rank
*   Active and completed projects
*   Monthly earnings (ICX and USD)
*   Votes (number and percentage)

A user can click any of the headings to reorder the list, although Rank, Earnings, and Votes will return the same result.

A user can click a P-Rep to view their page. Here they'll see more details about the team, all their latest updates, and their complete list of projects, organised within several tabs. The user can click between them to find the information they're most interested in.

If the user wants to vote for a P-Rep while still on their page, they can click Vote to open a pop-up, and use a slider to set the amount. They'll then click Confirm and sign the transaction.

When the user wants to go back to the main Vote page, they can click Vote in the breadcrumb navigation, or in the sidebar.


### Trade

The Trade page consists of 4 sections:



*   Trade Manager
*   Order Book (Buy Orders and Sell Orders)
*   Open Orders
*   Trade History

With them, a user can:



*   Swap assets
*   Place a limit order
*   Retire pegged assets
*   Provide liquidity to earn Balance Tokens
*   View and cancel open orders
*   View trade history


#### Swap assets

The Trade Manager defaults to the Swap interface. With it, a user can quickly swap one asset for another at the current market price. To choose which assets to swap, the user clicks the asset selector in each field. It displays the user's balance for each asset, and sorts them by highest to lowest value.

After one asset has been chosen, the other asset selector will only list the assets available to swap it with.

When the user fills in one of the fields, the other field will automatically fill with the correct value. The user then clicks Swap and signs the transaction.


#### Place a limit order

To set a specific price, a user opens the Limit tab and clicks the pair selector. It includes the current price for each trading pair to help them preview the information faster. After they choose a pair, they can use the chart to view the current price, the price action for different time intervals, and mouse over the chart to see the price at any moment.

If the user wants to reverse the pair, they can click Reverse. They can then set the price and amount, click Buy or Sell, and sign the transaction. Their order will appear in the Open Orders section until it's either filled or cancelled.


#### Retire pegged assets

To retire a pegged asset, a trader can click a button next to the Earn tab to sell it for an equivalent amount of sICX. They choose their pegged asset and see the approximate amount of sICX they'll receive. After they complete the order, they'll be prompted to trade their sICX for ICX for immediate liquidity.


#### Provide liquidity to earn Balance Tokens

If the user goes to the Earn tab, they'll learn that they can earn Balance Tokens if they buy sICX through the sICX/ICX pair within 3% of the actual price, which is determined by the amount of ICX in the staking pool. They can place their orders from the Limit tab, or use the Earn tab to also monitor their open orders within the 3% price range.

When the user executes trades at a price within 3% of the actual price of sICX, they’ll see the expected amount of BAL they could earn at the end of the day. This is based on their trading volume within the 3% range relative to the total trading volume within the 3% range.

When a liquidity provider receives Balance Tokens, a notification directs them to the Home page to claim and stake them. The first time they stake BAL, they'll be prompted to go to the Vote page to choose P-Reps to vote for.


#### View and cancel orders

From the Open Orders section, a user can view all their open limit orders, including:

*   Date and time opened
*   Trading pair
*   Order type
*   Asking price
*   Amount
*   Percentage filled

The user can click any heading to reorder the list. If they no longer want an order, they can click Cancel and sign a transaction to close it.


#### View trade history

The Trade History section shows information for all of a user’s trades. To reorder the list, the user clicks any of the headings. To view the details on the blockchain, the user can click a transaction address.


## Backend Module


### Overview

The Balanced backend will be event-driven from the ICON blockchain. The goal of this service is multiple, easily-deployable services so the Balanced team or a community member can run the UI application. The backend module only stores data that can be derived from the blockchain.

The service is broken into multiple components:



1. **Blockchain Worker** - This is a miniature “explorer” service that parses blocks and pulls out relevant transactions. It also has a wrapper to submit signed transactions to a node, so users can interact with a citizen node to pull data from the Balanced blockchain without running their own copy.
2. **Balanced Core** - This receives and processes messages from the blockchain worker, updating user balances and related information inside a centralized datastore. Loans, DEX, and other messages will all be computed and updated here.
3. **API Service** - This will power a websocket and REST API to inform end users about different events in the Balanced Network. This will include depth charts, order books, and trades on the DEX.
4. **Liquidation Monitor** - This will receive data from Balanced core & API service, and will call operations, such as liquidate, on a particular user account, and any other operation in the platform with a bounty on it. This service will likely be the most downloaded by the Balanced community, as anyone running it on a small server could stand to earn money in liquidations on the network.


#### Architecture

Each of the services in Balanced will be implemented as standalone microservices inside of containers. This allows for the simplest community deployment of the product, with a single container orchestration file needed to download the entire stack and run locally. In production, these services will be spread across multiple instances. 

**Backend:** Nodejs microservices - A collection of services will parse data from the blockchain and inform end users

**Message Queue:** Kafka - Messages will be passed between services through kafka topics 

**Datastore:** Redis - Rapidly accessed and changing data will be held in redis

**Permanent Datastore (backups/restores):** MySQL - Long term histories and a permanent record will be stored in MySQL


### Blockchain Worker

The Blockchain Worker consists of a Block Parser (parses out incoming blocks) and a Transaction Parser (parses out incoming transactions).


#### Block Parser

The Block Parser holds the pending block in redis and checks to see if any transactions held in the block contain information relevant to Balanced. Such relevant information includes:



*   Invocation/token transfer of a Balanced ecosystem token
    *   sICX
    *   Balanced (BAL)
    *   ICD (or any loaned asset)
*   Staking Contract Address
*   Loans Contract Address
*   DEX Contract Address

When the Block Parser finds relevant information, it emits a message on the transactions topic containing the below information:



*   txn hash
*   block number
*   block hash
*   transaction type (matched contract address)
*   from address
*   to address
*   status = pending

Once the block is fully processed, it will be updated in redis and a message will be sent to the blocks topic noting that the block is finished processing.


#### Transaction Parser

The Transaction Parser handles all incoming transactions and grabs detailed transaction information about the result, informs users of token transfers, and interacts with the Loans smart contract.

If the transaction has failed, the Transaction Parser will update the transactions topic with a failed status. If the transaction is successful, the Transaction Parser will grab information from the transaction event logs and results in order to determine how to handle the result.

The Transaction Parser is also responsible for handling token transfers. It updates the address’s balance with the new amount and sends a message to the balance.update topic to inform the user of the token transfer.

The Transaction Parser interacts with the Loans Contract when users add collateral, when transactions must be replayed, when users repay their debt, and when users are liquidated. 

When adding collateral, the Transaction Parser will send a message to balance.update to inform users of their new balance. When a user has transactions that must be replayed, the Transaction Parser will replay transactions, note the user’s current spot in the operation log, send a positions.update, and send a balance.update message. 

When a user repays their debt, the Transaction Parser will replay transactions to get the user up to date in the operation log, replay any activity on the Loans contract and send the remaining balance to the user. Finally, in the case of a liquidation, the Transaction Parser will update the balance of the user and update their position in the Loans smart contract.


### API & WS endpoints

The Balanced API will be primarily websocket driven, with some RESTful API endpoints for submitting transactions. Users can subscribe to topics at the level of a particular address to get informed of balance updates or others. All APIs will be public, therefore a user could subscribe to another address if they so choose. API endpoints and their responses are outlined below.

**Asset Subscription** 

Endpoint: assets.subscribe, params [“hx…3”]

Responses (per asset): assets.update dexFree [20000], dexLocked [10000], wallet [“hx...3”], collateral [50000], unrealizedLosses [-1.232]

**DEX Price Subscription**

Endpoint: prices.subscribe [“ICDICX”, “SICXICX”]

Message (per trading pair): prices.update Last [.33], Mark [.30]

**DEX Market Depth Subscription**

Endpoint: depth.subscribe [“ICXICD”]

Responses (per trading pair): depth.update Market [“ICXICD”], Bids [.88, 10000], Asks [.89, 9000]

**DEX Open Orders Subscription**

Endpoint: order.subscribe [“hx….2”, “SICXICX”] (* for all markets, market name otherwise)

Responses: order.update Market [“SICXICX”], id [123], type [buy, sell], price [11.22], amount [23], filled [14], user [“hx...2”], last [14], event [created, updated, filled, cancelled]

**Oracle Price Subscription**

Endpoint: oracle.subscribe

Reponses: oracle.update Market [“ICXUSD”], Price [.34112], Time [19281820192]

**Loans Contract Subscription**

Endpoint: loans.subscribe

Responses: loans.update totalCollateral [11.2], totalDebt [11.5], badDebt [1]

**Overall Balanced Subscription**

Endpoint: balanced.subscribe

Response: balanced.update liquidationPool [11.2 ICX]

**sICX Price Subscription**

Endpoint: staking.subscribe

Response: staking.update Rate [0.889]

**Transactions Subscription**

Endpoint: transact.subscribe, params [“hx...2”]

Response: transact.update Status [“confirmed”, “error”]


#### Architecture

The architecture of Balanced leverages an API + Websocket design and utilizes the following technology stack:

**Message Bus - Kafka:** Kafka topics are used to receive updates from the core backend service

**Historic & Recovery Database - Mysql:** Keep a journal of historic events on a mysql to read out in the event of a crash

**In-memory Database - Redis:** Store data within redis to track user balances

**Backend - Nodejs and uSocket:** API layer to manage subscriptions and push events to users


#### Integration

Users will authenticate through a public websocket API. Subscriptions will be handled at the level of global subscribes to market-level data, such as depth, or user level data, such as a particular user’s open orders.

The Balanced frontend will use this layer to get a real-time feed of user data and will receive updates based on the current state of the network. The frontend will also submit signed transactions to the backend via the API.


### Liquidation & arbitrage monitors


#### Liquidation

The liquidation monitor will be an external package, subscribing to different socket APIs and processing the outputs. It will run as an installable docker image, with a few params configurable by a user:



*   Keystore
*   Password
*   Network ID
*   Balanced Contract addresses

It will subscribe to the following socket APIs to determine if liquidation criteria are met:



*   Oracle - On each new prices event, it will multiply the debt vs price of each position times collateral, to determine if liquidation criteria is met
*   Loans - On each new position increased, repay or retire event, it will update the price of a position
*   Liquidations - On each liquidation event it will update the retired amount

As soon as the threshold is met, the liquidation bot will sign and submit a liquidation transaction to an ICON citizen node. After submitting a liquidation transaction, the bot will determine whether or not it was first to trigger liquidation and earn a reward; if not, it will note an error to the console.


#### Arbitrage

The arbitration bot will hedge the tokens on the Balanced DEX against a centralized exchange, and provide an open API for handling this behavior (similar to the Kelp bot on the stellar network). The bot will have three modes of operation:



1. Terminate against centralized exchanges 
2. Keep within a tolerable range to unstake ICX (sicx pairs)
3. Retire on the loans contract (pegged asset pairs)

If it can terminate a similar order on the orderbook of another exchange, it will place an order on the Balanced DEX. Minimum profit margin will be set (in terms of percentage) of ICX.


## Smart Contracts


### Overview

Balanced is structured as a DAO and as such will be powered almost entirely by smart contracts. The diagram below details the relationship between the 6 core smart contracts that power Balanced: Governance, Staking, Loans, Rewards, Dividends, and DEX. Outside of the 6 core smart contracts, Balanced will have IRC2 contracts tracking ownership and supply of the tokens associated with Balanced, which will initially be Balance Tokens (BAL), Staked ICX (sICX) and ICON Dollars (ICD).


![alt_text](/assets/contract_model.png "Balanced Contract Model")


Contract Model Diagram

When any state-changing external method is called, the transaction may also be used for driving the operations of the DAO. By spreading the work needed to operate the DAO over many transactions, the additional cost per transaction is kept low. While we will have to wait until implementation on the testnet to get a more accurate assessment of these fees, it is expected that they will be on the order of .008 ICX or less. Exceptions to that may occur during distribution of BAL tokens or ICX dividends, when the fees may be as high as .1 ICX if there are few transactions and a lot of calculations to complete.


### Token contracts


#### sICX

Standard IRC2 token contract plus mint and burn methods controlled by the sICX Staking Management contract


#### ICD

Standard IRC2 token contract plus mint and burn methods controlled by the Loans contract. The transfer function for this token contract will include a configurable fee parameter that will initially be set to zero. This will allow for a transfer fee to be imposed at a later date, as may be decided by DAO vote. Such a change will be controlled by the governance contract. Any collected fees will be directly transferred to the dividends contract. An oracle method on this contract will retrieve and update a timestamped value for the asset in ICX.


#### BAL

Standard IRC2 contract plus staking control and a mint method controlled by the Rewards contract. This contract will maintain two synchronized copies of token balances. When the Dividends contract needs a snapshot of the balances and staking status for calculation of dividend distributions, one of the copies will be frozen. Once distributions are complete, the copies will be brought back into synchronization.

As other pegged assets are added to Balanced, they will have the same specifications as ICD, with an independently defined transfer fee requirement.


### Core system contracts


#### sICX Staking Management

This contract will hold all ICX on deposit with Balanced. It will maintain an exchange rate between ICX and sICX, which is strictly determined by the ratio between ICX held on the contract and sICX total supply. 

When ICX is received, the contract will call for sICX to be minted and returned in an amount determined by going exchange rate. Any holder of sICX will be able to return it to this contract in exchange for ICX at the going exchange rate. A return will burn the sICX and start an unstaking process on the amount of ICX due.

This contract will also offer stake and vote methods to manage the staked ICX. The stake method will be an external method that can be called by anyone to check for, claim and restake I-score to the currently voted P-reps.

A vote method, which will only be callable by the Governance contract, will specify vote allocation of the staked ICX among the top 100 P-Reps. Restaking of I-score will also be driven by any other transaction called on the contract. When a deposit or withdrawal operation is called, the contract will first call the stake method to claim and stake any I-score, which ensures that the exchange rate for sICX is up to date.


#### Rewards

The Rewards contract sends out daily mining rewards of Balance Tokens as an incentive for certain behavior, including holding debt positions and providing DEX liquidity. The daily allocation of Balance Tokens will be distributed according to configurable parameters, currently {“debt_position”: 55, “DEX_liquidity”: 10, “continued_operations”: 30, “emergency_fund”: 5}, and controlled by the governance contract. 

The Rewards contract will also be responsible for governing the number of Balance Tokens mined per day. At the beginning of each new day, the Rewards contract sends a request to the Balance Token contract to mint the current day’s allocation of tokens, sends the tokens to the DEX (all miners will claim their BAL from the DEX), then begins reading batches of data from the Loans contract and the DEX on subsequent transactions. 

There are two ways to mine BAL, debt mining and DEX mining, and the Rewards contract is responsible for tracking both. As the Rewards contract reads batches of debt position data from the Loans contract and liquidity provider transaction data from the DEX contract, it adds entries of debt position mining power and DEX liquidity mining power for each address and sums up the total system mining power for each type of mining.

Allocation of tokens to each address is calculated and written to the DEX in batches. The formulae for distribution of newly mined BAL are as follows:

_Debt Position Mining_: Mining Rewards for an Address = 0.55 x Total Daily BAL x debt mining power for the address / Total debt mining power for that day.

_DEX Liquidity Mining_: Mining Rewards for an Address = 0.10 x Total Daily BAL x DEX liquidity mining power for the address / Total DEX liquidity mining power

_Mining Rewards for Continued Operations_: Allocation = 0.30 x Total Daily BAL

_Mining Rewards for the Emergency Reserve Fund_: Allocation = .05 x Total Daily BAL

Balanced distributes fees to token holders on a weekly basis. At the end of each week, this contract will call the Dividends contract to trigger a batch of distributions. If distributions have not completed, it will continue to call the Dividends contract on successive transactions until distributions are complete. Upon completion of dividends distribution, it will continue to the allocation of newly mined Balance Tokens.


#### Dividends

The Dividends contract receives fees and allocates fees to staked BAL on a weekly basis. All dividends will be held on the Dividends contract until they are claimed. Each individual participant must claim their dividends. Fees will be sent to the Dividends contract as they are incurred by transactions on the Loans, DEX, ICD, and other pegged asset contracts.

This contract will calculate and assign all dividend allocations at the end of each week. As with the Rewards contract, a snapshot of the database of staked BAL holders will be taken at the end of the week and will be used to allocate dividends. 

The snapshot will occur prior to the distribution of newly mined BAL and will be stored on the BAL token contract. When a transaction triggers distributions, a batch of staked token balances will be read from the BAL token contract, then the allocation of dividends will be calculated and sent for each of those addresses. This will repeat with successive transactions until the distributions are complete.

A read-only method will be provided to get dividends payouts for any previous week in order to serve historical data requests.


#### Loans

The Loans contract keeps track of debt positions for all pegged assets that have been minted by borrowers, and all sICX held as collateral will reside on this contract. The standing of a participant’s account will be determined by the ratio between the value of collateral deposited and the sum of the values of all outstanding pegged assets borrowed by the user. At the end of each day, the standing for every account will be calculated to determine whether the participant’s account is:



*   In good standing and earning Balance Tokens.
*   In good standing, but not earning Balance Tokens.
*   Locked due to insufficient collateral.
*   Tagged as scheduled for liquidation.

The standing of a borrower’s account will be affected by two types of actions; those initiated by the borrower and those initiated by a trader retiring assets in exchange for collateral. When a trader retires assets it changes the collateralization ratio of all accounts at once. Since there is a limit to how much computation can be completed in one transaction, the effect of these retirements on accounts will be calculated in batches.

However, actions taken by the borrower will only affect their own collateralization ratio, and a calculation must be done to bring their account up to date immediately before the action they wish to take is applied. The standing of each account will be brought up to date before distribution of Balance Tokens at the end of the day, and must be up to date for a particular account before a liquidation is executed on it.

**Receipt of tokens by the contract**

When a SCORE receives tokens, the token contract calls the tokenFallback method on the receiving SCORE. Data can be sent with the token transfer to direct the tokens once they are received. The Loans contract will decode the data that is sent with the tokens to determine what to do with them. If the data field does not contain valid instructions for how to handle the tokens, the transaction will revert.

**Retirement event replay batching**

In order to keep accounts current, every transaction will check and update accounts up to the most recent retirement event, unless it is otherwise loaded with another task such as dividends or Balance Token distribution. A pointer will be maintained for a cycle through the whole list of accounts and each transaction will pick up where the previous one left off, bringing accounts up to date.

**Provisions for positions data transfer to the Rewards contract**

The state of all debt positions at the end of the day, which will be used to calculate the mining rewards, will be too large to transfer to the Rewards contract in one transaction; it must be done in batches. Because the distribution will be done in batches, a snapshot of the full state of all positions is required for mining reward calculations. Without this snapshot mechanism there would be differences in state between distribution batches. To achieve this, the Loans contract will maintain two copies of its position data. One copy will be frozen at end of day to be used as a snapshot. The snapshot copy will provide consitent data for the Rewards Contract. The other copy will continue to update based on real-time activity on Balanced. Upon completion of all distributions for the day, the two copies will be synchronized.


#### **Methods**

**User Initiated State-Changing External methods**

Transactions on any of these methods will be used to carry out Balanced operational processing as needed in addition to the activity the method is called to perform.

_deposit_collateral()_: This method is accessed through the tokenFallback method by transferring the tokens to the contract and including handling instructions in the data field. If the data field specifies the _deposit_collateral()_ method, it will take the following actions:



*   Check if the received tokens are sICX. Other tokens will cause the transaction to revert.
*   Accept the sICX tokens and increase the collateral recorded for the sending address in the positions DictDB.

_originate_loan(_asset_name)_: When a borrower initiates this method, the contract will send the requested loan amount to the transaction origination address if there is sufficient collateral held on the loans contract for the borrower. It will take the following actions:



*   Check if the account is up to date with the events in the replay log. If not, it will apply those events until the account is up to date.
*   Check the stored price for the asset, and update it if it is not sufficiently recent.
*   Check the collateral balance of the Borrower.
*   Calculate the maximum allowed loan.
*   If the requested loan is less than the maximum allowed loan
    *   The asset will be minted and sent to the originating address.
    *   The positions DictDB will be updated with the new debt position for the originating address.
*   If the requested loan exceeds the maximum allowed loan
    *   An OriginationFailure event will be issued.
    *   The transaction will not revert so the applied updates will not be lost.

_repay_loan()_: This method is also accessed through the tokenFallback method. It will accept any type of token that is listed in the Asset Dictionary. Transfers of assets not listed in the Asset Dictionary will revert. A successful call to the repay_loan() method will take the following actions:



*   Reduce the debt position of the account by the amount of the token transfer or the value of the current debt position, whichever is less.
*   The _asset_burn()_ method will be called for the repaid asset.
*   If there is any excess token amount transferred beyond that needed to repay the current debt, it will be returned to the sender.

_withdraw_collateral(_amount)_: When this method is called it will perform the following actions:



*   Check if the account is up to date with the events in the replay log. If not, it will apply those events until the account is up to date.
*   If there are any outstanding debt positions, it will check the stored prices for any issued assets and update them if they are not sufficiently recent.
*   Calculate the maximum sICX that may be withdrawn without exceeding the account locking threshold.
*   If the requested withdrawal does not exceed the amount available, it will transfer the requested sICX amount to the sending address.
*   If the requested amount exceeds the amount available, it will issue a CollateralWithdrawalFailure event but the transaction will not revert so that any applied updates will not be lost.
*   If withdrawal was successful, it will update the collateral balance for the account in the positions DictDB.

**Trader initiated State-Changing External Methods**

Transactions on any of these methods will be used to carry out Balanced operational processing as needed, in addition to the activity the method is called to perform.

_retire_asset()_: When a trader sends a transaction to retire an asset it will come through the tokenFallback method. It will accept any type of token that is listed in the Asset Dictionary. Transfers of tokens not listed in the Asset Dictionary will revert. A successful call to retire_asset() will take the following steps:



*   Calculate the value of the asset being retired using the stored price if it is sufficiently recent, otherwise it will update the asset price with an oracle call, then calculate the value.
*   Check if the forced liquidation pool contains enough sICX to cover the retirement. If not it will then;
*   Check the debt positions of the borrowers of the pegged asset. If liquidating all of the debt of the borrowers still doesn’t cover the retirement, it will then;
*   Check and draw from the Emergency Reserve Fund.
*   If some portion of the retired asset is bad debt, and that portion is less than the total in the liquidation pool, then up to 110% of the retired asset value will be returned to the trader with the condition that no more than the full value of the liquidation pool will be returned.
*   A record will be entered in the retirement event log of the total outstanding debt for the asset, the amount retired, and the price of the asset in ICX
*   The _asset_burn()_ method will be called for the retired assets.

**Incentivized State-Changing External Methods**

Balanced offers an incentive for anyone who triggers a forced liquidation. An oracle price must have been obtained within a limited time window (e.g. 10 minutes) of the liquidation request for the collateralization ratio calculation to be considered accurate and the liquidation to be executed. Transactions on any of these methods will be used to carry out Balanced operational processing as needed in addition to the activity the method is called to perform.

_threshold_check()_: Updates batches of accounts, checking for any that are below the liquidation threshold. This will call the _event_replay()_ method on batches of addresses. The _event_replay()_ method will issue a ThresholdReached event in case of an account dropping below a threshold. 

_liquidate()_: This method will execute the following steps:



*   Checks if the account is up to date with events in the replay log.
*   Checks that asset prices were recently enough obtained from price oracles.
*   If the account is not up to date, replay and prices will be brought up to date.
*   The collateralization ratio is calculated, if necessary.
*   If the account collateralization is under the liquidation threshold the value of the debt positions will be calculated and 5% of that will be sent to the calling address in the form of sICX.
*   The liquidated amount is added to the bad debt for the asset.
*   The liquidation event is added to the replay log.
*   If the asset is for a dead market, the remaining collateral will be sent to the Emergency Reserve Fund, otherwise it will be added to the Liquidation Pool.

**Balanced Management State-Changing External Methods**

These methods will be called to set up, configure, and extend Balanced. In the initial phase of Balanced operation these methods will be called manually. As Balanced transitions to DAO operations they will eventually be called through the governance contract as a result of vote outcomes. Transactions on any of these methods will be used to carry out Balanced operational processing as needed, in addition to the activity the method is called to perform.

_add_asset(_token_address)_: When a new asset is added an entry will be created in the Asset Dictionary.

_read_position_data_batch()_: Provided for the Rewards contract to access the position data. When this method is called and the position DictDB is in the operating state, this call will transition it into the frozen state. In the frozen state only one of the two db copies is operational, while the second is used for transferring batches to the Rewards contract. Successive calls will return successive batches from the positions DictDB. A final call that returns an empty batch will also transition the positions DictDB back to the operating state. 

**Read-only External Methods**

_get_account_positions(_address)_: Reads out the collateral and debt positions for the given account. Only readable by account owner and Balanced management wallet.

_get_available_assets()_: This method will return a list of assets available to lend against deposited collateral.

**Internal methods**

_update_asset_value()_: Call the price oracle to retrieve the latest price for the pegged asset. Records the price in sICX, a timestamp, and a hash of the oracle request transaction.

_event_replay(_address)_: Applies retirement events executed by arbitrage traders since the last time the account was updated.

_asset_mint(_ticker, amount)_: This method will only be called in combination with a matching loan position increase for the participant’s address.

_asset_burn(_ticker, _amount)_: This method will only be called in combination with a matching loan position decrease for the participant’s address, or a matching entry in the retirement event log.


#### Data

**Positions**

This will be a depth 3 DictDB with the first key being a switch between two copies of the positions data. The second level key will be the address of the participant. Third level keys and values will be formatted as: {“sICX”: 30000, “ICD”: 100, “ICG”: 10, “standing”: 1, “replay”: 23928, “timestamp”: 56293028109}, where the amount of sICX collateral deposited is 30000, the outstanding ICD debt is 100, the ICON Gold (ICG) debt is 10, the standing of 1 indicates the position is mining Balance Tokens, replay gives the index of 23928 for the last entry from the retirement event log that was applied to the account, and the timestamp is the last time the account standing was calculated.

**Emergency Reserve Fund**

The Emergency Reserve Fund will be tracked with a DictDB containing balances for ICX, sICX, and BAL tokens.

**Asset Dictionary**

The asset dictionary will be a depth 2 DictDB with entries for all pegged assets on Balanced. It will have the token contract addresses as the first keys and properties for the asset such as ‘creation_time’, ‘ICX_price’, ‘price_update_time’, ‘bad_debt’, ‘active’ and ‘dead_market’

**Bad debt**

Bad debt for an asset will be tracked in the asset dictionary DictDB.

**Retirement and liquidation event log**

An ArrayDB object containing a string entry for each asset retirement or liquidation. Each entry will contain the amount retired or liquidated, the total outstanding debt for the asset at that time, and the sICX exchange rate for the asset at that time.

**Forced Liquidation Pool**

VarDB integer value of sICX held from forced liquidations.


### Supporting contracts


#### Governance

The Governance contract will be a central place for setting of parameters that control the operation of Balanced. Eventually the Governance contract will manage DAO votes to determine operation parameters, but we will begin with manual control until we find stable operating parameters and processes.


#### DEX

The basic functionality of the DEX will allow for trading of sICX, BAL, ICD, and ICX. The trading pairs will be sICXICX, BALICX, and ICDICX, although trading of BALICX may not be supported initially, since there will not be enough BAL minted to provide sufficient liquidity.

Each DEX user will be tracked by their address. For each user address there will be a dictionary entry listing all of the assets they hold on the DEX. Any fraction of a held asset that is committed to a listed order will be considered locked. The remaining portion of the held asset will be available for transfer back to the owning wallet at any time.

**Provision for transferring liquidity mining data to the Rewards contract**

Anyone completing limit orders for sICXICX on the DEX within 3% of the true value of sICX will earn Balance Tokens. Trading of pegged assets may also be incentivized with mining rewards. Each address completing orders will accumulate calculated mining power throughout the day with each additional trade. Each day will accumulate mining power into a new dictionary so the dictionary for the previous day may be used by the Rewards contract to calculate Balance Token earnings.


![alt_text](/assets/sequence_diagram.png "Core Functional Sequence Diagrams")

Core Functional Sequence Diagrams


## Equations and Algorithms

A few select algorithms/pseudocode equations used by the balanced network are available below. These are subject to change.

### Operation Replay

While we may face constraints in amount of operations on the ICON Blockchain, step costs and performance to reduce each users position with each operation, we have employed an operation log. We will write operations with the type (retire, take out loan, liquidate) to a ledger, and users who wish to take additional loans will have to replay operations until their position is caught up with the head pointer.


```
1. Setup: get user's current sequence number and loans sequence
2. While user's sequence is less than header sequence, check next sequence and apply to user
3. If at blockchain max iterations and not current, return with a status code to re-run the operation

apply_retire(...):
  1. Get price at that sequence number from oracle
  2. Reduce user's debt by ratio of total loans to their loan
  3. Reduce collateral reduced debt time snapshotted oracle price
  4. Increase user's sequence number

apply_liquidate(...):
  1. Reduce total debt by liquidated debt amount
  2. If user was liquidated, set collateral to 0
  3. Increase user's sequence number
```

### Bankruptcy & Liquidation

A position can be considered bankrupt when the total value of the debt (expressed in ICX) is equal to the value of the collateral. The balanced network is designed with avoiding this in mind, so will assign a liquidation price at 150% of this.

For assets that do not have a direct ICX price but do have a USD price, we will determine the value by pathing through USD  (e.g. `oracle_price[gold]=GOLDUSD / ICXUSD`)

Inputs to each require:
* `debt` user's debt (per-asset)
* `oracle_price` oracle price (per-asset, in pairs to ICX)
* `sicxicx_price` SICX-ICX converstion ratio defined in the staking contract
* `collateral` user's collateral, expressed in SICX

**Bankruptcy**

A position can be considered bankrupt when the following condition is met:

`sum(debt[asset] * oracle_price[asset]) = collateral * sicxicx_price`

**Liquidation**
For liquidation, user collateral should be 150% of position value:

`sum(debt[asset] * oracle_price[asset]) = 1.5 * collateral * sicxicx_price`

**Single asset liquidation price**

For a an isolated loan of a single asset, this can be solved against the oracle price:

`liquidation_price = (1.5 * collateral * sicxicx_price / debt)`
## Flow of a Position

This section will cover the lifecycle of a position from opening to closing, and various events that may occur while a position is open. For simplicity, this will discuss 1 pegged asset (ICON Dollars) and everything will be quoted in terms of dollar value. In future iterations, Balanced will support many different types of pegged assets and the dollar value of collateral will be less relevant. The more relevant number will be the ICX value of a borrower’s outstanding loan versus the amount of ICX collateral deposited.


### Opening a position

In order to open a position, a user must first convert their ICX into sICX. This is done automatically upon depositing ICX into Balanced.

After the user has deposited ICX into the Balanced Staking Pool, their sICX will be sent to the loan contract. With sICX in the loan contract, the oracle solution leveraged by Balanced will provide the user with the value of their collateral.

A user is now able to mint pegged assets based on the value of their collateral. Users pay a 25 basis point origination fee to borrow pegged assets from Balanced. 

For this walkthrough, let’s assume that Bob deposits $500 worth of sICX. With a deposit of $500 worth of collateral, Bob is now able to borrow up to $125 worth of ICON Dollars (ICD). Bob decides to borrow 125 ICD, and his open position will therefore be 125 ICD. Bob will only receive 124.6875 ICD because of the origination fee.


### Managing a position

To properly manage his position, Bob should pay close attention to the three key collateral ratios: the mining ratio (500%), the mandatory collateral ratio (400%), and the liquidation ratio (150%). Bob also should be aware of outside traders retiring pegged assets and the effect on his position.


#### Mining

Bob’s current collateral ratio is 400%, which meets the mandatory collateral ratio. In order for Bob to start mining Balance Tokens (BAL), he must raise his collateral ratio to 500%. Bob would like to mine BAL, so he decides to pay off 25 ICD worth of debt. With $500 worth of collateral and 100 ICD of debt, Bob now meets the mining ratio and will start mining BAL. The amount of BAL that Bob will mine is based on the following formula:

(Bob’s total debt / total debt owed to Balanced) x BAL allocated to miners on day_n


#### Locked collateral

Let’s now assume the value of Bob’s collateral has gone down to $300 and he still has 100 ICD of debt. Bob’s collateral ratio is now 300%, which is below the mandatory collateral ratio. Bob’s collateral is now locked. Bob cannot withdraw any collateral from Balanced and cannot borrow more ICD. In order to withdraw collateral from Balanced or borrow more ICD, Bob must increase his collateral ratio above 400% via paying off debt or depositing more collateral.


#### Liquidations

Let’s now assume the value of Bob’s collateral has gone down to $125 and he still has 100 ICD of debt. Bob’s collateral ratio is now 125%, which is below the liquidation ratio. Bob is now eligible to be liquidated by other users. Other users are incentivized to liquidate Bob because they will receive 5% of the value of Bob’s bad debt. 

In this scenario, Alice triggers Bob’s liquidation and will receive 5 ICD worth of Bob’s collateral. Bob’s position will be closed, all of Bob’s collateral will be sent to the Forced Liquidation Pool, and the 100 ICD of outstanding debt will be converted to bad debt. 

Any user (including Bob) will now have the opportunity to pay off the 100 ICD of bad debt in exchange for up to a 10% bonus on the repayment depending on the value of the collateral held in the Forced Liquidation Pool. Any leftover collateral upon paying off all bad debt will be sent to the Emergency Reserve Fund.

For example, Charlie comes to Balanced and pays off the 100 ICD of bad debt caused by Bob’s liquidation. Charlie then receives 110 ICD worth of collateral from the Forced Liquidation Pool.

To summarize, 5 ICD worth of Bob’s collateral is given to Alice for triggering the liquidation, 120 ICD worth of Bob’s collateral was sent to the Forced Liquidation Pool, Charlie received 110 ICD worth of the collateral from the Forced Liquidation Pool, bad debt is now 0, and the remaining 10 ICD worth of collateral is sent to the Emergency Reserve Fund.


#### Retiring pegged assets

When a Trader retires pegged assets, all current borrowers are automatically deleveraged based on their percentage of debt relative to the value of all debt on Balanced. Let’s assume that Bob has $500 worth of collateral and makes up 1% of the total debt on Balanced.

Alice has managed to purchase 100 ICD for only $80 and would like to take advantage of the arbitrage opportunity offered by Balanced. When Alice retires 100 ICD she will receive $99 worth of collateral because of the 1% arbitrage fee. 1 ICD will be sent to the Balanced Rewards Pool and 99 ICD will be retired. 

Since Bob makes up 1% of the total debt on Balanced, Bob will lose $0.99 worth of collateral and 0.99 ICD worth of his debt will be automatically paid off. 


### Closing a position

A position can be closed three ways: retirements, repayments, and liquidations.

Let’s assume Bob has 100 ICD of debt and $500 of collateral. 

Bob’s position can be closed with retirements if 100% of circulating ICD supply is retired. Balanced would sell $100 worth of collateral, Bob’s debt would be 0 ICD, and his position would be closed.

Bob’s position can be closed with repayments if Bob repays 100 ICD to Balanced. Bob would repay his debt, none of his collateral would be sold, and his position would be closed.

Bob’s position can be closed by liquidation if he drops below the liquidation ratio of 150% collateralization. If Bob gets liquidated, all of Bob’s collateral would be taken from him, his debt would be 0, and his position would be closed.
