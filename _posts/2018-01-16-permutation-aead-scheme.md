For post #2, I will be talking about a permutation based authenticated encryption with associated data (AEAD) scheme. This is mostly an excuse to practice including images into my posts.

Standard disclaimer: This is not a formal research paper and should not be held to standards of rigorous unconditional correctness.

The motivation
----
AEAD schemes are a trendy topic due to a few reasons:
    
- It is not necessarily obvious how to [combine a MAC and encryption](https://crypto.stackexchange.com/questions/202/should-we-mac-then-encrypt-or-encrypt-then-mac)
- There are multiple possible configurations when combining a MAC and encryption , and they have all been employed in one place or another. Quoting the answer by the crypto.stackexchange user M.S. Dousti:
    
    Encrypt then Authenticate (EtA) used in IPsec;
    
    Authenticate then Encrypt (AtE) used in SSL;
    
    Encrypt and Authenticate (E&A) used in SSH.
    
These assume that you are using either a block cipher based mode of operation to encrypt, or a stream cipher. A block cipher is usually more versatile than a stream cipher.

A block cipher typically interleaves applications of a publicly known permutation with the addition of secret key material.

- Internally, the key material is often times generated from an initial master key by a simpler and weaker transformation than the actual permutation

Additionally, to actually use a block cipher, you need to implement one or more modes of operation, such as CBC, CTR, etc.

A MAC may often times use either a block cipher or a permutation under the hood.

So to encrypt-then-authenticate, you need to implement:

- A block cipher or a stream cipher
    - Requires a permutation (or perhaps a PRF for a stream cipher)
    - A key alternating cipher built from the permutation    
- A mode of operation for the cipher
    - not required for a stream cipher
- A MAC
    - Usually built from some kind of hash        
        - Often times uses a permutation or block cipher under the hood
        - May or not be the same permutation/cipher used to encrypt with
            - Almost certainly is not if you use a stream cipher instead of a block cipher
    - May require the HMAC construction as well
    
Needless to say, the more that you have to implement, the larger the possibility for failure becomes. [Neglecting to include something as simple as `nonce++`](https://lwn.net/Articles/423747/) can cause catastrophic and complete failure.

AEAD schemes are designed to simplify the process of providing authenticity/integrity to a cryptogram. 

Some AEAD schemes are designed to be secure even when a nonce is accidentally repeated, or a properly random IV cannot be generated. These schemes are called nonce-misuse resistant, and aim to fail gracefully even in the worst case scenario where an adversary controls all inputs except for the key (i.e. the adversary can choose to repeat nonces at will).

The sponge construction
-------
The sponge duplex construction offered by the designers of keccak is an example of a permutation based AEAD scheme. A sponge construction basically uses a truncated permutation to provide confidentiality, hashing, and MAC functionality, all based off of a single primitive. This can simplify implementation complexity, which reduces the chances of something going wrong accidentally. The sponge duplex construction functions similarly to an authenticated stream cipher.

The sponge duplex construction is not the focus of this post, I mention it as prior art to indicate that the following ideas do not stray too wildly from the already beaten path.


permutation encryption algorithm
-----
Instead of using a traditional key alternating cipher, we are going to encrypt data using only a permutation. The design uses a randomly generated encryption key as the initialization vector for every message. A curious quirk of this scheme is that knowing the key used to begin the encryption process is not sufficient information to decrypt the resultant ciphertexts (assuming no information about the message is known). An additional quirk is that the decryption IV is not the same as the encryption IV.

On to the design:

![permutation based encryption algorithm]({{ "/img/permutationencryption.png" | https://github.com/erose1337/erose1337.github.io/blob/master/img/permutationencryption.png }})

First, a random key is generated for use as an initialization vector. Then, the first block of the plaintext is loaded into the left half of the permutation, while the key is loaded into the right half of the permutation. The permutation is then applied, and the output is split into two blocks. The left half is output as the first ciphertext block, while the right half is output as a psuedorandom key to be used to encrypt the next block in the same manner. After all of the plaintext has been processed, the final psuedorandom key is locked via exclusive-or with the master key. The left half and right half are analogous to the rate and capacity of the sponge function. They could probably be referred to as such.

There are a few advantages to this construction. 
- First, no block cipher is needed - only a permutation is required. This may seem like a small advantage, but implementation complexity adds up
- Second, the key that is used to process the data is processed by the full randomizing power of the permutation, instead of an ad-hoc weaker key schedule. 
- Third, the master key is only ever used on the output of a permutation. 

Assuming that the initial random key is generated correctly and uniquely for each message, this design appears solid as a rock, at least as far as confidentiality of the message and security of the master key is concerned. 

That's not to say that there are no downsides at all:
- Since the block is split in half, and you want at least an 128-bit key (if not a 256-bit key), it will require a somewhat large permutation, probably 384 or 512 bits

For the sake of completeness, here is the decryption algorithm:

![permutation based decryption algorithm]({{ "/img/permutationdecryption.png" | https://github.com/erose1337/erose1337.github.io/blob/master/img/permutationdecryption.png }})

AEAD algorithm
----
You might be wondering where the authentication comes into play. Don't worry, we're not done yet!

Consider what would happen if an adversary flipped some ciphertext bits, and then the decryption process is applied. Clearly, due to the diffusion caused by the permutation, any blocks to the left of the blocks that are modified will become corrupted. As a result, so will the initial random key. We can use that key for a MAC as well, and use that tag to authenticate all of the information. Since we're processing a MAC, this provides an entry point to include some non-encrypted associated data.

![permutation based AEAD encryption]({{ "/img/permutationmooe.png" | https://github.com/erose1337/erose1337.github.io/blob/master/img/permutationmooe.png }})

Wow, that was easy! We just tacked on a MAC to the beginning and it took care of authentication and integrity for us. There are a few advantages to this authentication technique:
- The MAC can be processed in parallel during the encryption process
- The time required to compute the MAC is independent of the length of the encrypted message

Again, that's not to say that there are no downsides:
- During decryption, the MAC cannot be processed until after decryption is complete

So encryption could be made blazingly fast, but decryption will necessarily be a bit slower. And ideally, you would like to verify the MAC *before* you bother to decrypt the ciphertexts - after all, if the cryptogram has been modified and you have to throw it out, you're wasting cycles to decrypt a garbage message. 

For the sake of completness again, here is the corresponding decryption algorithm:

![permutation based AEAD decryption]({{ "/img/permutationmood.png" | https://github.com/erose1337/erose1337.github.io/blob/master/img/permutationmood.png }})

Failure and repeated values
-------
Note: In this section, "the key" refers to the randomly generated encryption key, and not to the master key.

The worst case failure is when both a key, message, and associated data are all repeated. In this case, the cryptograms will be equivalent. No scheme can prevent this when all inputs are identical, so the scheme fares no worse than others in this regard.

In the case that a key and message are repeated but the associated data changes, the tag will still be unique. The ciphertext portion of the cryptograms will be identical. Confidentiality is partially breached in that an adversary may determine whether or not two ciphertexts encrypt the same message, but they may not learn anything new about the message.

In the case that a key and associated data are repeated, the authentication tag will remain the same. However, it does not appear that this yields the ability to break authenticity/integrity - attempting to supply a modified ciphertext or associated data to the decryption algorithm will still cause the tag generated during decryption to be different than the correct tag. Perhaps this could be exploited to create an authenticated scheme with minimal ciphertext expansion by re-using the same tag for different ciphertexts.

In the case that the message and associated data are repeated, but the key is properly generated, then the resultant cryptogram will unique.


A nonce-misuse resistant variant
-----
If we are willing to forego some performance benefits, we can craft a fully deterministic nonce-misuse resistant scheme. This can be done by computing the MAC over the associated data and plaintext first, and then using the output as a source of entropy to encrypt the message. This is pretty much the standard method to create a nonce-misuse resistant scheme. This post is a bit long already, and the parallel construction appears to fail relatively gracefully as-is, so I will save this idea for later.

