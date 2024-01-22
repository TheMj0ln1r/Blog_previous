---
layout: post
title:  "DEX Token Swap"
command: cat dexswap
---

A simple DEX Token Swap Project developed by me for an internship task.

Project Repo :  <a href="https://github.com/TheMj0ln1r/DEX-Token-Swap" target=_blank>DEX Token Swap</a>


***

# **ERC20 Token Swap**

**This TokenSwap smart contract facilitates the swapping of one ERC-20 token for another at a predefined exchange rate**

## Documentation

**Note : Most of the functionality of this pool is inspired by the design of `UniswapV2`**

## Features

The smart contract include the following features:

- Users can swap Token A for Token B and vice versa.

- The exchange rate between Token A and Token B is fixed `(1.2)`.

- Users can provide liquidity to the pool in return they get `shares`.

- Users can remove their liquidity from the pool.


## Design Choices

The `TokenSwapPool` contract is a simplified liquidity pool for swapping between two ERC-20 tokens (tokenA and tokenB) at a **fixed exchange rate**. It is an ERC-20 token itself, representing **shares** in the pool, which users get when they provide liquidity.

**Immutables:** `token0` and `token1` are immutable variables storing the addresses of the ERC-20 tokens that can be swapped. Once set in the constructor, their values cannot be changed.

**Reserves:** `reserves0` and `reserves1` keep track of the amounts of `token0` and `token1` in the pool, respectively.

**Swap Function:** Allows users to swap between `token0` and `token1`. 

- User have to send the amount of token that has to be swapped in the same transaction that calls `swap`

- It checks the balance difference before and after the transfer to find the amount sent by the user and then performs the swap based on the fixed exchange rate.

- Then it checks that which token amount was sent by the user by checkng the following condition.
    
    `if (_tokenIn == address(token0))`

- If the user sent `tokenA` and wants to get `tokenB` the `if` block will execute.

- Then the `amountToSend` will be calculated with the fixed exchange rate. 

- The exchange rate I used is `1.2`, so to handle with decimals in solidity I multiplied with `10` and devided with `12` in case of `tokenA`.

- If the user sent `tokenB` and wants to get `tokenB` the `else` block will execute. 

- To calculate `amountToSend`, the `amountIn` will be multiplied with `12` and divided with `10`.

- This will maintain the `fixed exchange rate` for the exchange.

**addLiquidity Function:** Add liquidity to the pool by sending both `token0` and `token1` to the contract. 

- The function calculates the number of share tokens (liquidity tokens) to mint based on the amounts added and the current pool size.

- The both tokens amounts should be sent in the same transaction with calls the `addLiquidity` function.

- When liquidity is added for the first time, the contract calculates the initial liquidity shares based on the `square root` of the product of the amounts of the two tokens.

- A minimum liquidity amount `(_mint(address(0xdeadbeef), 1000))` is minted initially to lock some liquidity. 

- The address `0xdeadbeef` is taken to assume that no one has the private key of it.

- In `Uniswap V2` it was `address(0)`, but if we use `address(0)` here the `ERC20` contract of `OpenZeppelin` reverts if the address passed to `_mint` is `0` then it will revert.

- To solve this `Uniswap V2` implements its own `UniswapERC20` contract. 

**removeLiquidity:** Allows users to burn their share tokens and withdraw a proportional amount of `token0` and `token1` from the pool.

- It is assumed that the burner sent in share tokens before calling removeLiquidity, but advisably as part of one transaction.

- If the total supply of share tokens is 1,000, and they burn 100 share tokens, then they get 10% of the token0 and token1 held by the pool. 

- _shareTokens / totalSupply is their burned share of the total supply of share tokens.

**_update function:** Updates the reserves of the pool.

**getReserves function:** Getter function for the `reserves0` and `reserves1`.

**Events:** 

    ```solidity
    event TokenSwap_Liquidity_Minted(address _to, uint _shareTokens);
    event TokenSwap_Liquidity_Burned(address sender, uint amount0, uint amount1, address _to);
    event TokenSwap_Swap(address, address, uint);
    ```

**Errors:**

    ```solidity
    error TokenSwap_Invalid_Address();
    error TokenSwap_Invalid_Exchange_Rate();
    error TokenSwap_TokenA_Transfer_Failed();
    error TokenSwap_TokenB_Transfer_Failed();
    error TokenSwap_Invalid_Amount();
    error TokenSwap_No_Liquidity_Shares();
    error TokenSale_No_Liquidity_Output();
    ```

## Security Considerations

**SafeERC20 Usage:** The SafeERC20 library for token transfers, which helps guard against potential vulnerabilities such as reentrancy attacks by ensuring that ERC-20 token transfers are executed safely.

**Custom Error Types:** Custom error types are defined to provide specific and clear error messages when certain conditions are not met. This can help users understand the reason for a transaction failure.

**Input Validation:** Checks for valid addresses in the constructor and various functions, preventing potential issues related to invalid or zero addresses.

**Minimum Liquidity Lock:** A minimum liquidity amount is minted initially to lock some liquidity. This prevents an `inflation attack` where an attacker could artificially inflate the liquidity.

**Liquidity Minting Approach:** The contract calculates initial liquidity shares using a `square root` function, inspired by `Uniswap V2`. This approach aims to mitigate the "**doubling liquidity**" problem that could lead to unintended consequences.

**Fixed Exchange Rate:** Used a fixed exchange rate between token0 and token1 This simplifies the contract, it is important to note that in dynamic market conditions, fixed exchange rates may expose users to `impermanent loss`.

**Minimum Liquidity Check:** The `addLiquidity` function checks that the calculated liquidity shares are not zero before minting new liquidity tokens. This prevents users from adding liquidity with extremely low or zero values.

**Reentrancy Protection:** Ensuring that functions are not susceptible to reentrancy attacks by applying `nonReentrant` modifier.

**Drawbacks** The `swap` function transfers tokens with the predetermined exchange rate(12). So, the tokens of exchange will always be same. i.e If user sent X amount of tokens at one time he will get Y amount of tokens. This will hold for any time. That is user will will get same amount of tokens for every swap he made. And there is no check that liquidity of the pool is maintained or not. We cant use `constant product AMM` because the exchange rate will be varied with the amount od tokens present in the pool. And the `liquidators` wont receive any fee as the fee is not used in this contract.


## Test Cases

**testConstructor:** Verifies that the constructor of the TokenSwapPool contract reverts if invalid addresses (zero addresses) are passed as parameters.

**testAddLiquidityFailing:** Checks that adding liquidity to the TokenSwapPool fails and reverts when the address provided is invalid or if there are no liquidity shares to add (i.e., the balances of tokens transferred to the pool are zero).

**testAddLiquidity:** Tests the successful addition of liquidity to the pool. It checks the initial state of the pool, transfers tokens to the pool, and then adds liquidity from the user. It verifies if the minted shares and total supply are updated correctly.

**testAmountAddLiquidityMultipleTimes:** Confirms that liquidity can be added multiple times and calculates the expected shares based on the pool's reserves and total supply. It ensures that minted shares are correctly accounted for after each addition.

**testAmountAddLiquidityDifferentRatio:** Similar to the previous test, but with a different ratio of TokenA to TokenB, ensuring that the minted shares are calculated correctly based on the provided ratio.

**testRemoveLiquidityFail:** Ensures that attempting to remove liquidity fails when the address provided is invalid or if there's no liquidity output expected (i.e., the shares to be burned are zero).

**testRemoveLiquidity:** Tests the successful removal of liquidity from the pool. It verifies that the tokens are returned to the user correctly and that the total supply of liquidity shares is updated.

**testSwapMyA:** Simulates a swap of TokenA for TokenB by a user and verifies that the user's token balances are updated correctly after the swap.

**testSwapMyB:** Simulates a swap of TokenB for TokenA by a user and ensures that the user's token balances are updated correctly after the swap.

**testSwapFails:** Checks that attempting to swap with an invalid amount (zero) or an invalid address causes the contract to revert.

**testGetReserves:** Confirms that the getReserves function returns the correct reserves of TokenA and TokenB in the pool.

**testDeployScript:** Verifies that the deployment script works correctly by deploying the contracts and checking that the addresses of the deployed contracts are non-zero.

```shell
mj0ln1r@Linux:~/dex-token-swap$ forge coverage
[⠢] Compiling...
[⠔] Compiling 34 files with 0.8.23
[⠊] Solc 0.8.23 finished in 8.26s
Compiler run successful!
Analysing contracts...
Running tests...
| File                | % Lines        | % Statements   | % Branches     | % Funcs       |
|---------------------|----------------|----------------|----------------|---------------|
| script/Deploy.s.sol | 100.00% (4/4)  | 100.00% (4/4)  | 100.00% (0/0)  | 100.00% (1/1) |
| src/TokenSwap.sol   | 98.31% (58/59) | 98.86% (87/88) | 95.45% (21/22) | 100.00% (7/7) |
| Total               | 98.41% (62/63) | 98.91% (91/92) | 95.45% (21/22) | 100.00% (8/8) |
```

## Usage

### Build

```shell
$ forge build
```

### Test

```shell
$ forge test
```

### Format

```shell
$ forge fmt
```

### Deploy

```shell
$ forge script script/Deploy.s.sol:TokenSwapScript --rpc-url <your_rpc_url> --private-key <your_private_key>
```
