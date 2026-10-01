# Why bitcoin core does not even trust the computer

Since i will have to manage keys/identities in my own project, i was looking at how they are implemented in
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
The CKey class manages a bitcoin private key. the *keydata* member stores the private key,
and various operations on it are provided by the methods of the class.
For example, the method *GetPubKey()* returns the public key associated with the secret value in *keydata*.
the result of GetPubKey() is basically the multiplication by the point G by the secret. G is not just a simple numeric value, but a point on the elliptic curve. So it's not really a cheap multiplication, but still a 'simple' one.

Now look at what *VerifyPubKey()* is doing (let's ignore notions of compression for now).
The goal of *verifyPubKey()* is to theck if a public key corresponds to the secret of this CKey object. For this:
- It generates a string with a random part
- Then it signs the message using the private key
- And finally, it checks that the public key can be associated with this signature.
If the signature can be associated with the public key given in argument, then we can attest that this public key corresponds to that private key.

This is similar to what is done to allow spending bitcoin coins. When a transaction sends satoshis, it sends them to some bitcoin address, which is a public key.
And when the recipient wants to spend those satoshis, what they have to provide is the proof that they own this address.
From the private key only them know, they can sign the transaction data.
Then, bitcoin nodes are able to check if the signature of that data matches the address (public key) to allow spending this transaction output.

That's all good.

But wasn't there a really simpler (and faster?) way to do the public key verication?

When spending coins, only the owner knows the private key
