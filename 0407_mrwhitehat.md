# Apr 7th 2022

Apr 7th 2022

Thursday? Already? How time flies! 🛫 What I’m curious about today is “Is there any Web 3.0 application that can help my best friend quit smoking and drinking”. Feel free to suggest... 

<br />

*Poly Network* *exploit* on August 10, 2021

---

"I hope my life can be composed of unique adventures, so I like to learn and hack," the hacker sent messages via the Ethereum blockchain, which can be viewed publicly. 
An anonymous hacker dubbed ‘Mr. White Hat’ returned the stolen assets ($600M+ worth of tokens) and even did Q & A sessions - [link](https://twitter.com/tomrobin/status/1425487745166753794). The hacker also returned the $500k bug bounty, as well as other donations.

<br />

🐸 (fyi) - A smart contract is a program / a piece of software; it can be executed only when after is authorized by a digital signature of the “owner” of the contract and must be validated by the consensus and then can change the state of the blockchain.

<br />

What is Poly Network?

> Poly Network is a cross-chain protocol that allows users to transfer or swap tokens across different blockchains. (e.g., Bitcoin ↔ Ethereum)

<br />

Cross-chain networks

- they allow a variety of chains to interact and transfer data (”cross-chain interoperability bridge”) in a distributed and autonomous way
- to provide unexpectedly large liquidity, they usually store a large amount of liquidity in their underlying wallets

<br />

so Poly Network?

- has a master wallet for each underlying Layer-1 network (e.g., Bitcoin wallet, Ethereum wallet) and each of them stores a large amount of tokens
- runs smart contracts which contain users’ instructions (e.g., “Here is 1 BTC, can you exchange this into ETH?”)

<br />

I’ve already said it twice but again, those master wallets contain a huge fund, so they should be kept secure, right? BTW who then manages these master wallets? 

> “authenticator nodes” aka “keepers” manage the wallets. 

<br />

And there are 2 important Poly smart contracts that were responsible for setting and managing keepers... and well yes, the hacker exploited vulnerabilities in these two contracts: `EthCrossChainManager` and `EthCrossChainData` 

<br />

![Untitled](https://user-images.githubusercontent.com/99378245/162302585-eb593521-a421-4840-97dd-fca558ee795f.png)

<br />

#### [EthCrossChainData.sol](https://github.com/polynetwork/eth-contracts/blob/d16252b2b857eecf8e558bd3e1f3bb14cff30e9b/contracts/core/cross_chain_manager/data/EthCrossChainData.sol#L45)

- this is a highly privileged contract
- verifies that it’s being communicated with by the right accounts
- `bytes public ConKeepersPkBytes` stores Consensus book Keepers Public Key Bytes which is like a whitelist and addresses in this list are approved to tell the system about the state of transactions on different chains (so if the attacker can add his/her address to this list...)
- it does `onlyOwner` check 
- and this contract is owned by `EthCrossChainManager`...

<br />

🐸 - the whitelist feature in the contract: enabling crypto withdrawals to go only to address already designated in the address book 

<br />

#### [EthCrossChainManager.sol](https://github.com/polynetwork/eth-contracts/blob/d16252b2b857eecf8e558bd3e1f3bb14cff30e9b/contracts/core/cross_chain_manager/logic/EthCrossChainManager.sol)

- `verifyHeaderAndExecuteTx` function - verify a given relay chain message (with the related header and the associated Merkle proof) and execute the cross-chain transaction on the destination chain
    
    → which means someone from another chain can invoke this function and can execute somewhat arbitrary codes
    
    Here, what if the hacker sends a transaction to this contract like “add me to the whitelist” which means “register me as a keeper”? As this `Manager` contract owns the `Data` contract above, this contract actually has sufficient privileges to do that.
    

- One more thing, this `EthCrossChainManager` contract would not call any function within the target contract, but only the one that conformed to the “*function signature*”
    
    → So for example, if an attacker could call the function below (which is in `EthCrossChainData`) by finding **the string** that matches with the first 4 bytes of the keccak256 hash of the name and arguments of `putCurEpochConPubKeyBytes`, the attacker could register his or her address as a keeper (add her/his address to the known list of authorized node addresses) 
    
      ethers.utils.id ('putCurEpochConPubKeyBytes(bytes)').slice(0, 10)` == `ethers.utils.id (```the string```).slice(0, 10)
   

```solidity
// Store Consensus book Keepers Public Key Bytes
    function putCurEpochConPubKeyBytes(bytes memory curEpochPkBytes) public whenNotPaused onlyOwner returns (bool) {
        ConKeepersPkBytes = curEpochPkBytes;
        return true;
    }    
```


→ The attacker could find the string ('f1121318093(bytes,bytes,uint64)') and called the function through `EthCrossChainManager`, as `EthCrossChainManager` owns `EthCrossChainData`, they could pass the `onlyOwner` check and add their address to the keeper list 
    
→ and could drain a large number of tokens (around $610M) from Poly’s master wallets!!

<br />
<br />

Who are you, Mr. White Hat! Can we be friends someday

<br />
<br />
<br />


References:

[https://twitter.com/polynetwork2/status/1425073987164381196?lang=en](https://twitter.com/polynetwork2/status/1425073987164381196?lang=en)

[https://github.com/polynetwork/eth-contracts/blob/master/contracts/core/cross_chain_manager/data/EthCrossChainData.sol](https://github.com/polynetwork/eth-contracts/blob/master/contracts/core/cross_chain_manager/data/EthCrossChainData.sol)

[https://github.com/polynetwork/eth-contracts/blob/d16252b2b857eecf8e558bd3e1f3bb14cff30e9b/contracts/core/cross_chain_manager/logic/EthCrossChainManager.sol](https://github.com/polynetwork/eth-contracts/blob/d16252b2b857eecf8e558bd3e1f3bb14cff30e9b/contracts/core/cross_chain_manager/logic/EthCrossChainManager.sol)

[https://ethereum.stackexchange.com/questions/106943/how-does-poly-network-hack-work/107213#107213](https://ethereum.stackexchange.com/questions/106943/how-does-poly-network-hack-work/107213#107213)

[https://peckshield.medium.com/polynetwork-bug-review-and-patch-analysis-88bde8441297](https://peckshield.medium.com/polynetwork-bug-review-and-patch-analysis-88bde8441297)

[https://www.ft.com/content/ee970452-02de-4d66-84a1-f2b1345f57d1](https://www.ft.com/content/ee970452-02de-4d66-84a1-f2b1345f57d1)

[https://twitter.com/tomrobin/status/1429764560064552964](https://twitter.com/tomrobin/status/1429764560064552964)

[https://medium.com/coinmonks/ethereum-under-the-hood-part-7-blocks-c8a5f57cc356](https://medium.com/coinmonks/ethereum-under-the-hood-part-7-blocks-c8a5f57cc356)

[https://www.youtube.com/watch?v=-SMliFtoPn8](https://www.youtube.com/watch?v=-SMliFtoPn8)

[https://research.kudelskisecurity.com/2021/08/12/the-poly-network-hack-explained/](https://research.kudelskisecurity.com/2021/08/12/the-poly-network-hack-explained/)
