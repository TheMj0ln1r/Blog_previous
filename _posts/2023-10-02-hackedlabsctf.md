---
layout: post
title:  "0xHackedLabs x OtterSec CTF 2023"
command: cat blockchainctf
---

Hello peers, I participated in a blockchain ctf conducted by 0xHacked Labs and OtterSec. These are the challenge writeups which i solved. We dont submit any flag to solve the challenge, we need to generate the ZK Proof of our exploit then that ZK Proof should be submitted to solve the challenge.


# BabyOtter

```
As easy as 1, 2, 1337..

rpc: http://18.207.142.64:38545
contract: 0x4e309C767Acc9f9366d75C186454ed205d5Eeee3
block_number: 3

Attached Files : [ https://github.com/0xHackedLabs/ctf/tree/main/src/BabyOtter/BabyOtter.sol ]
```


`BabyOtter.sol`

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

contract BabyOtter {
    bool public solved = false;

    function solve(uint x) public {
        unchecked {
            assert(x * 0x1337 == 1);
        }
        solved = true;
    }
}
```

The observation is that the contract uses `unchecked` block. unchecked block wont check for overflow of integer arithmetic. That means a `uint256` variable can store values less than `2**256-1`. If we tried to store values more than this value the value will be wrapped to mod `2**256`. 

Simply, `uint256 x = 2**256 + 1` will become `0`. `uint256 x = 2**256 + 2` will be `1`.

So, we need to find value of `x` such that which can satisfy the equation `x * 0x1337 = 1`. We can think of this as a modular multiplicative inverse. 

`x = pow(0x1337, -1, 2**256)`

In solidity we can find this as `x  =  0x1337 ** (2**254 -1)`. So, simply calling solve function with this value with the x value to solve the chalenge.

`Exploit.sol`

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

interface IBabyOtter {
    function solve(uint x) external;
}
contract Exploit {
    function exploit() public {
        // write code here
        address target = 0x4e309C767Acc9f9366d75C186454ed205d5Eeee3;
        uint leet = 0x1337;
        uint x;
        unchecked{x  =  leet ** (2**254 -1);}
        IBabyOtter(target).solve(x);
    }
}
```

We can test this challenge with foundry test script.

`Exploit.t.sol`

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import {Exploit as BoE} from "../src/BabyOtter/Exploit.sol";
interface IChallenge {
    function solved() external returns(bool);
}

contract ExploitTest is Test {

    function setUp() public {
        // fork the ctf chain
        vm.createSelectFork("http://18.207.142.64:38545", 3);
    }

    function test_babyotter() public {
        address target = 0x4e309C767Acc9f9366d75C186454ed205d5Eeee3;

        BoE exp = new BoE();
        exp.exploit();
        assertTrue(IChallenge(target).solved());
    }
}
```

`forge test --match-test test_babyotter -vv` will confirm the challenge is solved. Submitting the exploit ZK Proof will credit some points in CTF.


# ChildOtter

```
rpc: http://18.207.142.64:38545
contract: 0x63461D5b5b83bD9BA102fF21d8533b3aad172116
block_number: 3

Attached Files : [ https://github.com/0xHackedLabs/ctf/tree/main/src/ChildOtter/ChildOtter.sol ]
```

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

contract ChildOtter {
    mapping(uint => mapping(uint => uint)) val;
    bool public solved = false;

    function solve(uint x) public { 
      uint target;

      val[0][0] = x;
      assembly {
        target := mload(32)
      }
      assert(target == x);
      solved = true;
    }
}
```

This is the challenge for which I learned a lot of new things. To solve this I revised entire EVM internals from many sources(see references).

The challenge goal is to call the solve function with the value which is store at the EVM memory 32 bytes to 64 bytes. That is what actually `mload(32)` is doing.

Solidity reserves four 32-byte slots, with specific byte ranges (inclusive of endpoints) being used as follows:

`0x00 - 0x3f` (64 bytes): scratch space for hashing methods

`0x40 - 0x5f` (32 bytes): currently allocated memory size (aka. free memory pointer)

`0x60 - 0x7f` (32 bytes): zero slot

Scratch space can be used between statements (i.e. within inline assembly). The zero slot is used as initial value for dynamic memory arrays and should never be written to (the free memory pointer points to 0x80 initially).

The challenge contract defines a nested mapping and stored a value in the mapping at `key [0][0]`.

The variable `val` is assigned to storage slot 0 (simply because it is the first one). But the mapped elements are elsewhere.

To compute the storage slot or location of the mapped elements, the following is used :

`slot = keccak256([key, mappingSlot]) // concatenate key and mapping slot`

So, when storing values in a mapping EVM will compute keccak256 hash of the key and mapping slot. EVM will use `scratch space` to perform hashing. So, that means `0x00-0x3f`(0 - 64 bytes) will store the `keccak256([0,0])`.

The actual `value[0][0] = x` will be stored at `storage` slot `uint(keccak256([0,keccak256([0,0]))))`. In memory from `32 bytes to 64 bytes` are used to store the `keccak256([0,0])`. 

Lets confirm this with remix debugger.

<img src ="/assets/img/post_img/hackedlabsctf_1.png" alt="childOtter remix" class="autoimg">

Calling solve function with this value will solve the challenge.

`Exploit.sol`

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

interface IChildOtter {
    function solve(uint x) external;
}
contract Exploit {
    function exploit() public {
        // write code here
        address target = 0x63461D5b5b83bD9BA102fF21d8533b3aad172116;
        uint x = uint(keccak256(abi.encode(0,0))); // slot 0, key 0
        IChildOtter(target).solve(x);
    }
}
```

We can test this challenge with foundry test script.

`Exploit.t.sol`

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import {Exploit as CoE} from "../src/ChildOtter/Exploit.sol";
interface IChallenge {
    function solved() external returns(bool);
}

contract ExploitTest is Test {

    function setUp() public {
        // fork the ctf chain
        vm.createSelectFork("http://18.207.142.64:38545", 3);
    }

    function test_childotter() public {
        address target = 0x63461D5b5b83bD9BA102fF21d8533b3aad172116;
        CoE exp = new CoE();
        exp.exploit();
        assertTrue(IChallenge(target).solved());
    }
}
```

`forge test --match-test test_childotter -vv` will confirm the challenge is solved. Submitting the exploit ZK Proof will credit some points in CTF.

***

### References

1. <a href="https://ethereum.stackexchange.com/questions/114186/how-does-ethereum-fit-a-mapping-into-storage" target=_blank>StackOverflow</a>
2. <a href="https://docs.soliditylang.org/en/latest/internals/layout_in_memory.html" target=_blank>Solidity lang docs</a>
3. <a href="https://youtu.be/i_LwhlFNSkI?feature=shared" target=_blank>How mappings stored in EVM - MappingsJesper Kristensen YT</a>
4. <a href="https://youtu.be/Ru3inmu1FuQ?feature=shared" target=_blank>EVM Explained - Owen Thurm YT</a>
5. <a href="https://youtu.be/kCswGz9naZg?feature=shared" target=_blank>EVM Guide - Jordan McKinney YT</a>



