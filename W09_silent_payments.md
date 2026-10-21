Silent payments will enable one to share a single static address to receive payments, and all payments will not be linkable to each other.

In this article, let's concentrate on the math that make this possible. We will try to start from the 'need', and build up until the final formulas.

The main goal we want to achieve is that given a same 'destination address' (in quotes, because it is not strictly a bitcoin adress), each payment will be go to an actual bitcoin adress that is unique each time.
The second goal is that the payer does not have to talk to the receiver to find that destination address. And that the receiver can find that this adress corresponds to a payment to them. Finally, then only the payer and the receiver can link this adress to a payment between them; noone else.

Some conventions:
A : big single letters are public keys (or points on the elliptic curve)
a : small single letters are private keys
Alice is the payer and has key pairs that are variations of a/A
Bob is the receiver and has key pairs that are variations of b/B
|| marks concatenation
. is a multiplication.

The silent payment address
Bob gives a receive address to Alice. Silent payments address have the form sp.........
What does it contain?
To receive silent payments, bob will need two things : 
- A secret to share with people wanting to send him money, so they both can compute the same destination adresses
- A secret allowing him only to spend the output address that has been paid.µ
So Bob will compute on his side two key pairs :
- one to *scan* for adresses that correspond to payments to him : a_spend / A_spend
- one to *spend* those adresses

