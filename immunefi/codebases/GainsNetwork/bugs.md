## Bug Description

- In GFarm contract there is a function `POOL2_unstake()`, where `msg.sender.transfer(amount)` is invoked. However `.transfer()` `.send()` calls are limited in terms of the gas usage. The limit is ~2300 units. 

- So, what does it mean for us? It means that if the `msg.sender` is not an EOA(Externally owned account) this `.trasfer()` will end up with failure, because the gas provided is not enough to go through `fallback()` function in `msg.sender's` contract. Therefore, the user is not able to somehow unstake his funds. 

  -  ```Solidity
      `function POOL2_unstake(uint amount) external{
            User storage u = users[msg.sender];
            require(amount > 0, "Unstaking 0 ETH.");
            require(u.POOL2_provided >= amount, "Unstaking more than currently staked.");

            _POOL2_harvest(msg.sender, 0);
            msg.sender.transfer(amount);

            u.POOL2_provided = u.POOL2_provided.sub(amount);
            u.POOL2_rewardDebt = u.POOL2_provided.mul(POOL2_accTokensPerETH).div(1e18);
        }

## Another assets with the same issue:

- In GNSTradingVaultV5 there is a function `harvest()`, where `payable(msg.sender).transfer(pendingMatic)` is invoked. Due to the same issue this `.transfer()` will also unfortunately end up with a failure. 

      ```Solidity
      function harvest() public{
        User storage u = users[msg.sender];

        require(storageT.dai().transfer(msg.sender, pendingRewardDai()));
        u.debtDai = u.daiDeposited * accDaiPerDai / 1e18;

        uint pendingMatic = pendingRewardMatic();
        accMaticPerDai = pendingAccMaticPerDai();
        maticLastRewardBlock = block.number;
        u.debtMatic = u.daiDeposited * accMaticPerDai / 1e18;
        payable(msg.sender).transfer(pendingMatic);
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

Thanks!


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
