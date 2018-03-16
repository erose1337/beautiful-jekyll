Cryptography is a skill. Nobody is born with the knowledge. Everyone who wants the knowledge has to learn it.

Non-Standard disclaimer: This is a light-hearted rant that is not intended to seriously offend anyone or their sensibilities.

Cargo cult advice: Don't roll your own
-----
The phrase "don't roll your own" has become a sort of cargo-cult response to individuals attempting to practice algorithm design. One of the points of this post is to say that we should  be striving for better accuracy in the advice that we give. I argue that algorithm design is not really the issue. It is algorithm *use* that is where problems and damages begin. 


"Forget design - analysis is what you should practice"
-----
Analysis is crtiical, I agree - but the counter argument that I present is that "algorithm *design*" necessarily includes "analysis". If your "design" process does not incorporate "analysis", then you are not really "designing" anything. If you don't know what problems need to be solved, you cannot possibly construct an appropriate solution to those problems.


Don't *use* your own
-----
Cryptography is like martial arts for the information world. In order to prevent attackers from stealing your information, you need to be able to defend it. Cryptography attempts to design techniques that provide certainty in defense against known adversarial manuevers (aka "attacks"). 

Algorithms such as AES and the SHA family are effectively equivalent to police and the military - they were designed by professionals to ensure the highest level of security guarantees possible against all adversaries. Continuing with this analogy, adversaries are equivalent to criminals and foreign military (which is sometimes literally true, and not an analogy!). 

With this in mind, let's suppose you choose to start a vigorous exercise schedule, learn some survival skills, and start taking some self-defence classes.

How long will it be before you are comfortable foregoing the protection of the police/military against criminals and foreign invaders? Remember, your adversaries are nation state level threats with effectively infinite budgets and armies of trained professional attackers.

![not a fair match]({{ "/img/archivevotto.png" | https://github.com/erose1337/erose1337.github.io/blob/master/img/archievotto.png }})

Clearly, any rational individual with functioning self-preservation instincts would not forego the state-provided protection.

But that doesn't mean that there is no value to what you learned. And it *especially* doesn't mean that we should actively be telling people *not* to learn these things!

The real advice that we should be giving is "*Don't fight battles that you don't have to*". Learning the skills and problem solving abilities that algorithm design (which includes analysis!) provides are hugely beneficial, even if you aren't using them to protect your own personal information from adversaries (or worse, other peoples information).


Telegram
------
Cryptography is a skill. Nobody is born with the knowledge, and nobody acquires it without making learning from dedicated pursuit of the subject. There are certain skills, such as mathematics and computer science, that can help make learning cryptography easier. But being a mathematician who can program a computer does not automagically make you a cryptographer. Case in point: Telegram. The blog post [here](https://web.archive.org/web/20171229010558/https://moxie.org/blog/telegram-crypto-challenge/) does a good job of explaining. So I will summarize with the following image:

![cryptography is a dedicated skill]({{ "/img/cryptographersprofession.png" | https://github.com/erose1337/erose1337.github.io/blob/master/img/cryptographersprofession.png }})


Conclusion
-----
Hopefully that wasn't *too* contentious. Basically, I have personally benefitted immensely from studying algorithm design, and the thought of someone (let alone everyone) convincing me (or anyone else) not to simply seems tragic. Cryptography will teach you to look for the weakest links and think critically, which are valuable skills to have in general. We should not expect people to produce exceptional, landmark algorithms such as RSA and Diffie-Hellman - but given enough time and dedication, practicioners of the skill very well may end up producing new algorithms that aren't woefully doomed and horribly broken. But until they are > level 1 in cryptography, it's guaranteed to not happen. And as long as they stick to actually using constructions that are understood to work, then there is no harm in playing the game.


