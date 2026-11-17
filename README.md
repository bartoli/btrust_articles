A portfolio of my journey during BTrust Fellowship of 2026

## About me
French C++ developper for 20 years, I got interested in the Bitcoin ecosystem for 5 years. I have been a Lightning node manager for 4 years and developped open managment tools for LN nodes.
This year, i have followed programs from Chaincode Labs, Bitshala, and BTrust, In order to learn more deeply about Bitcoin and the Ligning Network, and to be able to contributre more to the ecosystem.

## Personal projects:
- [LNShortcut](https://amboss.space/node/02c521e5e73e40bad13fb589635755f674d6a159fd9f7b248d286e38c3a46f8683) : 
  My Lightning Node, operating since 2022. Aims to be a cheap altruistic routing node, and designed to select and tweak channels to require no rebalancing at all. Terminal Score 300
  
- [LNShortcut.xyz](https://lnshortcut.xyz): a service associated to my LN Node for analyzing the LN network graph, and suggesting interesting nodes to connect to for routing nodes.

- Wab3 / Echo : Current new project in the process of getting birth<br>
  **Wab3** is a C++ library of web3 primitives (Bitcoin, LN, Nostr,..), for native builds, WASM, or Android NDK<br>
  **Echo** is a platform for building dApps, whose goal is that the same C++ code builds an app that runs on desktop, WASM, or Android; with the particularity that multiple *echoes* of an app running on any of those platforms can combine into a complete app

## Articles written during the fellowship
- [Week 1 - Why SegWit Was Critical to Making the Lightning Network Practical](W01_Segwit_LN.md)
- [Week 3 - Rene Pickardt'scontributuons to the Lightning Network](W03_rene_pickard_contributions_to_LN.md)
- [Week 6 - Why Bitcoin Core Doesn't Even Trust Its Own Code](W06_Bitcoin_Defensive_Programming.md)
- [Week 9 - Silent Payments from scratch](W09_silent_payments_from_scratch.md)
- [Week 11 - Contributiong to Bitcoin](W11_Contributing_to_Bitcoin.md)

## Researched subject during the Fellowship
As a group of 3, we studied the subject of Silent Payments, and gave a presentation about how it solves the problems of address reuse.

## Bitcoin Core Contributions during the Fellowship
- [PR #36501](https://github.com/bitcoin/bitcoin/pull/36501) reduces memory usage by 8 bytes per block on the chain, saving a few MB of memory usage for most Bitcoin Core nodes
- [PR #36690](https://github.com/bitcoin/bitcoin/pull/36690) improves the Bech32 unhit tests, in order to check the guarantees of error detection in Bech32 and Bech32m strings, for the standard size limit and the new size limit of silent payment addresses.
- [PR #36839](https://github.com/bitcoin/bitcoin/pull/36839) makes cmake warn that building in Visual Studio using cl.exe compiler is now not supported anymore

## What's next
- Keep Contributing to Bitcoin Core
- Try to start contributing to the [CLBOSS](https://github.com/ksedgwic/clboss) project
- Work on a first functioning prototype for Echo
