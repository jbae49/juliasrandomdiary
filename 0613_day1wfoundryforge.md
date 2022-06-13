# Day1 with Foundry (Forge)

<br />

**[Foundry](https://www.paradigm.xyz/2021/12/introducing-the-foundry-ethereum-development-toolbox) is a blazing fast, portable and modular toolkit for Ethereum application development written in Rust.**


Foundry consists of:

- **[Forge](https://github.com/foundry-rs/foundry/blob/master/forge)**: Ethereum testing framework (like Truffle, Hardhat and DappTools). Forge lets you write your tests in Solidity, not Javascript or Typescript
- **[Cast](https://github.com/foundry-rs/foundry/blob/master/cast)**: Swiss army knife for interacting with EVM smart contracts, sending transactions and getting chain data.
- **[Anvil](https://github.com/foundry-rs/foundry/blob/master/anvil)**: local Ethereum node, akin to Ganache, Hardhat Network.

<br />

I wanted to test the difference between `public` and `external` modifier in terms of gas usage, mentioned in [here](https://ethereum.stackexchange.com/questions/19380/external-vs-public-best-practices?answertab=active#tab-top). I’m pretty new to Foundry and Solidity so had to face several errors even with a very simple contract.

This was the very first draft code of the contract and the testing script. 

<br />

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.7;

contract Contract {
	function test1(uint[10] memory array_a) public pure returns(uint){
		return array_a[5]*2;
	}
}
```

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.7;

import "../lib/forge-std/src/Test.sol";
import "../src/Contract.sol";

contract ContractTest is Test {
	Contract public sampleContract;
	uint[10] sampleArray;

	function setUp() public {
		sampleContract = new Contract();
		sampleArray = [1,2,3,4,5,6,7,8,9,10];
	}

	function test_test1() public {
		uint result1 = sampleContract.test1(sampleArray);
		assertEq(result1, 12);
	}
}
```

```solidity
forge build
forge test
```
<br />

Did you catch the error?

<br />

![Untitled](https://user-images.githubusercontent.com/99378245/173440300-dc87a7b8-611b-4849-84ac-8d7009b64b93.png)

<br />

The error was due to when Foundry tried to test the function with several random HUGE numbers and it overflowed the range of uint256. This `Arithmetic over/underflow` issue can be solved by using `Unchecked` will allow the number to wrap around so not give the error above. It may require a little more gas and there’s another way to solve this → I forgot; what was it

I could solve this by simply wrapping the arithmetic operation

<br />

```solidity
return array_a[5]*2;
```

with `unchecked` like the below

```solidity
unchecked { return array_a[10]*2; }
```

<br />

Time to test the fact that `external` is more gas-efficient than `public` 

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.7;

contract Contract {
    function test1(uint[20] memory array_a) public pure returns(uint){
        unchecked { return array_a[10]*2; }
    }

    function test2(uint[20] memory array_b) external pure returns(uint){
        unchecked { return array_b[10]*2; }
    }
}
```

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.7;

import "../lib/forge-std/src/Test.sol";
import "../src/Contract.sol";

contract ContractTest is Test {
    Contract public SampleContract;
    uint[20] SampleArray;

    function setUp() public {
        SampleContract = new Contract();
        SampleArray = [1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20];
    }

    function test_test1() public {
        uint result1 = SampleContract.test1(SampleArray);
        assertEq(result1, 22);
    }

    function test_test2() public {
        uint result2 = SampleContract.test2(SampleArray);
        assertEq(result2, 22);
    }

}
```
<br />

Results were different from what I expected 

<br />

![Untitled](https://user-images.githubusercontent.com/99378245/173440619-d3a8c811-a444-4c36-aada-70401befed5e.png)

<br />

test_test1() (gas: 28245) (that uses `public`) spent less gas than test_test2() (gas: 28267) (that uses `external`). According to [this](https://ethereum.stackexchange.com/questions/19380/external-vs-public-best-practices?answertab=active#tab-top), when using large arrays of data, external functions are likely more efficient than public functions. 

<br />
<br />
<br />

I still don’t figure out 

- Why in this case external function spent more gas than public function?
- Does Solidity have a method like `range()` to generate numbers within the range?
    - ex) `list(range(1,10))` Or `[1:10]` etc → [1,2,3,4,5,6,7,8,9,10]
    - Need to use for loop by pushing numbers in a dynamic array?
    - What’s the best practice for this
- I still don’t know why people declare the value first then assign later in Solidity
