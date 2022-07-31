## Bug Description

- In EstateSale.sol there is a function, where `msg.sender.transfer()` is invoked. However `.transfer()` `.send()` calls are limited in terms of gas usage. The limit is ~2300 units. 

- So, what does it mean for us? It means that if the `msg.sender` is not an EOA(Externally owned account) this `.trasfer()` will end up with failure, because the gas provided is not enough to enter `fallback()` function in `msg.sender` contract. Therefore, if `msg.sender` provide a value which is more then `ETHRequired`, he will likely loose `msg.value - ETHRequired` and will not get back remaining value.

    ```Solidity
        function buyLandWithETH(
            address buyer,
            address to,
            address reserved,
            uint256 x,
            uint256 y,
            uint256 size,
            uint256 priceInSand,
            bytes32 salt,
            uint256[] calldata assetIds,
            bytes32[] calldata proof,
            bytes calldata referral
        ) external payable {
            require(_etherEnabled, "ether payments not enabled");
            _checkValidity(buyer, reserved, x, y, size, priceInSand, salt, assetIds, proof);

            uint256 ETHRequired = getEtherAmountWithSAND(priceInSand);
            require(msg.value >= ETHRequired, "not enough ether sent");

            if (msg.value - ETHRequired > 0) {
                msg.sender.transfer(msg.value - ETHRequired); // refund extra
            }

            handleReferralWithETH(ETHRequired, referral, _wallet);

            _mint(buyer, to, x, y, size, priceInSand, address(0), ETHRequired);
            _sendAssets(to, assetIds);
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
- Personal recommendation is to use `.call()`
  
## References

  - Check out [this, where new EIP was proposed in order to increase the gas limit for .transfer() .send()](https://github.com/ethereum/solidity/issues/4630#event-1764469844). 
  - And [this to read more about it](https://ethereum.stackexchange.com/questions/28759/transfer-to-contract-fails)