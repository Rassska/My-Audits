## Bug Description

- In DEEPSPACE there is a function, where `payable(owner()).transfer(address(this).balance)` is invoked. However `.transfer()` `.send()` calls are limited in terms of the gas usage. The limit is ~2300 units. 

- So, what does it mean for us? It means that if the `owner()` is not an EOA(Externally owned account) this `.trasfer()` will end up with failure, because the gas provided is not enough to enter `fallback()` function in `owner's` contract. Therefore, it requires transferring ownership to the EOA, which takes some extra time for exacution through out the DAO.

    ```Solidity
        function withdrawalBNB() external onlyOwner() {
            payable(owner()).transfer(address(this).balance);
        }
    ```

## Impact
  - I appreciate this section, but everything was described above. 

## Risk Breakdown
Difficulty to Exploit: Easy
Weakness:
CVSS2 Score:
- I appreciate this section, but everything was described above. 

## Recommendation
- Personal recommendation is to use `.call()` or the library `Address.sol` from OZ to send funds. 
  
## References

  - Check out [this, where new EIP was proposed in order to increase the gas limit for .transfer() .send()](https://github.com/ethereum/solidity/issues/4630#event-1764469844). 
  - And [this to read more about it](https://ethereum.stackexchange.com/questions/28759/transfer-to-contract-fails)