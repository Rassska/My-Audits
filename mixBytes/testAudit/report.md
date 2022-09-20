# XDAO Security Audit Report

## Introduction

### Project Overview
 - XDAO is a decentralized autonomous organization which provides an ability for xProtocol users to propose some changes within the core protocol.

### In Scope
  - **The scope of the audit includes the following smart contracts at:**
    - ~/contracts:
        - [AccessControlled.sol](https://github.com/Rassska/SC-Audits/blob/main/mixBytes/testAudit/contracts/AccessControlled.sol)
        - [AccessControlledRoles.sol](https://github.com/Rassska/SC-Audits/blob/main/mixBytes/testAudit/contracts/AccessControlledRoles.sol)
        - [Constants.sol](https://github.com/Rassska/SC-Audits/blob/main/mixBytes/testAudit/contracts/Constants.sol)
        - [Errors.sol](https://github.com/Rassska/SC-Audits/blob/main/mixBytes/testAudit/contracts/Errors.sol)
        - [Proposal.sol](https://github.com/Rassska/SC-Audits/blob/main/mixBytes/testAudit/contracts/Proposal.sol)
        - [ProposalQueue.sol](https://github.com/Rassska/SC-Audits/blob/main/mixBytes/testAudit/contracts/ProposalQueue.sol)
        - [Uint256Helpers.sol](https://github.com/Rassska/SC-Audits/blob/main/mixBytes/testAudit/contracts/Uint256Helpers.sol)
        - [VAccessControl.sol](https://github.com/Rassska/SC-Audits/blob/main/mixBytes/testAudit/contracts/VAccessControl.sol)
        - [VBeacon.sol](https://github.com/Rassska/SC-Audits/blob/main/mixBytes/testAudit/contracts/VBeacon.sol)
        - [VBeaconProxy.sol](https://github.com/Rassska/SC-Audits/blob/main/mixBytes/testAudit/contracts/VBeaconProxy.sol)
        - [VetoNFT.sol](https://github.com/Rassska/SC-Audits/blob/main/mixBytes/testAudit/contracts/VetoNFT.sol)
        - [VotingDAOV2.sol](https://github.com/Rassska/SC-Audits/blob/main/mixBytes/testAudit/contracts/VotingDAOV2.sol)
    - ~/interfaces:
        - [IMiniMeToken.sol](https://github.com/Rassska/SC-Audits/blob/main/mixBytes/testAudit/interfaces/IMiniMeToken.sol)

  - Prior commit identifier: **9a89bc739ec01ade279ae27d797f27e9924efa09**
  - Audited commit identifier: **Todo**
  
## Codebase Summary & Key Improvement Opportunities:

### Key Features
  - The in-scope system was found to be low in complexity, with the key features being as follows:
    - [createProposal()](https://github.com/Rassska/SC-Audits/blob/35413d01ab0a6dd1ea42ce6d5343ca97f524a109/mixBytes/testAudit/contracts/VotingDAOV2.sol#L61-L63) creates empty proposal template
      - [in order to withdrawETH()](https://github.com/Rassska/SC-Audits/blob/35413d01ab0a6dd1ea42ce6d5343ca97f524a109/mixBytes/testAudit/contracts/VotingDAOV2.sol#L72-L78)
      - [in order to withdrawToken()](https://github.com/Rassska/SC-Audits/blob/35413d01ab0a6dd1ea42ce6d5343ca97f524a109/mixBytes/testAudit/contracts/VotingDAOV2.sol#L88-L95)
    - [vote()](https://github.com/Rassska/SC-Audits/blob/35413d01ab0a6dd1ea42ce6d5343ca97f524a109/mixBytes/testAudit/contracts/VotingDAOV2.sol#L124-L150) allows for users to share their support on active proposals
    - [veto()](https://github.com/Rassska/SC-Audits/blob/35413d01ab0a6dd1ea42ce6d5343ca97f524a109/mixBytes/testAudit/contracts/VotingDAOV2.sol#L104-L115) provides an ability to veto a specific active proposal for someone having veto tokens.
    - [execute()](https://github.com/Rassska/SC-Audits/blob/35413d01ab0a6dd1ea42ce6d5343ca97f524a109/mixBytes/testAudit/contracts/VotingDAOV2.sol#L159-L177) makes possible for anyone to execute certain proposal.

### Code Quality and Test Coverage
  - In summary, the code quality of the XDAO is pretty high. The codes were also found to be well-documented and team took the efforts to document the NatSpec for all the functions within the `VotingDAOV2.sol` contract. However, there are still some room of improvement as NatSpec was not documented for the `Proposal.sol` contract. For completeness and readability, it is recommended to document the NatSpec for all the functions in the contracts where feasible.

  - The lack of test coverage is something which makes it difficult for auditors to review the codebase. It is always mandatory point to have nearly ~100% test coverage of the crucial parts. 

### Re-entrancy Risks

  - The key features (e.g. `withdrawETH()`, `withdrawToken()`) were not found to be following the “Checks Effects Interactions” pattern rigorously, which leads to having some possiblities for re-entrancy attacks. Therefore, further improvement can be made to guard against future re-entrancy attacks. As the key features relies on many external contract calls via token/eth transfers, the risks of re-entrancy attack is significantly higher for XDAO compared to other protocols. Thus, it is prudent to implement additional reentrancy prevention wherever possible by utilizing the nonReentrant modifier from [Openzeppelin Library](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/security/ReentrancyGuard.sol) to block possible re-entrancy as a defense-in-depth measure.

## Summary of Findings:

- **[[C-01] Re-entrancy in `execute()` for ETH withdrawal proposals leads to draining the protocol](C-01)**
- **[[C-02] It's possible to drain a protocol by taking a flashloan over governance token to reach the quorum in a single tx](C-02)**
- **[[C-03] Delegate call into malicious contract could lead the proxy to be destroyed](C-03)**
- **[[C-04] Unnecessary decrease of votes in `vote()` leading to DoS](C-04)**

- **[[H-01] createProposal() could be front-run by an attacker with dummy proposals leading to DoS](H-01)**

- **[[M-01] VetoNFT token might be minted to non-ERC721 compatible receivers](M-01)**
- **[[M-02] Unchecked return value for ERC20 `.transfer()`](M-02)**
- **[[M-03] Unchecked return value for delegate call](M-03)**


- **[[L-01] Redundant `.isAccepted()` in `execute()`](L-01)**

- **[[G-01] Cache state variables, `MLOAD` << `SLOAD`](G-01)**
- **[[G-02] Consider `a = a + b` instead of `a += b`](G-02)**
- **[[G-03] Consider marking functions with VETO_MINT role as payable](G-03)**
- **[[G-04] Use `private/internal` for `constants/immutable/state` variables instead of `public`](G-04)**
- **[[G-05] Use `calldataload` instead of `MLOAD`/`MSTORE`](G-05)**


## **[[C-01] Re-entrancy in `execute()` for ETH withdraw proposals leads to draining the protocol](C-01)**
### ***Description:***
  - If the destination address is not an EOA, it's possible for an attacker to simply put some logic into `fallback()` function in order to get back and execute the proposal second/third/fourth and so on times. Since the `execute()` doesn't follow to the pattern "Checks Effects Interactions", therefore, all described above is possible.
  
### Proof of Concept
  - XDAO wanted to send some credits in ETH to zkAlice for writing exellent posts about xProtocol. But zkAlice wasn't agree about the amount of credits that XDAO was willing to pay. So, she makes a plan to attack the whole DAO. After noticing that it's possible to re-entrant the `execute()` function, she decided to put a logic in a `fallback()` function in order to get back to the XDAO and call the `execute()` again and again. Fortunately, zkBob disclosed that critical issue before that and all funds were saved. 

  - **All occurances:**

    - contracts/VotingDAOV2.sol 
  
    ```Solidity
      function execute(uint256 hash) external {
          (bool found, uint8 index) = find(hash);
          require(found, Errors.ERROR_NOT_FOUND);

          Proposal storage proposal = proposals[index];
          require(!proposal.vetoed, Errors.ERROR_VETOED);
          require(!proposal.isExpired(), Errors.ERROR_EXPIRED);
          require(!proposal.isExecuted, Errors.ERROR_ALREADY_EXECUTED);
          require(proposal.isQuorumReached(), Errors.ERROR_QUORUM_IS_NOT_REACHED);
          require(proposal.isAccepted(), Errors.ERROR_CANNOT_EXECUTE_REJECTED_PROPOSAL);

          if (proposal.payment.amount > 0 && proposal.isAccepted()) {
              _withdraw(proposal.payment);
          }

          proposal.isExecuted = true;

          emit ProposalExecuted(hash);
      }

      function _withdraw(Payment storage payment) internal {
          if (payment.sendETH) {
              _withdrawETH(payment.destination, payment.amount);
          } else {
              _withdrawToken(IERC20(payment.token), payment.destination, payment.amount);
          }
      }

      function _withdrawETH(address destination, uint256 amount) internal {
          // mistype check
          require(destination != address(0), Errors.ERROR_ZERO_ADDRESS);

          // ensure there are enough funds
          // leave that requirement for testing purposes
          // to unify revert's error messages
          require(address(this).balance >= amount, Errors.ERROR_TREASURY_INSUFFICIENT_BALANCE);

          // send eth
          destination.call{value: amount}("");
      }


    ```

### Recommended Mitigation Steps
  - Short term: it's possible to mark `execute()` as nonReentrant by using [ReentrancyGuard](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/security/ReentrancyGuard.sol) from OZ. 
  - Long term: consider putting `proposal.isExecuted = true` before invoking  `_withdraw(proposal.payment)`

## **[[C-02] It's possible to take a flashloan over governance token to reach the quorum in a single tx](C-02)**

### ***Description:***
  - An attacker has an ability to take a flash loan in order to buy 50%+ governance tokens, which is pretty much enough to reach the quorum. Well, after that he can create a proposal to drain the entire XDAO and successfully vote and execute that in a single transaction. After receiving the tokens and ETH he simply repays the flashloan(e.g selling back governance tokens) + some part from his profit. 
  
### Proof of Concept

  - **All occurances:**

    - contracts/VotingDAOV2.sol 
  
    ```Solidity
      function execute(uint256 hash) external {
          (bool found, uint8 index) = find(hash);
          require(found, Errors.ERROR_NOT_FOUND);

          Proposal storage proposal = proposals[index];
          require(!proposal.vetoed, Errors.ERROR_VETOED);
          require(!proposal.isExpired(), Errors.ERROR_EXPIRED);
          require(!proposal.isExecuted, Errors.ERROR_ALREADY_EXECUTED);
          require(proposal.isQuorumReached(), Errors.ERROR_QUORUM_IS_NOT_REACHED);
          require(proposal.isAccepted(), Errors.ERROR_CANNOT_EXECUTE_REJECTED_PROPOSAL);

          if (proposal.payment.amount > 0 && proposal.isAccepted()) {
              _withdraw(proposal.payment);
          }

          proposal.isExecuted = true;

          emit ProposalExecuted(hash);
      }

      function _withdraw(Payment storage payment) internal {
          if (payment.sendETH) {
              _withdrawETH(payment.destination, payment.amount);
          } else {
              _withdrawToken(IERC20(payment.token), payment.destination, payment.amount);
          }
      }

      function _withdrawETH(address destination, uint256 amount) internal {
          // mistype check
          require(destination != address(0), Errors.ERROR_ZERO_ADDRESS);

          // ensure there are enough funds
          // leave that requirement for testing purposes
          // to unify revert's error messages
          require(address(this).balance >= amount, Errors.ERROR_TREASURY_INSUFFICIENT_BALANCE);

          // send eth
          destination.call{value: amount}("");
      }


    ```

### Recommended Mitigation Steps
  - Short term: making it impossible to `create()` the proposal and `execute()` it within the same block. 
  - Long term: designing a mechanism that will set the time lock for n days before proposal execution process.

## **[[C-03] Delegate call into malicious contract could lead the proxy to be destroyed](C-03)**

### ***Description:***
  - It's possible for anyone to call `__fallback()` in a `VBeaconProxy` which simply delegate calls into the caller. However, if the caller is a smart-contract which simply invokes selfdestruct in his `fallback()` function. The proxy will be destroyed. 
  
### Proof of Concept

  - **All occurances:**

    - contracts/VotingDAOV2.sol 
  
    ```Solidity
      function __fallback() external virtual {
          msg.sender.delegatecall("TODO: implement an emergency method to temporarily pause the contract in the future...");
      }
    ```

### Recommended Mitigation Steps
  - Short term: Delegate calls should be proceeded only into trusted contracts(e.g. implementation). 
  - Long term: carefully re-disign beacon proxy pattern.

## **[[C-04] Unnecessary decrease of votes in `vote()` leading to DoS](C-04)**

### ***Description:***
  - In `VotingDAOV2.sol` there is a function `vote(uint256, bool)` which allows to anyone to vote for any active proposal by. However, before the line `proposal.vote(support, votingPower)` it descreses `proposal.yeas -= votingPower` and `proposal.nays -= votingPower` for no reason. And it may cause underflow each time, when vote() is invoked. 
  
### Proof of Concept

  - **All occurances:**

    - contracts/VotingDAOV2.sol 
  
    ```Solidity
      function vote(uint256 hash, bool support) external {
          (bool found, uint8 index) = find(hash);
          require(found, Errors.ERROR_NOT_FOUND);

          Proposal storage proposal = proposals[index];
          require(!proposal.vetoed, Errors.ERROR_VETOED);
          require(!proposal.isExpired(), Errors.ERROR_EXPIRED);
          require(!proposal.isExecuted, Errors.ERROR_ALREADY_EXECUTED);

          uint32 votingPower = votingToken.balanceOfAt(msg.sender, proposal.createdBlockNumber).toUint32();
          require(votingPower > 0, Errors.ERROR_INSUFFICIENT_BALANCE);

          if (voted[msg.sender][hash] == VoteType.YEA) {
              proposal.yeas -= votingPower;
          }

          if (voted[msg.sender][hash] == VoteType.NAY) {
              proposal.nays -= votingPower;
          }

          proposal.vote(support, votingPower);
          voted[msg.sender][hash] = support ? VoteType.YEA : VoteType.NAY;

          if (proposal.isQuorumReached()) {
              emit ProposalQuorumReached(proposal.hash, proposal.isSupported());
          }
      }
    ```

### Recommended Mitigation Steps
  - Short term: Remove the following code, since it doesn't make any sense.
    -  ```Solidity 
            if (voted[msg.sender][hash] == VoteType.YEA) {
                proposal.yeas -= votingPower;
            }

            if (voted[msg.sender][hash] == VoteType.NAY) {
                proposal.nays -= votingPower;
            }
        ```
  - Long term: N/A.

## **[[H-01] createProposal() could be front-run by an attacker with dummy proposals leading to DoS](H-01)**
### ***Description:***
  -  In `VotingDAOV2.sol` there is a function `createProposal(uint256)` in order to propose some changes. This function invokes an internal `_createProposal(uint256, Payment memory)` to execute some logic along with creating proposal. Imagine that XDAO wants to commit new changes by creating new proposal, but an attacker is spamming the queue of those proposals(e.g by creating dummy proposals). So, the queue can have at most about 3 active proposals, therefore it's easy to proceed described above. 
  
### Proof of Concept
  - XDAO is creating a proposal to commit new changes. An attacker front-run that transaction with 3 dummy `createProposal()` transaction and leads the XDAO's transaction to be failed, because it's not possible to have 3+ active proposals at the same time.

  - **All occurances:**

    - contracts/VotingDAOV2.sol 
  
    ```Solidity
      function createProposal(uint256 hash) external {
          _createProposal(hash, ProposalLibrary.emptyPayment());
      }

      function _createProposal(uint256 hash, Payment memory payment) internal {
          require(!proposalExisted[hash], Errors.ERROR_COLLISION);
          proposalExisted[hash] = true;

          require(votingToken.balanceOf(msg.sender) > 0, Errors.ERROR_INSUFFICIENT_BALANCE);

          if (payment.amount > 0) {
              require(payment.destination != address(0), Errors.ERROR_ZERO_ADDRESS);
              if (!payment.sendETH) {
                  require(payment.token != address(0), Errors.ERROR_ZERO_ADDRESS);
              }
          }

          super.enqueue(
              Proposal({
                  hash: hash,
                  createdTimestamp: block.timestamp.toUint32(),
                  createdBlockNumber: block.number.toUint32(),
                  payment: payment,
                  updatedBlockNumber: 0,
                  yeas: 0,
                  nays: 0,
                  isExecuted: false,
                  vetoed: false
              })
          );

          emit ProposalCreated(hash);
      }

    ```

### Recommended Mitigation Steps
  - Short term: design a mechanism that allows for new proposals to be in pending state. 
  - Long term: N/A 


## **[[M-01] VetoNFT token might be minted to non-ERC721 compatible receivers](M-01)**
  
### ***Description:***
  - `VetoNFT.mint()` allows for those who has a **VETO_MINT** role to mint Veto tokens by specifing the address of a certain receiver. However, when minting NFT, the function does not check whether a receiving contract implements `onERC721Received()` or not. Therefore, the vetoNFT could be not transferrable in some cases.
  
### Proof of Concept
  - Alice has a **VETO_MINT** role. Now she wants to mint some VoteNFTs to Bob so that Bob can controll over all upcoming proposals. But Bob is pretty secure guy, he doesn't use his EOA in order to interact with DAO, instead he created a smart-contract for himself to execute certain functions. Well in this case, Bob might have some problems with receiving those Veto tokens and Alice will think that Bob successfully gets Veto tokens. 

  - **All occurances:**

    - contracts/VetoNFT.sol:
  
    ```Solidity
        function mint(address user) role(VETO_MINT) external returns (uint256) {
          _tokenIds.increment();

          uint256 newItemId = _tokenIds.current();
          _mint(user, newItemId);

          return newItemId;
        }
    ```

### Recommended Mitigation Steps
  - Short term: use `_safeMint()` instead of `_mint()`
  - Long term: token receiving contracts should provide a mechanism for interacting with XDAO's proposal vetoing process.

  - Also `_safeMint()` uses slightly more gas compared to `_mint()` and could be vulnerable to re-entrancy attacks, since the receiver can put some logic in `onERC721Received()` function. Since, only one who has a **VETO_MINT** role can mint tokens, the re-entrancy vector attacks are not possible. Be aware of all trade-offs.

## **[[M-02] Unchecked return value for ERC20 `.transfer()`](M-02)**

### ***Description:***
  - Some ERC20 compatible tokens like **USDT** return bool instead of reverting in case of the failure. Thus, it's always important to check for failures in order to consider such edge cases.
  
### Proof of Concept
  - XDAO wanted to reward zkBob for disclosing critical bug related to re-entrancy attack in proposal execution process. So, zkBob sends his address to receive some tokens from XDAO. XDAO created a proposal and executed it successfully, but zkBob didn't receive that bug bounty. It turns out, that XDAO used USDT to send the reward, therefore they were unaware, since in `_withdrawToken()` there is no check for return value.

  - **All occurances:**

    - contracts/VotingDAOV2.sol 
  
    ```Solidity
      function _withdrawToken(
          IERC20 token,
          address destination,
          uint256 amount
      ) internal {
          // mistype check
          require(address(destination) != address(0), Errors.ERROR_ZERO_ADDRESS);

          // if there are not enough funds on the token
          // safeTransfer is still may not revert
          require(token.balanceOf(address(this)) >= amount, Errors.ERROR_TREASURY_INSUFFICIENT_BALANCE);

          // send token
          token.transfer(destination, amount);
      }
    ```

### Recommended Mitigation Steps
  - Short term: it's possible to wrap `token.transfer()` into `require(token.transfer())`. 
  - Long term: N/A.

## **[[M-03] Unchecked return value for delegate call](M-03)**

### ***Description:***
  - Delegate calls like usual calls tend to return a bool and bytes after proceeding the call. However in proxy there is no checks for bool, thus it's impossible to know whether the .delegatecall() successed or not. 
  
### Proof of Concept
  - **All occurances:**

    - contracts/VotingDAOV2.sol 
  
    ```Solidity
      function __fallback() external virtual {
          msg.sender.delegatecall("TODO: implement an emergency method to temporarily pause the contract in the future...");
      }
    ```

### Recommended Mitigation Steps
  - Short term: simply check the bool value after the call. 
  - Long term: N/A.

## **[[L-01] Redundant `.isAccepted()` in `execute()`](L-01)**

### ***Description:***
  - In the line above it was previously checked. Since, though double check doesn't make any sense. 
    - ```Solidity
        function execute(uint256 hash) external {
            (bool found, uint8 index) = find(hash);
            require(found, Errors.ERROR_NOT_FOUND);

            Proposal storage proposal = proposals[index];
            require(!proposal.vetoed, Errors.ERROR_VETOED);
            require(!proposal.isExpired(), Errors.ERROR_EXPIRED);
            require(!proposal.isExecuted, Errors.ERROR_ALREADY_EXECUTED);
            require(proposal.isQuorumReached(), Errors.ERROR_QUORUM_IS_NOT_REACHED);
            require(proposal.isAccepted(), Errors.ERROR_CANNOT_EXECUTE_REJECTED_PROPOSAL);

            if (proposal.payment.amount > 0 && proposal.isAccepted()) {
                _withdraw(proposal.payment);
            }

            proposal.isExecuted = true;

            emit ProposalExecuted(hash);
        }
      ```

## **[[G-01] Cache state variables, `MLOAD` << `SLOAD`](G-01)**
### ***Description:***

- `MLOAD`/`MSTORE` costs only 3 units of gas, `SLOAD`(warm access) is about 100 units. Therefore, cache, when it's possible.

### ***All occurances:***

- Contracts:
  
    ```Solidity
      file: contracts/VotingDAOV2.sol 
      ...............................

        // Lines: [104-115]
          // Comment: It's possible to cache proposal for state reading.

            function veto(uint256 hash) external {
              // ...
                require(!proposal.vetoed, Errors.ERROR_VETOED);
                require(!proposal.isExpired(), Errors.ERROR_EXPIRED);
                require(!proposal.isExecuted, Errors.ERROR_ALREADY_EXECUTED);
              // .. 
            }

        // Lines: [124-150]
          // Comment: It's possible to cache proposal for state reading.

            function vote(uint256 hash, bool support) external {
              // ...
                require(!proposal.vetoed, Errors.ERROR_VETOED);
                require(!proposal.isExpired(), Errors.ERROR_EXPIRED);
                require(!proposal.isExecuted, Errors.ERROR_ALREADY_EXECUTED);
              // ...
                if (proposal.isQuorumReached()) {
                  emit ProposalQuorumReached(proposal.hash, proposal.isSupported());
                }
            }

        // Lines: [159-177]
          // Comment: It's possible to cache proposal for state reading.

            function execute(uint256 hash) external {
              // ...
                require(!proposal.vetoed, Errors.ERROR_VETOED);
                require(!proposal.isExpired(), Errors.ERROR_EXPIRED);
                require(!proposal.isExecuted, Errors.ERROR_ALREADY_EXECUTED);
                require(proposal.isQuorumReached(), Errors.ERROR_QUORUM_IS_NOT_REACHED);
                require(proposal.isAccepted(), Errors.ERROR_CANNOT_EXECUTE_REJECTED_PROPOSAL);

                if (proposal.payment.amount > 0 && proposal.isAccepted()) {
                    _withdraw(proposal.payment);
                }
              // ...
            }

        // Lines: [225-231]
          // Comment: It's possible to receive args directly from a calldata 

          function _withdraw(Payment storage payment) internal {
              if (payment.sendETH) {
                  _withdrawETH(payment.destination, payment.amount);
              } else {
                  _withdrawToken(IERC20(payment.token), payment.destination, payment.amount);
              }
          }

      file: contracts/Proposal.sol 
      ...............................
      
        // Lines: [38-56]

          function isExpired(Proposal storage proposal) internal view returns (bool) {
              return proposal.createdTimestamp + PROPOSAL_TTL < block.timestamp;
          }

          function isQuorumReached(Proposal storage proposal) internal view returns (bool) {
              return isAccepted(proposal) || isRejected(proposal);
          }

          function isRejected(Proposal storage proposal) internal view returns (bool) {
              return proposal.nays >= Constants.MIN_QUORUM;
          }

          function isAccepted(Proposal storage proposal) internal view returns (bool) {
              return proposal.yeas >= Constants.MIN_QUORUM;
          }

          function isActive(Proposal storage proposal) internal view returns (bool) {
              return proposal.createdBlockNumber > 0 && !isRejected(proposal) && !proposal.isExecuted;
          }

        ```

## **[[G-02] Consider `a = a + b` instead of `a += b`](G-02)**
### ***Description:***

- It has an impact on the deployment cost and the cost for distinct transaction as well.


### ***All occurances:***

- Contracts:
  
    ```Solidity
      file: contracts/Proposal.sol 
      ...............................
      
        // Lines: [63-67]
          if (support) {
              proposal.yeas += votingPower;
          } else {
              proposal.nays += votingPower;
          }

## **[[G-03] Consider marking functions with VETO_MINT role as payable](G-03)**
### ***Description:***

- This one is a bit questionable, but you can try that out. So, the compiler adds some extra conditions in case of non-payable, but we know that `onlyOwner` modifier will be reverted, if the user invoke following methods.

### ***All occurances:***

- Contracts:
  
    ```Solidity
      file: contracts/VetoNFT.sol 
      ...............................
        // Lines: [17-25]
          function mint(address user) role(VETO_MINT) external returns (uint256) {
              _tokenIds.increment();

              uint256 newItemId = _tokenIds.current();
              _mint(user, newItemId);

              return newItemId;
          }

## **[[G-04] Use `private/internal` for `constants/immutable/state` variables instead of `public`](G-04)**
### ***Description:***

- Optimization comes from not creating a getter function for each `public` instance. Try to define them as private/internal if it's possible in specific case.
  
### ***All occurances:***

- Contracts:
  
    ```Solidity
      file: contracts/Proposal.sol 
      ...............................
      
        // Lines: [32-32]
            uint32 constant PROPOSAL_TTL = 3 days;

      file: contracts/ProposalQueue.sol 
      .................................
        // Lines: [14-14]
            uint8 public constant MAX_ACTIVE_PROPOSALS = 10;


      file: contracts/VotingDAOV2.sol 
      .................................
        // Lines: [32-33]
            IMiniMeToken public votingToken;
            VetoNFT public vetoes;

        // Lines: [35-36]
          mapping(address => mapping(uint256 => VoteType)) voted;
          mapping(uint256 => bool) proposalExisted;


## **[[G-05] Use `calldataload` instead of `MLOAD`/`MSTORE`](G-05)**
### ***Description:***

- After Berlin hard fork, to load a non-zero byte from calldata dropped from 64 units of gas to 16 units, therefore if you do not modify args, use a calldata instead of memory. Here you need to explicitly specify `calldataload`, or replace `memory` with `calldata`. If the args are pretty huge, allocating args in memory will cost a lot. 
  
### ***All occurances:***

- Contracts:
  
    ```Solidity
      file: contracts/VotingDAOV2.sol 
      ...............................
      
        // Lines: [195-223]
          // Comment: Payment could be load from calldata directly, since it's only utilized for reading operations.  
           
            function _createProposal(uint256 hash, Payment memory payment) internal {}

        // Lines: [225-231]
          // Comment: Payment could be load from calldata directly, since it's only utilized for reading operations.  
            
            function _withdraw(Payment storage payment) internal {}

      file: contracts/ProposalQueue.sol 
      ...............................
        // Lines: [50-55]
          // Comment: Proposal could be load from calldata directly, since it's only utilized for reading operations.  

            function enqueue(Proposal memory proposal) internal {
              (bool found, uint8 index) = findInactive();
              require(found, Errors.ERROR_QUEUE_IS_FULL);
              
              proposals[index] = proposal;
          }

## Kudos for the quality of the code! It was pretty nice to go through
