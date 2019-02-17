---
layout: page
title: About me
subtitle: Who in the world am I?
---

My name is Ella Rose. I am a fast learner who's always keen to pick up new technologies and concepts. 

I am a top user on the [Cryptography Stack Exchange](https://crypto.stackexchange.com/users/29554/ella-rose) site with over 100 answers and over 100,000 visitors reached.

I have some skill in the following subjects:

- Computer programming/software development 
  - 5 years of experience
  - I developed my own [application framework in python](https://github.com/erose1337/pride) that provides apis for audio, gui, networking, database, and security 
- Cryptography
  - 2.5 years experience
  - Extensive experience with [algorithm design and analysis](https://github.com/erose1337/crypto), including symmetric and asymmetric algorithms
- Cooking 
  - 10+ years experience
- Music 
  - 10+ years experience
  - I have [my own compositions](https://www.bitchute.com/video/Nmptt4DaC4vq/) of varying genres that I play on guitar
  
Additionally, I have many interests that I enjoy researching and reading about:

- Technology
- History
- Physics
- Psychology

As well as some subjects that I would love to develop skill with, instead of just reading about:

- Digital Signal Processing (DSP)
- Machine Learning
- FPGA/Hardware Development

I am [interested in freelancing, consultation, and full-time employment](https://stackoverflow.com/users/story/3103584) for the right kind of work.

You can add me on [keybase](https://keybase.io/ella_rose) if you are so inclined.

### my history

I initially started learning python because I had an idea for a game that I wanted to develop. I began by developing buttons and windows using the pygame package, which eventually became fleshed out into a more fully featured and general purpose api for building graphical applications. I then became acquainted with some audio related packages for python, and incorporated those into my as-of-yet undeveloped application framework. I quickly learned to put the program to work for me, developing techniques that enabled me to integrate an arbitrary amount of concurrent functionality using consistent design patterns and processes. 

Along with audio I began to learn about networking and socket programming, which I also integrated into my existing code base. Next I created my own remote python shell, which was the beginning of the rabbit hole for my study of computer security and cryptography. I needed to learn how to handle passwords properly, so I did a search and found my way to stackexchange. I began to read questions and answers like mad while incorporating my new knowledge into my program. 

Eventually, after constructing a remote procedure call layer and a data transfer service on top of that, I became so enamoured with cryptography and algorithm design that my time spent programming became split between my application framework and exploration of cryptography. 

I began to practice the forbidden art of cryptographic algorithm design. My first constructions were of course hopelessly insecure and totally inefficient, but the purpose of the operation was to learn, not to create any proposals. And learn I did, and my results began to improve. I mostly designed ciphers and permutations of various kinds, with occasional forays into the design of hash functions. I learned to program in C, read assembly, and learned some of the details about how the CPU functions. I also began to answer more questions on crypto.stackexchange.

Eventually, I happened to construct a design with a most curious feature. My original goal with the design was to have something extremely efficient in hardware. The design utilized only XOR and a secret transposition, which I reasoned could be implemented with nothing but XOR gates and crossed wires. However, I quickly noticed strange patterns in the output of this particular cipher. It turns out that I had accidentally designed a cipher that was homomorphic with respect to XOR. This began my foray into homomorphic encryption, which began my foray into public key cryptography.

About that same time, I stumbled upon the paper Fully Homomorphic Encryption Over The Integers, which casually mentioned something that was incredible to me. These cryptographers not only had a technique to turn a secret key cipher into a public key one, but they barely mentioned it in passing! I recognized that I could utilize the cipher I had designed to instantiate a public key cryptosystem, and began to work closely on these ideas with someone who would become my go-to cryptanalyst. 

Together, with most of the initial credit going to my partner, we broke my initial design. We then proceeded to design and break countless others, and eventually the secret-key based designs gave way to proper number theory based asymmetric cryptosystems... and then I went full circle, back to a secret-key based design. 

This more or less brings us to today, where I am maybe halfway to understanding how to apply the magical cipher-destroying black-box known as the LLL algorithm. I have learned an incredible amount about computer science, mathematics, and design/engineering principles in general along the way, and I hope and believe that there is plenty more still to come.

