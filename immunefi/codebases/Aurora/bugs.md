## Bug Description

- In NearBridge there is a function, where `payable(msg.sender).transfer(amount)` is invoked. However `.transfer()` `.send()` calls are limited in terms of the gas usage. The limit is ~2300 units. 

- So, what does it mean for us? It means that if the `msg.sender` is not an EOA(Externally owned account) this `.trasfer()` will end up with failure, because the gas provided is not enough to enter `fallback()` function in `msg.sender's` contract. Therefore, the user is not able to somehow withdraw his funds. 
  - Also, the function adminSendEth will not resolve the problem, because `.transfer()` is used there as well.
  - The function `challenge()` will not also transfer the `lockEthAmount` according to the same problem.

  -  ```Solidity
      function deposit() public payable override pausable(PAUSED_DEPOSIT) {
          require(msg.value == lockEthAmount && balanceOf[msg.sender] == 0);
          balanceOf[msg.sender] = msg.value;
      }

      function withdraw() public override pausable(PAUSED_WITHDRAW) {
          require(msg.sender != lastSubmitter || block.timestamp >= lastValidAt);
          uint amount = balanceOf[msg.sender];
          require(amount != 0);
          balanceOf[msg.sender] = 0;
          payable(msg.sender).transfer(amount);
      }

  -
    ```Solidity
      function challenge(address payable receiver, uint signatureIndex) public override pausable(PAUSED_CHALLENGE) {
          require(block.timestamp < lastValidAt, "No block can be challenged at this time");
          require(!checkBlockProducerSignatureInHead(signatureIndex), "Can't challenge valid signature");

          balanceOf[lastSubmitter] = balanceOf[lastSubmitter] - lockEthAmount;
          receiver.transfer(lockEthAmount / 2);
          lastValidAt = 0;
      }
  -
    ```Solidity
      function adminSendEth(address payable destination, uint amount) public onlyAdmin {
            destination.transfer(amount);
      }
    ```
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

## All Occurances: 
  - NearBridge (0x3be7Df8dB39996a837041bb8Ee0dAdf60F767038): 
    - AdminControlled.sol 
      - Lines: [44-46]
    - NearBridge.sol
      - Lines: [83-89]
      - Lines: [91-98]
  - NearProvider (0x051AD3F020274910065Dcb421629cd2e6E5b46c4):
    - AdminControlled
      - Lines: [265-267] 
  - EthCustodian(0x6BFaD42cFC4EfC96f529D786D643Ff4A8B89FA52):
    - EthCustodian.sol
      - Lines: [107-126]
    - AdminControlled.sol 
      - Lines: [43-45]
  - ERC20Locker (0x23Ddd3e3692d1861Ed57EDE224608875809e127f):
    - AdminControlled.sol 
      - Lines: [602-604]
  - eNear (0x85F17Cf997934a597031b2E18a9aB6ebD4B9f6a4):
    - AdminControlled.sol 
      - Lines: [34-36]