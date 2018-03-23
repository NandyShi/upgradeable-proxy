# Upgradeable Proxy

This repository tests the proxy upgradeability mechanism. It is a simplified version of the system being used by the
[AragonOS](https://github.com/aragon/aragonOS), [Level K](https://github.com/levelkdev/master-storage) and [ZepplinOS](https://github.com/zeppelinos/core) systems. The core upgradeability mechanism has been copied and a few features have been removed.

The results of tests will be summarised here. The contributions of this repository are made for general knowledge only. No contributors are to be held liable for any damages occurred from using code or information from this repository. Do your own thorough testing before deploying any upgradeable smart contract mechanisms.

## 1. Getting started

To get up to speed on upgradeable smart contract strategies, please read [Summary of Ethereum Upgradeable Smart Contract R&D](https://blog.indorse.io/ethereum-upgradeable-smart-contract-strategies-456350d0557c).

As noted in the article, **implementing an upgradeable smart contract requires a well thought out governance strategy**. Section 3.4 advise on how this can be done.

## 2. Running the tests

```
git clone https://github.com/jackandtheblockstalk/upgradeable-proxy
cd upgradeable-proxy
truffle test
```

## 3. A **tested guide** to how to create an upgradeable Ethereum smart contract

### 3.1 The upgrade mechanism

The three main contracts that are used in the upgradeable proxy mechanism are:
1. [Proxied.sol](https://github.com/jackandtheblockstalk/upgradeable-proxy/blob/master/contracts/Proxied.sol)
2. [Proxy.sol](https://github.com/jackandtheblockstalk/upgradeable-proxy/blob/master/contracts/Proxy.sol)
3. [Upgradeable.sol](https://github.com/jackandtheblockstalk/upgradeable-proxy/blob/master/contracts/Upgradeable.sol)

Please see in-code contract and function descriptions for how these contracts allow you to make an upgradeable smart contract.
**TODO:** write descriptions of contracts above into code...

### 3.2 How to make a simple Uint getter/setter smart contract upgradeable

To see the simplest way of implementing an upgradeable smart contract, check out UintSimple.sol and it's test suite.

There are several ways to structure a smart contract that will be upgradeable. The following three sections explain these different structures and their pros and cons. In each of the structures, it was found that the gas cost increase was the same (~3% or 1100 gas). For each mechanism, the following smart contract will be upgraded with new logic for the setValue() function:
```
contract UintSimpleV1 is Upgradeable {
    uint value;

    function getValue() view public returns (uint) {
        return value;
    }

    function setValue(uint _value) public {
        value = _value;
    }
}
```

#### 3.2.1 One smart contract containing all desired storage and logic
See tests for contracts _UintSimple_

```
contract UintSimpleV2 is Upgradeable {
    uint value;

    function getValue() view public returns (uint) {
        return value;
    }

    function setValue(uint _value) public {
        value = 2*_value;
    }
}
```

#### 3.2.2 A modular smart contract design
See tests for contracts _UintSimpleModular_

See [UintSimpleModular.sol](https://github.com/jackandtheblockstalk/upgradeable-proxy/blob/master/contracts/test/UintSimpleModular.sol)

#### 3.2.3 An inherited smart contract
See tests for contracts _UintInherited_

```
contract UintInheritedV2 is UintSimpleV1 {
    function setValue(uint _value) public {
        value = 2*_value;
    }
}
```

### 3.3 What can and can't you do when upgrading a contract with the Proxy

#### 3.3.1 You can

You can do the following changes on an upgraded contract and it will behave as if you have replaced the contract and kept the state.

1. Change function logic of pre-existing functions so long as the signature does not change.
   - See contracts _UintSimple_, _UintAdvancedV2g_OverrideFunctionGetter_ and _UintAdvancedV2h_OverrideFunctionSetter_
2. Add new functions to the upgraded contract.
   - See contract _UintAdvancedV2a_NewFunction_
   - **Note:** the new function can be added in any place in the contract. It does not need to be added as the last function. See contract _UintAdvancedV2i_NewFunction_
   - **Note:** applications or other smart contracts will need to know about the upgrade to be able to call the new function with the new ABI.
3. Add new events to the upgraded contract.
   - See contract _UintAdvancedV2c_NewEvent_
   - **Note:** applications or other smart contracts will need to know about the upgrade to be able to watch for the new event with the new ABI.
4. Change the order of transactions in a function.
   - See contract _UintAdvancedV2d_ReverseFunctionOrder_
5. Change visibility for upgraded functions.
   - **Note:** Only the following changes were tested
     - public --> external. Functions were still callable externally. See contract _UintAdvancedV2j_ChangeVisibility_
     - public --> internal. Function calls could no longer be made. see contract _UintAdvancedV2k_ChangeVisibility_
   - **TODO:** test this more
6. Change access modifier view to pure.
   - See contract _UintAdvancedV2i_ChangeKeyword_
7. Remove events from the contract
   - See contract _UintEventV2a_RemovedEvent_
8. Change the return type for function calls.
   - Changing to return solidity value types (uint, string etc), as well as arrays and tuples were successful
   - See contracts _UintAdvancedV2k_ChangeReturn_ to _UintAdvancedV2s_ChangeReturn_ (all letters k-s)
   - Changing to return structs was not tested due to limitations of the javascript web3 object at the time. See contract _UintAdvancedV2t_ChangeReturn_
   - **TODO** do above test when web3.js is updated
   - **Note:** applications or other smart contracts will need to know about the upgrade to be able to correctly interpret the new return values with the new ABI.

#### 3.3.2 You can't

You cannot do the following changes on an upgraded contract and expect that it will behave like an updated contract with the same state.

1. Add in new storage variables to the upgraded smart contract and use it in functions.
   - See contracts _UintAdvancedV2b_NewStorage_, _UintAdvancedV2e_NewStorage_, _UintAdvancedV2f_NewStorage_ and _UintAdvancedV2j_NewStorage_.
   - You can add new storage variables, however, these new variables cannot be used in **any** of the contract's pre-existing functions. They can be used in any new function.
   - **Note:** a workaround for this would be to create another [separate] contract with new state variable which can be called by the new contract. The initial upgradeable contract would have to have a pre-existing address in place so that this could be used to point to such a separate smart contract...
   - **TODO:** test workaround as stated above
2. Change the order of state variables in the upgraded smart contracts
   - See contract _DoubleUintV2_
3. Declare any variables with initialized values. Declare any constant state variables keyword.
   - See contract _UintInherited_
   - **Note:** state variables must be initialized using the intialize() function.

If you find a way to do any of the above, please send [me](https://twitter.com/theblockstalk) or the team at [Indorse](https://twitter.com/joinindorse) a message, or submit an issue or PR to this repo.

#### 3.3.3 Still to research

**TODO:**
* initialized variables in the target are not initialized in the target.
* CHECK WHY CONTRACTS THAT DO HAVE INITIALIZED VARIABLES ARE WORKING
* apply any intialize() functions to ownable contract as well
* Change access modifier pure to view.
* Test upgradeability of data structures: strings, mappings, structs, arrays
* upgrade a contract twice with many things that can be upgraded upgraded
* do a spellcheck of this readme.

### 3.4 Creating a permissioned (Ownable) proxy upgrade

The proxy upgrade mechanism was combined with the Zepplin [Ownable](https://github.com/OpenZeppelin/zeppelin-solidity/blob/master/contracts/ownership/Ownable.sol) contract standard to allow for the upgradeTo() function to only be called by the owner. The owner of the proxy can be set as a multisig or DAO-like contract to provide distributed governance.

The permissioned upgradeable contracts can be seen in the [/contracts/ownable](https://github.com/jackandtheblockstalk/upgradeable-proxy/tree/master/contracts/ownable) folder. Please see _UintOwnable_ contract tests for details.
