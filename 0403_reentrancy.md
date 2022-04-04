# Apr 3rd 2022



I stumbled across a blog post that says it is actually possible for anyone to recover Uniswap liquidity tokens that are accidentally sent to the pair contract itself by calling the `burn` function. I stopped reading the blog and looked at the `burn` function (UniswapV2Pair.sol). As I’m new to Solidity and there was an unfamiliar modifier called `lock`. I was curious why this is needed in many of their external functions as it’s just seemed to be used as it looks; checks whether the `unlocked` variable equals 1 and then changes it to 0 and then changes it again to 1 after executing the function. The answer was a simple preventative technique against reentrancy attacks.

<br />

```js
uint private unlocked = 1;
    modifier lock() {
        require(unlocked == 1, 'UniswapV2: LOCKED');
        unlocked = 0;
        _;
        unlocked = 1;
    }
```

<br />

Reentrancy is one of the most well-known vulnerabilities in Solidity-based Ethereum smart contracts. A famous exploitation case of reentrancy is the DAO attack; 3.6M Ether was stolen. 

<br />

![Untitled](https://user-images.githubusercontent.com/99378245/161455914-267694a8-094a-4abf-b966-b591eac45392.png)

<br />

Reentrancy can simply be done like the above. 4 simple steps

1. The attacker 🙂 initiates a transaction by calling the withdraw function (`attack()` here)
2. Contract A checks whether the balance of contract B is greater than 0 and as there is no `payable` function in contract B that in general receives Ether, the `fallback()` function is called
3. The fallback function recursively calls the `withdraw()` function again.
4. Repeated until the fund stored in contract A is completely drained then finally can break out of that loop

<br />

How can we prevent our contract from Reentrancy attacks?

- Update state variables before making any external calls to other contracts
    
    → In the above example, you can update the balance (`balance = 0`) before you make a call to another contract (`send Ether`) then once the fallback function is called, as the balance is already updated to 0, so the withdraw function can not be called again.
    
- Use a modifier called **ReentrancyGuard** that can prevent reentrancy during certain functions
    
    → the `lock` modifier above 
    
 <br />

Curiosity solved 😊


<br />
<br />

References:

[https://www.youtube.com/watch?v=4Mm3BCyHtDY&t=3s](https://www.youtube.com/watch?v=4Mm3BCyHtDY&t=3s)

[https://arxiv.org/pdf/1908.04507.pdf](https://arxiv.org/pdf/1908.04507.pdf)

[https://arxiv.org/pdf/2105.06974.pdf](https://arxiv.org/pdf/2105.06974.pdf)

[https://www.paradigm.xyz/2020/08/ethereum-is-a-dark-forest](https://www.paradigm.xyz/2020/08/ethereum-is-a-dark-forest)

[https://github.com/Uniswap/v2-core/blob/master/contracts/UniswapV2Pair.sol#L140](https://github.com/Uniswap/v2-core/blob/master/contracts/UniswapV2Pair.sol#L140)

[https://ethereum.stackexchange.com/questions/97512/why-is-modifier-lock-needed-in-uniswapv2pair-smart-contract](https://ethereum.stackexchange.com/questions/97512/why-is-modifier-lock-needed-in-uniswapv2pair-smart-contract)
