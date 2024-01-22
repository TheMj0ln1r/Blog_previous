---
layout: post
title:  "Decentralized Voting"
command: cat dvoting
---

A simple Decentralized Project developed by me for an internship task.

Project Repo :  <a href="https://github.com/TheMj0ln1r/Decentralized-Voting-System" target=_blank>Decentralized Voting Project</a>

***

# **Decentralized Voting System**


The system has following features:

- Users can register to vote.

- The owner of the contract can add candidates.

- Only owner can start an election.

- Multiple Elections can be conducted.

- Registered voters can cast their votes for a specific candidate.

- The voting process should be transparent, and the results should be publicly accessible.

## Design Choices

The Election contract is designed to manage a simple voting process, with features to register voters and candidates, cast votes, and reveal election results. 

**Access Control:** The contract uses an onlyOwner modifier to restrict certain functions (like registering candidates and revealing results) to the owner of the contract, which is set during contract deployment.

**Voter Tracking:** Voters are tracked using mappings that associate their addresses with respective Voter. This design allows for efficient lookups and updates.

**Unique Identifiers:** Both voters and candidates are assigned unique IDs to differentiate them within their respective categories.
Vote Casting Mechanism: Voters can cast votes for registered candidates, and the contract ensures that each voter can only vote once.

**Election Results:** The contract allows the owner to calculate and reveal the winner based on the number of votes each candidate received. The reveal results feature is a bit complex as it supports if the multiple candidates got the same majority.

**Election Duration:** The contract checks whether the election is active before allowing certain operations (e.g., voter registration, voting). This prevents these operations from occurring outside the specified election duration.

**Events:** The contract emits events for candidate registration, voter registration, vote casting, and winner declaration, which helps in tracking activities on the blockchain.

**Error Handling:** The contract uses error messages to clearly communicate the reasons for transaction failures. For example, it raises errors like Election_Not_Owner(), Election_Voter_Already_Registered(), etc. This helps users and developers understand the context of the failure.

**Time-Based Logic:** The contract uses `electionDuration` and `electionStartTime` to manage the duration of the election.

**Dynamic Arrays**: The `candidateDetails` array is used to store a dynamic list of `Candidate` structs. This allows for an arbitrary number of candidates to be registered in the election.

**State Variables:**

`owner`: The address of the contract owner.

`totalNumOfVoters`: A counter for the total number of registered voters.

`totalNumOfCandidates`: A counter for the total number of registered candidates.

`voterDetails`: A mapping from voter addresses to Voter structs.

`candidateDetails`: A mapping from candidate addresses to Candidate 

**Structs**:

`candidIdToAddr`: A mapping from candidate IDs to their addresses.
Structs:

`Voter`: Contains voter ID and a boolean indicating whether the voter has voted.

`Candidate`: Contains candidate ID and the number of votes received.

**Modifiers**:

`onlyOwner`: Restricts function access to the contract owner.

**Functions**:

`registerVoter`(): Allows an address to register as a voter.

`registerCandidate(address _candidate)`: Allows the owner to register a new candidate.

`castVote(address _candidate)`: Allows a registered voter to cast a vote for a registered candidate.

`revealResults()`: Allows the owner to calculate and reveal the winner of the election.

`getMajority(address _candidate)`: Returns the number of votes for a given candidate.

**Events**:

`Election_Candidate_Registered`: Emitted when a new candidate is registered.

`Election_Voter_Registered`: Emitted when a new voter is registered.

`Election_Voted`: Emitted when a vote is cast.

`Election_Winner_Declared`: Emitted when the election winner is declared.

`Election_Election_Started` : Emitted when election was started by owner.


## Security Considerations

The Election smart contract incorporates several security considerations to protect the integrity of the election process:

**Access Control:** The contract uses an onlyOwner modifier, which employs a simple authorization check to restrict access to sensitive functions such as registerCandidate and revealResults. This ensures that only the owner can perform certain actions 1.

**Checks-Effects-Interactions Pattern:** The contract adheres to the Checks-Effects-Interactions pattern by performing checks first (such as verifying if a voter or candidate is already registered), updating the contract state, and then emitting events 13.

**State Variable Visibility:** State variables such as owner, totalNumOfVoters, and totalNumOfCandidates are declared public, allowing transparency and verification of contract state by external observers 25.

**Use of Custom Errors:** The contract uses custom error messages for reverting transactions when certain conditions are not met. This helps in reducing gas costs compared to using require with string messages and improves clarity regarding why a transaction failed 1.

    ```solidity
    error Election_Not_Owner();
    error Election_Voter_Already_Registered();
    error Election_Candidate_Already_Registered();
    error Election_Invalid_Candidate();
    error Election_Already_Voted();
    error Election_Voter_Not_Registered();
    error Election_Election_Running();
    error Election_Election_Completed();
    error Election_Invalid_Duration();
    ```

**Event Emission:** Events are emitted for key actions such as voter registration, candidate registration, vote casting, and winner declaration. This allows for off-chain monitoring and verification of contract interactions 3.

**Minimal Use of Loops:** The contract avoids extensive use of loops, which can lead to out-of-gas errors. The only loop present is in the revealResults function, which iterates through candidates to determine the winner. This loop is bound by the number of candidates, which is expected to be a manageable number 13.

**No External Calls:** The contract does not make external calls to other contracts, which reduces the risk of reentrancy attacks and unexpected interactions with untrusted contracts 12.

**Immutable Owner:** The owner is set once upon contract deployment and is not changeable. This prevents potential takeover attacks where the ownership could be transferred to a malicious party 1.

**No Ether Handling:** The contract does not handle Ether transactions, which means there's no risk associated with sending and receiving Ether, nor any need for a fallback function 1.

## Test Cases

**setUp:** This function initializes the test environment. It creates instances of the Election contract and assigns addresses to voters and candidates.

**testInitialElectionState:** This test checks the initial state of the Election contract. It asserts that the owner is set correctly, and that the initial number of voters, election duration, and election start time are all zero.

**testRegisterVoter:** This test verifies the functionality of the registerVoter function. It checks that a voter can be registered, and that attempting to register the same voter twice causes a revert.

**testRegisterVoterDuringElection:** This test checks that a voter cannot be registered during an election. It attempts to register a voter while an election is running and expects a revert.

**testRegisterCandidate:** This test verifies the registration of candidates. It checks that a candidate can be registered, that attempting to register the same candidate twice causes a revert, and that attempting to register an invalid candidate also causes a revert.

**testRegisterCandidateDuringElection:** This test checks that a candidate cannot be registered during an election. It attempts to register a candidate while an election is running and expects a revert.

**testStartElection**: This test checks the startElection function. It verifies that an election can be started, and that starting an election with an invalid duration causes a revert.

**testStartElectionAlreadyRunning:** This test verifies that an election cannot be started if another one is already running. It attempts to start an election while another one is running and expects a revert.

**testCastVote:** This test checks the castVote function. It verifies that a vote can be cast by a registered voter, and that attempting to cast a vote with an unregistered voter or an invalid candidate causes a revert.

**testGetMajority:** This test checks the getMajority function. It verifies that the majority of votes is calculated correctly for a single candidate.

**testIsElectionActive:** This test checks the isElectionActive function. It verifies that the function correctly identifies whether an election is currently active.

**testRevealResults:** This test verifies the revealResults function. It checks that the function correctly declares the winner of an election.

**testRevealResultsMultipleWinners:** This test checks that the revealResults function correctly handles elections with multiple winners.

**testRevealResultsNotOwner:** This test checks that the revealResults function can only be called by the owner of the election.

**testScript:** This test verifies the execution of the ElectionScript. It checks that the script runs successfully and that the Election contract instance it creates is valid.


```shell
mj0ln1r@Linux:~/decentralized_voting_system$ forge coverage
[⠢] Compiling...
[⠊] Compiling 24 files with 0.8.23
[⠔] Solc 0.8.23 finished in 7.80s
Compiler run successful!
Analysing contracts...
Running tests...
| File                  | % Lines         | % Statements   | % Branches     | % Funcs       |
|-----------------------|-----------------|----------------|----------------|---------------|
| script/Election.s.sol | 100.00% (3/3)   | 100.00% (3/3)  | 100.00% (0/0)  | 100.00% (1/1) |
| src/Election.sol      | 100.00% (55/55) | 96.34% (79/82) | 88.24% (30/34) | 100.00% (7/7) |
| Total                 | 100.00% (58/58) | 96.47% (82/85) | 88.24% (30/34) | 100.00% (8/8) |
```

## Usage

### Clone

```shell
$ git clone 
$ cd 
```

### Build

```shell
$ forge build
```

### Test

```shell
$ forge test
```

### Deploy

```shell
$ forge script script/Election.s.sol:ElectionScript --rpc-url <your_rpc_url> --private-key <your_private_key>
```