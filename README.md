Cohort : 2026<br>
Fellow : Olivier BARTHELEMY [Github](https://github.com/bartoli)<br>
Group Lead : Simon Njeru<br>
Mentor : Abubakar Sadiq Ismail<br>

*A portfolio of my journey during BTrust Fellowship of 2026*

## About me
French C++ developper for 20 years, I got interested in the Bitcoin ecosystem for 5 years. I have been a Lightning node manager for 4 years and developped open managment tools for LN nodes.<br>
This year, I have followed programs from Chaincode Labs, Bitshala, and BTrust, in order to learn more deeply about Bitcoin and the Ligning Network, and to be able to contributre more to the ecosystem.

## Bitcoin Core Contributions during the Fellowship
Proposed changes:
- [PR #36501](https://github.com/bitcoin/bitcoin/pull/36501) reduces memory usage by 8 bytes per block on the chain, saving a few MB of memory usage for most Bitcoin Core nodes
- [PR #36690](https://github.com/bitcoin/bitcoin/pull/36690) improves the Bech32 unit tests, in order to check the guarantees of error detection in Bech32 and Bech32m strings, for the standard size limit and the new size limit of silent payment addresses.
- [PR #36839](https://github.com/bitcoin/bitcoin/pull/36839) makes cmake warn that building in Visual Studio using cl.exe compiler is now not supported anymore

Other review of other maintainers' work:
- Test and ACK of [PR #35923](https://github.com/bitcoin/bitcoin/pull/35923) for accounting unbroadcast txids in memory usage
- Review of [PR #36835](https://github.com/bitcoin/bitcoin/pull/36835) to add a python test for scanning Silent Payments
- Discussion for specification of [Issue #36605](https://github.com/bitcoin/bitcoin/issues/36605) on how to bump fee on external transactions that might be Silent Payments

## Researched subject during the Fellowship
As a group of 3 (with Brndon Votteler and Chisom Nwike), we studied the subject of Silent Payments, and gave a presentation about how it solves the problems of address reuse.<br>
I particularly deep-dived into th eformulas that make Silent Payments possible (see Week 9 article below)

## Articles written during the fellowship
- [Week 1 - Why SegWit Was Critical to Making the Lightning Network Practical](W01_Segwit_LN.md)
- [Week 3 - Rene Pickardt's contributions to the Lightning Network](W03_rene_pickardt_contributions_to_LN.md)
- [Week 6 - Why Bitcoin Core Doesn't Even Trust Its Own Code](W06_Bitcoin_Defensive_Programming.md)
- [Week 9 - Silent Payments from scratch](W09_silent_payments_from_scratch.md)
- [Week 11 - Contributiong to Bitcoin](W11_Contributing_to_Bitcoin.md)

## Personal projects:
- [LNShortcut](https://amboss.space/node/02c521e5e73e40bad13fb589635755f674d6a159fd9f7b248d286e38c3a46f8683) : 
  My Lightning Node, operating since 2022. Aims to be a cheap altruistic routing node, and designed to select and tweak channels in order to require no rebalancing. Terminal Score 300
  
- [LNShortcut.xyz](https://lnshortcut.xyz): a service associated to my LN Node for analyzing the LN network graph, and suggesting interesting nodes to connect to for routing nodes.

- Wab3 / Echo : New projects currently in the early design and development stages.<br>
  **Wab3** is a C++ library of web3 primitives (Bitcoin, LN, Nostr,..), for native builds, WASM, or Android NDK<br>
  **Echo** is a platform for building dApps, whose goal is that the same C++ code builds an app that runs on desktop, WASM, or Android

## What's next
- Keep Contributing to Bitcoin Core
- Try to start contributing to the [CLBOSS](https://github.com/ksedgwic/clboss) project
- Work on a first functioning prototype for Echo
