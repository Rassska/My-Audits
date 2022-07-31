## Bug Description

- In RcaTreasury.sol, there is a function, where ` _to.transfer(_amount);` is invoked. Where the Governance has an ability to withdraw all funds to the specified address. It might ends up with a failure in case, when _to is not an EOA, because the gas provided(which is ~2300 units) is not enough to enter `fallback()` function in `_to's` contract. 

  - ```Solidity
      function withdraw(address payable _to, uint256 _amount) external onlyGov {
          _to.transfer(_amount);
      }

## Impact
  - Unexpected behavior. 

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
