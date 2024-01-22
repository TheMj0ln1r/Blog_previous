---
layout: post
title:  "MultiSig Project"
command: cat multisig
---

A simple MultiSig Wallet Project developed by me for an internship task.

Project Repo :  <a href="https://github.com/TheMj0ln1r/MultiSig-Wallet" target=_blank>MultiSig Project</a>

***

## **Multi Signature Wallet**

**A multi-signature (multisig) wallet is a smart contract that requires multiple private keys to authorize and execute a transaction. This provides an additional layer of security compared to traditional single-signature wallets, where only one private key is needed. Multisig wallets are commonly used in various decentralized applications, including cryptocurrency platforms and decentralized autonomous organizations (DAOs).**


## Documentation

### Features
This multi-sig contract has the following features:

- The contract allows multiple owners to collectively control the funds in the wallet. 

- Wallet has multiple owners, each having their own private key.

- A specified number of owners (threshold) is required to approve and execute a transaction.

- Owners can submit, approve, and cancel transactions.

### Design Choices

I have designed this MultiSig contract with the following functionalities with my design choices to make this efficient.

1. **Structs for Transactions** : This choice makes it easy to manage and organize transaction data. The struct encapsulates the details of each transaction, including the owner initiating it, whether it has been executed, the recipient address, value, and data.

2. **Approval Status** : Used a mapping to track approval status for each transaction. Storing approval status in a mapping (`isApproved`) allows for efficient lookup and management of transaction approvals. It helps in quickly determining whether an owner has already approved a specific transaction.

3. **Several Modifiers**: Implemented modifiers like `validTxId`, `onlyOwner`, `hasToExecuted`, and `hasToApproved` for access control. Modifiers help in code reusability and readability. They enforce certain conditions before executing a function, making the code more secure and preventing unauthorized access. 

4. **ThreshHold** :  Used a `threshold` value to determine when a transaction is considered confirmed. The contract requires a minimum number of owner approvals (`threshHold`) to execute a transaction.

5. **Revoking Approve**: Implemented a `revokeApprove` function. Allowing owners to revoke their approvals provides additional control and flexibility. If an owner changes their mind before a transaction is executed, they can revoke their approval, preventing the transaction from going through.

6. **Cancellation of Tx** : Implemented a `cancelTx` function. Owners have the ability to cancel transactions they initiated, providing a way to clean up the transaction list and prevent the execution of unwanted transactions. I have implemented this `cancelTx` function in such a way that it can only be callable by the transaction owner. i.e, who submits that transaction.

7. **Many Events** : Events are emitted to provide transparency and allow external systems to react to state changes.

    ```solidity
    // Events
    event MultiSig_Transaction_Submit(uint indexed txId, address indexed from, address _to, uint value, bytes data);
    event MultiSig_Transaction_Approved(uint indexed _txId, address indexed _owner);
    event MultiSig_Transaction_Executed(uint indexed _txId);
    event MultiSig_EthDeposited(address indexed sender, uint indexed value);
    event MultiSig_Approve_Revoked(address indexed _owner, uint indexed _txId);
    ```

8. **Custom Errors** : I have used custom error types to be more specific to the user when they encounter any error. 

    ```solidity
    // Errors
    error MultiSig_Not_Enough_Owners();
    error MultiSig_Invalid_ThreshHold_Specified();
    error MultiSig_Invalid_Owner();
    error MultiSig_Not_Owner();
    error MultiSig_Invalid_To();
    error MultiSig_Invalid_TransactionID();
    error MultiSig_Tx_Already_Approved();
    error MultiSig_Tx_Already_Executed();
    error MultiSig_Transaction_Not_Yet_Confirmed();
    error MultiSig_Transaction_Failed();
    error MultiSig_Tx_Not_Approved();
    error MultiSig_Not_Tx_Owner();
    error MultiSig_Insufficient_Wallet_Balance();
    ```
9. **Added a receive function** : To be able to receive deposited ether to the wallet. Upon receiving ether the Wallet will emit Deposit event.

### Security Considerations

I have taken some important security considerations in the `MultiSig` contract which handles many edge cases and eliminates vulnerabilities in the contract.

1. **Access COntrol** : Access to the functions were handled by modifiers, for example : Submitting a transaction can only possible for a owner. This check done my `onlyOwner()` modifier.

2. **Transaction State Checks** : Modifiers like executed and approved prevent actions such as double approvals or double executions of transactions.

3. **Constructor validation** : The owners specified in constructor were checked again duplicate owner addresses and zero addresses. The threshold is ensured to less than total number of owners and atleast 2.

4. **Custom Errors** : Custom errors provide informative feedback and help save gas compared to traditional require statements with error messages. 

5. **Event Emission** : The contract emits events for significant actions like transaction submission, approval, execution, and fund deposits.

6. **Cancelling Transaction** : The cancellation of transaction can only done before the transaction executes and only by the owner who submits that transaction.

7. **Executing Transaction** : The transaction can only executed by a owner when the transaction get enough approvals (threshold). The transaction existence is checked by `validTxId` modifier.

8. **Low level call handle** : The low level call is used to send the valut to the address. This `call` is handled properly by ensuring its return value. And the `checks-effect-interactions` is followed to mitigate `Reentrancy`. 

### Test Cases

I have implemented near 100% test coverage for the MultiSig contract. Which ensures following test cases.

Here's an outline of the test cases we will cover:

1. Test the contract initialization and owner settings.

2. Test submitting a transaction by an owner.

3. Test approving a transaction by an owner.

4. Test approving a transaction reverts if already approved.

5. Test executing a transaction after reaching the threshold.

6. Test executing a transaction reverts if not enough approvals.

7. Test revoking an approval.

8. Test revoking an approval reverts if not previously approved.

9. Test transaction execution reverts if it's already executed.

10. Test canceling a transaction by its owner.

And even more...

```bash
$ forge coverage

Running tests...
| File                  | % Lines         | % Statements    | % Branches     | % Funcs       |
|-----------------------|-----------------|-----------------|----------------|---------------|
| script/MultiSig.s.sol | 100.00% (9/9)   | 100.00% (10/10) | 100.00% (0/0)  | 100.00% (1/1) |
| src/MultiSig.sol      | 100.00% (24/24) | 97.06% (33/34)  | 92.86% (13/14) | 100.00% (6/6) |
| Total                 | 100.00% (33/33) | 97.73% (43/44)  | 92.86% (13/14) | 100.00% (7/7) |
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

### Gas Snapshots

```shell
$ forge snapshot
```

### Anvil

```shell
$ anvil
```

### Deploy

```shell
$ forge script script/MultiSig.s.sol:MultiSigScript --rpc-url <your_rpc_url> --private-key <your_private_key>
```
