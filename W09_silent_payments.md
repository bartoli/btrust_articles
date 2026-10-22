# Silent Payments from scratch

Silent Payments (BIP-352) will enable one to share a single static address to receive payments, and all payments will not be linkable to each other.

In this article, let's concentrate on the math that makes this possible. 
We will not start from the final equations and explain what each part does.
We will try to start from the **need**, and build up what is needed until the final formula is reached and understood.
This way:  **don't trust, verify!**

The main goal we want to achieve is that when someone wants to receive payments, given the same 'destination address' (in quotes, because it is not strictly a Bitcoin address), each payment will go to an actual Bitcoin address that is unique each time.<br>
The second goal is that the payer does not have to talk to the receiver to find that destination address.<br>
And that the receiver can find that this address corresponds to a payment to them. Finally, only the payer and the receiver can link this address to a payment between them; no one else.

Some conventions:
- **A** : big single letters are public keys (or points on the elliptic curve)
- **a** : small single letters are private keys
- Alice is the payer and has key pairs that are variations of **(a, A)**
- Bob is the receiver and has key pairs that are variations of **(b, B)**
- **||** marks concatenation
- **.** is multiplication.

## The silent payment address
Bob gives a receive address to Alice. Silent payment addresses have the form spxxxxxxxxxxx.

What does it contain?<br>
To receive silent payments, Bob will need two things : 
- A secret to share with people wanting to send him money, so they both can compute the same destination addresses
- A second secret allowing him only to spend the output address that has been paid using the two secrets.

So Bob will compute on his side two key pairs :
- one to *scan* for addresses that correspond to payments to him : **(b_scan, B_scan)**
- one to *spend* those addresses : **(b_spend, B_spend)**

The silent payment address is some special encoding that actually stores the public values of **B_scan** and **B_spend**. And Bob keeps the associated private values.

## The shared secret
We talked about how Alice and Bob can't interact to receive a new payment address. There must be some calculation of a formula that only Alice and Bob can compute the same result of, based on data only Alice and Bob can know before having to exchange anything extra. What does each know from the other one? Their public key. What do only they know? Each one knows their own private key.

Also, if Bob is giving his public key as data in his public silent payment address, how is a secret value coming from him?

Let's tinker..
Only Alice has all the values of (1) *A = a.G*
Only Bob has all the values of (2) *B = b.G*
Let's rearrange (1) : *G = A/a*
Let's put that in (2) : *B = b*(A/a)
Then we can obtain *a.B = b.A*.

That's it! We have a formula that Alice can compute one side, and Bob can compute the other side, to the same value! And both sides depend on Alice or Bob's secret key, so we know only one person can compute them.
Here, a and A are Alice's keys used to pay Bob (the keys that own the input she wants to spend).
And B and b are Bob's scan key, which we named B_scan and b_scan earlier.
That shared secret will be named S. A will compute a.B_scan to get it, and Bob will compute A.b_scan.

## The base formula
We want to be able to calculate two things : 
P : the public key Alice needs to send to. Alice and Bob, and only them, must be able to compute them. <br>
p : the associated private key to spend outputs sent to P. Only Bob must be able to compute it.<br>
Again, both must satisfy the equation *P = p.G*.<br>
p must depend on b_spend only bob knows, so P should depend on B_spend, which is b_spend.G<br>

- p = b_spend
- P = b_spend * G = B_spend

P must also depend on the shared secret S. Let's just add it, keeping a ratio of G between both keys
 - p = b_spend + S
 - P = B_spend + (S.G)

## Now let's generate unique addresses per payment!
We have a base formula that allows only Alice and Bob to respectively decide or detect that it is a silent payment from Alice to Bob. And that only Bob can compute the secret key to spend its output.

We can see that depending on who is paying, the (a, A) values will be different, thus generating different destination addresses!

But wait, what if Alice wants to pay multiple times? Oops, P will be the same if she spends outputs owned by the same pubkey. Which means some form of tracing can be done. 
Actually, there are multiple things that might make multiple payments generate the same P. We will see a few, and how we can find a better calculation for P

### Adding some variability from Alice's side
We just saw that 2 payment transactions using inputs owned by the same pubkey might go to the same destination address.

#### First, without changing the formula, what can Alice do?
She could spend other outputs (owned by her other addresses) for each payment. Or spend a different combination of outputs. For example, to also obfuscate the actual number of people paying, if she uses outputs owned by unique addresses for each input of the transaction. And actually, the coin selection algorithm of her wallet might choose multiple UTXOs as inputs of the payment. So we have to account for this in the formula.

For silent payments, a or A is used in the formula to calculate S earlier.
When there is a single input in the transaction, a and A are directly the private and public keys of the address owning the UTXO that is spent.
When there are multiple inputs, we could do a simple thing and just take (a, A) from the first spent UTXO, and be done with it. But we might as well add some variability to the result by accounting for all inputs.

We must end up with final a and A values that keep the ratio G. We know that any key pair of Alice's addresses, (A1, a1), (A2, a2), ..., (An, an), all also have this factor. So we can simply aggregate them by adding them : 
- A = A1+A2+..+An = G.(a1+a2+..+an)
- a = (a1+a2+..+an)
- S = (A1+A2+..+An).b_scan = (a1+a2+..+an).B_scan

What if multiple outputs being spent are owned by the same pubkey?
We aggregate them multiple times. That added variability is cheap to obtain, just another addition per input in the transaction.

Why not use a more complex aggregation that also accounts for the order of the pubkeys, so multiple sets of keys spending in multiple orders would generate different P addresses?

- this is not a cheap calculation, and we will need to do it a LOT when scanning for silent payment outputs
- we still are not really fixing the issue that if that set of outputs is used again, they would pay to the same destination address; merely reducing it. So it's really not worth it.

*On multisig payments:*
Here, we can see one of the reasons silent payments are more complicated to implement for payments with inputs from multiple people : to compute S on the side of all the ones paying, each peer paying a part would need to know the private keys owning the outputs the other payers are spending in order to compute the aggregated A value!

#### The real Variability on Alice's side
So we did cheap tricks to reduce the issue when Alice is using multiple UTXOs to pay, but did not really solve the problem.

We really need to add some variable in the equation that depends on alice's spent output and is unique for each payment.
The pubkey owning Alice's UTXOs might not be, but the outpoint (txid+output number) of those outputs is!
And since the outpoint values will be unique, no need to include all outpoints of the UTXOs being spent. A formula selects the outpoint from one of the inputs of the paying transaction, and transforms it into what we will call *input_hash*

The formulas become:

- p = b_spend + (S.input_hash)
- P = B_spend + (S.input_hash).G 

### More variability from the output side

Wait what? Did we not just make sure any payment would send to a different address because no two transactions will spend exactly the same combination of UTXOs? Yes, we did. For any two transactions, payments to Bob's SP address will send to a different address. But what if the transactions pay to Bob's SP address twice? Then, exactly the same UTXOs would be sent to the same destination address.
And if one is using silent payments to avoid linkability, which whole point was that each received output is sent to a different address, this defeats the purpose!
We want that even in the same transactions, spending the same UTXOs, if two outputs are sent to the same sp address, then this will result in money being sent to two different addresses. That will hide a bit more about which output goes to whom, and what the actual amount being exchanged is.
For this, we include a last variable in the formula: k. k will be a counter, starting at 0 for the first output going to Bob's SP address in the transaction. Then, for each new output of the transaction going to Bob's SP address, k is incremented.

- p = b_spend + H(S.input_hash || k)
- P = B_spend + H(S.input_hash || k).G

And finally, that's it! Every payment going to Bob's SP address will go to a different Taproot output address; only Alice and Bob will be able to know that they correspond to a payment from Alice to Bob's SP address, and only Bob will be able to spend them!








