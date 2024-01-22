---
layout: post
title:  "Token Sale Project"
command: cat tokensale
---

A simple Token Pre Sale and Public Sale Project developed by me for an internship task.

Project Repo :  <a href="https://github.com/TheMj0ln1r/Token-Sale-Project" target=_blank>Token Sale Project</a>

***

# **Token Pre-Sale & Public Sale**

**Token Sale is a fundraising mechanism used by blockchain projects to distribute and sell their native tokens to the public.**

## Documentation

### Features

This Token sale project has the following features : 

**Presale:**

- Users can contribute Ether to the presale and receive project tokens in return.

- The presale has a maximum cap on the total Ether that can be raised.

- The presale has a minimum and maximum contribution limit per participant.

- Tokens are distributed immediately upon contribution.

**Public Sale:**

- After the presale ends, the public sale begins.

- Users can contribute Ether to the public sale and receive project tokens in return.

- The public sale has a maximum cap on the total Ether that can be raised.

- The public sale has a minimum and maximum contribution limit per participant.

- Tokens are distributed immediately upon contribution.

**Token Distribution:**

- The smart contract should have a function to distribute project tokens to a specified address. This function can only be called by the owner of the contract.

**Refund:**

- If the minimum cap for either the presale or public sale is not reached, contributors should be able to claim a refund.

**Emergency Withdraw:**

- If there was any emergency situation that owner has to withdraw all the funds from contract. This can be done with the Emergency withdraw function.


### Design Choices

**Inheritance and Libraries:** 

- The contract inherits from the `Ownable`, `SafeERC20` contracts, which are part of the OpenZeppelin library. This ensures that only the owner of the contract has certain privileges and safely interact with ERC-20 tokens.

**State Variables:** 

- The contract has various state variables to manage the token sale parameters, such as minimum and maximum caps for both presale and public sale, contribution limits, durations, start times, and amounts raised.

- Separate variables are used to track contributions for presale and public sale, and mappings are used to associate contributions with specific addresses.

**Events:**

- The contract emits events to log important changes and actions, making it easier for external systems to track and respond to contract events.

**Modifiers:**

- Modifiers are used to ensure that certain conditions are met before executing functions. For example, isValidCapacities ensures that the minimum and maximum values are valid, and isValidDuration ensures that the sale duration is greater than zero.

**Constructor:**

- The constructor initializes the contract with the address of the ERC-20 token being sold. If the provided token address is zero, an error is triggered.

**Presale and Public Sale Functions:**

- Separate functions are implemented to start the presale and public sale, each with parameters such as maximum and minimum caps, duration, and contribution limits. These functions can only be called by the owner.

- The functions perform checks to ensure that the presale and public sale are not already active, and that the presale has occurred before starting the public sale.

**Token Purchase Functions:**

- Functions like buyTokensInPresale and buyTokensInPublicsale allow participants to buy tokens during the presale and public sale, respectively. Checks are in place to ensure the sale is active, contribution limits are adhered to, and the minimum and maximum caps are not exceeded.

**Refund Function:**

- The claimRefund function allows participants to claim a refund if the minimum cap for the presale or public sale is not reached. Refunds are only possible after the respective sale has ended.

**Secure Value Transfer:**

- The contract includes a secure internal function (sendValue) to transfer Ether to an address. It checks the contract's balance and ensures that the transfer is successful.

**Setters and Getters:**

- Various setter functions allow the owner to change parameters such as capacities, durations, and contribution limits. Getter functions are provided to check the status of the presale and public sale.

**Emergency Withdraw Function:**

- An emergency withdrawal function (withdraw) is implemented, allowing the owner to withdraw any remaining tokens and Ether from the contract.

**Fallback Function:**

- The contract includes a fallback function that emits an event when it receives Ether when an unexpected ether was sennt to this contract.


### Security Considerations

**Inheritance from OpenZeppelin Contracts:** The contract uses OpenZeppelin's `Ownable` and `SafeERC20` contracts, which are well-tested and widely accepted in the Ethereum community for managing ownership and safe token transfers, respectively.

`Access Control:` The `onlyOwner` modifier from the `Ownable` contract is used throughout the TokenSale contract to restrict certain sensitive functions to be executed only by the contract owner.

**Use of SafeERC20:** The contract uses the `SafeERC20` library for safe token interactions, which helps prevent issues related to token transfers, such as `reentrancy` and token transfer edge cases.

**Custom Errors:** The contract defines custom errors for various failure conditions, making it easier to understand why transactions fail and saving gas compared to traditional `require` statements with error messages.
    ```solidity
    error TokenSale_Invalid_Max_Min();
    error TokenSale_Invalid_Sale_Duration();
    error TokenSale_Invalid_Token();
    error TokenSale_Presale_Already_Active();
    error TokenSale_Presale_Not_Started();
    error TokenSale_Contribution_Limit_Passed();
    error TokenSale_Not_Enough_Contribution();
    error TokenSale_Presale_Max_Cap_Passed();
    error TokenSale_Insufficient_Token_Balance();
    error TokenSale_Publicsale_Not_Started();
    error TokenSale_Publicsale_Max_Cap_Passed();
    error TokenSale_Publicsale_Already_Active();
    error TokenSale_Invalid_To_Address();
    error TokenSale_Sale_Min_Cap_Reached();
    error TokenSale_InsufficientBalance(address);
    error TokenSale_Refund_Tx_Failed();
    error TokenSale_Not_Contributed();
    error TokenSale_Owner_Cant_Buy();
    ```

**Input Validation:** The contract includes input validation using modifiers such as `isValidCapacities` and `isValidDuration` to ensure that the parameters for presale and public sale are within logical and acceptable ranges.

**State Checks:** Functions like `buyTokensInPresale` and `buyTokensInPublicsale` check the state of the sale using conditions to ensure that tokens can only be bought when the sales are active and not otherwise.

**Contribution Tracking:** The contract tracks contributions in mappings, allowing for individual refund logic and ensuring that contributions can be returned if certain conditions (like not reaching the minimum cap) are met.

**Refund Logic:** The contract provides a claimRefund function that allows users to claim refunds after the presale or public sale ends if the minimum cap is not reached, with checks to prevent refunds while the sale is active.

**Emergency Withdrawal:** The `withdraw` function allows the contract owner to withdraw all funds and tokens from the contract in case of emergency, using the `onlyOwner` modifier for access control.

**Visibility and State Mutability Restrictions:** The contract uses appropriate visibility (public, internal, etc.) and state mutability (view, payable, etc.) specifiers to restrict access and modifications to functions and state variables.

**Event Emission:** The contract emits events for significant actions like starting sales, token purchases, and refunds, which helps in tracking contract activity and debugging.

    ```solidity
    event TokenSale_PreSale_Contribution_Limits_Changed(uint indexed min, uint indexed max);
    event TokenSale_PublicSale_Contribution_Limits_Changed(uint indexed min, uint indexed max);
    event TokenSale_Thanks_For_Donation(address indexed sender);
    event TokenSale_Pre_Sale_Started(uint);
    event TokenSale_Public_Sale_Started(uint);
    event TokenSale_Tokens_Sent_In_Presale(address indexed receiver, uint indexed amount);
    event TokenSale_Tokens_Sent_In_Publicsale(address indexed receiver, uint indexed amount);
    event Token_Presale_Refund_Success(address indexed receiver, uint);
    event Token_Publicsale_Refund_Success(address indexed receiver, uint);
    event TokenSale_Emergency_Withdraw(address indexed owner);
    event TokenSale_PublicSale_Cap_Set(uint min, uint max);
    event TokenSale_PreSale_Cap_Set(uint min, uint max);
    ```

**Time-Based Logic:** The contract uses block timestamps to manage the durations of the presale and public sale, ensuring that they only last for the intended period.

**Receive Function:** A `receive` function is included to handle plain unexpected Ether transfers to the contract.


### Test Cases

1. **setUp:** Initializes the testing environment by deploying the EliteToken and TokenSale contracts and transferring some tokens to the TokenSale contract.

2. **testTokenBalance:** Confirms that the TokenSale contract received the correct number of tokens and that the remaining balance is with the test contract (the owner).

3. **testInitialization:** Validates that the TokenSale contract's owner is set correctly.

4. **testSetPresaleCapacity:** Checks that the presale capacity (minimum and maximum caps) can be set correctly.

5. **testSetPresaleCapacityNotOwner:** Ensures that only the owner can set the presale capacity by expecting a revert when a non-owner account attempts to set it.

6. **testSetPresaleCapacityInvalidParams:** Tests various invalid parameters for setting the presale capacity to confirm that the contract correctly reverts these transactions.

7. **testSetPublicsaleCapacity:** Similar to testSetPresaleCapacity, but for the public sale capacity.

8. **testSetPublicsaleCapacityNotOwner:** Similar to testSetPresaleCapacityNotOwner, but for the public sale capacity.

9. **testSetPublicsaleCapacityInvalidParams:** Similar to testSetPresaleCapacityInvalidParams, but for the public sale capacity.

10. **testSetPresaleDuration:** Checks that the presale duration can be set correctly.

11. **testSetPresaleDurationNotOwner:** Ensures that only the owner can set the presale duration.

12. **testSetPresaleDurationInvalidDuration:** Tests that setting an invalid presale duration results in a revert.

13. **testStartPresaleInvalidContribution:** This test checks that the startPreSale function reverts when called with invalid contribution limits. It validates that contributions cannot be zero, and the minimum contribution cannot be higher than the maximum contribution.

14. **testStartPresaleAgain:** This test ensures that attempting to start a new presale while one is already active should fail. It tests that the contract prevents overlapping presales.

15. **testStartPresaleAgainAfterEnd:** This test checks that a new presale can be started after the previous one has ended. It uses vm.warp to simulate time passing and ensures that starting a new presale is allowed only after the previous one has concluded.

16. **testStartPublicsale:** This test verifies that the public sale can start correctly after the presale has ended. It simulates the end of the presale by advancing the block timestamp and then starts the public sale.

17. **testStartPublicsaleNotOwner:** This test confirms that only the owner can start the public sale. It uses vm.prank to simulate a non-owner account attempting to start the sale, expecting the transaction to revert.

18. **testStartPublicsalePresaleRunning:** This test ensures that the public sale cannot start if the presale is still active, expecting the relevant error to be thrown if this rule is violated.

19. **testStartPublicsalePresaleNotStarted:** This test checks that the public sale cannot start if the presale has not been initialized, expecting a revert with the appropriate error.

20. **testStartPublicsaleAgain:** This test verifies that once the public sale has started, attempting to start it again should fail to prevent duplicate sales.

21. **testStartPublicsaleAgainAfterEnd:** This test checks that a new public sale can start after the previous one has ended, similar to the presale tests.

22. **testStartPublicsaleInvalidCap:** This test is similar to testStartPresaleInvalidContribution, but for the public sale. It checks for reverts when invalid presale caps are provided.

23. **testStartPublicsaleInvalidDuration:** This test checks that the public sale cannot be started with a duration of zero, expecting a revert with the appropriate error.

24. **testStartPublicsaleInvalidContribution:** This test is similar to testStartPresaleInvalidContribution, but it checks the public sale's contribution limits for validity.

25. **testBuyTokensInPresale:** This test verifies that tokens can be purchased in the presale. It uses hoax to simulate an account with Ether and checks the token and Ether balances after the purchase.

26. **testBuyTokensInPresaleMaxContribute:** This test checks that an account cannot exceed the maximum contribution limit during the presale by making multiple purchases.

27. **testBuyTokensInPresaleAfterSomeTime:** This test ensures that contributions can still be made after some time has passed in the presale, using `vm.warp` to simulate the passage of time.

28. **testBuyTokensInPresaleContributeLimits:** This test checks that the presale contribution limits are enforced, expecting reverts if the contribution is too high or too low.

29. **testBuyTokensInPresaleNotStarted:** This test ensures that attempting to buy tokens before the presale has started should fail.

30. **testBuyTokensInPresaleOver:** This test confirms that once the presale is over, no more tokens can be purchased, simulating the end of the presale with `vm.warp`.

31. **testBuyTokensInPresaleCapLimits:** This test verifies that the presale respects the maximum cap set for the sale. It simulates three different buyers trying to purchase tokens during the presale, with the third purchase exceeding the presale cap and expecting a revert.

32. **testBuyTokensInPresaleInsufficientBal**: This test checks that the TokenSale contract cannot sell more tokens than it holds. It sets up a scenario where the contract has only a small number of tokens and attempts to sell more than that amount, expecting the transaction to revert due to insufficient token balance.

33. **testBuyTokensInPresaleWhileAndAfterPublicSale:** This test ensures that once the public sale has started, and after it has ended, no purchases can be made through the presale function. It does this by starting the presale, then the public sale, and then trying to make a presale purchase, which should fail.

34. **testBuyTokensInPublicsale**: This function tests the basic functionality of buying tokens during the public sale. It starts with the presale, advances time to allow the public sale to start, and then simulates a purchase of tokens during the public sale.

35. **testBuyTokensInPublicsaleMaxContribute:** Similar to the presale test, this test checks that an individual cannot exceed the maximum contribution limit during the public sale, even with multiple purchases.

36. **testBuyTokensInPublicsaleContributeLimits:** This test ensures that the public sale enforces both minimum and maximum contribution limits, expecting transactions that fall outside these limits to revert.

37. **testBuyTokensInPublicsaleNotStarted:** This test ensures that the public sale function cannot be used before the public sale has officially started, expecting the transaction to revert if attempted.

38. **testBuyTokensInPublicsaleOver:** This test verifies that no purchases can be made through the public sale function once the public sale has ended.

39. **testBuyTokensInPublicsaleCapLimits:** This test is similar to testBuyTokensInPresaleCapLimits, but for the public sale. It ensures that the public sale respects its maximum cap and that attempts to buy beyond this cap will fail.

40. **testBuyTokensInPublicsaleInsufficientBal:** This test, akin to testBuyTokensInPresaleInsufficientBal, checks that the TokenSale contract cannot sell more tokens than it possesses during the public sale.

41. **testBuyTokensInPrePublicsales:** This test checks the scenario where an investor buys tokens during the presale and then again during the public sale, verifying the token balances after each purchase and ensuring that no purchases can be made once the public sale is over.

42. **testsendTokensTo:** This function tests the sendTokensTo functionality of the contract, which is likely a way to distribute tokens to a specific address. It ensures that only authorized accounts (presumably the contract owner) can call this function.

43. **testReceive:** This test seems to check the behavior of the contract when it receives a plain Ether transfer, which might trigger a fallback function designed to handle such transactions. It expects a specific event to be emitted, acknowledging the receipt of the donation.

44. **testIsPresaleActive:** This test checks the isPresaleActive function of the TokenSale contract. It starts the presale and asserts that isPresaleActive returns true. Then it warps the blockchain time by one day and checks that isPresaleActive returns false, indicating the presale has ended.

45. **testIsPublicsaleActive:** This test verifies the isPublicsaleActive function. It starts the presale, then warps time by one day to start the public sale. It checks that isPresaleActive returns false and isPublicsaleActive returns true. After warping time by an additional two days, it asserts that isPublicsaleActive returns false, indicating the public sale has ended.

46. **testIsPublicsaleHappened:** This test checks if the isPresaleHappened function correctly tracks whether the presale has occurred. Initially, it asserts the function returns false. After starting the presale and warping time by three days, it asserts the function returns true.

47. **testClaimRefundPresale:** This test ensures that participants can claim refunds after the presale ends. It starts the presale, has an account buy tokens, then warps time by one day and calls claimRefund. It checks that the account's balance is refunded.

48. **testClaimRefundPresaleNotEnd:** This test confirms that trying to claim a refund while the presale is still active should fail and cause a revert with the appropriate error.

49. **testClaimRefundPublicsale:** Similar to the presale refund test, this checks that participants can claim refunds after the public sale ends. It starts the presale, starts the public sale after one day, has an account buy tokens, warps time by two days, and then calls claimRefund, checking for a refund.

50. **testClaimRefundPrePublicsalesNotEnd:** This test verifies that attempting to claim a refund from the public sale while it's still active should fail, resulting in a revert with the appropriate error.

51. **testClaimRefundPrePublicsalesMinCapReached:** This test ensures that if the minimum cap of the sale is reached, participants cannot claim refunds, expecting a revert with the appropriate error.

52. **testWithdraw:** This test checks the withdraw function of the TokenSale contract. It expects an event to be emitted indicating an emergency withdrawal and confirms that only authorized accounts can call this function, expecting a revert for unauthorized accounts.

53. **testDeployScript:**This test ensures that the deployment script works correctly. It deploys the contracts using the DeployScript and checks that the token and sale contract addresses are non-zero and that the sale contract has the correct token balance.

And 10 more tests....

### Usage

```shell
$ forge build
```

### Test

```shell
$ forge test
```

### Gas Snapshots

```shell
$ forge snapshot
```

### Deploy

```shell
$ forge script script/DeployScript.s.sol:DeployScript --rpc-url <your_rpc_url> --private-key <your_private_key>
```