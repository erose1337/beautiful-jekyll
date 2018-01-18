The goal of this post is to blow your mind. Today we will do the impossible.

Standard disclaimer: This is not a formal research paper and should not be held to standards of rigorous unconditional correctness. 

Secret Key Based Public Key Encryption
-------
If you're familiar with cryptography, you are probably already thinking "this is not possible". So to begin, I will cite prior art by real cryptographers to indicate that once more we are not straying too far from an already beaten path.

[Fully Homomorphic Encryption Over The Integers](https://eprint.iacr.org/2009/616.pdf)
-------
The first time I read this paper, I was stunned. No further than the top of page 2 is the following quote:

    "So far we only described a symmetric scheme, but turning it into a public key encryption scheme is easy..."
    
I don't know about you, but I had to re-read this statement half a dozen times to make sure that it says what it looks like it says. I had to ask, what kind of secret sauce do they have that makes this unbelievably hard problem "easy"?

The answer is homomorphic encryption.

Homomorphic Encryption
-------
Homomorphic encryption preserves enough structure to allow you to perform meaningful operations on ciphertexts. For example, some schemes allow you to add ciphertexts together, and the result upon decryption will be equivalent to the addition of the plaintexts.

Homomorphic encryption effectively provides a world where a single person has read permissions, but everyone has write permissions. Coincidentally, this many-to-one structure happens to more or less be the defintion of public key cryptography. So it actually follows naturally that you can build public key encryption from secret key homomorphic encryption.

Preliminaries
------
We will assume the existence of a secure secret key cipher that allows us to add ciphertexts together. That is:

    D(E(m1) + E(m2)) = m1 + m2
    
A consequence of this is that you may also multiply ciphertexts by a plaintext scalar:

    D(E(m1) * 2) = D(E(m1) + E(m1)) = m1 * 2
    
Constructions
------
Not only can we build public key encryption from a secret key cipher, there are actually a whole bunch of ways to go about it. Some of them are more accomodating to certain types of ciphers, depending on the details of the design. For example, some ciphers may not be able to effectively encrypt 0s, which rules out the ability to use certain schemes.

The first idea we will explore is the one presented in the aforementioned paper. The cipher in that paper happens to support an extra feature, namely the ability to add a plaintext bit to a ciphertext:

    D(E(m1) + m2) == m1 + m2
    
This is a tad abnormal, as most homomorphic ciphers don't offer this feature. But it is useful, as we will see.

Encryptions of 0
-------
A public key consists of many encryptions of 0. The public key operation selects a random set of elements from the public key, then adds the plaintext bit to the result:

    ciphertext = pub[r_1] + pub[r_2] + pub[r_3] + ... + m
    
If we examine the resultant expression more closely, we see the following:
    
    ciphertext = E(0) + E(0) + E(0) + ... + m
    D(ciphertext) = 0 + 0 + 0 + ... m = m
    
Upon decryption, the zeros simply disappear, leaving us with the message bit. Pretty cool!

This particular public key operation forms a [subset sum problem](https://en.wikipedia.org/wiki/Subset_sum_problem). Many schemes in the past have been based on this, and it appears to be challenging to craft a secure variant. There are some important details that need to be examined, namely the so-called "density": the ratio of the size of the set to the size of the largest element in it. (thanks to [this paper](https://www.cypherpunks.ca/~iang/pubs/SSP-IJIS.pdf) for the succinct description); This post will not delve into such details.

Encodings
-------
This section will detail various different techniques that can be used to encode a plaintext value.

Not all ciphers function correctly/securely if you attempt to encrypt zeros. Additionally, the ability to add a plaintext to a ciphertext appears to be uncommon. These two points can be circumvented by various alternative encodings.

One such encoding is to distribute the encrypted ordered sequence 1, 2, 3, ... N as the public key. The public key operation then sums a random subset of these elements, and the payload is equivalent to the sum of the plaintext values:
    
    ciphertext = pub[1] + pub[5] + pub[7]
    D(ciphertext) = 1 + 5 + 7 = 13

Another encoding that may be more efficient in regards to the public key size is to distribute an ordered sequence of powers of two, 1, 2, 4, 8, ... N. A power of two has only a single bit set, and so the public key would look like this:
    
    0000 0001
    0000 0010
    0000 0100
    0000 1000
    0001 0000
    0010 0000
    0100 0000
    1000 0000
    
This encoding has you form your message one bit at a time by summing the appropriate elements to be equal to the desired message. Due to the deterministic nature of this encoding, it is helpful to distribute encryptions of 0 as well to help randomize the resultant ciphertext. Try encoding the same message twice and you'll see what I mean. Either that or just use it as a KEM to send a random value and then use that as a key to encrypt your message, which is arguably a better/simpler idea anyways.

A stronger public key operation
-------
Many cryptosystems based off of subset sum problems have been broken. It would be nice to be able to use a stronger public key operation. We can exploit the ability to multiply ciphertexts by a plaintext value to accomplish this. Let's distribute an encryption of 0 and an encryption of 1 as our public key, then try the following:
    
    ciphertext = (E(0) * r_0) + (E(1) * r_1)
    D(ciphertext) = (0 * r_0) + (1 * r_1) = r_1
    
This is nice for a whole bunch of reasons: We can use a smaller public key, and it actually makes a type of lattice problem. The example above only uses two values for the sake of simplicity; You could distribute many 0s instead of just one and increase the dimension of the problem.

"The augmented subset sum"
------
Note: The "augmented subset sum" is not really an official name. You probably won't find anything if you search for it. 

For the sake of completeness, you could combine the ideas into the following:
    
    ciphertext = (pub[r_0] * r_1) + (pub[r_2] * r_3) + (pub[r_4] * r_5) + ... + (pub[1] * r_n)
    D(ciphertext) = (0 * r_1) + (0 * r_2) + (0 * r_3) + (0 * r_5) + (1 * r_n) = r_n
    
The public key consists of encryptions of 0, as well as an encryption of 1. Encryption involves selecting a random subset of 0s, scaling them by a random plaintext value, summing them, and then finally adding a multiple of the encryption of 1. 

Beware
------
This post might make creating such constructions look easy - conceptually this may even be true. But the details matter greatly in regards to the security of the constructions. Things like the sizes of the numbers involved, and how many terms are used influence the security - the algebra only tells part of the story. You might be surprised at the ability of the [LLL algorithm](https://www1.lip6.fr/~joux/pages/papers/ToolBox.pdf) to pull these ciphertexts apart - I know I have been, many times!

