# Silent Payments from scratch

Silent payments will enable one to share a single static address to receive payments, and all payments will not be linkable to each other.

In this article, let's concentrate on the math that make this possible. 
We will not start from the final equations, and explain what each part does.
We will try to start from the 'need', and build up what is needed until the final formula is reached and understood.
this way, can 'not trust, verify!'

The main goal we want to achieve is that given a same 'destination address' (in quotes, because it is not strictly a bitcoin adress), each payment will be go to an actual bitcoin adress that is unique each time.
The second goal is that the payer does not have to talk to the receiver to find that destination address. And that the receiver can find that this adress corresponds to a payment to them. Finally, then only the payer and the receiver can link this adress to a payment between them; noone else.

Some conventions:
A : big single letters are public keys (or points on the elliptic curve)
a : small single letters are private keys
Alice is the payer and has key pairs that are variations of a/A
Bob is the receiver and has key pairs that are variations of b/B
|| marks concatenation
. is a multiplication.

## The silent payment address
Bob gives a receive address to Alice. Silent payments address have the form spxxxxxxxxxxx.

What does it contain?
To receive silent payments, bob will need two things : 
- A secret to share with people wanting to send him money, so they both can compute the same destination adresses
- A secret allowing him only to spend the output address that has been paid.µ
So Bob will compute on his side two key pairs :
- one to *scan* for adresses that correspond to payments to him : a_scan / A_scan
- one to *spend* those adresses : a_spend / A_spend
The silent payment address is some special encoding that actually stores the public values of B_scan and B_spend. ANd bob keeps the associated private values

## The shared secret
We talked about how Alice and bob can't interact to receive a new payment address. There must be some calculation of a formula that only Alice and Bob can compute the same result of, based on data only Alice and Bob can know before having to exchange anything extra. What does each know from the other one? Their public key. What do only them know? Each onw know their own private key.

ALso, if bob is giging his public keys as data in his public silent payment address, how is a secret value coming from him?

Let's tinker..
Only Alice has all the values of (1) *A = a.G*
Only Bob has all the values of (2) *B = b.G*
Let's rearrange (1) : *G = A/a*
Let's put that in (2) : *B = b*(A/a)
Then we can obtain *a.B = b.A*.

That's it! We have a formula that Alice can compute one side, and Bob can Compute the other side, to the same value! ANd both sides depend on Alice or Bob's secret key, so we know only one person can compute them.
Here, a and A are Alice's keys used to pay bob (the keys that own the input she wants to spend).
And B and b are Bob's scan key. Which we named B_scan and b_scan earlier.
That shared secret will be named S. A will compute a.B_scan to get it, and bob will compute A.b_scan.

## The base formula
We want to be able to calculate two things : 
P : the pubkey Alice needs to send to. Alice and Bob, and only them, must be able to compute them<br>
p : the associated private key to spend ouptuts sent to P. Only Bob must be able compute it.<br>
Again, both must satisfy the equation *P = p.G*, so ratio between them must be the factor G.<br>
p must depend on b_spend only bob knows, so P should depend on B_spend, which is b_spend.G<br>

- p = b_scan
- P = B_scan * G

P must also depend on the shared secret S. Let's just add it, keeping a ratio of G between both keys
 - p = b_scan + S
 - P = B_scan + (S.G)

## Now let's generate unique adresses per payment!
We have a base formula that allows only Alice and Bob to respectively decide or detect that it a silent payment from Alice to Bob. And that only Bob can compute the secret key to spend it's output.

We can see that depending on who is paying them, the (a,A) values will be different, thus generating different destination adresses!

But wait, what if Alice wants to pay multiple times ? oops, P will be the same if she spends outputs owned by the same pubkey. Which means some mform of tracing can be done. 
Actually, there are multiple things that might make multiple payments generate the same P. We will see a few, and how we can find a better calculation for P

### Adding some variability from Alice's side
We just saw that 2 payments transactions using inputs owned by the same pubkey might go to the same destination address.

#### First, without changing the formula, what can Alice do?
She could spend other outputs (owned by her other adresses) for each payment payment. Or spend a different combination of outputs. For example, to also obfuscate the actual number of person paying, if she uses outputs owned by unique adresses for each input of the transaction. And actually, the coin selection algorithm of her wallet might choose to use multiple outputs of the payment. So we have to account for this in the formula.

For silent payments, a or A was used in the formula to calculate S earlier.
When there is a single input in the transaction, a and A are directly the private and public keys of theadress owning the utxo that is spent.
When there are multiple inputs, we could do a simple thing and just take (a,A) from the first spent utxo, and be done with it. But we might as well add some variability to the result by acounting for all inputs.

We must stick with some final a and A values that keep the ratio G. We know that any key pair of ALices adddresses, (A1,a1),(A2,a2),..,(An,an) all also have this factor. So we can simply aggregate them by adding them : 
- A = A1+A2+..+An = G.(a1+a2+..+an)
- a = (a1+a2+..+an)
- S = (A1+A2+..+An).b_scan = (a1+a2+..+an).B_scan

What if multiple outputs being spent are owned by the same pubkey?
We aggregate them multiple times. That added variability is cheap to obtain, just another additon per input in the transaction.

Why not use a more complex aggregation that also accounts for the order of the pubkeys, so multiple sets of keys spending in multiple ordzer would generate different P addresses?

- this is not a cheap calculation, and we will need to do it a LOT when scanning for silent payment outputs
- we still are not really fixing the issue that if that set of outputs is used again, they would pay to the same destination address; merley reducing it. So it's really not worth it.

*On multisig payments:*
Here, we can see one of the reason the chosen formulas don't allow multisig payments (inputs from multiple people) : to compute S on the side of all the Alices paying, each Alice would need to know the private keys woning the outputs the other Alices are spending!

#### The real Variability on Alice's side
So we did cheap tricks to reduce the issue when Alice is using multiple utxos to pay, but did not really solve the problem.

We really need to add some variable in the equation that depends on alice's spent output and is unique for each payment.
The pubkey owning Alice's utxos might not be, but the outpoint (txid+output number) of those outputs is!
And since the outpoint values will be unique, no need to include the outpoint of utxo being spent. A formula selects the outpoint from one of the inputs of the paying transaction, and transforms it into what we will call *input_hash* 








