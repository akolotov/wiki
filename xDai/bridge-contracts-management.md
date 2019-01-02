xDai bridge contracts management
=====

## Action flow

There are four groups of operations with the xDai birdge that can be performed by the bridge admin. All operations are performed by owners of the Multisignature Wallet which requires that several of accounts will need to confirm the operation transaction.

The addresses of the Multisignature Wallet:
  * the ETH Mainnet: `0xff1a8EDA5eAcdB6aAf729905492bdc6376DBe2dd`, [ABI](https://github.com/poanetwork/poa-chain-spec/blob/4fe29a700b1331c570e20c5424523e522f5ab4d7/abis/bridge/MultiSigWallet.json), [Etherscan](https://etherscan.io/address/0xff1a8EDA5eAcdB6aAf729905492bdc6376DBe2dd)
  * the xDai chain: `0x0d3726e5a9f37234d6b55216fc971d30f150a60f`, [ABI](https://github.com/poanetwork/poa-chain-spec/blob/4fe29a700b1331c570e20c5424523e522f5ab4d7/abis/bridge/MultiSigWallet.json), [Blockscout](https://blockscout.com/poa/dai/address/0x0d3726e5a9f37234d6b55216fc971d30f150a60f/transactions)

The common action flow could be described as:

1. One of the multisig wallet owner encodes the call of method with set of parameters (if any). For example this can be done with [the ABI Enconding Service](https://abi.hashex.org/).
2. The endcoded sequence of bytes is used to create a transaction for the multisig wallet contract. It is done with `submitTransaction` method of the multisig wallet contract. The method raises the event `Submission` containing the index of the registered transaction. The index is shared with other owners of the wallet.
3. The rest of owners confirm the transaction by invoking `confirmTransaction` from the multisig wallet contract. As soon as enough number of the confirmation received the method encoded in the step 1 will be invoked automatically. This is important to know in order to set adequate gas limit for that transaction with confirmation which sent by the wallet owner finalizing the operation.
4. If the method was not invoked due to exceeding of gas limit, the owners could execute the confirmed transaction manually by sending `executeTransaction`. 

The particular action flow will differ for every group of management operations, that's why they are considered in details below.  

### Upgrade 

There are two possible scenarios how the bridge or validators contracts can be upgraded:
  * a security fix when only the contract implementations is changed
  * an improvement when the contract implemmentation upgrade requires initialization of storage values

#### Security upgrade

1. Deploy a new implementation of the bridge or validators contract.
2. Depending on the contract and the chain use one of the link below to get the current `version` of the contract implementation:
  * The bridge contract_ on _the ETH Mainnet_: [Etherscan](https://etherscan.io/address/0x4aa42145aa6ebf72e164c9bbc74fbd3788045016#readContract), 
  * The validators contract_ on _the ETH Mainnet_: [Etherscan](https://etherscan.io/address/0xe1579dEbdD2DF16Ebdb9db8694391fa74EeA201E#readContract) 
  * The bridge contract_ on _the xDai chain_: [Blockscout](https://blockscout.com/poa/dai/address/0x7301cfa0e1756b71869e93d4e4dca5c7d0eb0aa6/read_contract)
  * The validators contract_ on _the xDai chain_: [Blockscout](https://blockscout.com/poa/dai/address/0xb289f0e6fbdff8eee340498a56e1787b303f1b6d/read_contract)
3. Use the `upgradeTo` method from `EternalStorageProxy` ABI, the address of the new implementation and the incremented version number to encode the data for the transaction. Example of the data: `3ad06d160000000000000000000000000000000000000000000000000000000000000004000000000000000000000000f097137c7ec5e582b5704065f72ac5903d0b526d`.
4. Use NiftyWallet (or another tool that could build transactions based on the contract source code or ABI) to invoke `submitTransaction` of the mutlisig wallet contract (`0xff1a8EDA5eAcdB6aAf729905492bdc6376DBe2dd` on the Mainnet ETH, `0x0d3726e5a9f37234d6b55216fc971d30f150a60f` on the xDai chain). The data field must be filled with the bytes received on the previous step. The destination depends on the contract:
  * `0x4aa42145Aa6Ebf72e164C9bBC74fbD3788045016` if the security upgrade is made for _the bridge contract_ on _the ETH Mainnet_. 
  * `0xe1579dEbdD2DF16Ebdb9db8694391fa74EeA201E` if the security upgrade is made for _the validators contract_ on _the ETH Mainnet_. 
  * `0x7301cfa0e1756b71869e93d4e4dca5c7d0eb0aa6` if the security upgrade is made for _the bridge contract_ on _the xDai chain_.
  * `0xb289f0e6fbdff8eee340498a56e1787b303f1b6d` if the security upgrade is made for _the validators contract_ on _the xDai chain_.
5. Identify the index of the transaction returned in the `Submission` event as soon as the transaction from the previous step inclided into a block and share it with other multisig wallet owners.
6. (for the rest of owners) Use NiftyWallet (or another tool that could build transactions based on the contract source code or ABI) to invoke `confirmTransaction` of the mutlisig wallet contract (`0xff1a8EDA5eAcdB6aAf729905492bdc6376DBe2dd` on the Mainnet ETH, `0x0d3726e5a9f37234d6b55216fc971d30f150a60f` on the xDai chain). Set the gas limit in three times bigger than the gas estimation facility suggests.

#### Improvement introduction

1. Identify the method that needs to be called as part of the new implementation intialization. In the further steps it is assumes that the method's name is `upgradeFrom3to4` which takes no arguments. 
2. Use the method mentioned above from the new contract implementation code or ABI to encode the data to be passed to `upgradeToAndCall` method. Example of the data: `50d28adb`. 
2. Deploy a new implementation of the bridge or validators contract.
3. Depending on the contract and the chain use one of the link below to get the current `version` of the contract implementation:
  * The bridge contract_ on _the ETH Mainnet_: [Etherscan](https://etherscan.io/address/0x4aa42145aa6ebf72e164c9bbc74fbd3788045016#readContract), 
  * The validators contract_ on _the ETH Mainnet_: [Etherscan](https://etherscan.io/address/0xe1579dEbdD2DF16Ebdb9db8694391fa74EeA201E#readContract) 
  * The bridge contract_ on _the xDai chain_: [Blockscout](https://blockscout.com/poa/dai/address/0x7301cfa0e1756b71869e93d4e4dca5c7d0eb0aa6/read_contract)
  * The validators contract_ on _the xDai chain_: [Blockscout](https://blockscout.com/poa/dai/address/0xb289f0e6fbdff8eee340498a56e1787b303f1b6d/read_contract)
4. Use the `upgradeToAndCall` method from `EternalStorageProxy` ABI, the address of the new implementation and the incremented version number to encode the data for the transaction. Example of the data: `0xa9c45fcb0000000000000000000000000000000000000000000000000000000000000004000000000000000000000000692a70d2e424a56d2c6c27aa97d1a86395877b3a0000000000000000000000000000000000000000000000000000000000000060000000000000000000000000000000000000000000000000000000000000000450d28adb00000000000000000000000000000000000000000000000000000000`.
5. Use NiftyWallet (or another tool that could build transactions based on the contract source code or ABI) to invoke `submitTransaction` of the mutlisig wallet contract (`0xff1a8EDA5eAcdB6aAf729905492bdc6376DBe2dd` on the Mainnet ETH, `0x0d3726e5a9f37234d6b55216fc971d30f150a60f` on the xDai chain). The data field must be filled with the bytes received on the previous step. The destination depends on the contract:
  * `0x4aa42145Aa6Ebf72e164C9bBC74fbD3788045016` if the security upgrade is made for _the bridge contract_ on _the ETH Mainnet_. 
  * `0xe1579dEbdD2DF16Ebdb9db8694391fa74EeA201E` if the security upgrade is made for _the validators contract_ on _the ETH Mainnet_. 
  * `0x7301cfa0e1756b71869e93d4e4dca5c7d0eb0aa6` if the security upgrade is made for _the bridge contract_ on _the xDai chain_.
  * `0xb289f0e6fbdff8eee340498a56e1787b303f1b6d` if the security upgrade is made for _the validators contract_ on _the xDai chain_.
6. Identify the index of the transaction returned in the `Submission` event as soon as the transaction from the previous step inclided into a block and share it with other multisig wallet owners.
7. (for the rest of owners) Use NiftyWallet (or another tool that could build transactions based on the contract source code or ABI) to invoke `confirmTransaction` of the mutlisig wallet contract (`0xff1a8EDA5eAcdB6aAf729905492bdc6376DBe2dd` on the Mainnet ETH, `0x0d3726e5a9f37234d6b55216fc971d30f150a60f` on the xDai chain). Set the gas limit in four times bigger than the gas estimation facility suggests.

### Configuration

Definition of all configuration parameters that could change the bridge behavior are available in the separate section below.
Here are described the steps for changing the daily limit of the Bridge Home contract (`0x7301cfa0e1756b71869e93d4e4dca5c7d0eb0aa6`, deployed in the xDai chain). These steps could be easily to be interpolated on other parameters of this or another contract:

1. Use the `setDailyLimit` method from `HomeBridgeErcToNative` ABI and a new daily limit (in this example the new limit is 50000 xDai coins) to encode the data for the transaction. Example of the data: `b20d30a9000000000000000000000000000000000000000000000a968163f0a57b400000`.
2. Use NiftyWallet (or another tool that could build transactions based on the contract source code or ABI) to invoke `submitTransaction` of the mutlisig wallet contract (`0x0d3726e5a9f37234d6b55216fc971d30f150a60f` on the xDai chain). The destination must be `0x7301cfa0e1756b71869e93d4e4dca5c7d0eb0aa6`. The data field must be filled with the bytes received on the previous step. 
3. Identify the index of the transaction returned in the `Submission` event as soon as the transaction from the previous step inclided into a block and share it with other multisig wallet owners.
4. (for the rest of owners) Use NiftyWallet (or another tool that could build transactions based on the contract source code or ABI) to invoke `confirmTransaction` of the mutlisig wallet contract (`0x0d3726e5a9f37234d6b55216fc971d30f150a60f`). Set the gas limit twice bigger than the gas estimation facility suggests.
5. Since any modificatio of the daily limit on one side must be reflected on another side as it is described below, the steps 1 - 4 need to be executed on the ETH Mainnet. So, use the `setExecutionDailyLimit` method from `ForeignBridgeErcToNative` ABI and the same daily limit as was used in the step 1 to encode the data for the transaction. Example of the data: `3dd95d1b000000000000000000000000000000000000000000000a968163f0a57b400000`.
6. Use NiftyWallet (or another tool that could build transactions based on the contract source code or ABI) to invoke `submitTransaction` of the mutlisig wallet contract (`0xff1a8EDA5eAcdB6aAf729905492bdc6376DBe2dd` on the ETH Mainnet). The destination must be `0x4aa42145Aa6Ebf72e164C9bBC74fbD3788045016`. The data field must be filled with the bytes received on the previous step. 
7. Identify the index of the transaction returned in the `Submission` event as soon as the transaction from the previous step inclided into a block and share it with other multisig wallet owners.
8. (for the rest of owners) Use NiftyWallet (or another tool that could build transactions based on the contract source code or ABI) to invoke `confirmTransaction` of the mutlisig wallet contract (`0xff1a8EDA5eAcdB6aAf729905492bdc6376DBe2dd`). Set the gas limit twice bigger than the gas estimation facility suggests.

### Admin privileges management

Prior to go through an example of the flow to change admin privileges for bridge contracts it is worth to describe types of admin privileges.

1. **Admin to manage the validators set** - the account that has rights to add and remove validators, change the number of required confirmations. 
2. **Admin to manage brigde parameters** - the account that has rights to adjust the transactions limits, the failback gas price and finalization threshold.
3. **Upgradability admin** - the account that has rights to upgrade the bridge contract and the validators contract. Since this type of admin is very powerful, only the upgradability admin has permissions to claim tokens sent to the bridge contract by mistake.

By default the multisig wallet of the xDai bridge validators is assositated with all three roles.

#### Change the admin to manage the validators set

1. Use the `transferOwnership` method from `BridgeValidators` ABI and a new admin account to encode the data for the transaction. Example of the data: `f2fde38b000000000000000000000000715e5837c903d0b52ec2b576d70406f095f72ac5`.
2. Use NiftyWallet (or another tool that could build transactions based on the contract source code or ABI) to invoke `submitTransaction` of the mutlisig wallet contract (`0xff1a8EDA5eAcdB6aAf729905492bdc6376DBe2dd` on the Mainnet ETH, `0x0d3726e5a9f37234d6b55216fc971d30f150a60f` on the xDai chain). The destination must be either `0xe1579dEbdD2DF16Ebdb9db8694391fa74EeA201E` if the new admin is for the validators contract on the ETH Mainnet side or `0xb289f0e6fbdff8eee340498a56e1787b303f1b6d` if the new admin is for the contract on the xDai chain. The data field must be filled with the bytes received on the previous step. 
3. Identify the index of the transaction returned in the `Submission` event as soon as the transaction from the previous step inclided into a block and share it with other multisig wallet owners.
4. (for the rest of owners) Use NiftyWallet (or another tool that could build transactions based on the contract source code or ABI) to invoke `confirmTransaction` of the mutlisig wallet contract (`0xff1a8EDA5eAcdB6aAf729905492bdc6376DBe2dd` on the Mainnet ETH, `0x0d3726e5a9f37234d6b55216fc971d30f150a60f` on the xDai chain). Set the gas limit twice bigger than the gas estimation facility suggests.  

#### Change the admin to manage brigde parameters

1. Use the `transferOwnership` method from `ForeignBridgeErcToNative` or `HomeBridgeErcToNative` ABI and a new admin account to encode the data for the transaction. Example of the data: `f2fde38b000000000000000000000000715e5837c903d0b52ec2b576d70406f095f72ac5`.
2. Use NiftyWallet (or another tool that could build transactions based on the contract source code or ABI) to invoke `submitTransaction` of the mutlisig wallet contract (`0xff1a8EDA5eAcdB6aAf729905492bdc6376DBe2dd` on the Mainnet ETH, `0x0d3726e5a9f37234d6b55216fc971d30f150a60f` on the xDai chain). The destination must be either `0x4aa42145Aa6Ebf72e164C9bBC74fbD3788045016` if the new admin is for the bridge contract on the ETH Mainnet side or `0x7301cfa0e1756b71869e93d4e4dca5c7d0eb0aa6` if the new admin is for the contract on the xDai chain. The data field must be filled with the bytes received on the previous step. 
3. Identify the index of the transaction returned in the `Submission` event as soon as the transaction from the previous step inclided into a block and share it with other multisig wallet owners.
4. (for the rest of owners) Use NiftyWallet (or another tool that could build transactions based on the contract source code or ABI) to invoke `confirmTransaction` of the mutlisig wallet contract (`0xff1a8EDA5eAcdB6aAf729905492bdc6376DBe2dd` on the Mainnet ETH, `0x0d3726e5a9f37234d6b55216fc971d30f150a60f` on the xDai chain). Set the gas limit twice bigger than the gas estimation facility suggests.  

#### Change the upgradability admin

1. Use the `transferProxyOwnership` method from `EternalStorageProxy` ABI and a new admin account to encode the data for the transaction. Example of the data: `ff1739cae000000000000000000000000715e5837c903d0b52ec2b576d70406f095f72ac5`.
2. Use NiftyWallet (or another tool that could build transactions based on the contract source code or ABI) to invoke `submitTransaction` of the mutlisig wallet contract (`0xff1a8EDA5eAcdB6aAf729905492bdc6376DBe2dd` on the Mainnet ETH, `0x0d3726e5a9f37234d6b55216fc971d30f150a60f` on the xDai chain). The data field must be filled with the bytes received on the previous step. The destination depends on the contract:
  * `0x4aa42145Aa6Ebf72e164C9bBC74fbD3788045016` if the upgradability admin is being changed for _the bridge contract_ on _the ETH Mainnet_. 
  * `0xe1579dEbdD2DF16Ebdb9db8694391fa74EeA201E` if the upgradability admin is being changed for _the validators contract_ on _the ETH Mainnet_. 
  * `0x7301cfa0e1756b71869e93d4e4dca5c7d0eb0aa6` if the upgradability admin is being changed for _the bridge contract_ on _the xDai chain_.
  * `0xb289f0e6fbdff8eee340498a56e1787b303f1b6d` if the upgradability admin is being changed for _the validators contract_ on _the xDai chain_.
3. Identify the index of the transaction returned in the `Submission` event as soon as the transaction from the previous step inclided into a block and share it with other multisig wallet owners.
4. (for the rest of owners) Use NiftyWallet (or another tool that could build transactions based on the contract source code or ABI) to invoke `confirmTransaction` of the mutlisig wallet contract (`0xff1a8EDA5eAcdB6aAf729905492bdc6376DBe2dd` on the Mainnet ETH, `0x0d3726e5a9f37234d6b55216fc971d30f150a60f` on the xDai chain). Set the gas limit twice bigger than the gas estimation facility suggests.  

### ERC20 tokens release

1. Use the `claimTokens` method from `ForeignBridgeErcToNative` or `HomeBridgeErcToNative` ABI, an address of an ERC20 token contract and receiver of the tokens to encode the data for the transaction. Example of the data: `69ffa08a0000000000000000000000000dcd2f752394c41875e259e00bb44fd505297caf000000000000000000000000583031d1113ad414f02576bd6afabfb302140225`.
2. Use NiftyWallet (or another tool that could build transactions based on the contract source code or ABI) to invoke `submitTransaction` of the mutlisig wallet contract (`0xff1a8EDA5eAcdB6aAf729905492bdc6376DBe2dd` on the Mainnet ETH, `0x0d3726e5a9f37234d6b55216fc971d30f150a60f` on the xDai chain). The destination must be either `0x4aa42145Aa6Ebf72e164C9bBC74fbD3788045016` if the token contract is on the ETH Mainnet or `0x7301cfa0e1756b71869e93d4e4dca5c7d0eb0aa6` if the token contract is on the xDai chain. The data field must be filled with the bytes received on the previous step. 
3. Identify the index of the transaction returned in the `Submission` event as soon as the transaction from the previous step inclided into a block and share it with other multisig wallet owners.
4. (for the rest of owners) Use NiftyWallet (or another tool that could build transactions based on the contract source code or ABI) to invoke `confirmTransaction` of the mutlisig wallet contract (`0xff1a8EDA5eAcdB6aAf729905492bdc6376DBe2dd` on the Mainnet ETH, `0x0d3726e5a9f37234d6b55216fc971d30f150a60f` on the xDai chain). Set the gas limit three times bigger than the gas estimation facility suggests.  

## The xDai bridge management API

The following management interfaces are available for the validators of the xDai bridge:

### Upgrade of the bridge contracts implementation:

#### The contract on the ETH Mainnet: 
| Address | Explorers | ABI |
| --------| -------- | -------- |
| `0x4aa42145Aa6Ebf72e164C9bBC74fbD3788045016` | [Blockscout](https://blockscout.com/eth/mainnet/address/0x4aa42145aa6ebf72e164c9bbc74fbd3788045016/transactions) | [Etherscan](https://etherscan.io/address/0x4aa42145aa6ebf72e164c9bbc74fbd3788045016) | [EternalStorageProxy ABI](https://github.com/poanetwork/poa-chain-spec/blob/4fe29a700b1331c570e20c5424523e522f5ab4d7/abis/bridge/EternalStorageProxy.json) |

| Method | Description |
| --------| -------- |
| `upgradeTo` | It allows to upgrade the current implementation of the bridge contract to a new one. The method expects an integer numbering the next implementation version and an Ethereum address where the next implementation was deployed. |  
| `upgradeToAndCall` | It allows to upgrade the current implementation of the bridge contract to a new one and to call automatically a method in the new implementation. The method expects an integer numbering the next implementation version, an Ethereum address where the next implementation was deployed and set of bytes containing the selector of the method with the parameters ABI-encoded. The encoded method will be called on benhalf of the proxy contract. |
| `transferProxyOwnership` | It changes the upgrade admin account whose opearations are allowed to upgrade the bridge contract implementation. The method expects a new Ethereum address. |

#### The contract on the xDai chain: 
| Address | Explorers | ABI |
| --------| -------- | -------- |
| `0x7301cfa0e1756b71869e93d4e4dca5c7d0eb0aa6` | [Blockscout](https://blockscout.com/poa/dai/address/0x7301cfa0e1756b71869e93d4e4dca5c7d0eb0aa6/transactions) | [EternalStorageProxy ABI](https://github.com/poanetwork/poa-chain-spec/blob/4fe29a700b1331c570e20c5424523e522f5ab4d7/abis/bridge/EternalStorageProxy.json) |

| Method | Description |
| --------| -------- |
| `upgradeTo` | It allows to upgrade the current implementation of the bridge contract to a new one. The method expects an integer numbering the next implementation version and an Ethereum address where the next implementation was deployed. |  
| `upgradeToAndCall` | It allows to upgrade the current implementation of the bridge contract to a new one and to call automatically a method in the new implementation. The method expects an integer numbering the next implementation version, an Ethereum address where the next implementation was deployed and set of bytes containing the selector of the method with the parameters ABI-encoded. The encoded method will be called on benhalf of the proxy contract. |  
| `transferProxyOwnership` | It changes the upgrade admin account whose opearations are allowed to upgrade the bridge contract implementation. The method expects a new Ethereum address. |

### Upgrade of the validators contracts implementation:

#### The contract on the ETH Mainnet: 
| Address | Explorers | ABI |
| --------| -------- | -------- |
| `0xe1579dEbdD2DF16Ebdb9db8694391fa74EeA201E` | [Blockscout](https://blockscout.com/eth/mainnet/address/0xe1579debdd2df16ebdb9db8694391fa74eea201e/transactions), [Etherscan](https://etherscan.io/address/0xe1579dEbdD2DF16Ebdb9db8694391fa74EeA201E) | [EternalStorageProxy ABI](https://github.com/poanetwork/poa-chain-spec/blob/4fe29a700b1331c570e20c5424523e522f5ab4d7/abis/bridge/EternalStorageProxy.json) |

| Method | Description |
| --------| -------- |
| `upgradeTo` | It allows to upgrade the current implementation of the bridge contract to a new one. The method expects an integer numbering the next implementation version and an Ethereum address where the next implementation was deployed. |  
| `upgradeToAndCall` | It allows to upgrade the current implementation of the bridge contract to a new one and to call automatically a method in the new implementation. The method expects an integer numbering the next implementation version, an Ethereum address where the next implementation was deployed and set of bytes containing the selector of the method with the parameters ABI-encoded. The encoded method will be called on benhalf of the proxy contract. |  
| `transferProxyOwnership` | It changes the upgrade admin account whose opearations are allowed to upgrade the validators contract implementation. The method expects a new Ethereum address. |

#### The contract on the xDai chain: 
| Address | Explorers | ABI |
| --------| -------- | -------- |
| `0xb289f0e6fbdff8eee340498a56e1787b303f1b6d` | [Blockscout](https://blockscout.com/poa/dai/address/0xb289f0e6fbdff8eee340498a56e1787b303f1b6d/transactions) | [EternalStorageProxy ABI](https://github.com/poanetwork/poa-chain-spec/blob/4fe29a700b1331c570e20c5424523e522f5ab4d7/abis/bridge/EternalStorageProxy.json) |

| Method | Description |
| --------| -------- |
| `upgradeTo | It allows to upgrade the current implementation of the bridge contract to a new one. The method expects an integer numbering the next implementation version and an Ethereum address where the next implementation was deployed. |  
| `upgradeToAndCall` | It allows to upgrade the current implementation of the bridge contract to a new one and to call automatically a method in the new implementation. The method expects an integer numbering the next implementation version, an Ethereum address where the next implementation was deployed and set of bytes containing the selector of the method with the parameters ABI-encoded. The encoded method will be called on benhalf of the proxy contract. |  
| `transferProxyOwnership` | It changes the upgrade admin account whose opearations are allowed to upgrade the validators contract implementation. The method expects a new Ethereum address. |

### Management of the bridge validators list and the number of required signatures:

#### The contract on the ETH Mainnet: 
| Address | Explorers | ABI |
| --------| -------- | -------- |
| `0xe1579dEbdD2DF16Ebdb9db8694391fa74EeA201E` | [Blockscout](https://blockscout.com/eth/mainnet/address/0xe1579debdd2df16ebdb9db8694391fa74eea201e/transactions), [Etherscan](https://etherscan.io/address/0xe1579dEbdD2DF16Ebdb9db8694391fa74EeA201E) | [BridgeValidators ABI](https://github.com/poanetwork/poa-chain-spec/blob/4fe29a700b1331c570e20c5424523e522f5ab4d7/abis/bridge/BridgeValidators.json) |

| Method | Description |
| --------| -------- |
| `addValidator` | It adds a new validator. The method expects an Ethereum address of the new validator. It is recommended to stop all bridge instances prior to add the new validator. Make sure that the corresponding validator is added on another side of the bridge. |
| `removeValidator` | It removes a validator. The method expects the address of the validator that needs to be removed. The operation will be successful only if the number of remaining validator is still greater of equal the number of required signatures. It is recommended to stop all bridge instances prior to remove the validator. Make sure that the corresponding validator is added on another side of the bridge. |
| `setRequiredSignatures` | It sets the number of required signatures (the number of validators' confirmations). The method expects an integer. It is recommended to stop all bridge instances prior to remove the validator. Make sure that the number of required signatures is updated on another side of the bridge to the corresponding value. |
| `transferOwnership` | It changes the management account whose opearations are allowed to change parameters of the validators contract. The method expects a new Ethereum address. |

#### The contract on the xDai chain: 
| Address | Explorers | ABI |
| --------| -------- | -------- |
| `0xb289f0e6fbdff8eee340498a56e1787b303f1b6d` | [Blockscout](https://blockscout.com/poa/dai/address/0xb289f0e6fbdff8eee340498a56e1787b303f1b6d/transactions) | [BridgeValidators ABI](https://github.com/poanetwork/poa-chain-spec/blob/4fe29a700b1331c570e20c5424523e522f5ab4d7/abis/bridge/BridgeValidators.json) |

| Method | Description |
| --------| -------- |
| `addValidator` | It adds a new validator. The method expects an Ethereum address of the new validator. It is recommended to stop all bridge instances prior to add the new validator. Make sure that the corresponding validator is added on another side of the bridge. |
| `removeValidator` | It removes a validator. The method expects the address of the validator that needs to be removed. The operation will be successful only if the number of remaining validator is still greater of equal the number of required signatures. It is recommended to stop all bridge instances prior to remove the validator. Make sure that the corresponding validator is added on another side of the bridge. |
| `setRequiredSignatures` | It sets the number of required signatures (the number of validators' confirmations). The method expects an integer. It is recommended to stop all bridge instances prior to remove the validator. Make sure that the number of required signatures is updated on another side of the bridge to the corresponding value. |
| `transferOwnership` | It changes the management account whose opearations are allowed to change parameters of the validators contract. The method expects a new Ethereum address. |

### Management of the bridge parameteres:

#### The contract on the ETH Mainnet: 
| Address | Explorers | ABI |
| --------| -------- | -------- |
| `0x4aa42145Aa6Ebf72e164C9bBC74fbD3788045016` | [Blockscout](https://blockscout.com/eth/mainnet/address/0x4aa42145aa6ebf72e164c9bbc74fbd3788045016/transactions), [Etherscan](https://etherscan.io/address/0x4aa42145aa6ebf72e164c9bbc74fbd3788045016) | [ForeignBridgeErcToNative ABI](https://github.com/poanetwork/poa-chain-spec/blob/4fe29a700b1331c570e20c5424523e522f5ab4d7/abis/bridge/ForeignBridgeErcToNative.json) |

| Method | Description |
| --------| -------- |
| `setExecutionDailyLimit` | It sets the daily limit of tokens amount that can be requested by the bridge validators to be unlocked through the confirmations. The method expects an integer - the amount of tokens. The value must be the same as the daily limit set on another side of the bridge. This parameter is needed to secure the bridge from malicious validators actions. |
| `setExecutionMaxPerTx` | It sets the maximum of tokens amount that can be requested by the bridge validators to be unlocked in one transaction. The method expects an integer - the amount of tokens. The value must be the same as the daily limit set on another side of the bridge. This parameter is needed to secure the bridge from malicious validators actions. |
| `setGasPrice` | It sets the gas price that will be used by the bridge instance to send transactions to the ETH Mainnet if the gas price oracle is not accessible. The method expects an integer - the gas price in Wei. |
| `setRequiredBlockConfirmations` | It sets the finalization threshold - the number of blocks after which the bridge instance considers the chain as finalized and sends the confirmation for the corresponding transfer request. The method exepects an integer. |
| `transferOwnership` | It changes the management account whose opearations are allowed to change parameters of the bridge contract. The method expects a new Ethereum address. |
| `claimTokens` | It allows to transfer ether or ERC20 tokens that was sent to the bridge address by mistake. The method expects an Ethereum address of an ERC20 token contract and an Ethereum address of a recipient of tokens. If ether should be sent instead of tokens the token address account must be `0`. The method will fail if the token address is the same as the address of the token contract that is used for the bridge operations. The method is allowed to be called only from the account that is able to upgrade the bridge contracts. |

#### The contract on the xDai chain: 
| Address | Explorers | ABI |
| --------| -------- | -------- |
| `0x7301cfa0e1756b71869e93d4e4dca5c7d0eb0aa6` | [Blockscout](https://blockscout.com/poa/dai/address/0x7301cfa0e1756b71869e93d4e4dca5c7d0eb0aa6/transactions) | [HomeBridgeErcToNative ABI](https://github.com/poanetwork/poa-chain-spec/blob/4fe29a700b1331c570e20c5424523e522f5ab4d7/abis/bridge/HomeBridgeErcToNative.json) |

| Method | Description |
| --------| -------- |
| `setDailyLimit` | It sets the daily limit of xDai coins amount that can be transferred to the ETH Mainnet by the xDai chain users. The method expects an integer - the amount of coins in Wei. If this parameter is changed `ExecutionDailyLimit` must be set in the same value on the ETH Mainnet side. |
| `setMaxPerTx` | It sets the maximum of xDai coins amount that can be transferred to the ETH Mainnet by a chain user in one transaction. The method expects an integer - the amount of coins in Wei. If this parameter is changed `ExecutionMaxPerTx` must be set in the same value on the ETH Mainnet side. |
| `setMinPerTx` | It sets the minimus of xDai coins amount that can be transferred to the ETH Mainnet by a chain user in one transaction. The method expects an integer - the amount of coins in Wei. This parameter is needed to secure validators account to be dried out by transactions those values are less than network fees. |
| `setExecutionDailyLimit` | It sets the daily limit of xDai coins amount that can be requested by the bridge validators to be minted through the confirmations. The method expects an integer - the amount of tokens. This parameter is needed to secure the bridge from malicious validators actions. |
| `setExecutionMaxPerTx` | It sets the maximum of xDai coins amount that can be requested by the bridge validators to be minted in one transaction. The method expects an integer - the amount of tokens. This parameter is needed to secure the bridge from malicious validators actions. |
| `setRequiredBlockConfirmations` | It sets the finalization threshold - the number of blocks after which the bridge instance considers the chain as finalized and sends the confirmation for the corresponding transfer request. The method exepects an integer. |
| `transferOwnership` | It changes the management account whose opearations are allowed to change parameters of the bridge contract. The method expects a new Ethereum address. |
| `claimTokens` | It allows to transfer xDai coins or ERC20 tokens that was sent to the bridge address by mistake. The method expects an Ethereum address of an ERC20 token contract and an Ethereum address of a recipient of tokens. If ether should be sent instead of tokens the token address account must be `0`. The method is allowed to be called only from the account that is able to upgrade the bridge contracts. |

## Usage NiftyWallet for the xDai bridge management

Starting from the version 4.10, [NiftyWallet](https://github.com/poanetwork/nifty-wallet) can invoke methods of the contracts by taking ABIs either from verified contracts code on a block explorer or from the JSON data entered manually.

An example how to use the new functionality is available [here](https://medium.com/poa-network/nifty-wallet-now-supports-interactions-with-smart-contracts-5e8c43c19e3a)
