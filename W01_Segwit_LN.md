# How Segwit was the last building block to enable the Lightning Network
In January of 2016, the [Lightning Network Paper](https://lightning.network/lightning-network-paper.pdf) was published by Joseph Poon and Thaddeus Dryja.

Later in 2016, Bitcoin core activated BIP112 and BIP68 (OP_CHECKSEQUENCEVERIFY, and the ability to have relative lock times in utxo spending conditions). This made the implementation. This allowed Lightning to have a more precise system for timelocks on revocation transactions, allowing them to have a delay relative to the moment the transaction was published onchain, to better control the time t push penalty transactions).

Every light seemed to be green for the actual implementation of lightning Network.<br>
Everything except the transaction malleability issue.

## What is the transaction malleability issue?
Let's first understand some fundamentals
### How is the txid of a transaction determined
To make it simple, the txid (transaction identifier), is the hash of all the transaction's data.
All data, meaning even signatures that allow to spend the inputs are serialized and taken into account for the transaction identifier.

We take all fields from the transaction (version, inputs, outputs,...), serialized them as a byte array, and perform a SHA256 hash of the data twice. The result is a 256-bit/32-byte value, which printed as a hex stings of 64 characters will be used as the txid

### How transactions signatures work (before SegWit)
I will not fully explain how signing tranctions work here, but the properties that help uderstand the problem.

Using ECDSA, a signature basically consists in 2 integers r and s, noted (r,s)

For ECDSA (on secp256k1), it can be proved that, if (r,s) is a valid signature, then (r, -s mod n) is also a valid signature (n is the scep256k1 curve order value)

### The problem
As you probably know, noone can sign a transaction for someone else without knowing their private key. But, as we just saw, they can take any signed transaction in the mempool, and have an easy way to calculate another valid transaction from it.
- If they take a published transaction, it is easy to extract (r,s) from the signature field of the transaction. 
- they can as easily compute (r, n-s), which will be another valid signature for the same message.
- Then reconstruct a full transaction from it.
- As the txid depends on all transaction data (including the new signature value), the txid for this transaction now has a different value
- They can now publish another transaction, with another id, that spends the same inputs to the same outputs.

### Real life consequences
One could think : what is the problem? Bitcoin solves the double spend problem, so only one of the transactions can end up in the mempool, the the wanted inputs will be sent to the wanted outputs eventually regardless of which transaction is confirmed.

The simplest example to understand is some retail service that sells something for bitcoin. They will tell the buyer to pay them to some bitcoin address. The buyer will send the seller the trénsaction id to monitor the success of the payment. They agree that it is considered paid when that transaction has N confirmations.

Imagine the buyer, as soon as he sees that transaction, actually publishes the 'malleated' transaction equivalent, and tries to make it confirmed first (for example by also publishing a transaction that uses the utxo they own with a high fee).
The wrong transaction will get confirmed first. The buyer might only notice that his transaction failed, but not notice that some other transaction still sent his satoshis to the sender address. At that point, they might not even have a possibility to see the original transaction data since it was removed as invalid from mempools. the seller can now say : your payment did not work, send it again. ANd get paid twice!

Or it can be done in reverse, if some service offers withdrawals. The recipient gives a destination address. The service sends them the withrdrawal tx. The recipient malleates the transaction and makes it confirmed first. they will receive the funds, but the service will thing the withdrawl fail and issue another transaction. the recipient has now been paid twice

### For the Lightning Network
For the Lightning network, this becomes a big problem : channels rely on a funding transaction id : the multisig transaction between the 2 peers that cooperate to create that channel.
Lightning relies on the fact that at any point from the channel opening confirmation, both peers can unilaterally close the channel, because for each state update of the channel, they atomically exchanged closing transactions (that spend the funding txid), that would give each peer their share of the value they have in the channel.
Now, if someone just malleates the funding transaction, this transaction might get confirmed, and the LN nodes lose track of the channel and both peers can't recover the funds.
To do so, they have to the manage to collaborate to manually re-craft a closing transaction. But the whole point of Lightning's trustless construction is that either party should be able to enforce the latest state unilaterally

## How SegWit fixes this
We saw that the malleability issue comme from the dissimetry or serialized data used for the signature, and for the transaction ID. 
- The signature cannot by definition depend on the value of the signature (itself), but the txid does, since it includes all transaction data. - We can easily find another signature for an aleadt signed tranction
-> transaction can be malleated.

Here comes segwit (Segregated Witness)

One of the points of SegWit is that signatures are needed to validate transactions, but do not really need to be in the UTXO set for later operations.
Before segwit, about 60% of the blockchain data consisted of signatures!
With SegWit, signature stuff is now moved into a new block of data called **witness data**.

Because SegWit was a soft fork, this new data area is after the legacy transaction data, and will only be transmitted to nodes with SegWit enabled.

Segwit also defines a new consensus on serialization of segwit transaction to define the txid : it takes all previous data areas, and NOT the new withess area.

So now the transaction id does not depend anymore on the transaction's signatures, and this kind of transaction malleability attack can't work anymore (as long as the output being spent is a segwit one and not a legacy one)!

I say **this kind** of transactin malleability, because there are other ways transaction can be malleated, for example in multisig transactions construction. But that's a story for another time...

