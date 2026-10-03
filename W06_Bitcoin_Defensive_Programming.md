# Why Bitcoin Core Doesn't Even Trust Its Own Code
## Why is this check here?
Since I will have to manage keys/identities in my own project, I was looking at how they are implemented in
[bitcoin core repository](https://github.com/bitcoin/bitcoin/).
I stumbled upon this method : 
```
bool CKey::VerifyPubKey(const CPubKey& pubkey) const {
    if (pubkey.IsCompressed() != fCompressed) {
        return false;
    }
    unsigned char rnd[8];
    std::string str = "Bitcoin key verification\n";
    GetRandBytes(rnd);
    uint256 hash{Hash(str, rnd)};
    std::vector<unsigned char> vchSig;
    Sign(hash, vchSig);
    return pubkey.Verify(hash, vchSig);
}
```
The *CKey* class manages a bitcoin private key. Its' *keydata* member stores the private key.
And various operations on that private key are provided by the methods of the class.

For example, the method *GetPubKey()* returns the public key associated with the secret value in *keydata*.
the result of GetPubKey() is basically the multiplication of the point G by the secret. G is not just a simple numeric value, but a point on the *secp256k1 elliptic curve*. So it's not really a cheap multiplication, but still a 'simple' operation.

Now look at what *VerifyPubKey()* above is doing (let's ignore notions of compression or x-only pubkeys for now).
The goal of *VerifyPubKey()* is to theck if a public key corresponds to the secret of this CKey object. For this:
- It generates a string with a random part
  ```
    std::string str = "Bitcoin key verification\n";
    GetRandBytes(rnd);
    uint256 hash{Hash(str, rnd)};
  ```
- Then it signs the message using the private key
  ```
      Sign(hash, vchSig);
  ```
- And finally, it checks that the public key can be associated with this signature.
  ```
  return pubkey.Verify(hash, vchSig);
  ```
If the signature can be associated with the public key given in argument, then we can attest that this public key corresponds to that private key.

This is similar to what is done to allow spending bitcoin coins. When a transaction sends satoshis, it sends them to some bitcoin address, which is a public key.
And when the recipient wants to spend those satoshis, what they have to provide is the proof that they own this address.
From the private key only them know, they can sign the transaction data.
Then, bitcoin nodes are able to check if the signature of that data matches the address (public key) to allow spending this transaction output.

That's all good.

But wasn't there a really simpler (and faster?) way to do the public key verication?

## Couldn't we just compute the public key?
When spending coins, only the owner knows the private key. So they (nodes validating the transaction spending an output) have to, for example, check the signature of some known data of the transaction against a pubkey.

But in VerifyPubKey(), the class DOES know the private key. So could we not simply compute the public key associated with that private key, then just compare it with the public key given in argument? As seen before, this would only cost one multiplication and one comparison?

Mathematically, we absolutely can. It's also how an AI would implement this method. Because it is mathematically correct. But there is a reason why the Bitcoin core team chose a 'slower' way.

## Why does Bitcoin Core do this?
And that reason can be seen in the Git history on the project.
In commit [d0c41a73501a0bf94fca91be5fb38ab039490843](https://github.com/bitcoin/bitcoin/commit/d0c41a73501a0bf94fca91be5fb38ab039490843) from Pieter Wuille, we can see that compating pubkeys was exactly what was done up to 2014 / v0.10. And the reason why it was changed is explained in the commit message :
```
Add sanity check after key generation
Add a sanity check to prevent cosmic rays from flipping a bit in the
generated public key, or bugs in the elliptic curve code. This is
simply done by signing a (randomized) message, and verifying the
result.
```
So it's not that just comparing the argument to the public key that can be computed is wrong.

Bitcoin's value comes from the proofs mathematics can provide that it will behave as intended for all participants. But bitcoin core is not opening some portal to a world where pure mathematics rule. Bitcoin core is implemented in a programming language, compiled to some machine code, for some physical processor, that makes advanced use of the laws of physics or our world.

Bitcoin core has to make sure there is no fault anywhere in this chain, from some other part of the source code that would later become buggy, to some cosmic ray flipping a bit somewhere in the procesor's circuitry.

## So, where else does Bitcoin Core make this assumption?
Once i understood the issue, i spent some time investigating the rest of the code base, in case some other part of the code still had the issue, or if some newer code additions had mistakenly added back the same fragility. Fortunately, this was not the case. there was merley a single place where we were still compating the result of GetPubKey() to en expected public key in MuSig code, but the end of the function still had a proper verification later.
This was like a treasure hunt for me. In the end, i was slightly disappointed to not have found a place in the code I could have contributed to fix, but i think i also was more pleasantly surprised at the quality of the existing code, thanks to everyone that has been there before us.

## What does this have to do with contributing?
THIS, is how much rigor has to go into writing code fo Bitcoin core. And also the kind of thing you discover by actually reading and reviewing Bitcoin Core.

This is also an example of why contributing to Bitcoin Core isn't necessarily about writing hundreds of lines of new code.
Reading existing code, asking why something apparently redundant exists, looking at its history, and verifying the reasoning behind it is already valuable contribution work.
In fact, Bitcoin Core explicitly encourages new contributors to start with review and testing, because these are among the most important bottlenecks in the project.

In this case, a few lines of apparently redundant code led me from a cryptographic operation, through Bitcoin's history, to a much broader lesson about defensive programming in security-critical software.
