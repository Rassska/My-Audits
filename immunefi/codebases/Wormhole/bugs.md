## Bug Description

- In Governance.sol contract there is a function `submitTransferFees(bytes memory _vm) public`, where `recipient.transfer(transfer.amount)` is invoked in order to transfer fees to the certain recipient. However `.transfer()` `.send()` calls are limited in terms of the gas usage. The limit is `0x08FC` units of gas. 

- It means that, if the `recipient` is not an EOA(Externally owned account), `.trasfer()` might end up with failure in some cases, because the gas provided is not enough to go through the `fallback()` function in `recipient's` contract. 

  -  ```Solidity
        function submitTransferFees(bytes memory _vm) public {
            Structs.VM memory vm = parseVM(_vm);

            // Verify the VAA is valid before processing it
            (bool isValid, string memory reason) = verifyGovernanceVM(vm);
            require(isValid, reason);

            // Obtains the transfer from the VAA payload
            GovernanceStructs.TransferFees memory transfer = parseTransferFees(vm.payload);

            // Verify the VAA is for this module
            require(transfer.module == module, "invalid Module");

            // Verify the VAA is for this chain
            require(transfer.chain == chainId() || transfer.chain == 0, "invalid Chain");

            // Record the governance action as consumed to prevent reentry
            setGovernanceActionConsumed(vm.hash);

            // Obtains the recipient address to be paid transfer fees
            address payable recipient = payable(address(uint160(uint256(transfer.recipient))));

            // Transfers transfer fees to the recipient
            recipient.transfer(transfer.amount);
        }
      
- However, if the recipient pretend to be only EOA, that would begreat to add some corresponded checks, like: 

  - ```Solidity
      function isContract(address addr) returns (bool) {
        uint size;
        assembly { size := extcodesize(addr) }
        return size > 0;
      }

## Clarification
  - I think, it's pretty useful to clarify that: If we have an ideal case, where the account has smth like that:
  ```
  event Received(address, uint);

      receive() external payable {
          emit Received(msg.sender, msg.value);
      }
  ```
  - There are no any issues, everything works fine.

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

Thanks!


## Impact
  - Smart-contract doesn't deliver a promised value. 

## Recommendation
- Personal recommendation is to use `.call()` or the library `Address.sol` from OZ to send funds, which uses the `.call()` underhood. And do not forget about handling the return value!

## Real mitigated examples

- Check out 2PI team having the same issue, they decided to use Address.sol from OZ in order to replace .transfer() with .call(): https://github.com/2pinetwork/contracts/blob/master/contracts/Archimedes.sol#L265 

## References

  - Check out [this, where new EIP was proposed in order to increase the gas limit for .transfer() .send()](https://github.com/ethereum/solidity/issues/4630#event-1764469844). 
  - And [this to read more about it](https://ethereum.stackexchange.com/questions/28759/transfer-to-contract-fails)
