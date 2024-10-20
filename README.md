# QuestionHound
Question Hound ($QHND) - This is Fine | Meme Token
Welcome to the official repository for Question Hound ($QHND), a meme token designed to embody the spirit of the crypto community during tough bear markets. Built on the BNB Chain, $QHND rewards users who embrace the chaos of volatile markets with a game-like transaction system. This repository includes the smart contract, website source code, and relevant resources.

Overview
Question Hound is a community-driven token that operates with the following core mechanics:

100 million total supply
5% tax on every transaction
2% is allocated to liquidity
3% is redistributed to the previous two buyers/sellers
Rewards are based on whether the next transaction is a buy or sell:
If the next transaction is a buy, the last two participants receive BNB.
If it's a sell, they receive more $QHND tokens.
Features
Liquidity Growth: 2% of every transaction is added to liquidity, helping to stabilize the market.
Dynamic Rewards System: The token distribution system creates a game where timing transactions properly can maximize rewards.
Meme and Community Powered: Built for those who find humor and opportunity in market downturns.
Fast and Low-Cost Transactions: Optimized for BNB Chain.
Smart Contract Details
The smart contract for $QHND is built using Solidity and can be deployed on the BNB Chain. Key features include:

Renounced Ownership: Ensures that no changes can be made after deployment, making it truly community-driven.
Reward Logic: Embedded to incentivize active trading by redistributing a portion of transaction taxes to the last two traders.
Repository Contents
contracts/:

Contains the Solidity smart contracts, including the RewardTracker and Liquidity logic for the $QHND token.
website/:

The source code for the token’s official website. It includes sections for:
About the token
Key features
How to buy the token
Links to BscScan, CoinMarketCap, and CoinGecko
scripts/:

Web3.js scripts for interacting with the deployed smart contract:
Fetch the latest buyer
Update the latest buyer and rewards
images/:

Logos for $QHND, BscScan, CoinMarketCap, and CoinGecko.
Installation and Setup
Prerequisites
Node.js and npm for running web3.js scripts
Truffle or Hardhat for deploying smart contracts
MetaMask or Trust Wallet for connecting to the BNB Chain
Deploying Smart Contract
Clone the repository:
git clone https://github.com/yourusername/QHND-Token.git
cd QHND-Token

Install dependencies:
npm install

Compile and deploy the contract using Truffle or Hardhat:
truffle compile
truffle migrate --network bnbChain

Configure the contract’s ABI and address in the website/scripts/web3.js file for interaction.

Running the Website
Navigate to the website/ folder:
cd website

Open index.html in your browser to view the website.

To make modifications, edit the HTML, CSS, and JavaScript files.

How to Contribute
We welcome contributions from the community to improve the project. Here's how you can contribute:

Fork this repository.
Create a new branch (git checkout -b feature-branch).
Make your changes and commit them (git commit -m "Added a new feature").
Push to the branch (git push origin feature-branch).
Open a pull request and describe your changes.
License
This project is licensed under the MIT License. See the LICENSE file for details.
