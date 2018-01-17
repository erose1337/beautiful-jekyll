Standard disclaimer: This is not a formal research paper and should not be held to standards of rigorous unconditional correctness. 

The order of operations in a permutation
-------
The design of permutation generally consists of the following steps:
- inclusion of round constants into the state    
- application of an inexpensive linear operation 
- application of an operation designed to add non-linearity
    
This post outlines the goals of each of these steps, argues for an optimal order for the application of the steps, and concludes by examining equations for different orders of operations. Additionally, there is some advice on how to generate round constants, as that is not a subject that has been as thoroughly researched as the design of linear/non-linear layers have.

Round Constants
-------
Goal of operation:
- produce asymmetry in the state
    - symmetry will be very noticeable with bit-sliced designs if you don't apply constants
    - constants causes different sections of the state to have different terms in them
    
The important questions are: How should constants be generated, and when should they be applied?
    
The Source and Application of Constant Values  
-----  
When examining an implementation of a permutation, it becomes easy to see that round constants occupy extra space. Extra space does not necessarily mean memory, but it does necessarily mean registers. There are really two options: dedicate a register to round constants and update them in place, or perform a MOV every round to load constants from memory. Due to the relatively simplistic design of most permutations, MOV instructions can quickly become the limiting factor in the efficiency of the algorithm. Where possible it is better to keep the constants loaded in a register, and apply instructions to the register to advance the value of the constants in place.

- the extra space required by constants can imply the need for more MOV instructions
- lots of MOV operations (especially into SIMD registers) can eat into CPU budget quickly and reduce throughput and latency

Round constants are usually chosen somewhat arbitrarily, in a manner that makes it at least plausible that they do not provide a backdoor. This post advocates the following manner to generate constants:

For a permutation with a small state (i.e. 128 bits), the round counter makes a great source for constants:
- The information is readily available, most likely already in a register
- Guaranteed to be unique each round
- No extra code or resources required to implement the evolution of the constants
- 0 degrees of freedom in selecting constants 
    - provides plausability that the constants do not enable a backdoor
        
While that may seem ideal, it is hard to compete on the throughput front with such a small state. A general purpose CPU really needs to utilize SIMD operations to achieve maximum throughput.
    
For a permutation that uses SIMD operations, using the round counter is slightly more complicated:
- SIMD operations quickly lose their efficiency when MOVing data frequently to/from SIMD registers
    - The need to minimize the number of MOVs is even more crucial
- The round counter is a single word, not an SSE register
    - Need to devote an entire SIMD register to constants

Given this information, this post concludes the following:    
- The round constants should be intialized into one SIMD register, and updated in place
    - operations like XOR and ADD require 2 operands
        - have to MOV the other operand into an SIMD register before using, which is slow
- solution: initialize SIMD register to 1, 2, 4, 8 and use shift left 1 to evolve constants
    - limited by word size and number of rounds   
    
The "Diffusion Layer"
------
The diffusion layer is usually linear, but may be fused with the non-linear layer, as in the case of ARX ciphers. The goal of this operation is to spread the terms of each expression evenly across all of the other expressions. This causes the number of terms in each expression to increase. The faster this rate of increase, the fewer applications/rounds will be required to propagate changes evenly throughout the state. Note that a heavier linear layer will require less rounds, but more time to apply. A cipher with fewer, more complex rounds is not necessarily faster than a cipher with more, simple rounds, and there is a balance to be struck between the two approaches.

The goals and design strategies for diffusion has been extensively studied elsewhere and will not be further detailed in this post. From this point on, we will simply assume the existence of an appropriate linear function when speaking of the diffusion layer.

The "Non-Linear Layer"
------
Equations of degree 1 are easily solved, regardless of how many terms are involved. The goal of the non-linear layer is to increase the degree of terms and produce expressions that are maximally complex and unworkable. 

The design of non-linear functions for cryptography has been extensively studied elsewhere and will not be detailed in this work. From this point on, we will assume the existence of an appropriate non-linear function when speaking of the non-linear layer.


The Order of Operations
-----------
This works argues that the ideal order of operations is as follows:
- add round constants
- update round constants
- linear diffusion layer
- non-linear layer

This arrangement has a few advantages:
- First, despite the simplicity of the round constants, they will be expanded to cover more of the state by the linear layer. 
    - Technically, one could argue that the constants are actually the values produced after applying the linear layer to the round counter, instead of the round counter itself. 
- The expanded round constants will contribute to an increase of the complexity of the equations after the application of the non-linear layer
    - If the constants are added after the non-linear layer, they do not increase the complexity of the equations
    
Consider alternative ordering of the operations:
- If the constants are added last, they are not expanded by the linear layer, and they do not contribute to the non-linear layer at all
- If the linear layer is applied after the non-linear layer, then fewer differences will be fed into the non-linear layer
    - This is easily visible during the first few rounds when operating on a state with a low initial hamming weight

Arguably, the least ideal order of operations is the reverse:
- non-linear layer
- linear layer
- add round constants
- update round constants

Admittedly, some of these arguments break down as soon as the second round is applied. For example, even if the constants were added last in round 1, the linear layer of round 2 will be applied to them. However, if the two designs are compared side-by-side and round-for-round, the recommended order of operations should produce clearly superior output. This may be more obvious by looking at the equations that represent the cipher, which are not too complex to view after only 1 round. Each bit of the state can be seen as the combination of XOR/AND of other bits in the state. Consider the following, using example layers that are not ideal, but serve to illustrate the point:

- With the supposedly optimal order, given input bits a, b, c, d, and round constants w, x, y, z:
    - First, xor in the round constants (and update them, which is not shown)        
        - `a ^ w`
        - `b ^ x`
        - `c ^ y`
        - `d ^ z`
    - Second, apply the linear layer
        - `a ^ w ^ b ^ x` 
        - `b ^ x ^ c ^ y` 
        - `c ^ y ^ d ^ z`
        - `d ^ z ^ a ^ w`
    - Third, apply the non-linear layer
        - `(a ^ w ^ b ^ x) & (c ^ y ^ d ^ z)`
        - `(b ^ x ^ c ^ y) & (d ^ z ^ a ^ w)`
        - `(c ^ y ^ d ^ z) & (b ^ x ^ c ^ y)`
        - `(d ^ z ^ a ^ w) & (a ^ w ^ b ^ x)`
        
- With the supposedly sub-optimal order, given input bits a, b, c, d, and round constants w, x, y, z:
    - First, apply the non-linear layer
        - `a & c`
        - `b & d`
        - `c & b`
        - `d & a`
    - Second, apply the linear layer:
        - `(a & c) ^ (b & d)`
        - `(b & d) ^ (c & b)`
        - `(c & b) ^ (d & a)`
        - `(d & a) ^ (a & c)`
    - Third, apply the constants:
        - `(a & c) ^ (b & d) ^ w`
        - `(b & d) ^ (c & b) ^ x`
        - `(c & b) ^ (d & a) ^ y`
        - `(d & a) ^ (a & c) ^ z`
        
Now, examining the two results side-by side (or rather, top and bottom for formatting purposes):
- `(a ^ w ^ b ^ x) & (c ^ y ^ d ^ z)`
- `(b ^ x ^ c ^ y) & (d ^ z ^ a ^ w)`
- `(c ^ y ^ d ^ z) & (b ^ x ^ c ^ y)`
- `(d ^ z ^ a ^ w) & (a ^ w ^ b ^ x)`         


- `(a & c) ^ (b & d) ^ w`
- `(b & d) ^ (c & b) ^ x`
- `(c & b) ^ (d & a) ^ y`
- `(d & a) ^ (a & c) ^ z`   
    
The first set of equations certainly looks more complicated, as it includes twice as many terms in between each AND operation. That `^ w` at the end of the second set of equations doesn't appear to really do much - supposing we were limited to one round, it would be trivial to peel those constants off. 