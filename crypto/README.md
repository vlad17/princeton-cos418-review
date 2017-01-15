# Summary
We use cryptography to hide messages between the sender and receiver. 

## Symmetric Keys
Sender and recipient share a common key, which can be used for confidentiality (encryption), and message authentication (integrity). The challenge is how to distribute this key, but there are different algorithms to do this (such as Diffie-Hellman).

## Public-Key Cryptography
Each part has a public key and a private key. Alice and Bob's public key can be accessed to anyone, and each uses private keys to decrypt ciphertexts and generate new signatures on messages.

### Details
RSA is a simple application of public/private keys. Basically just generate a number **n = p * q**, where `p` and `q` are primes. Then, pick an exponent `e` and solve for `d` such that **d*e = 1 (mod (p-1)(q-1))**. This is really secure because factoring is hard.

## Crytographic Hash Functions
A hash function is a one-way function that takes a message `m` of arbitrary length and produce a short number `H(m)`. It is efficient, such that it is easy to compute. It also has the **hiding property**, such that given `H(m)`, it is hard to find `m`. 

Collisions do exist, but are hard to find. 

## Hash Pointer
Use a hash function to point to data. There are various applications, such as in P2P. If we name a file `F = H(data)`, then participants can verify that `H(downloaded) == F`.

## Hash Chains
Creates a "tamper-evident" log of data, because if any of the data changes, all the subsequent hash points change (otherwise a hash collision is found, which has very low probability).
  
![serial schedule image](/transactions/hash_chain.png)


TODO: Consider adding this in another unit (such as block chains). This is in the conflict resolution lecture.