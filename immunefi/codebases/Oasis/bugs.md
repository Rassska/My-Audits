## Bug Description

- In AutomationExecutor.sol there is a function, where `payable(msg.sender).transfer(etherUsed)` is invoked. However `.transfer()` `.send()` calls are limited in terms of the gas usage. The limit is 0x08FC units. 

- So, what does it mean for us? It means that if the authorized `msg.sender` is not an EOA(Externally owned account) this `.trasfer()` will end up with failure, because the gas provided is not enough to enter `fallback()` function in `msg.sender's` contract. Therefore, the user is not able to somehow receive his funds. 

  -  ```Solidity
      function execute(
            bytes calldata executionData,
            uint256 cdpId,
            bytes calldata triggerData,
            address commandAddress,
            uint256 triggerId,
            uint256 daiCoverage,
            uint256 minerBribe,
            int256 gasRefund
        ) external auth(msg.sender) {
            uint256 initialGasAvailable = gasleft();
            bot.execute(executionData, cdpId, triggerData, commandAddress, triggerId, daiCoverage);

            if (minerBribe > 0) {
                block.coinbase.transfer(minerBribe);
            }
            uint256 finalGasAvailable = gasleft();
            uint256 etherUsed = tx.gasprice *
                uint256(int256(initialGasAvailable - finalGasAvailable) - gasRefund);

            if (address(this).balance > etherUsed) {
                payable(msg.sender).transfer(etherUsed);
            } else {
                payable(msg.sender).transfer(address(this).balance);
            }
        }

- Also there is a function withdraw within the same contract, which unfortunately will end up with failure according to the described issue.
  - ```Solidity
        function withdraw(uint256 wad) public {
            require(balanceOf[msg.sender] >= wad);
            balanceOf[msg.sender] -= wad;
            payable(msg.sender).transfer(wad);
            emit Withdrawal(msg.sender, wad);
        }

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
