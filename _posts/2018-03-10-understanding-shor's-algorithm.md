In this post we will understand the part of Shor's algorithm that uses classical computation. 

But Shor's algorithm is a *quantum* algorithm?
-----
Shor's algorithm consists of two parts, a classical computation part, and a quantum subroutine invoked during the classical computation part. So while it incorporates a quantum algorithm, the entire algorithm does not require quantum circuits, only part of it.


So what is Shor's algorithm and why is it useful?
-----
Shor's algorithm is an efficient algorithm for factoring integers. When input some number $N$, it outputs the factors of $$N$$ - the numbers $$p, q$$ such that $$p * q = N$$. 

When $$p, q$$ are sizeable prime numbers, separating $$N$$ into the constituent $$p, q$$ is a very hard and prohibitively time consuming problem. It should be noted that not *all* instances of integer factorization are hard. For example, if $$N = 2 * 3 * p$$, then it is easy to separate $$N$$ into the constituent factors even when using a naive factoring algorithm.

Through some math that I do not yet understand, Shor's algorithm can [also be used](https://en.wikipedia.org/wiki/Shor%27s_algorithm#Discrete_logarithms) to solve the discrete logarithm problem. I won't be commenting on this in this post as I don't get it yet.


Why are these hard problems important?
-----
Some of the algorithms that secure the worlds communication are based on the intractability of solving the problems of integer factorization and discrete logarithms. The ability to solve these problems implies the ability to break the algorithms that are used to secure communications on the internet. The existence of Shor's algorithm is largely responsible for NIST calling for submissions for cryptography algorithms that are not based off of these problems. 


So how does it work?
-----
Basically, the algorithm tries to find some $$x$$ such that $$x^2 \equiv 1 \bmod N$$. After doing so, it retrieves factors of $$N$$ by computing $$gcd(x - 1, N)$$ and $$gcd(x + 1, N)$$. Why does this work? 

There is an identity that says $$x^2 \equiv 1 \bmod (x + 1) * (x - 1)$$. Let's try some examples with small numbers:

$$2^2 \equiv 1 \bmod (3 * 1)\\
3^2 \equiv 1 \bmod (4 * 2)\\
4^2 \equiv 1 \bmod (5 * 3)\\
5^2 \equiv 1 \bmod (6 * 4)\\
\dots$$


Ok, so clearly that works out that way. But after finding $$x$$ why do we compute $$gcd(x + 1, N)$$ instead of simply taking $$x + 1$$? The answer is simple: the $$x$$ in question most likely squares to be a multiple of $$N * r + 1$$ instead of $$N + 1$$. This means that each $$x + 1$$ and $$x - 1$$ will have some extra factors, namely those that constitute the $$r$$ value.

In the following example, we will take $$x=3$$, $$N = 8$$, and we will assume the value $$r = 4$$. This is basically just a random choice for $$r$$. $$r$$ is not a value that is chosen by the algorithm, it is a by-product of the way the result is computed:

- $$3^2 * 4 \equiv 1 \bmod (4 * 2 * 4)$$
    - $$gcd(4, 4 * 2 * 4) = 4$$


So how do you find such an $$x$$?
-----
This is where Shor's algorithm comes into play. It searches for some $$a^b$$ such that $$a^b \equiv 1 \bmod N$$ and $$b$$ is even. The first condition combines with the second condition so that we can divide $$b$$ by $$2$$ and obtain some $$a^{b/2}$$ that when squared is congruent to $$1$$ modulo $$N$$. Then we can compute $$gcd(a^{b/2} + 1, N)$$ and $$gcd(a^{b/2} - 1, N)$$ to obtain the factors of $$N$$.

The quantum algorithm comes into play when searching for $$b$$. Finding the period of $$a^1$$, $$a^2$$, $$a^3$$, ... $$a^b$$ is apparently a hard problem [^1], but it is what we need to do to find $$b$$. The [quantum phase estimation algorithm](https://en.wikipedia.org/wiki/Quantum_phase_estimation_algorithm) is used to find $$b$$. I don't understand the actual quantum algorithm part of Shor's algorithm yet, so I can't comment on it. But I can recommend this [blog](http://algassert.com). This guy *really* knows what he's talking about in regards to quantum computation. 


[^1]: I say "apparently a hard problem", because I could not figure out/find what specific "hard problem" order-finding actually is based on. I suppose it could be solved with discrete logarithms, but I don't know that discrete logarithms are the only way to solve the problem.