## Bug Description

- In vault.sol there is a function, where `payable(receiver).transfer(amount)` is invoked. However `.transfer()` `.send()` calls are limited in terms of the gas usage. The limit is 0x08FC units. 

- So, what does it mean for us? It means that if the `receiver` is not an EOA(Externally owned account) this `.trasfer()` will end up with failure, because the gas provided is not enough to enter `fallback()` function in `msg.sender's` contract. Therefore, the user is not able to somehow withdraw his funds. 

  -  ```Solidity
      function fundTransfer(address receiver, uint256 amount) public {
            
            require(msg.sender == strategist, "Not strategist");
            require(receiver != address(0), "Please provide valid address");

            payable(receiver).transfer(amount);
        }
- 
## Impact
  - Smart-contract freezes the user's funds. 

## Risk Breakdown
Difficulty to Exploit: Easy
Weakness:
CVSS2 Score:
- I appreciate this section, but everything was described above. 

## Recommendation
- Personal recommendation is to use `.call()` or the library `Address.sol` from OZ to send funds, which uses the `.call()` underhood. 
  
## References

  - Check out [this, where new EIP was proposed in order to increase the gas limit for .transfer() .send()](https://github.com/ethereum/solidity/issues/4630#event-1764469844). 
  - And [this to read more about it](https://ethereum.stackexchange.com/questions/28759/transfer-to-contract-fails)
