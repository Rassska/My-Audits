## Bug Description

- In Maximillion.sol contract there is a function `repayBehalfExplicit(address borrower, MGlimmer mGlimmer_) public payable`, where `msg.sender.transfer(received - borrows)` is invoked in order to refund excesses. However `.transfer()` `.send()` calls are limited in terms of the gas usage. The limit is `0x08FC` units of gas. 

- It means that if the `msg.sender` is not an EOA(Externally owned account), `.trasfer()` might end up with failure in some cases, because the gas provided is not enough to go through `fallback()` function in `msg.sender's` contract. Therefore, the user who tries to repay the borrower's debt will not be able to make that happen due to the failure. 

  -  ```Solidity
      function repayBehalfExplicit(address borrower, MGlimmer mGlimmer_) public payable {
          uint received = msg.value;
          uint borrows = mGlimmer_.borrowBalanceCurrent(borrower);
          if (received > borrows) {
              mGlimmer_.repayBorrowBehalf.value(borrows)(borrower);
              msg.sender.transfer(received - borrows);
          } else {
              mGlimmer_.repayBorrowBehalf.value(received)(borrower);
          }
      }

## Clarification
  - I think, it's pretty useful to clarify that: If we have an ideal case, where the account has smth like that:
  ```
  event Received(address, uint);

      receive() external payable {
          emit Received(msg.sender, msg.value);
      }
  ```
  - There no any issues, everything works fine.

  - But what is the problem though? The problem is that we can't just simply wish something to happen in DeFi. Bob easily could have some logic like that:
```
  event Received(address indexed from, uint indexed amount, bytes providedData);

      fallback() external payable {
          emit Received(msg.sender, msg.value, msg.data);
      }
```

  - or smth else, like:
```
mapping(address => uint256) received;

    fallback() external payable {
        received[msg.sender] += msg.value
    }
```

  - Where the gas limit provided with a `.transfer()` call is not enough. And everything will be messed up :( And the issue is not relies on the Bob's side.

> My personal opinion on that is following:

  - If we expect that only EOA's will interact with the contract, using `.transfer()` is fine!
  - If smart-contracts also have an ability, it's better to think about such pitfalls and mitigate them, if there's such opportunity.
  - I don't think that using `.transfer()` in order to close any opportunities for re-entrancy is a good practice, since it causes some issues like that. Use simple mutex based re-entrancy guard instead.

Thanks!


## Impact
  - Smart-contract doesn't deliver a promised value. 

## Risk Breakdown
Difficulty to Exploit: Easy
Weakness:
CVSS2 Score:
- I appreciate this section, but everything was described above. 

## Recommendation
- Personal recommendation is to use `.call()` or the library `Address.sol` from OZ to send funds, which uses the `.call()` underhood. And do not forget about handling the return value!
  
## References

  - Check out [this, where new EIP was proposed in order to increase the gas limit for .transfer() .send()](https://github.com/ethereum/solidity/issues/4630#event-1764469844). 
  - And [this to read more about it](https://ethereum.stackexchange.com/questions/28759/transfer-to-contract-fails)
