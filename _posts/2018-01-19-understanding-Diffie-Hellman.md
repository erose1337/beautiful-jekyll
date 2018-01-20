The goal of this post is to understand the Diffie-Hellman key agreement algorithm.

Standard disclaimer: This is not a formal research paper and should not be held to standards of rigorous unconditional correctness. 

Standard Diffie-Hellman
-----
Classic Diffie-Hellman key agreement uses modular exponentiation to generate public keys and shared secrets. The private key is a random integer which serves as the exponent. Here is an example using small numbers.

$x \leftarrow random()\\
y \leftarrow random()\\
pub_x \leftarrow g^x\\
pub_y \leftarrow g^y\\
share_x \leftarrow pub_y^x\\
share_y \leftarrow pub_x^y\\
share_x = share_y$

Exponentation is equivalent to repeated multiplication by $g$. The private key effectively indicates the quantity of $g$ in this sequence. If we unpackage the exponentation step and use small numbers, it is easy to see why the algorithm functions correctly:
    
$x \leftarrow 4\\
y \leftarrow 3\\
pub_x \leftarrow g * g * g * g\\
pub_y \leftarrow g * g * g\\
share_x \leftarrow pub_y * g * g * g * g \equiv (g * g * g) * (g * g * g * g)\\
share_y \leftarrow pub_x * g * g * g \equiv (g * g * g * g) * (g * g * g)\\
share_x = share_y = g * g * g * g * g * g * g$
                

The group structure
-----                
Applying a function successively forms a [group][1] - Each time the function is applied, it is effectively equivalent to incrementing the group value by 1. This allows for an indirect way to compute an addition operation, by enumerating the number of times that the function has been applied. In classic DH, the function $f$ is simple multiplication by the generator $g$. Effectively, a DH private key is the number of times that the function has been applied. 

You can actually instantiate [Diffie-Hellman][2] with an arbitrary deterministic function $f$ - correctness is guaranteed, but security is not. 

It is easy to see that when you apply the function $x$ times, and then apply the function $y$ more times, then the function has been applied in total $x + y$ times. Provided that both parties use the same $f$ and initial starting value (the generator $g$), both parties will arrive at the same value. 

An example with small numbers, generalized to use some function $f$ instead of the specific case of multiplication by $g$:

$priv_1 \leftarrow 4\\
priv_2 \leftarrow 3\\
pub_1 \leftarrow F(F(F(F(g))))\\
pub_2 \leftarrow F(F(F(g)))\\
share_1 \leftarrow F(F(F(F(F(F(F(g)))))))\\
share_2 \leftarrow F(F(F(F(F(F(F(g)))))))$

Clearly, $share_1$ and $share_2$ yield the same output, as the function has been applied the same number of times. Thus correctness is guaranteed - irrespective of the actual function used to instantiate the protocol. So what if we used say, sha256 as our $f$?

$f(x) = sha256(x)$
-----
To begin, we select an initial seed, some arbitrary bitstring labeled as $g$. For security, we require the private keys $\text{x, y}$ to be unguessable, which implies that they must be random bitstrings of size $>= 2^{128}$. This quickly presents a problem: Since the private key indicates the number of times that the function $f$ is to be applied, this implies that each party will need to invoke on the order of $2^{128}$ sha256 evaluations to generate a public key - this is clearly computationally intractable.

So while an arbitrary function can technically be used to instantiate the DH protocol, there are additional requirements the function must fulfill to ensure both computability and security.

Challenges
-----
There are three challenges that I have encountered to create a DH protocol this way. 

The first is finding some $f$ that is difficult to recover the number of times that $f$ has been applied, given some output $F(F(F....(g)))$.

- Failing to fulfill this property implies the ability of an adversary to recover the private key
       
The second is that the public keys must not be usable to reach the shared secret. 

- I.e. if $share_{secret} = pub_1 * pub_2$, then recovery of the private key is not necessary to break the system

The third requirement is that the function must offer an algebraic shortcut that allows you to compute the $N_{th}$ term without actually invoking $f$ $N$ times. This was the problem that we had with our sha256 example.

- Without an algebraic shortcut, computing a public key and/or the shared secret is computationally intractable. 

There is a actually fourth requirement that may or may not be trivial to fulfill: Finding a generator for the group must be "easy". (thank you to crypto.stackexchange [user MarquisDeSitruce](https://crypto.stackexchange.com/users/55071/marquisdesitruce) for pointing this out)

- If there is no generator, you cannot generate group elements, and hence no key agreement can occur

Conclusion
-----
The Diffie-Hellman key agreement algorithm can be generalized and instantiated with functions other than modular exponentiation - the challenge is really in finding an appropriate function. I hear [elliptic curves](https://en.wikipedia.org/wiki/Elliptic_curve_cryptography) work pretty well... but the details of those are still way over my head.

Note
-----
This post was adopted from an answer of mine on crypto.stackexchange

  [1]: https://en.wikipedia.org/wiki/Group_(mathematics)
  [2]: https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange#Generalization_to_finite_cyclic_groups
