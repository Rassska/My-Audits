## Bug Description

- In XBERouter there is a function `addLiquidity(uint256, uint256)`, where `payable(msg.sender).transfer(half - liquidityWETH)` is invoked. However `.transfer()` `.send()` calls are limited in terms of the gas usage. The limit is 0x08FC units. 

- So, what does it mean for us? It means that if the `msg.sender` is not an EOA(Externally owned account) this `.trasfer()` will end up with failure, because the gas provided is not enough to enter `fallback()` function in `msg.sender's` contract. Therefore, the user is not able add liquidity. 

  -  ```Solidity
            function addLiquidity(uint256 _deadline, uint256 _tokenOutMin)
                public
                payable
                ensure(_deadline)
            {
                require(msg.value > 0, "ZERO_ETH");
                uint256 half = msg.value / 2;
                require(_getAmountXBE(half) >= _tokenOutMin, "PRICE_CHANGED");
                uint256 XBEfromSwap = _swapETHforXBE(half);
                (
                    uint256 liquidityXBE,
                    uint256 liquidityWETH,
                    uint256 liquidityTokens
                ) = _addLiquidity(XBEfromSwap, half);
                if (XBEfromSwap - liquidityXBE > 0)
                    XBE.transfer(msg.sender, XBEfromSwap - liquidityXBE);
                if (half - liquidityWETH > 0)
                    payable(msg.sender).transfer(half - liquidityWETH);
                vault.depositFor(liquidityTokens, msg.sender);
            }

   - Decompiled version to analyze, what is underneath: 
     - ```
        function func_009B(var arg0, var arg1) {
            var var0 = arg0;
        
            if (var0 < block.timestamp) {
                var temp19 = memory[0x40:0x60];
                memory[temp19:temp19 + 0x20] = 0x461bcd << 0xe5;
                memory[temp19 + 0x04:temp19 + 0x04 + 0x20] = 0x20;
                memory[temp19 + 0x24:temp19 + 0x24 + 0x20] = 0x07;
                memory[temp19 + 0x44:temp19 + 0x44 + 0x20] = 0x11561412549151 << 0xca;
                var temp20 = memory[0x40:0x60];
                revert(memory[temp20:temp20 + (temp19 + 0x64) - temp20]);
            } else if (msg.value > 0x00) {
                var var1 = 0x00;
                var var2 = 0x01d3;
                var var3 = 0x02;
                var var4 = msg.value;
                var2 = func_09D7(var3, var4);
                var temp0 = var2;
                var1 = temp0;
                var2 = arg1;
                var3 = 0x01df;
                var4 = var1;
                var3 = func_0397(var4);
            
                if (var3 >= var2) {
                    var2 = 0x00;
                    var3 = 0x0228;
                    var4 = var1;
                    var3 = func_04D2(var4);
                    var temp1 = var3;
                    var2 = temp1;
                    var3 = 0x00;
                    var4 = var3;
                    var var5 = 0x00;
                    var var6 = 0x0239;
                    var var7 = var2;
                    var var8 = var1;
                    var6, var7, var8 = func_066A(var7, var8);
                    var temp2 = var6;
                    var3 = temp2;
                    var4 = var7;
                    var5 = var8;
                    var6 = 0x00;
                    var7 = 0x024c;
                    var8 = var3;
                    var var9 = var2;
                    var7 = func_09F9(var8, var9);
                
                    if (var7 <= var6) {
                    label_02DF:
                        var6 = 0x00;
                        var7 = 0x02eb;
                        var8 = var4;
                        var9 = var1;
                        var7 = func_09F9(var8, var9);
                    
                        if (var7 <= var6) {
                        label_0329:
                            var temp3 = memory[0x40:0x60];
                            memory[temp3:temp3 + 0x20] = 0x36efd16f << 0xe0;
                            memory[temp3 + 0x04:temp3 + 0x04 + 0x20] = var5;
                            memory[temp3 + 0x24:temp3 + 0x24 + 0x20] = msg.sender;
                            var6 = storage[0x03] & (0x01 << 0xa0) - 0x01;
                            var7 = 0x36efd16f;
                            var8 = temp3 + 0x44;
                            var9 = 0x00;
                            var var10 = memory[0x40:0x60];
                            var var11 = var8 - var10;
                            var var12 = var10;
                            var var13 = 0x00;
                            var var14 = var6;
                            var var15 = !address(var14).code.length;
                        
                            if (var15) { revert(memory[0x00:0x00]); }
                        
                            var temp4;
                            temp4, memory[var10:var10 + var9] = address(var14).call.gas(msg.gas).value(var13)(memory[var12:var12 + var11]);
                            var9 = !temp4;
                        
                            if (!var9) { return; }
                        
                            var temp5 = returndata.length;
                            memory[0x00:0x00 + temp5] = returndata[0x00:0x00 + temp5];
                            revert(memory[0x00:0x00 + returndata.length]);
                        } else {
                            var6 = msg.sender;
                            var7 = 0x08fc;
                            var8 = 0x02ff;
                            var9 = var4;
                            var10 = var1;
                            var8 = func_09F9(var9, var10);
                            var temp6 = memory[0x40:0x60];
                            var temp7 = var8;
                            var temp8;
                            temp8, memory[temp6:temp6 + 0x00] = address(var6).call.gas(var7 * !temp7).value(temp7)(memory[temp6:temp6 + 0x00]);
                            var6 = !temp8;
                        
                            if (!var6) { goto label_0329; }
                        
                            var temp9 = returndata.length;
                            memory[0x00:0x00 + temp9] = returndata[0x00:0x00 + temp9];
                            revert(memory[0x00:0x00 + returndata.length]);
                        }
                    } else {
                        var6 = storage[0x01] & (0x01 << 0xa0) - 0x01;
                        var7 = 0xa9059cbb;
                        var8 = msg.sender;
                        var9 = 0x026e;
                        var10 = var3;
                        var11 = var2;
                        var9 = func_09F9(var10, var11);
                        var temp10 = memory[0x40:0x60];
                        memory[temp10:temp10 + 0x20] = (var7 << 0xe0) & ~((0x01 << 0xe0) - 0x01);
                        memory[temp10 + 0x04:temp10 + 0x04 + 0x20] = var8 & (0x01 << 0xa0) - 0x01;
                        memory[temp10 + 0x24:temp10 + 0x24 + 0x20] = var9;
                        var8 = temp10 + 0x44;
                        var temp11 = memory[0x40:0x60];
                        var temp12;
                        temp12, memory[temp11:temp11 + 0x20] = address(var6).call.gas(msg.gas)(memory[temp11:temp11 + var8 - temp11]);
                        var9 = !temp12;
                    
                        if (!var9) {
                            var temp13 = memory[0x40:0x60];
                            var temp14 = returndata.length;
                            memory[0x40:0x60] = temp13 + (temp14 + 0x1f & ~0x1f);
                            var6 = 0x02dd;
                            var8 = temp13;
                            var7 = var8 + temp14;
                            var6 = func_0A10(var7, var8);
                            goto label_02DF;
                        } else {
                            var temp15 = returndata.length;
                            memory[0x00:0x00 + temp15] = returndata[0x00:0x00 + temp15];
                            revert(memory[0x00:0x00 + returndata.length]);
                        }
                    }
                } else {
                    var temp16 = memory[0x40:0x60];
                    memory[temp16:temp16 + 0x20] = 0x461bcd << 0xe5;
                    memory[temp16 + 0x04:temp16 + 0x04 + 0x20] = 0x20;
                    memory[temp16 + 0x24:temp16 + 0x24 + 0x20] = 0x0d;
                    memory[temp16 + 0x44:temp16 + 0x44 + 0x20] = 0x14149250d157d0d2105391d151 << 0x9a;
                    var2 = temp16 + 0x64;
                
                label_0182:
                    var temp17 = memory[0x40:0x60];
                    revert(memory[temp17:temp17 + var2 - temp17]);
                }
            } else {
                var temp18 = memory[0x40:0x60];
                memory[temp18:temp18 + 0x20] = 0x461bcd << 0xe5;
                memory[temp18 + 0x04:temp18 + 0x04 + 0x20] = 0x20;
                memory[temp18 + 0x24:temp18 + 0x24 + 0x20] = 0x08;
                memory[temp18 + 0x44:temp18 + 0x44 + 0x20] = 0x0b48aa49ebe8aa89 << 0xc3;
                var1 = temp18 + 0x64;
                goto label_0182;
            }
        }
        ```

      - As we can see, in the line: `temp8, memory[temp6:temp6 + 0x00] = address(var6).call.gas(var7 * !temp7).value(temp7)(memory[temp6:temp6 + 0x00]);` we're manually specifying the limit by the var7, which is exactly equals to `var7 = 0x08fc`.
  

## Impact
  - Smart-contract is unable to add liquidity, if the `msg.sender` is not an EOA. 

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
