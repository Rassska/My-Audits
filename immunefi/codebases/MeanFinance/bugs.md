## Bug Description

- In DCAHubCompanionWTokenPositionHandler.sol there is a function, where `_recipient.transfer(_amount)` is invoked. However `.transfer()` `.send()` calls are limited in terms of the gas usage. The limit is ~2300 units. 

- So, what does it mean for us? It means that if the `_recipient` is not an EOA(Externally owned account) this `.trasfer()` will end up with failure, because the gas provided is not enough to enter `fallback()` function in `_recipient's` contract. Therefore, the user is not able to somehow withdraw his funds. 

  -  ```Solidity
      function _unwrapAndSend(uint256 _amount, address payable _recipient) internal {
        if (_amount > 0) {
          // Unwrap wToken
          wToken.withdraw(_amount);

          // Send protocol token to recipient
          _recipient.transfer(_amount);
        }
      }

  - This internal function could be invoked from: 
    - ```Solidity
      function withdrawSwappedManyUsingProtocolToken(uint256[] calldata _positionIds, address payable _recipient)
        external
        payable
        returns (uint256 _swapped)
      {
        for (uint256 i; i < _positionIds.length; i++) {
          _checkPermissionOrFail(_positionIds[i], IDCAPermissionManager.Permission.WITHDRAW);
        }
        IDCAHub.PositionSet[] memory _positionSets = new IDCAHub.PositionSet[](1);
        _positionSets[0].token = address(wToken);
        _positionSets[0].positionIds = _positionIds;
        uint256[] memory _withdrawn = hub.withdrawSwappedMany(_positionSets, address(this));
        _swapped = _withdrawn[0];
        _unwrapAndSend(_swapped, _recipient);
      }
    
    - ```Solidity
      function increasePositionUsingProtocolToken(
        uint256 _positionId,
        uint256 _amount,
        uint32 _newSwaps
      ) external payable checkPermission(_positionId, IDCAPermissionManager.Permission.INCREASE) {
        _wrap(_amount);
        hub.increasePosition(_positionId, _amount, _newSwaps);
      }

      /// @inheritdoc IDCAHubCompanionWTokenPositionHandler
      function reducePositionUsingProtocolToken(
        uint256 _positionId,
        uint256 _amount,
        uint32 _newSwaps,
        address payable _recipient
      ) external payable checkPermission(_positionId, IDCAPermissionManager.Permission.REDUCE) {
        hub.reducePosition(_positionId, _amount, _newSwaps, address(this));
        _unwrapAndSend(_amount, _recipient);
      }

    - ```Solidity
      function reducePositionUsingProtocolToken(
        uint256 _positionId,
        uint256 _amount,
        uint32 _newSwaps,
        address payable _recipient
      ) external payable checkPermission(_positionId, IDCAPermissionManager.Permission.REDUCE) {
        hub.reducePosition(_positionId, _amount, _newSwaps, address(this));
        _unwrapAndSend(_amount, _recipient);
      }

    - ```Solidity
      function terminateUsingProtocolTokenAsFrom(
        uint256 _positionId,
        address payable _recipientUnswapped,
        address _recipientSwapped
      ) external payable checkPermission(_positionId, IDCAPermissionManager.Permission.TERMINATE) returns (uint256 _unswapped, uint256 _swapped) {
        (_unswapped, _swapped) = hub.terminate(_positionId, address(this), _recipientSwapped);
        _unwrapAndSend(_unswapped, _recipientUnswapped);
      }

    - ```Solidity
      function terminateUsingProtocolTokenAsTo(
        uint256 _positionId,
        address _recipientUnswapped,
        address payable _recipientSwapped
      ) external payable checkPermission(_positionId, IDCAPermissionManager.Permission.TERMINATE) returns (uint256 _unswapped, uint256 _swapped) {
        (_unswapped, _swapped) = hub.terminate(_positionId, _recipientUnswapped, address(this));
        _unwrapAndSend(_swapped, _recipientSwapped);
      }

  - All of the following functions unfortunately will end up with a described failure. 

## Another assets with the same issue:

  - DCAHubSwapper(0xBF5C27CC7c1c91E924fA7B4df7928371e8f713a6)
    - DCAHubSwapperSwapHandler.sol
      ```Solidity
      function _handleSwapForCallerCallback(IDCAHub.TokenInSwap[] calldata _tokens, bytes memory _data) internal {
        CallbackDataCaller memory _callbackData = abi.decode(_data, (CallbackDataCaller));
        for (uint256 i; i < _tokens.length; i++) {
          IDCAHub.TokenInSwap memory _token = _tokens[i];
          if (_token.toProvide > 0) {
            if (_token.token == address(wToken) && _callbackData.msgValue != 0) {
              // Wrap necessary
              wToken.deposit{value: _token.toProvide}();
              // Return any extra tokens to the original caller
              if (_callbackData.msgValue > _token.toProvide) {
                payable(_callbackData.caller).transfer(_callbackData.msgValue - _token.toProvide);
              }
            }
            IERC20(_token.token).safeTransferFrom(_callbackData.caller, address(hub), _token.toProvide);
          }
        }
      }

    - CollectableDust.sol
      ```Solidity
      function _sendDust(
        address _to,
        address _token,
        uint256 _amount
      ) internal {
        require(_to != address(0), 'CollectableDust: zero address');
        require(!_protocolTokens.contains(_token), 'CollectableDust: token is part of protocol');
        if (_token == PROTOCOL_TOKEN) {
          payable(_to).transfer(_amount);
        } else {
          IERC20(_token).safeTransfer(_to, _amount);
        }
        emit DustSent(_to, _token, _amount);
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
