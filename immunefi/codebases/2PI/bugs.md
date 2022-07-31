## Bug Description

- In Archimedes.sol there is a function, where `payable(msg.sender).transfer(__wantBalance)` is invoked. However `.transfer()` `.send()` calls are limited in terms of gas usage. The limit is ~2300 units. 

- So, what does it mean for us? It means that if the `msg.sender` is not an EOA(Externally owned account) this `.trasfer()` will end up with failure, because the gas provided is not enough to enter `fallback()` function in `msg.sender` contract. Therefore, `msg.sender` is not able to withdraw his funds.

    ```Solidity
       // Withdraw want token from Archimedes.
            function withdraw(uint _pid, uint _shares) public nonReentrant {
                require(_shares > 0, "0 shares");
                require(_userShares(_pid) >= _shares, "withdraw: not sufficient founds");

                updatePool(_pid);

                // Pay rewards
                _calcPendingAndPayRewards(_pid, msg.sender);

                PoolInfo storage pool = poolInfo[_pid];

                uint _before = _wantBalance(pool.want);
                // this should burn shares and control the amount
                uint withdrawn = _controller(_pid).withdraw(msg.sender, _shares);
                require(withdrawn > 0, "No funds withdrawn");

                uint __wantBalance = _wantBalance(pool.want) - _before;

                // In case we have WNative we unwrap to Native
                if (address(pool.want) == address(WNative)) {
                    // Unwrap WNative => Native
                    WNative.withdraw(__wantBalance);

                    payable(msg.sender).transfer(__wantBalance);
                } else {
                    pool.want.safeTransfer(address(msg.sender), __wantBalance);
                }

                // This is to "save" like the new amount of shares was paid
                _updateUserPaidRewards(_pid, msg.sender);

                emit Withdraw(_pid, msg.sender, _shares);
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