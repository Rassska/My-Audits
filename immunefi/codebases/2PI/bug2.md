## Bug Description

- In Archimedes.sol there is a function `withdraw()`, where `_calcPendingAndPayRewards(_pid, msg.sender)` is invoked. The following function triggers `_safePiTokenTransfer(_user, pending)` which uses ERC20 `.transfer()` in order to transfer piTokens. However ERC20's `transfer()` doesn't revert in case of failure, instead - return `bool`, but here there is no checks for that returned bool. 

    - ```Solidity
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


    -  ```Solidity
            function _calcPendingAndPayRewards(uint _pid, address _user) internal returns (uint pending) {
                uint _shares = _userShares(_pid, _user);

                if (_shares > 0) {
                    pending = ((_shares * poolInfo[_pid].accPiTokenPerShare) / SHARE_PRECISION) - paidRewards(_pid, _user);

                    if (pending > 0) {
                        _safePiTokenTransfer(_user, pending);
                        _payReferralCommission(_user, pending);

                        emit Harvested(_pid, _user, pending);
                    }
            }

    - ```Solidity
            // Safe piToken transfer function, just in case if rounding error causes pool to not have enough PI.

            function _safePiTokenTransfer(address _to, uint _amount) internal {
                uint piTokenBal = piToken.balanceOf(address(this));

                if (_amount > piTokenBal) {
                    _amount = piTokenBal;
                }

                // piToken.transfer is safe
                piToken.transfer(_to, _amount);
            }
    

## Impact
  - I appreciate this section, but everything was described above. 

## Risk Breakdown
Difficulty to Exploit: Easy
Weakness:
CVSS2 Score:
- I appreciate this section, but everything was described above. 

## Recommendation
- Try to handle bool value properly. 
  
## References
- Check out this: https://ethereum.stackexchange.com/questions/57723/why-use-returnsbool-in-a-transfer-function