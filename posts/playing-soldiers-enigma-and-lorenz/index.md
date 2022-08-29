---
title: "Playing Soldiers: An Exploration into Wartime Ciphers - Enigma and Lorenz"
date: 2022-08-29T15:27:07
draft: false
featuredImage: static/postimages/playing-soldiers-enigma-and-lorenz/Encrypt.svg
author: Luke Briggs
rssFullText: true
categories: [Cryptography, Software Development]
description: Rotors, Motors, and 5-hole paper tape
---

I had a bit of time over the Summer, and thought I would do something that I have been meaning to do for a long time.
Create a software implementation of an Enigma machine. 
While I was at it, I thought that I might as well take a look at how the machines could be broken.

For the programs themselves, head over to the [project page](/projects/enigma-lorenz).
The rest of this post is an explanation of how the machines work and the low-hanging fruit when it comes to vulnerabilities.

# Preface
Enigma and Lorenz are two series of electromechanical cipher machines that saw extensive use by the Axis powers during World War II. 
Enigma was primarily used by the German Navy, Air Force, and Army as a method to encrypt communications. 
Lorenz machines were used from 1940 onwards and were the preferred encryption method for communications between German High Command. 
The Enigma and Lorenz machines formed the vast majority of traffic decrypted at the British Government Code and Ciphers School (GC&CS) in Bletchley Park.

# Enigma
Enigma was the most widely used of the two machines, and the largest share traffic flowing through Bletchley Park was encrypted by Enigma. 
With over 40,000 machines constructed (compared to the approximately 200 Lorenz machines), Enigma was the priority of Allied codebreakers, especially during the early stages of the war.

## Workings
German military Enigma machines are made up of three distinct parts: the rotors, the reflector and the plugboard.

![Data flow through Enigma machine](static/postimages/playing-soldiers-enigma-and-lorenz/Enigma.svg)

### The Plugboard
![Enigma plugboard](static/postimages/playing-soldiers-enigma-and-lorenz/Plugboard.svg)

The plugboard is the simplest part of the encryption process but is what separates military Enigma from the commercial variant used before the war. 
The plugboard is a grid of contacts with each contact relating to a letter. 
Two ends of a plug would be inserted into two of the letters to electronically connect them. 
The connections on the plugboard would cause letters to be swapped before and after encryption. 
Each Enigma machine came with 13 plugs, with only up to 10 ever used at one time.

### The Rotors
The rotors are the most distinctive element of the Enigma machine. 
All German military Enigma machines featured three conventional rotors. 
A later Enigma model, the M4, is often referred to as having four rotors however this ‘fourth’ rotor behaves more like a reflector than a conventional rotor. 

![Enigma rotor](static/postimages/playing-soldiers-enigma-and-lorenz/Rotor.svg)

A rotor is very straightforward electronically: it maps a single input on one side, to an output on another side, through a wire. 
There were 26 mappings on each rotor, one for each letter of the alphabet, and each mapping was marked with either a number from 1 to 26 or the letters A to Z. 
The topmost marking could be seen through a small window on the machine, and this was used for setting an initial configuration. 
An Enigma rotor also has a “ring setting”. 
The markings on the rotor can be twisted independent of the rotor itself and so the mapping represented by “A” on ring setting 0 would be represented by a “Z” on ring setting 1. 
It effectively offsets the outer casing and notch from the inner wiring. 
The ring setting is set at the beginning of the message and remains the same throughout.

On each keypress the rightmost rotor is rotated towards the operator by a single step. 
Each rotor also has one or two small notches aligned with one or two of the steps. 
Usually, when the right rotor turns it blocks the other two rotors from spinning. 
However, if the rotor is in a position where the notch aligns with the mechanism, the rotor to the left of it can rotate one step. 
The same is true for the middle and left rotors. 
The effect is similar to something like the odometer on a car: at a certain point in the rotation of the right rotor, the middle rotor is increased by one, and at a certain point in the rotation of the middle rotor, the left rotor is increased by one.

#### Note on Double Stepping
There is a quirk with the mechanism of German Enigma machines which means the middle rotor will also rotate if it is at its notch position even if the rotor to the right of it is not.

### The Reflector
The reflector is electronically similar to a rotor in that it has a series of input and output contacts that are mapped with wiring, however there is no rotation of a reflector during transmission of the message. 
A reflector is set at the beginning of a message and remains static throughout. 
The ‘fourth’ rotor of Enigma M4 also remains static throughout the message and therefore it somewhat of a misnomer to call it a ‘fourth’ rotor. 
In fact, using the ‘beta’ wheel as the fourth rotor and the UKW-b wheel as the reflector in M4 Enigma is logically identical to the UKW-B reflector of three-wheel Enigma.

## Stages of Encryption

![Enigma encryption process](static/postimages/playing-soldiers-enigma-and-lorenz/Encrypt.svg)

To fully understand the stages of Enigma encryption, it is best to show a worked example. 
In this example we are going to encrypt the character ‘A’ with rotors III, II, and I in default position, with A mapped to T on the plugboard, and with the ring setting of rotor I set to 2. 
To run this configuration in the provided simulator the command would be:

`enigma -m "A" -l "III 1 0" -c "II 1 0" -r "I 1 2" -ukw "B" -plugs "A:T"`

### Character 'A' Pressed
When character A is pressed, the right most rotor rotates forward by one. 
The right rotor was not at its notch position, so it is the only rotor that moves. 
An electrical signal is then sent through the machine.

1.	The first stage is the plugboard. Our plugboard maps A to a T so our electrical signal comes out of contact T of the plugboard.
2.	Because rotor I has been shifted by one, output T of the plugboard actually goes into input U of rotor I. The input contact U maps to the output contact U and so this receives the electrical signal.
3.	Rotor I’s contact U is connected to Rotor II’s input contact T so this is the input point of the rotor. T is wired to N in this rotor, so a signal is outputted on this contact.
4.	Rotor II’s contact N is connected to Rotor III’s input contact N so this receives the signal. Input N is wired to output N so electricity is passed out at this contact.
5.	Output N goes into the reflector’s input N, travels to the mapped output point K and then travels back through all previous 3 rotor along whichever wirings are available in that direction.
6.	K is mapped to U so Rotor III feeds electricity to the contact U on Rotor II’s left hand side.
7.	U is mapped to H so Rotor II feeds electricity to its output contact H.
8.	Because Rotor I is rotated by 1 step, Rotor II’s right contact H is touching Rotor I’s left contact I so this is the input point. I maps to H so electricity is fed to this output contact. 
9.	Once again, because I is rotated by I, contact H is actually in connected to the G contact on the plugboard. Because we have no mapping for G, it passes through unchanged, and the G lamp lights ups. G becomes our ciphertext.

## Methods of Decryption
### Brute Force
The most naïve of decryption methods would be to brute force the initial settings. 
This would be incredibly difficult, even on a modern machine, due to the large number of possibilities:

Possible Rotor Combinations = 5×4×3=60 (Picking 3 rotors from 5)

Possible Rotor Settings = 26×26×26=17,576 (26 settings for each of the 3 rotors)

Possible Plugboard Settings = 26!/(6!10!2^10)=150,738,274,937,250 (26 letters, order of 10 pairs does not matter, 6 letters go unused, mappings are bi-directional)

Total Combinations = 158,962,555,217,826,360,000 

This is only using a three rotor Enigma machine where three rotors are chosen from five. 
The Navy could choose three rotors from eight, and the M4 Enigma had even more combinations with its two reflectors.

Brute forcing Enigma was impossible at the time and is still infeasible as a method of decryption on modern machines in a reasonable period.

### Analytical Methods
Because of the infeasibility of brute force methods, cryptanalysis had to be performed to reduce the search space to something reasonable. 

#### Known-plaintext Attacks
One of the key flaws in Enigma’s encryption process was that, due to the nature of the reflector, a letter in plaintext could never map to the same letter in the ciphertext. 
The primary method of utilising this flaw came in using either known plaintext or guessed plaintext. 
If a message was sent with partially known content in a guessable format, then you could use the flaw to work out where certain words may appear in the ciphertext and then reverse engineer the settings from that. 

An example of this might be a weather report. 
A weather report is likely to contain the German word for ‘weather report’, ‘wetterbericht’. 
You could see where ‘wetterbericht’ would plausibly fit in the message (no plaintext letters match ciphertext letters) and only go through combinations where that outcome is possible. 
A piece of known plaintext such as this was known as a ‘crib’.

Cribs could also be used to identify plugboard combinations. 
By going through the crib, we could make certain assumptions and develop trees of possible plugboard settings. 
If one of our assumptions is contradicted along the tree, then we can rule out that whole branch of plugboard settings without ever needing to try them. 
Techniques like plugboard checking can help to reduce the search space significantly. 
The search space became small enough that even in the 40s it was possible to use these techniques to reduce the problem electromechanically using devices known as Bombes.

#### Ciphertext-only Attack
All successful cryptanalysis on Enigma during the war focused on known-plaintext attack methods. 
Modern computing power combined with statistical methods allows for certain plaintext-only methods of decryption. 
On such plaintext-only attack involves calculating the index of coincidence. 
Index of coincidence (IoC) is a measure of how likely to it is to draw two letters from a piece of text, and for those two letters to be the same. 
Index of coincidence is useful in decrypting because the distribution of letters in a language is different to that of random characters (A piece of random text has an average IoC of 1.0 whereas German text has an average IoC of 2.05). 
Enigma is vulnerable to the use of IoC as a method of decryption since if one of either the rotors, ring settings, or plugboard are correct, the message will be slightly closer to the plaintext and therefore have a slightly higher IoC.

If you were to run all possible rotor combinations with default ring settings, no plugboard, and the message was sufficiently long, the IoC of the correct rotor combination would be slightly higher than random. 
Being able to solve, or at least narrow, the problem in individual elements removes the multiplicative element of brute force and makes the problem more computationally feasible. 
Likewise, having one of the plugboard settings correct will produce a better IoC than having none of them correct.
 
IoC is only useful when you have enough text to become statistically significant, with Enigma this would be somewhere between 600-1500 characters depending on plugboard settings. 
The German military purposely kept Enigma messages short for this very reason, as IoC was known at the time. 
There were, however, occasions where operational mistakes or multiple messages using the same settings would result in long strings of characters enciphered the same.

It is likely that IoC could be used in conjunction with known-plaintext attacks to create a much more efficient method of decryption since both use alternative ways to narrow the search-space.

# Lorenz

As previously mentioned, The Lorenz machine was used to encipher the most secret intelligence between Hitler and his generals. 
The nature of the traffic meant that the cryptanalysts at Bletchley Park had much fewer messages to try and decrypt, and the lack of machines meant that it was less likely for them to get a physical machine to test with. 
In fact, almost all cryptanalysis on Lorenz was done without any access to the physical hardware as the first machine was only captured by the allies late in the war as Europe was being liberated. 
However, messages did tend to be longer due to the fact the Lorenz machine was much easier to use.

## Workings
Unlike Enigma, which was sent by morse, the Lorenz machine used teleprinter code which was automatically ciphered and deciphered. 
An operator would type their message in plaintext and the receiver would see the incoming message in plaintext with decryption only occurring for transmission.

Lorenz signals used, what would now be referred to as, a 5-bit encoding scheme known as ITA2. 
Plaintext characters would be represented as 5 dots or spaces on 5-hole paper tape, and this is what would be enciphered. 
The encipherment is logically straightforward, the plaintext ITA2 stream would have a bitwise XOR operation performed on it with a pseudorandom keystream. 
It is the cracking of how the keystream was generated that would be vital in uncovering the message.

For UX reasons, the ITA scheme used by the Lorenz simulator is slightly different to the standard ITA2 schema.
The simulator uses the following mapping:

<div style="line-height: 1.1rem; margin-left: auto; margin-right: auto">

| **Binary Code** | **Letter Shift**    | **Figure Shift**    |
|:---------------:|:-------------------:|:-------------------:|
| `00000`           | ±                   | ±                   |
| `00001`           | T                   | 5                   |
| `00010`           | \_                  | \_                  |
| `00011`           | O                   | 9                   |
| `00100`           | SPACE               | \!                  |
| `00101`           | H                   | £                   |
| `00110`           | N                   | ,                   |
| `00111`           | M                   | \.                  |
| `01000`           | \|                  | \|                  |
| `01001`           | L                   | \)                  |
| `01010`           | R                   | 4                   |
| `01011`           | G                   | &                   |
| `01100`           | I                   | 8                   |
| `01101`           | P                   | 0                   |
| `01110`           | C                   | :                   |
| `01111`           | V                   | =                   |
| `10000`           | E                   | 3                   |
| `10001`           | Z                   | \+                  |
| `10010`           | D                   | \#                  |
| `10011`           | B                   | ?                   |
| `10100`           | S                   | \\                  |
| `10101`           | Y                   | 6                   |
| `10110`           | F                   | %                   |
| `10111`           | X                   | /                   |
| `11000`           | A                   | \-                  |
| `11001`           | W                   | 2                   |
| `11010`           | J                   | $                   |
| `11011`           | ^ \(Figure Shift\)  | ^ \(Figure Shift\)  |
| `11100`           | U                   | 7                   |
| `11101`           | Q                   | 1                   |
| `11110`           | K                   | \(                  |
| `11111`           | \* \(Letter Shift\) | \* \(Letter Shift\) |

</div>

### Keystream Generation
Like Enigma, the Lorenz machine used rotors, however there were 12 rather than the 3 found in Enigma. 
The rotors can be divided into three sets: the ψ wheels, the χ wheels, and the μ wheels.

![Lorenz encryption process](static/postimages/playing-soldiers-enigma-and-lorenz/Lorenz.svg)

Each rotor has a number of pins across its circumference. The number of pins on a rotor is co-prime with the number of pins on the other rotors, giving a very long period before a key sequence is repeated. The pins on the rotor can be set to either on or off (equivalent to 1 or 0 on a bit pattern). 

When a key is pressed, the 5-bit input is XORed with the bit pattern across the χ wheels, then again with the bit pattern across the ψ wheels. The wheels are then stepped depending on the position of the other wheels.

#### Rotor Stepping
-	After the XORing, all χ rotors step forward by one. 
-	The ψ wheels only step forward by one if μ2 is in a ‘on’ positions
-	Then, μ1 will always step forward by one
-	If μ1 is in an ‘on’ position then μ2 will also step forward by one.

Later models of the Lorenz machine had further limitations placed on when a rotor would step forward.

## Methods of Decryption
As with Enigma, Lorenz machines were far too complex to have any hope of using brute force techniques to acquire the initial settings.

### Depths

As with Enigma, having credible guesses of what parts of the plaintext might be can go a long way into decrypting the whole message. If two messages share the same key then it becomes a much easier task.

Consider the following, where C1 and C2 are two cipher texts, P1 and P2 are two plaintexts, and K is a key they have in common.


$$C_1=P_1\oplus K$$
$$C_2=P_2\oplus K$$
$$C_1\oplus C_2=P_1{\oplus P}_2\oplus K\oplus K$$
$$\mathrm{Since\ }K\oplus K=0:$$
$$C_1\oplus C_2=P_1{\oplus P}_2$$
$$\mathrm{let\ }F=C_1\oplus C_2$$

As can be seen, by having two messages that use the same key, you can achieve a combination (F) of the two plaintexts by XORing the two ciphertexts. The resulting amalgamation contains no part of the keystream.

The result of this is, if we can guess a part of one plaintext, and XOR it with F, then we will get a part of the other piece of plaintext. One powerful instance of this weakness proving useful came when a German officer sent two long messages using the same key with slightly different content. First a message was sent, then the receiver requested the message be sent again due to a failure on their end, the sender sent the message again but, because they were more impatient the second time around, started using abbreviations in their message.

#### Worked Example
Say we have the message: 
```
Plaintext:      REGIMENT NUMBER 5 WILL ATTACK WEST POINT AT DAWN
Ciphertext:     Z_ZFINNOV_YQM*PL*_JL^6?-1!::9=|+,(2(:1.$,1-1*OKVIL
```

And then our operator comes back to us and asks us to repeat the message. We might send back:

```
Plaintext:      REGIMENT NO 5 WILL ATK W POINT AT DAWN
Ciphertext:     Z_ZFINNOV_RKVCALQVI NT^=1£&_-03'^%8^-|13
```

If we XOR our two cipher text messages we get:
```
F:              ±±±±±±±±±±*O|ZY±_PFPQS|X±THI^_-£1|6£%6$4
```

In this particular ITA alphabet, ± is the null character and so we know that the characters in those positions are the same in both messages.

Let us suppose that we know that a message is going to start with a regiment number, but we don’t know where they will attack. We can XOR our crib with F to see if we can get some of the other message:
```
F:              ±±±±±±±±±±*O|ZY±_PFPQS|X±THI^_-£1|6£%6$4
Crib:           REGIMENT NUMBER
F XOR CRIB:     REGIMENT NO ^5*
```

We can see that be XORing our crib against the message, we get the contents of the other message. Since they are messages with the same intent, we can take a guess that our first message will also refer to regiment 5, this can form the basis of a new crib:

```
F:              ±±±±±±±±±±*O|ZY±_PFPQS|X±THI^_-£1|6£%6$4
Crib:           REGIMENT NUMBER ^5*
F XOR Crib:     REGIMENT NO ^5* WI
```

Here we have gathered two more letters. We can keep feeding cribs into each other to read out the entire message:
```
F:              ±±±±±±±±±±*O|ZY±_PFPQS|X±THI^_-£1|6£%6$4
Crib:           REGIMENT NUMBER ^5* WI
F XOR Crib:     REGIMENT NO ^5* WILL A
Crib:           REGIMENT NUMBER ^5* WILL A
F XOR Crib:     R REGIMENT NO ^5* WILL ATK W
```
At points like this we can see another likely deviation in the messages. We have found out that the second message uses the phrase ATK, one assumption we could make is that they used the full word ATTACK in the original message.

```
F:              ±±±±±±±±±±*O|ZY±_PFPQS|X±THI^_-£1|6£%6$4
Crib:           REGIMENT NUMBER ^5* WILL ATTACK
F XOR Crib:     REGIMENT NO ^5* WILL ATK W POIN
```

Likewise, a cryptanalyst might also assume that W could mean WEST. Realistically cryptanalysts would try various combinations to see which one creates reasonable plaintext.

```
F:              ±±±±±±±±±±*O|ZY±_PFPQS|X±THI^_-£1|6£%6$4
Crib:           REGIMENT NUMBER ^5* WILL ATTACK WEST POI
F XOR Crib:     REGIMENT NO ^5* WILL ATK W POINT AT DAWN
```

We now have both messages in the clear without ever needing to know what the settings were of the Lorenz machine or what the key stream was. The only outright guess we needed to make was that the message would start with the phrase “REGIMENT NUMBER”, and from that we found out which regiment it was, what they were doing, and when they were doing it.

This was not the only technique used in the breaking of Lorenz traffic, but it is one of the most potent. Such an attack also highlights how operational failures can severely jeopardise the use of a system that is quite secure from a cryptographic standpoint.

# Bibliography
Computerphile. (2014, December 9). Tackling Enigma (Turing's Enigma Problem Part 2) - Computerphile. Retrieved from YouTube: https://www.youtube.com/watch?v=kj_7Jc1mS9k

Computerphile. (2014, November 28). Turing's Enigma Problem (Part 1) - Computerphile. Retrieved from YouTube: https://www.youtube.com/watch?v=d2NWPG2gB_A

Computerphile. (2015, July 1). Fishy Codes: Bletchley's Other Secret - Computerphile. Retrieved from YouTube: https://youtu.be/Ou_9ntYRzzw

Computerphile. (2015, September 16). Zig Zag Decryption - Computerphile. Retrieved from YouTube: https://www.youtube.com/watch?v=yxx3Bkmv3ck

Computerphile. (2018, September 5). Exploiting the Tiltman Break - Computerphile. Retrieved from YouTube: https://www.youtube.com/watch?v=-hrNWRtDr7Y

Computerphile. (2021, April 21). Cracking Enigma in 2021 - Computerphile. Retrieved from YouTube: https://www.youtube.com/watch?v=RzWB5jL5RX0

Good, J., Michie, D., & Timms, G. (1945). General Report on Tunny. 

Jackson, J. (2022, August 15). The National Museum of Computing. Retrieved from The Enigma Machine: https://www.tnmoc.org/bh-2-the-enigma-machine

Numberphile. (2013, January 10). 158,962,555,217,826,360,000 (Enigma Machine) - Numberphile. Retrieved from YouTube: https://www.youtube.com/watch?v=G2_Q9FoD-oQ

Numberphile. (2013, January 14). Flaw in the Enigma Code - Numberphile. Retrieved from YouTube: https://www.youtube.com/watch?v=V4V2bpZlqx8

Owen, J. (2021, December 11). How did the Enigma machine work? Retrieved from YouTube: https://www.youtube.com/watch?v=ybkkiGtJmkM

Stichting Cryptomuseum. (2022, August 12). Retrieved from Crypto Museum: https://www.cryptomuseum.com/index.htm

Thaophialuang, L. (2019, January 24). Enigma Simulator. Retrieved from Enigma Cipher: https://piotte13.github.io/enigma-cipher/

