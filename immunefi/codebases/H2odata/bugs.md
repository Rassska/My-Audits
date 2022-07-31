## Bug Description

- In CoinJoin.sol there is a function, where `account.transfer(wad)` is invoked. However `.transfer()` `.send()` calls are limited in terms of the gas usage. The limit is ~2300 units. 

- So, what does it mean for us? It means that if the `account` is not an EOA(Externally owned account) this `.trasfer()` will end up with failure, because the gas provided is not enough to enter `fallback()` function in `account` contract. Therefore, exting ETH from the system is not possible.

    ```Solidity
        function exit(address payable account, uint256 wad) external {
            require(int256(wad) >= 0, "ETHJoin/overflow");
            safeEngine.modifyCollateralBalance(collateralType, msg.sender, -int256(wad));
            emit Exit(msg.sender, account, wad);
            account.transfer(wad);
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