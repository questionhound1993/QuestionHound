// SPDX-License-Identifier: MIT
pragma solidity ^0.8.28;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable2Step.sol";
import "@uniswap/v2-core/contracts/interfaces/IUniswapV2Factory.sol";
import "@uniswap/v2-periphery/contracts/interfaces/IUniswapV2Router02.sol";
import "@uniswap/v2-periphery/contracts/interfaces/IUniswapV2Router02.sol";
// Import the RewardTracker contract
import "./RewardTracker.sol";

contract QuestionHoundToken is ERC20, Ownable {
    uint256 private constant TOTAL_SUPPLY = 100_000_000 * 10**18;
    uint256 private constant TAX_PERCENTAGE = 5 *1e18; // 5% represented with 18 decimals
    uint256 private constant LIQUIDITY_TAX = 2 *1e18; // 2% represented with 18 decimals
    uint256 private constant REWARD_PREVIOUS_BUYER = 2 *1e18; // 2% to previous buyer
    uint256 private constant REWARD_BEFORE_PREVIOUS_BUYER = 1 *1e18; // 1% to buyer before the previous

    struct BuyerInfo {
        address previousBuyer;
        address beforePreviousBuyer;
    }

    BuyerInfo private buyerInfo;
    IUniswapV2Router02 private pancakeRouter;
    address public liquidityPair;
    address[] private path;
    RewardTracker private rewardTracker;


    mapping(address => bool) private isExcludedFromTax;

    constructor(address _pancakeRouter, address _rewardTracker) ERC20("Question Hound", "QHND") Ownable(msg.sender) {
	rewardTracker = RewardTracker(_rewardTracker); // Initialize reward tracker

        address contractAddress = address(this); // Cache address(this)

        pancakeRouter = IUniswapV2Router02(_pancakeRouter);

        // Mint total supply to the owner
        _mint(msg.sender, TOTAL_SUPPLY);

        // Create a liquidity pair for the token with WBNB (BNB)
        liquidityPair = IUniswapV2Factory(pancakeRouter.factory()).createPair(contractAddress, pancakeRouter.WETH());

        // Initialize the path array for swapping QHND to WBNB
        path = [contractAddress, pancakeRouter.WETH()];

        // Exclude owner and contract from tax
        isExcludedFromTax[owner()] = true;
        isExcludedFromTax[contractAddress] = true;
    }
    event TaxApplied(address indexed sender, uint256 amount);

    // Override transfer function to implement buy/sell logic
    function _transfer(address sender, address recipient, uint256 amount) internal override {
        address contractAddress = address(this); // Cache address(this)

        // Check if the transfer is a buy/sell (interacting with the liquidity pool) and NOT an internal contract function (like adding liquidity)
        if ((sender == liquidityPair || recipient == liquidityPair) && 
            sender != contractAddress && recipient != contractAddress) {
            // Apply tax and rewards only on buy/sell through liquidity
            rewardTracker.updateLatestBuyer(sender); // Update latest buyer
            _applyTaxAndReward(sender, recipient, amount);
        } else {
            // Regular transfers or internal transfers (like adding liquidity) are tax-free
            super._transfer(sender, recipient, amount);
        }
    }

    // Function to apply tax and distribute rewards
    function _applyTaxAndReward(address sender, address recipient, uint256 amount) private {
        uint256 transferAmount = amount;
        address contractAddress = address(this); // Cache address(this)

        // Check if tax should be applied (not excluded from tax)
        if (!isExcludedFromTax[sender] && !isExcludedFromTax[recipient]) {
            // Calculate tax amount and apply scaling by 1e18 to maintain precision
            uint256 taxAmount = (amount * TAX_PERCENTAGE) / 100 *1e18; // Corrected to use scaled percentage

            // 2% goes to liquidity
            uint256 liquidityAmount = (taxAmount * LIQUIDITY_TAX) / TAX_PERCENTAGE;
            // Check the success of the transfer using require and handle if false is returned
            require(_safeTransfer(contractAddress, liquidityAmount), "Transfer to contract for liquidity failed");
            addLiquidity(liquidityAmount);

            // 2% to the last buyer/seller and 1% to the one before, if buyers exist
	// Rewards distribution (2% + 1%)
            uint256 rewardAmount = (taxAmount * REWARD_PREVIOUS_BUYER) / TAX_PERCENTAGE;
            uint256 beforeRewardAmount = (taxAmount * REWARD_BEFORE_PREVIOUS_BUYER) / TAX_PERCENTAGE;

            address previousBuyer = buyerInfo.previousBuyer;
            address beforePreviousBuyer = buyerInfo.beforePreviousBuyer;

            if (previousBuyer != address(0)) {
                // Reward the previous buyer
                require(_safeTransfer(previousBuyer, rewardAmount), "Transfer to previous buyer failed");
	    rewardTracker.updateRewards(previousBuyer, rewardAmount); // Update reward tracking
            } else {
                // No previous buyer, reward goes to the owner
                require(_safeTransfer(owner(), rewardAmount), "Transfer to owner (no previous buyer) failed");
            
            }

            if (beforePreviousBuyer != address(0)) {
                // Reward the buyer before the previous one
                require(_safeTransfer(beforePreviousBuyer, beforeRewardAmount), "Transfer to buyer before previous failed");
	rewardTracker.updateRewards(beforePreviousBuyer, beforeRewardAmount); // Update reward tracking
            } else {
                // No buyer before previous, reward goes to the owner
                require(_safeTransfer(owner(), beforeRewardAmount), "Transfer to owner (no buyer before previous) failed");
            }

            // Update buyer info only if the new value is different
            if (buyerInfo.previousBuyer != sender) {
                buyerInfo.beforePreviousBuyer = previousBuyer;
                buyerInfo.previousBuyer = sender;
		rewardTracker.updateRewardReceivers(sender);
            }

            // Adjust the transfer amount after tax
            transferAmount = amount - taxAmount;
        

        super._transfer(sender, recipient, transferAmount);
      }


    
}

     // Add liquidity to the pool using the collected tokens
      function addLiquidity(uint256 tokenAmount) private {
        address contractAddress = address(this); // Cache address(this)

        // Track the contract's Ether balance before the swap
        uint256 initialBalance = contractAddress.balance;

        // Swap tokens for BNB (Ether)
        swapTokensForBNB(tokenAmount);

        // Calculate the amount of BNB received by subtracting the initial Ether balance from the current balance
        uint256 bnbReceived = contractAddress.balance - initialBalance;

        // Add liquidity to PancakeSwap pool (BNB/QHND pair)
        _addLiquidity(tokenAmount, bnbReceived);
    }

    // Swap tokens for BNB using the router
    function swapTokensForBNB(uint256 tokenAmount) private {
        address contractAddress = address(this); // Cache address(this)
        _approve(contractAddress, address(pancakeRouter), tokenAmount);

        // Perform the swap
         pancakeRouter.swapExactTokensForETHSupportingFeeOnTransferTokens(
            tokenAmount,
            0, // Accept any amount of BNB
            path,
            contractAddress, // The contract receives the BNB
            block.timestamp
        );
    }

    // Add liquidity to PancakeSwap using the collected BNB and QHND
    function _addLiquidity(uint256 tokenAmount, uint256 bnbAmount) private {
        address contractAddress = address(this); // Cache address(this)
        _approve(contractAddress, address(pancakeRouter), tokenAmount);

        // Add the liquidity to PancakeSwap (BNB/QHND)
        pancakeRouter.addLiquidityETH{value: bnbAmount}(
            contractAddress,
            tokenAmount,
            0,  // slippage is fine
            0,  // slippage is fine
            owner(),
            block.timestamp
        );
    }

    // Safe transfer function to handle tokens that return false on failure instead of reverting
   function _safeTransfer(address to, uint256 amount) private returns (bool){
    require(super.transfer(to,amount), "Token Transfer failed");
    // If super.transfer() reverted here and we didn't reach this line,
    // then it means something went wrong with token transfers.
    return false;
        }

    // Allow excluding addresses from tax (e.g., for exchanges)
    function excludeFromTax(address account, bool excluded) external onlyOwner {
        isExcludedFromTax[account] = excluded;
    }

    // Function to receive BNB when swapTokensForBNB is called
    receive() external payable {
        // This function now allows any address to send Ether to the contract
    }

    fallback() external payable {
        // Fallback function to accept Ether sent to the contract in case receive() is not triggered
    }
    function increaseAllowance(address spender, uint256 addedValue) public returns (bool) {
    _approve(_msgSender(), spender, allowance(_msgSender(), spender) + addedValue);
    return true;
    }

    function decreaseAllowance(address spender, uint256 subtractedValue) public returns (bool) {
    _approve(_msgSender(), spender, allowance(_msgSender(), spender) - subtractedValue);
    return true;
    }


    // Allow the contract owner to withdraw Ether from the contract
    function withdraw() external onlyOwner {
    uint256 contractBalance = address(this).balance;
    require(contractBalance != 0, "No Ether available to withdraw");

    // Use call instead of transfer for safe Ether withdrawal
    (bool success, ) = payable(owner()).call{value: contractBalance}("");
    require(success, "Ether withdrawal failed");
    }
    
}
