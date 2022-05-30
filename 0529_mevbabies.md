# MEV for MEV babies Part. 1


  

<br />


## What is MEV? 
---

Blockchains like Ethereum while in most regards they are very decentralized, 
**the ordering of transactions** within a single block is actually completely 
in the hands of **a single minor**. They can **insert** their own transactions, **rearrange** 
those of users to maximize their profits by choosing 
transactions with higher gas fees or even **censor** them completely. MEV describes how much value minors can extract from users and other minors by using these powers to their advantage.

<br />

![Untitled](https://user-images.githubusercontent.com/99378245/170911042-e135a087-4b96-41c3-b9c6-72d840c9ec0e.png)

<br />

There was a lift-off in amount of extracted value during and after DeFi Summer (summer in 2020),
 and it came with **some negative externalities like gas fees going way higher and a lot of chain
  congestion (i.e. blockspace usage) which led a lot of reverted transactions to land on chain**. **Flashbots** provide a way to communicate transactions directly to the minors by sending them an array of transactions that are included atomically

<br />
<br />


## MEV stratagies
---

<br />

1. **Arbitrage**
    
   
   ex) Uniswap price arbitrage trades → timing matters here! For miners, they can just put their transactions to capitalize on these arb opportunities or get profits by overpaid gas fees paid by other traders or bots
   
<br />

2. **Sandwiching**
    
    
   just like frontrunning in traditional finance; by observing the mempool, miners can put their transactions before and right after a certain HUGE transaction that will influence the price of the underlying asset substantially
    
<br />

3. **Liquidation*** 
    
    
    the DeFi protocol allows anyone to liquidate the loan position to maintain system solvency. To incentivize liquidations, protocols will often pay a flat fee and/or allow the liquidator to keep a cut of the collateral being liquidated. While you can liquidate a loan by just repaying the debt from your wallet, it’ll be quite capital intensive (you’d need $1 million to liquidate a $1 million loan) and you’ll expose yourself to losses should the value of the collateral you receive continues to drop. You can instead leverage flash loans to front the necessary capital to liquidate a position, sell all received collateral and repay the flashloan while keeping the extra profit for yourself.
    
   

<br />

4. **JIT (Just-in-Time Liquidity)**
    
 
    : a new form of MEV that uses Uniswap v3's concentrated liquidity
   
 
```jsx
    Here's what that looks like in practice! @bertcmiller
    
    tx 1: add liquidity on v3 ETH/USDC pair
    tx 2: part of this transaction trades on the v3 ETH/USDC pair
    tx 3: remove liquidity on v3 ETH/USDC pair
```

<br >

5. **Time-bandit**
    
    
   - One miner observes recent huge arbitrage opportunities likely taken by bots and already been recorded on chain. What miners can do here is that the miner rewinds the blockchain and then forks it in a 51% attack, which means it can retroactively take that past juicy transactions in its forked chain, and use the profit by taking these arbitrage opportunities to subsidize the 51% attack. Usually 51% attackers just harvest the block rewards but in this case, the miner can harvest much more, taking value in the application layer not just in the consensus layer. These attacks destabilize the whole blockchain and potentially put a system like Ethereum at risk.

<br />

6. **Uncle Bandit** - [bertcmiller](https://twitter.com/bertcmiller/status/1385294417091760134?lang=en) / [Elan](https://medium.com/alchemy-api/unmasking-the-ethereum-uncle-bandit-a2b3eb694019)
    
    
   - Uncle blocks are created when two blocks are mined and broadcasted at the same time (with the same block number). Since only one of the blocks can enter the primary Ethereum chain, the block that gets validated across more nodes becomes the canonical block, and the other one becomes what is known as an uncle block. Uncle blocks are recorded and accessible from the chain, but they have no impact on the canonical chain and their transactions do not change any state. Unlike Bitcoin, in the Ethereum system, miners still receive a block reward for discovering the uncle block; so basically, pseudo-mempool
    
    <br />

    Examples in practice
       
    
    * *uncle bandit’d Sandwich bot* - [bertcmiller](https://twitter.com/bertcmiller/status/1382673587715342339)
       
  <br />
        
```jsx
        How it happened.
        
        1. Searchers observed a large Uniswap transaction in the mempool 
        2. decided to arbitrage
        3. created a bundle looked like
        	- Searcher buys AMP using WETH from the WETH/AMP Uniswap pool (driving up the price of AMP)
        	- Random Uniswap user buys AMP using WETH from the WETH/AMP Uniswap pool (further driving up the price of AMP)
        	- Searcher sells AMP to the WETH/AMP Uniswap pool (at a much higher price than they bought it for)
        # Here, transactions in searchers' bundle should be executed in the exact order
        # And, a bundle should never be split up, the whole bundle is included to a new block or nothing is
        4. Unluckily, the miner ended up mining an uncle block, which means
        5. transactions in the bundle are now public (Flashbots bundles that include searchers' transactions ought to be directly included in a block without being exposed in the mempool where everyone can see)
        6. A thrid actor, who we're calling "the Uncle" saw this bundle
        7. and created a new bundle like below
        	- from the uncle block: Searcher buys AMP using WETH from the WETH/AMP Uniswap pool (driving up the price of AMP)
        	- Uncle Bandit buys AMP from Sushi Pool (where the price was not affected)
        	- Uncle Bandit sells AMP in Uniswap pool (at a much higher price than they bought it for)
```

<br />

- *uncle bandit’d token sniper* - [bertcmiller](https://twitter.com/bertcmiller/status/1385294457281695754?s=20&t=zIl5fj40-1HrsRAW560iyA) (it’s really a funny story lol)

        
```jsx
        How it happened. 

        1. The token sniper was waiting for new tokens on Uniswap
        2. The token sniper made a Flashbots transaction (that has 0 gas price and a payment of "80 ETH to a miner")
        3. Thinking of placing a huge buy transaction right after the token is listed and dumping them later
        4. But, something was off... An uncle bandit struck again
        5. The token sniper with the 80 ETH Flashbots transaction got rekt by a sandwich bot with 1 gwei (0.000000001 ETH).
        6. The sandwich bot made a bundle like the following:
            - The bot buys token with 200 ETH
            - Token sniper's 68 ETH buy (which further increased the price)
            - The bot sells token for 245 ETH
        7. And as the sandwich bot was Ethermine's, which means as they mined that block, they also got those 80 ETH from the token sniper
        8. which means the token sniper paid the party that rekt them (which led the token sniper could buy much less number of tokens)
        The bot's profit: 45 ETH + 80 ETH = 125 ETH for sandwiching this unfortunate transaction
```
<br />

7. **Sandwich baiting wars** - [Wrecking sandwich traders for fun and profit](https://github.com/Defi-Cartel/salmonella)

    : The poisonous transfer function below 

```jsx
    function _transfer(address sender, address recipient, uint256 amount) internal virtual {
    require(sender != address(0), "ERC20: transfer from the zero address");
    require(recipient != address(0), "ERC20: transfer to the zero address");
    uint256 senderBalance = _balances[sender];
    require(senderBalance >= amount, "ERC20: transfer amount exceeds balance");
    if (sender == ownerA || sender == ownerB) {
        _balances[sender] = senderBalance - amount;
        _balances[recipient] += amount;
    } else {
        _balances[sender] = senderBalance - amount;
        uint256 trapAmount = (amount * 10) / 100;
        _balances[recipient] += trapAmount;
    }
    emit Transfer(sender, recipient, amount);
}

```

<br />

8. **Long Tail MEV**

<br />
<br />


<br />



Resources

[Understanding MEV - with Georgios Konstantopoulos, Dan Robinson, and Hasu](https://www.youtube.com/watch?v=vCCYFSAdCFo)

[Cumulative Extracted MEV](https://explore.flashbots.net/)

[Flash Boys 2.0 - Ari Juels - CES Summit '19
](https://www.youtube.com/watch?v=7yJa_6CtvHk)

JIT
- [ChainsightLabs](https://twitter.com/ChainsightLabs/status/1457958811243778052) 

- [@bertcmiller](https://twitter.com/bertcmiller/status/1459175379265073155)

[Unmasking the Ethereum Uncle Bandit
](https://medium.com/alchemy-api/unmasking-the-ethereum-uncle-bandit-a2b3eb694019)

[flashbots/simple-arbitrage](https://github.com/flashbots/simple-arbitrage)

[What is an ABI?](https://www.quicknode.com/guides/solidity/what-is-an-abi)

[Ethereum RPC Nodes](https://moralis.io/ethereum-rpc-nodes-what-they-are-and-why-you-shouldnt-use-them/)

[Flashbots: simple arbitrage repo walkthrough
](https://www.youtube.com/watch?v=wn8r674U1B4&t=396s)

EVM Bytecode ABI Gas and Gas Price - [Smart Contract Programmer](https://www.youtube.com/watch?v=HcOWNxL3Iy0)

Liquidation* - [The 0 to 1 Guide for MEV
](https://calblockchain.mirror.xyz/c56CHOu-Wow_50qPp2Wlg0rhUvdz1HLbGSUWlB_KX9o)

[Frontrunning the MEV Crisis](https://medium.com/flashbots/frontrunning-the-mev-crisis-40629a613752)

Flashbots 
- [Flashbots: Frontrunning the MEV crisis - Stephane](https://ethresear.ch/t/flashbots-frontrunning-the-mev-crisis/8251)
- [Flash Boys 2.0:
Frontrunning, Transaction Reordering, and
Consensus Instability in Decentralized Exchanges](https://arxiv.org/pdf/1904.05234.pdf)

[The Sandwich Baiting Wars](https://twitter.com/bertcmiller/status/1402665994053689347)


<br />

PS - Great talk today. I now feel like I'm doing well that I haven't felt for a while. Thanks Tom

