# How Segwit was the last building blockt to enable the Lightning Network
In January of 2016, the ([Lightning Network Paper](https://lightning.network/lightning-network-paper.pdf) was published by Joseph Poon and Thaddeus Dryja.

Later in 2016, Bitcoin core activated BIP112 and BIP112 (OP_CHECKSEQUENCEVERIFY, and the ability to have relative lock times in utxo spending conditions). This made the implementation. This allowed Lightning to have a more precise system for timelocks on revocation transactions,allowing them to have a delay relative to the moment the transaction was published onchain.

Everything light seemed to be green for the actual implementation of lightning Network.<br>
Everything except the transaction malleability issue.

## What is the transaction malleability issue?
### How is the txid of a transaction determined
### How transactions signatures work (before SegWit)
### The problem
### Real life consequences

## How SegWit fixes this
