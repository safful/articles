# Vyper Vulnerability Anaylsis

On 30 July, reports surfaced of vulnerabilities in Vyper versions 0.2.15, 0.2.16 and 0.3.0 which left many pools on Curve vulnerable to a reentrancy attack. This exploit was possible due to a reentrancy vulnerability which allowed the attackers to call the add liquidity function during the remove liquidity process. In total $69.3 million was affected, with $16.7 million returned by white hat hackers. However, this still means that approximately $52 million was stolen, making it the largest reentrancy attack so far in 2023.

## Event Summary

On July 30, 2023 Vyper, a contract-oriented programming language designed for the Ethereum Virtual Machine (EVM), announced that Vyper compiler versions 0.2.15, 0.2.16 and 0.3.0 were vulnerable to malfunctioning reentrancy locks. As a result, several DeFi projects were affected by the vulnerability resulting in a total loss of $52 million. Upon investigation, Safful found the reentrancy attack targeting the pETH-ETH-f pool.
Safful has identified six wallets involved in the incident. The first wallet (0x172) failed to exploit the vulnerability in block 17806056. The original exploiter withdrew 0.1 ETH from Tornado Cash and proceeded to create the attack contract. However, a front runner (0x6Ec21) paid more gas and was able to execute their transaction first to acquire approximately 6.1k WETH worth over $11.4 million.
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1691882255232-6eea29a0-c4b5-4d14-89ff-e874a9911c4e.png#averageHue=%23fefcfc&clientId=u7de84afb-6a02-4&from=paste&id=ubc2a9865&originHeight=952&originWidth=2502&originalType=url&ratio=2&rotation=0&showTitle=false&size=631589&status=done&style=none&taskId=u87af7777-ba87-4096-917b-6307f99247a&title=) Image: Failed exploit transaction that was front run by a MEV bot. Source: Etherscan
The vulnerability led to further losses, with EOA 0xDCe5d acquiring approximately $21 million worth of assets. A break down of the involved wallets is shown below.
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1691882328932-46b385cd-aae4-4417-ad75-6fc4bdf14b8d.png#averageHue=%23efefef&clientId=u7de84afb-6a02-4&from=paste&height=514&id=u6535232c&originHeight=1028&originWidth=2736&originalType=binary&ratio=2&rotation=0&showTitle=false&size=386004&status=done&style=none&taskId=u8ddba04d-447a-4271-aaf2-85e95717e2e&title=&width=1368)Table: Assets transferred with associated wallets.
In total, six projects were affected, approximately $69.3 million was taken and $16.7 million has been returned, resulting in a total loss of $52 million.

## What is Vyper?

Vyper is a contract-oriented, pythonic programming language for the Ethereum Virtual Machine (EVM). Beta versions of Vyper have been around since 2017, but its first non-beta release came with version 0.2.1 in July 2020.
Solidity is the dominant language within the Ethereum ecosystem as it has existed much longer than Vyper, which has ultimately led many community members to create tools that operate specifically with Solidity. According to [DeFiLlama](https://defillama.com/languages), Vyper smart contracts make up $2.17 billion of the roughly $70 billion worth of Total Value Locked (TVL) in DeFi protocols. Solidity, on the other hand makes up the vast majority at $67.49 billion.
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1691882255104-64abf8e9-40f9-4099-87a9-dda110e93bde.png#averageHue=%231d1d1c&clientId=u7de84afb-6a02-4&from=paste&id=ucead3dbd&originHeight=1300&originWidth=2402&originalType=url&ratio=2&rotation=0&showTitle=false&size=399834&status=done&style=none&taskId=u8fc597c3-75cf-4cf7-80cd-29948c4bc21&title=) Image: TVL by programming languages. Source: [DeFiLlama](https://defillama.com/languages)
As of 10 May, 2023 Vyper’s usage dominance was down to 6.27% from its high of 30% in August 2020. Despite the TVL dominance of Vyper being significantly lower than Solidity, this incident still led to $62 million being impacted.
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1691882255145-956f55c3-8910-4b62-949b-6d58f2afe9e4.png#averageHue=%23b1c651&clientId=u7de84afb-6a02-4&from=paste&id=ueffbc187&originHeight=1392&originWidth=2488&originalType=url&ratio=2&rotation=0&showTitle=false&size=387263&status=done&style=none&taskId=ubbbc80b4-92c2-4011-a7d5-9e0da52520d&title=) Image: TVL dominance by programming languages. Source: [DeFiLlama](https://defillama.com/languages)

## What is a Compiler Version?

A compiler version refers to a specific release of a programming language's compiler, which translates human-readable source code into machine-readable code. Vulnerabilities were found in versions 0.2.15, 0.2.16, and 0.3.0 of Vyper, resulting in a reentrancy attack that impacted several DeFi projects, causing a total loss of $52 million. Compiler versions are periodically updated to introduce features, fix bugs, and enhance security. The Vyper language does not currently provide a white hack bug bounty program.

### Versions 0.2.15 - 0.3.0

The earliest vulnerable version of Vyper, v0.2.15, was released on 23 July, 2021. However, by December of the same year version 0.3.1 was released at which point the previous vulnerability was no longer present.

## Timeline of Events

The incident began at 13:10 +UTC with [the initial exploiter’s transaction](https://etherscan.io/tx/0xb5d91f1e0afc96a52f8c6c28eae405eda7fcc5d34d6d03bdd8b16bd58089e939) targeting the JPEG'd pool on Curve failing due to a front run transaction.
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1691882255437-a7693c89-0df0-477e-b3c3-114b904c76b9.png#averageHue=%23f1f2f4&clientId=u7de84afb-6a02-4&from=paste&id=ub437f885&originHeight=268&originWidth=1544&originalType=url&ratio=2&rotation=0&showTitle=false&size=48477&status=done&style=none&taskId=u77367666-5940-4d4c-8fa9-6466264b69f&title=)
[JPEG’d acknowledged](https://twitter.com/JPEGd_69/status/1685655792274341888?s=20) that the pETH-ETH Curve pool had been exploited at 14:17 UTC.
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1691882255596-c865d388-debe-4421-9882-c584aa5b95d7.png#averageHue=%23000000&clientId=u7de84afb-6a02-4&from=paste&id=ude6440af&originHeight=348&originWidth=599&originalType=url&ratio=2&rotation=0&showTitle=false&size=63872&status=done&style=none&taskId=u9db17563-b2f5-42cb-b8ab-4fc3cf8b882&title=)
[Vyper then announced](https://twitter.com/vyperlang/status/1685692973051498497) that versions 0.2.15, 0.2.16 and 0.3.0 contained a malfunctioning reentrancy lock.
Following Vyper’s tweet, Metronome and Alchemix were impacted.
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1691882255607-cdbcca5b-fc69-4a04-b23e-553e1cc614fb.png#averageHue=%23f1f2f4&clientId=u7de84afb-6a02-4&from=paste&id=uffa6abaa&originHeight=464&originWidth=1546&originalType=url&ratio=2&rotation=0&showTitle=false&size=95527&status=done&style=none&taskId=u4879dda9-4850-4b13-a139-5d2f7079931&title=)
Metronome DAO [then announced](https://twitter.com/MetronomeDAO/status/1685723675188772869?s=20):
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1691882255599-a64c5e4d-8915-47b0-9dbf-4351ea7eb584.png#averageHue=%23000000&clientId=u7de84afb-6a02-4&from=paste&id=u4df20bea&originHeight=245&originWidth=605&originalType=url&ratio=2&rotation=0&showTitle=false&size=43102&status=done&style=none&taskId=uc9338609-9ca3-49c5-85c5-1330767bea9&title=)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1691882255688-3eaa6662-daca-4f66-a59d-b8fb453c04a7.png#averageHue=%23000000&clientId=u7de84afb-6a02-4&from=paste&id=u7c153661&originHeight=456&originWidth=599&originalType=url&ratio=2&rotation=0&showTitle=false&size=97150&status=done&style=none&taskId=u470096f1-a1ca-49d5-8591-8f3b3954665&title=)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1691882255891-b1e1e1e2-0f6a-4e6f-a7ae-7ada463f44a3.png#averageHue=%23eff1f3&clientId=u7de84afb-6a02-4&from=paste&id=u9ad339d2&originHeight=306&originWidth=1550&originalType=url&ratio=2&rotation=0&showTitle=false&size=66326&status=done&style=none&taskId=u1e062ea5-4e8a-4dee-b96b-db655faf782&title=)
At 8:23 PM UTC, Curve Finance made an announcement on Discord stating that remaining pools are safe and unaffected by the Vyper bug.
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1691882256093-7d0889fe-aebf-4a84-a59d-5a78a0d6f8de.png#averageHue=%23373b41&clientId=u7de84afb-6a02-4&from=paste&id=u014be22d&originHeight=1092&originWidth=2338&originalType=url&ratio=2&rotation=0&showTitle=false&size=546171&status=done&style=none&taskId=ued2bbcab-0f31-438e-ad15-9408afe4b0e&title=)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1691882255892-d1437ba1-3460-400b-8ba5-4571993bff94.png#averageHue=%23f0f2f4&clientId=u7de84afb-6a02-4&from=paste&id=u9942598e&originHeight=354&originWidth=1546&originalType=url&ratio=2&rotation=0&showTitle=false&size=73860&status=done&style=none&taskId=u26c87602-ad71-40d7-9c5b-c9d506c2854&title=)
On 31 July, [Curve Finance announced on Twitter](https://twitter.com/CurveFinance/status/1685925429041917952) that there was a potential for a pool on Arbitrum to be affected, however there is no profitable exploit for malicious actors to execute, meaning that it's unlikely the pool will be attacked. Safful has not detected any other attack exploiting the Vyper bug.
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1691882255954-dd2aa5a4-f24e-4635-81e2-db3b79204c82.png#averageHue=%23000000&clientId=u7de84afb-6a02-4&from=paste&id=u1536a5ca&originHeight=363&originWidth=602&originalType=url&ratio=2&rotation=0&showTitle=false&size=53182&status=done&style=none&taskId=uabcda05c-f01a-4753-883e-a1c660e9d73&title=)
On 4th August, [Alchemix announced on twitter](https://twitter.com/AlchemixFi/status/1687531017161105408?s=20) that the Curve Finance exploiter had returned a total of 4,819 $alETH and 2259 $ETH so far. That they are also looking forward for the remaining funds to be returned.
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1691882255979-4efbc235-cd15-4afd-a2e4-b10a4c097281.png#averageHue=%23fdfdfd&clientId=u7de84afb-6a02-4&from=paste&id=ud9306945&originHeight=199&originWidth=395&originalType=url&ratio=2&rotation=0&showTitle=false&size=29152&status=done&style=none&taskId=uf1cfa13e-5738-440f-9353-9de48d99c9f&title=)

## Attack Flow

This example is based on the [transaction](https://etherscan.io/tx/0xa84aa065ce61dbb1eb50ab6ae67fc31a9da50dd2c74eefd561661bfce2f1620c) that targeted JPEG’d.
Attacker: [0x6ec21d1868743a44318c3c259a6d4953f9978538](https://etherscan.io/address/0x6ec21d1868743a44318c3c259a6d4953f9978538)
Attack Contract: [0x466b85b49ec0c5c1eb402d5ea3c4b88864ea0f04#code](https://etherscan.io/address/0x466b85b49ec0c5c1eb402d5ea3c4b88864ea0f04#code)

1. The attacker started by borrowing 80,000 WETH (~$149,371,300) from Balancer: Vault.
2. The attacker then unwrapped the WETH and called pETH-ETH-f.add_liquidity() and added 40,000 ETH (~$74,685,650) into the [pETH-ETH-f pool](https://etherscan.io/address/0x9848482da3ee3076165ce6497eda906e66bb85c5). In return the attacker received 32,431 pETH (pETH-ETH-f).
3. The attacker called remove_liquidity() to remove the liquidity added in step 2. 3,740 pETH and 34,316 ETH was transferred to the attack contract which triggered the fallback() function, giving control to the attacker. In the fallback function, the attacker added another 40,000 ETH to the pETH-ETH-f pool and received 82,182 pETH.
4. The attacker called remove_liquidity() again and removed 10,272 pETH, receiving 47,506 ETH and 1,184 pETH. The attacker then exchanged 4,924 pETH for 4,285 ETH in the pETH-ETH-f pool.

In total, the attacker obtained 34,316 ETH from step 3 and 47,506 and 4,285 from step 4 for a total of 86,107 ETH. After repaying the 80,000 ETH flash loan the attacker was left with 6,107 ETH ($11,395,506).

## Vulnerability

This exploit was possible due to a reentrancy vulnerability which allowed the attacker to call the add liquidity function during the remove liquidity process.
Although the functions should have been protected by the @nonreentrant('lock'), testing of the add_liquidty() and remove_liquidity() functions has proven it did not prevent reentrancy.
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1691882256392-f7f01861-7816-4728-80b8-ec3814c2c31f.png#averageHue=%23042654&clientId=u7de84afb-6a02-4&from=paste&id=uec19e8b0&originHeight=1600&originWidth=1403&originalType=url&ratio=2&rotation=0&showTitle=false&size=771916&status=done&style=none&taskId=u8f7bd047-fde6-49ba-b9d1-c0dd3281f6e&title=) Image: Vyper_contract for Curve.fi Factory Pool Source: [Etherscan](https://etherscan.io/address/0x9848482da3ee3076165ce6497eda906e66bb85c5#code)
Following the exploits on JPEG’d, Metronome, and Alchemix, Vyper confirmed that versions v0.2.15, v0.2.16 and v0.3.0 were vulnerable to reentrancy protection failing.
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1691882256167-ee64eea3-85ba-4501-8596-83371fd9d9b1.png#averageHue=%23000000&clientId=u7de84afb-6a02-4&from=paste&id=uef7a02ed&originHeight=188&originWidth=587&originalType=url&ratio=2&rotation=0&showTitle=false&size=20005&status=done&style=none&taskId=uad7dfc62-9dfe-4291-8ed7-a21bc066ec6&title=)

### Remediation

Projects that have been built using the vulnerable Vyper versions should reach out to Vyper, who can assist in mitigations. If possible, projects should upgrade to the latest version of Vyper which does not contain this bug.

## Conclusion

The incident affecting Vyper is the largest reentrancy exploit that we've detected in 2023. It accounts for 78.6% of such instances in terms of funds lost. Unfortunately, the two largest reentrancy incidents this year have affected contracts written in Vyper, albeit [different vulnerabilities](https://www.dlnews.com/articles/defi/conic-finance-suffers-exploit-similar-to-the-dao-hack/).
2023 has now seen over $66 million lost due to reentrancy attacks across all chains. This is approximately $4 million more than all of 2020 and just $1 million behind the amount lost in 2021. Notably, the total for 2023 also represents a 259.45% increase the amount lost to reentrancy attacks in 2022.
Losses to reentrancy attacks had been declining due to an increased understanding of security practices. However, July has now seen a total of four reentrancy attacks. Total losses caused by reentrancy attacks during the month of July have now reached $58.4 million.
